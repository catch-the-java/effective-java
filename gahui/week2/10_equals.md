# Item 10 equals는 일반 규약을 지켜 재정의하라

# equals 메서드
재정의하기 쉬워보이지만 함정이 많아 자칫하면 끔찍한 결과를 초래한다. 

## 아래와 같은 상황 중 하나라도 해당된다면 재정의하지 말자
### 1. 인스턴스가 본질적으로 고유하다
  - 값을 표현하는 것이 아닌 동작하는 클래스
  - ex) Thread


### 2. 인스턴스의 논리적 동치성을 검사할 일이 없다.


### 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

### 4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
- equals가 실수라도 호출되는 걸 막고 싶다면 아래와 같은 방식으로 구현하자.
```java
@Override
	public boolean equals(Object o) {
		//호출금지
		throw new AssertionError();
	}
```

# Equals를 재정의해야하는 경우
## 1. 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
- 주로 값 클래스(Integer, String)가 이에 해당한다. 
- 값 클래스라고 하더라도 인스턴스가 두개 이상 만들어지지 않으면 재정의를 하지 않아도 된다. 

# equals 일반규약
equals 메서드를 재정의할 때, **반드시** 규약을 따라야 한다. 
## 1. 반사성
- 자기 자신과 같아야 한다. 
- x와 x는 연관이 있다.

## 2. 대칭성
- x와 y가 연관이 있으면, y와 x도 연관이 있다.

```java
public class CaseInsensitiveString {
	private final String s;

	public CaseInsensitiveString(String s) {
		this.s = Objects.requireNonNull(s);
	}

	// 대칭성 위배
	@Override
	public boolean equals(Object o) {
		if( o instanceof CaseInsensitiveString)
			return s.equalsIgnoreCase(
					((CaseInsensitiveString) o).s);

		if (o instanceof String)
			return s.equalsIgnoreCase((String) o);
		return false
	}
}
```
- 위와 같은 케이스에서는 String type도 비교할 수 있도록 했다. 
- 하지만, String type의 경우, CaseInsensitiveString을 알지 못한다. 그렇기 때문에 대칭성에 위배된다. 
- String과 연동하려고 해서는 안된다.

## 3. 추이성
- x와 y가 관련이 있고, y와 z가 연관이 있으면, x와 z도 연관이 있다.
- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 **존재하지 않는다.**
- 상속을 하지 않고 컴포지션을 사용하여 해결할 수 있다. 

```java
public class ColorPoint {
	private final Point point;
	private final Color color;

	public ColorPoint(int x, int y, Color color) {
		point = new Point(x,y);
		this.color = Objects.requireNonNull(color);
	}

	public Point asPoint() {
		return point;
	}

	@Override
	public boolean equals(Object o) {
		if(!(o instanceof Point))
			return false;
		ColorPoint cp = (ColorPoint) o;
		return cp.point.equals(point) && cp.color.equals(color);
	}

```


## 4. 일관성
- 두 객체가 같다면 영원히 같아야 한다. 
- 불변 객체는 equals로 **항상** 동일한 값을 가져야한다. 
- 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다. 
- equals는 항상 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야한다.


## 5. null-아님
- 모든 객체가 null과 같지 않아야한다.
- NullPointException이 아닌 false의 값을 내야한다. 
- instnaceof로 null check를 할 수 있다. 


# equals 구현 방법
## 1. ==연산자를 사용해 입력이 자기 자신의 참조인지를 확인한다.
- 성능 최적화용

## 2. instanceof 연산자로 입력이 올바른 타입인지 확인하자

## 3. 입력이 올바른 타입으로 형변환한다.
- instanceof 검사를 했기 때문에 형변환은 무조건 된다. 

## 4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사한다.
- 하나라도 다르면 false 반환

## 추가
- 다를 가능성이 크거나, 비교하는 비용이 싼 필드를 먼저 비교하자. 

# 주의사항
## 1. equals 재정의할 때 hashcode도 재정의하자
## 2. 너무 복잡하게 해결하지 말자
## 3. Object 외의 타입을 매개변수르 받는 equals 메서드는 선언하지 말자.

# 용어 
## 1. 리스코프 치환 원칙 
- 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 