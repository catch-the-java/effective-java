# item 44 표준 함수형 인터페이스를 사용하라

## 이전의 자바

자바가 람다를 지원하기 시작하면서 API를 작성하는 모범 사례가 바뀌었다.

예를들어 템플릿 메서드 패턴 같은 것이 최근에는 매력이 많이 떨어졌다.

현대적인 해법은 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이다. 즉, 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다.

(단, 이때에는 함수형 매개변수 타입을 올바르게 선택해야한다.)

LinkedHashMap을 생각해보자. 이 클래스의 protected 메서드인 removeEldestEntry를 재정의하면 캐시로 사용할 수 있다.

맵에 새로운 키를 추가하는 put 메서드는 이 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다. 다음 정의와 같이 하면 size()가 100보다 커지면 가장 오래된 원소를 제거하게 된다.

```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
  return size() > 100;
}
```

잘 동작하지만 함수형 인터페이스를 이용해 다음과 같이 선언할 수 있다.

```java
@FunctionlInterface interface EldestEntryRemovalFunction<K, V> {
  boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
```

이 인터페이스는 잘 동작하지만 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있다.

즉, __필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.__ 이를 통해 API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다.

표준 함수형 인터페이스는 굉장히 많은데 다음을 보자.

### Predicate

Predicate는 test라는 추상 메서드를 정의함. T 객체를 받아서 boolean을 반환한다.

원형은 아래와 같으며 활용도 함께 적어놓았다.

```java
@FunctionalInterface
public interface Predicate<T> {
	boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
	List<T> results = new ArrayList<>();
	for(T t : list) {
		if(p.test(t)) {
			results.add(t);
		}
	}
	return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, noEmptyStringPredicate);
```

"and"나 "or" 같은 메서드가 추가로 존재하는데 해당 내용은 뒷 부분에서 살펴볼 것이다.

### Consumer

Consumer는 T객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다.

즉, 객체를 받아서 특정 행동을 수행하고자 할 때 사용할 수 있는 인터페이스이다. 특히 foreach와 람다를 이용해서 리스트의 항목을 출력하는 예제를 살펴보자.

```java
@FunctionalInterface
public interface Consumer<T> {
	void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
	for(T t: list) {
		c.accept(t);
	}
}
forEach(
	Arrays.asList(1,2,3,4,5),
	(Integer i) -> System.out.println(i)
);
```

### Function

Function 인터페이스는 제네릭 형식 T를 받아서 제네릭 형식 R을 반환하는 추상메서드 apply를 정의한다.

우리가 수학에서 생각하는 함수 생각하면 편하다. 예제를 보자.

```java
@FunctionalInterface
public interface Function<T, R> {
	R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
	List<R> result = new ArrayList<>();
	for(T t : list) {
		result.add(f.apply(t));
	}
	return result;
}

List<Integer> l = map(
			Arrays.asList("lambdas", "in", "action"),
			(String s) -> s.length() 
);//String을 받아서 Integer를 반환함
```

### 기본형 특화

3가지의 함수형 인터페이스 Predicate<T>, Consumer<T>, Function<T, R>을 살펴봤다. 하지만 조금 더 특화된 함수형 인터페이스가 있다.

자바는 기본적으로 **primitive type(기본형 타입)**과 **reference type(참조형 타입)**이 있는데 지금까지 봤다시피 **제네릭**을 이용해 구현하는 함수형 인터페이스는 어쩔 수 없이 참조형 타입을 사용해야 한다.

자바에서는 참조형을 기본형으로, 기본형을 참조형으로 바꾸는 기능을 지원한다.

**언박싱(unboxing)**, **박싱(boxing)**이라고 하며 이것을 묵시적으로 자동으로 박싱과 언박싱이 이루어 지는 기능을 **AutoBoxing(오토 박싱)** 이라고 한다.

다만 이런 박싱과 언박싱 과정이 메모리를 더 사용하고 참조 시간을 추가로 쓰게 된다. 이것을 피하기 위한 특별한 Interface를 제공한다.

```java
public interface IntPredicate {
	boolean test(int t);
}

IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000); //이건 참이며 박싱 과정이 없어.

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); //거짓이며 박싱과정이 존재.
```

이 외에도 다양한 인터페이스가 존재한다. 표로 정리한 내용은 책에 있으니 책을 참조하자.

(너무 많어..)

추가로 잘 정리해 놓은블로그를 함께 보도록 하자(https://bcp0109.tistory.com/313)



만약에 직접 만든 함수형 인터페이스가 있다면 @FunctionalInterface 애너테이션을 사용하여 표시해줘야 한다.

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
3. 그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.



## 주의점

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안된다.

클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 한다.

이에 대해서는 item52에 더 자세히 나타낸다.