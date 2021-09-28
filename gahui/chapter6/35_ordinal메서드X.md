# ordinal 메서드 대신 인스턴스 필드를 사용하라

## ordinal
- 열거 타입에서 기본적으로 제공하는 ordinal 메서드가 존재한다.
- 열거 타입 상수는 자연스럽게 하나의 정수값에 대응된다.
- ordinal 메서드는 해당 상수가 그 열거 타입에서 몇번 째 위치인지를 반환한다.
- Enum API 문서에도 프로그래머가 사용할 일은 거의 없다고 적혀있다. 
- 자료구조를 쓸 목적이 아니라면 사용하지 말자.


## 잘못 사용한 경우
```java
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SEPTET, OCTET, NONET, DECTET;
	
	public int numberOfMusicians() { return ordinal() +1; }
}
```
- 문제점
  - 유지보수가 힘들다.
  - 같은 상수를 가지고 있는 값은 추가할 수 없다.

## 인스턴스 필드를 사용한 경우
```java
public enum EnsembleInstance {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
	SEXTET(6), SEPTET(7), OCTET(8),
	NONET(9), DECTET(10), TRIPLE_QUARTET(12);

	private final int numberOfMusicians;
	EnsembleInstance(int size) {this.numberOfMusicians = size;}
	public int numberOfMusicians() { return numberOfMusicians; }
}
```