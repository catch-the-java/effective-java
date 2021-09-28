# int 상수 대신 열거 타입을 사용하라

## int 상수의 문제점
1. 타입 안전성을 보장하지 못함.
	- ex) 아래와 같을 때, 컴파일러는 눈치채지 못한다.
	```java
	APPLE_PRICE = 1000;
	ORANGE_PRICE = 1000;
	```
2. 평범한 상수를 나열한 것 뿐이라서 해당 상수값이 바뀌면 클라이언트도 반드시 다시 컴파일해야 한다.

3. 정수 상수는 디버깅/값 출력 시 단지 숫자로만 보여, 도움이 되지 않는다.

4. 해당 그룹에 속한 정수 상수가 총 몇 개인지 알 수 없다.

</br>

## 열거 타입(ENUM type) 특징
1. 클래스이다.
2. 밖에서 접근할 수 있는 생성자를 제공하지 않는다.(싱글턴)
3. 인스턴스 통제된다. 
4. 컴파일 타임 타입 안전성 제공
5. 각자의 이름 공간이 있어, 이름이 같은 상수도 공존한다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
```

</br>

## 열거타입에 메서드나 필드 추가 
- 각 상수와 연관된 데이터를 해당 상수 자체에 내제시키고 싶을 때
	- ex) Apple, Orange 와 같이 과일의 색, 과일의 이미지 반환 메서드
- 생성자에서 데이터를 받아서 인스턴스 필드에 저장하면 된다.
- 열거타입은 근본적으로 __불변__ 이라 모든 필드는 final이어야 한다. 
- 필드를 public으로 선언해도 되지만, private으로 두고 별도의 public 접근자 메서드로 하자

</br>

## 열거 타입에서 상수 하나 제거하면 어떻게 될까?
- 제거한 상수를 참조하지 않는 클라이언트에는 아무런 영향이 없다.
- 그렇다면, 참조한 클라이언트는?
	- 컴파일 오류
	- 다시 컴파일 하지 않으면 런타임에 오류

</br>

## 열거타입의 바람직한 사용
1. 해당 기능을 클라이언트에게 노출해야 할 합당한 이유가 없다면, private or package-private으로 선언하라
2. 널리쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 __멤버 클래스__ 로 만들어 사용하자.

</br>

## 상수별 다르게 동작하는 코드 구현
```java
public enum Operator {
	PLUS {public double apply(double x, double y) { return x+y; }},
	MINUS {public double apply(double x, double y) { return x-y; }},
	TIMES {public double apply(double x, double y) { return x*y; }},
	DIVIDE {public double apply(double x, double y) { return x/y; }};
	
	public abstract double apply(double x, double y);
}
```
- 추상메서드로 구현되어 있어, 새로운 ENUM type이 추가된다면, apply를 구현해야한다.

</br>

## 상수별 메서드 구현을 상수별 데이터와 결합하기
```java
public enum OperatorClassBody {
	PLUS("+") {public double apply(double x, double y) { return x+y; }},
	MINUS("-") {public double apply(double x, double y) { return x-y; }},
	TIMES("*") {public double apply(double x, double y) { return x*y; }},
	DIVIDE("/") {public double apply(double x, double y) { return x/y; }};

	private final String symbol;
	OperatorClassBody(String symbol) {this.symbol = symbol;}
	
	@Override 
	public String toString() {return symbol;}
	public abstract double apply(double x, double y);
}
```

## toString 재정의 시, fromString 메서드도 함께 제공하는 것을 고려하자.
- fromString 메서드 : toString이 반환하는 문자열을 해당 열거타입상수로 반환해주는 메서드

```java
private static final Map<String, OperatorClassBody> stringToEnum =
			Stream.of(values()).collect(
					toMap(Object::toString, e -> e)
			);

	public static Optional<OperatorClassBody> fromString(String symbol) {
		return Optional.ofNullable(stringToEnum.get(symbol));
	}
```
</br>

## ENUM Type 작성 시 switch 문의 단점
1. 유지보수의 어려움
```java
public enum PayrollDay {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
	SATURDAY, SUNDAY;
	
	private static final int MINUS_PER_SHIFT = 8 * 60;
	
	int pay(int minutesWorked, int payRate) {
		int basePay = minutesWorked * payRate;
		
		int overtimePay;
		switch (this) {
			case SATURDAY: case SUNDAY:
				overtimePay = basePay/2;
				break;
			default:
				overtimePay = minutesWorked <= MINUS_PER_SHIFT ?
						0 : (minutesWorked - MINUS_PER_SHIFT) * payRate /2;
		}
		return basePay + overtimePay;
	}
}
```

## 전략 열거 타입
```java
public enum PayrollDayPayType {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
	SATURDAY, SUNDAY;

	private static final int MINUS_PER_SHIFT = 8 * 60;

	//전략 열거 타입 패턴
	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked <= MINUS_PER_SHIFT ? 0 :
						(minsWorked - MINUS_PER_SHIFT) * payRate /2;
			}
		},
		WEEKEND {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked * payRate / 2;
			}
		};
		
		abstract int overtimePay(int minsWorked, int payRate);
		private static final int MINS_PER_SHIFT = 8 * 60;
		
		int pay(int minsWorked, int payRate) {
			int basePay = minsWorked * payRate;
			return basePay + overtimePay(minsWorked, payRate);
		}
	}
}
```

</br>

## switch문이 좋은 선택이 될 수 있는 예)
- 기존 열거 타입에 상수별 동작을 혼합해 넣을 때
```java
public static Operator inverse(Operator op) {
		switch (op) {
			case PLUS: return Operator.MINUS;
			case MINUS: return Operator.PLUS;
			case TIMES: return Operator.DIVIDE;
			case DIVIDE: return Operator.TIMES;
			
			default: throw new AssertionError("알수 없는 연산 :  " + op);
		}
	}

```

</br>

## 열거 타입을 사용해야 하는 경우
1. 필요한 원소를 컴파일 타임에 알 수 있는 상수 집합
	ex) 태양계 행성, 한 주의 요일, 체스 말 등
