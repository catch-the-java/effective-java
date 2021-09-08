# item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라
- ordinal 메서드는 상수가 열거 타입에서 몇 번째 위치인지를 반환한다.

<br/>

## ordinal메서드 단점
```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTER ...

	public int numberOfMusicians() {
		return ordinal()+1;
	}
}
```
#### 동작은 하지만 유지보수 끔찍하다.
- 상수 선언 순서 바꾸면 오동작
- 사용 중인 정수와 값이 같은 상수는 추가 못함
#### 값을 중간에 비워둘 수도 없다.

<br/>

## 해결책
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```
#### 열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하자.
