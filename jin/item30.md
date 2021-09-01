# item 30 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로 메서드도 제네릭으로 만들 수 있다. 

아래 코드는 컴파일이 되긴 하지만 warning을 발생시킬 것이다.

```java
public static Set union(Set s1, Set s2) {
	Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

__타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다.__ 위와 같은 코드를 제네릭 메서드로 나타내면 다음과 같다.

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

이런 식으로 하면 타입 안정성도 확보가 가능하고 경고 역시 지울 수 있다. 이를 이용한 예제 코드를 보자.

```java
public static void main(String[] args) {
  Set<String> guys = Set.of("톰", "딕", "해리");
  Set<String> stooges = Set.of("래리", "모에", "컬리");
  Set<String> aflCio = union(guys, stooges);
  System.out.println(aflCio);
}
```

이 union 메서드는 집합 3개의 타입이 모두 같아야 하는데 한정적 와일드 카드를 이용하여 더 유연하게 만들어 줄 수 있다.

항등함수를 담은 클래스를 만들고 싶다고 해보자. 자바 라이브러리의 Function.identity를 사용하면 되지만 직접 만들어 보자.

항등 함수 객체는 상태가 없으니 요청할 때마다 새로 만드는 것은 낭비이므로 제네릭을 이용하여 만들어 보자.

__제네릭 싱글턴 팩터리 패턴__

```java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
  return (UnaryOperator<T>) IDENTITY_FN;
}
```

IDENTITY_FN을 UnaryOperator\<T>로 형변환하면 비검사 형변환 경고가 발생한다. T가 어떤 타입이든 Object는 T타입이 아니기 때문이다.

하지만 항등함수란 입력값을 수정 없이 그대로 반환하는 함수이므로 T가 어떤 타입이든 값을 수정없이 그대로 반환하는 함수이므로 UnaryOperator\<T>를 반환해도 타입 안전하다.

다음 코드를 확인해보자.

```java
public static void main(String args) {
  String[] strings = {"삼베", "대마", "나일론"};
  UnaryOperor<String> samString = identityFunction();
  for (String s : strings)
    System.out.println(sameString.apply(s));
  
  Number[] numbers = {1, 2.0, 3L};
  UnaryOperator<Number> sameNumber = identityFunction();
  for (Number n : numbers)
    System.out.println(sameNumber.apply(n));
}
```


### 재귀적 타입 한정

타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다.

```java
public interface Comparable<T> {
  int compareTo(T o);
}
```

여기서 타입 매개변수 Comparable\<T>를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.

예를들어 Comparable\<String>은 비교하고 Comparable\<Integer>는 Integer를 구현하는 방식이다.

Comparable를 구현한 원소의 컬렉션을 입력받는 메서드 들은 주로 그 원소를 정렬 혹은 검색, 최대, 최소값을 구하는데 사용된다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c);
```

타입 한정인 <E extends Comparable\<E>>E max(Collection\<E> c)는 "모든 타입 E는 자신과 비교할 수 있다." 라고 읽을 수 있다.

다음은 이 메서드의 구현이다.

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if(c.isEmpty())
    throw new Illegal ArgumentException("컬랙션이 비어있습니다.");
  
  E result = null;
  for (E e : c)
    if(result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return result;
}
```

재귀적 타입 한정은 훨씬 복잡해 질 가능성이 있지만 그런일은 잘 일어나지 않는다.
