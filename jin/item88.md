# item 88 readObject 메서드는 방어적으로 작성하라

## 방어적 복사

이전에 가변한 Date를 방어적으로 복사하는 이전 예제를 보자.

```java
public final class Period {
  private final Date start;
  private final Date end;
  
  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
    if(this.start.compareTo(this.end) > 0)
      throw new IllegalArgumentException(start + " after " + end);
  }
  
  public Date start() { return new Date(start.getTime());}
  public Date end() { return new Date(end.getTime()); }
  public String toString() { return start + " - " + end; }
}
```

### 직렬화의 문제점

위 코드는 객체의 물리적 표현과 논리적 표현이 일치하므로 item 87 처럼 직렬화 하여도 괜찮지만 불변식을 보장하지 못한다.

그 이유는 다음과 같다.

- readObject 메서드가 실질적으로 또 다른 public 생성자 임(따라서 다른 생성자와 같은 수준으로 주의가 필요)
- 보통의 생성자 처럼 readObject 메서드에서도 인수가 유효한지 검사해야함
- 필요하다면 매개변수를 방어적으로 복사해야한다.
- readObject가 이 작업을 제대로 수행하지 못하면 공격자는 아주 쉽게 해당 클래스의 불변식을 깨트릴 수 있다.

__요약 : readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다. 비정상적인 객체를 주어 직렬화 하게 한다면 정상적으로는 만들어 낼 수 없는 객체를 생성하게 된다.__

(readObject와 writeObject는 자바 직렬화와 역직렬화 과정에서 별도 처리가 필요할 때 클래스 내부에 선언해줘야 한다.)



## 직렬화의 문제점을 해결?

- 위 Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역질렬화 된 객체가 유효한지 검사해야함.
- 이 유효성 검사를 통과하지 못하면 InvalidObjectException을 던짐으로서 역질렬화를 막아야 한다.
- 하지만, 이것 또한 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로 참조를 추가하면 가변 Period를 만들어 낼 수 있어 문제가 있다.

```java
//공격의 예
public class MutablePeriod {
// Period 인스턴스
public final Period period;
	// 시작 시각 필드 - 외부에서 접근할 수 없어야 한다. 
  public final Date start;
	// 종료 시각 필드 - 외부에서 접근할 수 없어야 한다. 
  public final Date end;
  
	public MutablePeriod() { 
    try {
      ByteArrayOutputStream bos = new ByteArrayOutputStream();
      ObjectOutputStream out = new ObjectOutputStream(bos);
    // 유효한 Period 인스턴스를 직렬화한다. 
      out.writeObject(new Period(new Date(), new Date()));
    /*
    * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다. * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고하자.
    */
      byte[] ref = { 0x71, 0, 0x7e, 0, 5 }; // 참조 #5 bos.write(ref); // 시작(start) 필드
      ref[4] = 4; // 참조 # 4
      bos.write(ref); // 종료(end) 필드
    // Period 역직렬화 후 Date 참조를 '훔친다'. ObjectInputStream in = new ObjectInputStream(
      new ByteArrayInputStream(bos.toByteArray())); 
      period = (Period) in.readObject();
      start = (Date) in.readObject();
      end = (Date) in.readObject();
    } catch (IOException | ClassNotFoundException e) {
  		throw new AssertionError(e); 
    }
 }
```

이를 다음과 같은 코드로 공격할 수 있다.

```java
public static void main(String[] args) { 
  MutablePeriod mp = new MutablePeriod(); 
  Period p = mp.period;
  Date pEnd = mp.end;
	// 시간을 되돌리자! 
  pEnd.setYear(78); System.out.println(p);
	// 60년대로 회귀! 
  pEnd.setYear(69); System.out.println(p);
}

```

- 문제의 근원은 Period의 readObject 메서드가 방어적 복사를 충분히 하지 않은데 있다. 
  __따라서, 객체를 역직렬화할 때는 클라이언트가 소유해서는 안되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.__

아래와 같이 방어적으로 복사한다.

```java
private void readObject(ObjectInputStream s) 
  throws IOException, ClassNotFoundException {
	
  s.defaultReadObject();
		// 가변 요소들을 방어적으로 복사한다. 
  start = new Date(start.getTime()); 
  end = new Date(end.getTime());
// 불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
		throw new InvalidObjectException(start +" after "+ end); 
}
```



## ReadObject 메서드를 써도 좋은 조건

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
  (아니요라면 custom readObject를 통해 유효성 검증을 해야한다.)
- 위 패턴은 직렬화 프록시 패턴을 사용할 수도 있다.
- 하지만 직간접적으로 재정의 할 수 있는 메서드는 호출하지 말자.



## Reference

https://madplay.github.io/post/what-is-readobject-method-and-writeobject-method