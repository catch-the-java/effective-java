# private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴이란?
- 인스턴스를 하나만 생성할 수 있는 클래스
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워진다. 

</br>

# 싱글턴을 만드는 방법
생성자를 private으로 감추고, public static 멤버를 마련해둔다. 

## 방법1. public static final 방식
- public static final 필드를 초기화할 때 딱 한번 호출된다.
- 예외) 권한이 있는 클라이언트가 리플렉션 API인 AccessibleObject.setAccessible을 사용해서 private 생성자를 호출할 수 있다.
  - 이는 예외를 던지도록 해결할 수 있다.
- 장점
  - 싱글턴임이 API에 드러난다.
  - 간결하다.
```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	public void leaveTheBuilding() {}
}
```

## 방법2. 정적 팩토리 방식 
- (위와 예외는 똑같이 적용 됨)
- 장점
  - 싱글턴이 아니게 변경할 수 있다.
  - 제네릭 싱글턴 팩터리로 만들수 있다.
  - 메서드 참조를 공급자(supplier)로 사용할 수 있다. 
```java
class Elvis2 {
	private static final Elvis2 INSTANCE = new Elvis2();
	private Elvis2() {}
	public static Elvis2 getInstance() { return INSTANCE; }
	public void leaveTheBuilding() {}
}

// 메서드 참조를 공급자로 사용하는 장점?! 
// QQ. 이렇게 쓰면 무엇이 좋은가??
Supplier<Elvis2> elvis2Supplier = Elvis2::getInstance;
```

## 방법3. 열거타입 선언
- 간결, 추가 노력없이 직렬화 가능
  - **QQ. 직렬화가 되면 좋은점?**
- 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법
```java
enum Elvis3 {
	INSTANCE;
	public void leaveTheBuilding() {}
}
```

</br>

# 기타용어
## 1.직렬화
- 자바 시스템 내부에서 사용되는 Object/Data를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하는 기술.
- java.io.Serializable 인터페이스를 상속하여야 한다.
- java.io.ObjectOutputStream 을 통해 직렬화를 진행

## 2.역직렬화
- byte 형태를 Object/Data로 변환
    
## 3. Supplier 
- 함수형 인터페이스 중 하나
- 람다/스트림과 약간 비슷해 보임(메소드를 함수형으로 만들 수 있는 듯!) 
- Supplier가 대표적으로 사용되는 것으로 Stream의 generate 메서드를 사용할 수 있다.

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```


</br>

# 참고 링크
- 직렬화 관련 : https://nesoy.github.io/articles/2018-04/Java-Serialize
- SUpplier : https://mkyong.com/java8/java-8-supplier-examples/