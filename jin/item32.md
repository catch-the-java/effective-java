# Item32 제네릭과 가변인수를 함꼐 쓸 때는 신중하라

## 가변인수 메서드

가변인수(Java varargs) 메서드와 제네릭은 자바 5에서 함께 추가 되었지만 두개는 잘 어울리지 못한다. 우선 가변 인수는 뭘까?

만약 매개변수의 개수가 쓰는 사람에 따라서 달라지는 경우는 어떻게 될까?

계속 개수를 추가해줄수는 없는 노릇이다. 이것을 해결하기 위해 이전에는 배열을 응용했으나 다음과 같이 할 수 있다.

참고로 가변인수는 메서드의 마지막에 선언해야한다.

```java
String printAll(String s1... str) {
  for(String item : str) {
    System.out.println(item);
  }
}
```



만약 가변인수 매개변수에 제네릭이나 매개변수화 타입이 포함된다면 경고가 발생한다.

```java
warning: [unchecked] Possible heap pollution from parameterized varage type List<String>
```

매개변수화 타입의 변수가 타입이 다른 객체를 참조한다면 힙 오염이 발생한다. 예를 보자.

```java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  object[0] = intList;
  String s = stringList[0].get(0); //ClassCastException
}
```

배열이 Object로 선언됐기 때문에 varargs인 stringList를 그대로 받아 들일 수 있고 Object 배열이어서 Integer로 타입이 선언된 List 또한 넣을 수 있다. 그리하여 타입이 위험해지고 힙 오염이 발생한다. __따라서 타입 안정성을 해치기 때문에 제네릭 varargs 매개변수는 쓰는 것은 위험하다.__

근데, 이는 자바에서 막아놓지 않은 이유는 실무에서 도움이 되기 떄문이다.

Arrays.asList(T... a), Collections.addAll(Collection<? super T> c, T... elements) 등의 예가 존재한다. 이 예시들은 앞서 보여준 예제와는 달리 타입이 안전하다. 또한 자바7 이전에는 SuppressWarings("unchecked") 어노테이션을 이용하여 타입 안정성 경고를 지워줘야 했지만 @SafeVarargs를 이용해서 타입 안정성을 보여줄 수 있다.



## 안정적인 가변인수 메서드

가변인수 메서드가 안전하다는 것이 확실하지 않으면 @SafeVarargs 어노테이션을 달아선 안된다. 어떻게 보장할 수 있을까?

- 메서드가 이 배열에 아무것도 저장하지 않음
- 그 배열의 참조가 밖으로 노출되지 않음

즉, 결론적으로 해당 메서드가 순수하게 인수들을 전달하는 일만 쓰인다면(varargs 목적대로만 사용한다면) 사용해도 좋다.

다음 코드를 보자.

```java
static <T> T[] toArray(T... args) {
  return args;
}
```

이 코드는 T형식을 배열에 담아서 반환해주는 좋은 메서드 같지만 아니다. 이 메서드의 반환 타입은 이 메서드에 인수를 넘기는 컴파일 타임에 결정이 되게 되는데 그 시점에는 컴파일러가 타입을 잘못 판단할 가능성이 있다. 자신의 varargs 매개변수를 그대로 반환하면 힙 오염이 이 메서드를 호출한 쪽의 콜 스택까지 전염될 수 있다. 구체적인 예를 보자.

```java
static <T> T[] pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a, b);
    case 1: return toArray(a, c);
    case 2: return toArray(b, c);
  }
  throw new AssertionError(); // 도달 불가.
}
```

위 예제만 봤을때는 큰 문제가 없어 보인다. 근데 이걸 쓰는 코드를 만들어 보자.

```java
public static void main(String[] args) {
  String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

이 코드 에러난다.

ClassCastException이 발생되는데 왜 그럴까? 타입 변환 로직도 없는데 말이다.

우리가 잊고 있는 사실이 한가지가 있는데 pickTwo의 반환값을 String[]으로 바꿔주기 위해 형변환 하는 코드를 자동으로 컴파일러가 넣어주는데 Object[]가 String[]의 하위타입이 아니므로 형 변환이 실패하게 된다. 이러한 예는 사실 알아차리기 쉽지 않으며 그러므로 사용할때 매우 주의해야한다.

하지만 다시 이야기 한 것 처럼 두가지 상황에서는 안전하다는 것을 기억하자.

- @SafeVarargs로 제대로 어노테이트 된 다른 varargs 메서드에 넘기는 것은 안전하다.
- 그저 이 배열 내용의 일부 함수를 호출만 하는(즉, varargs를 받지 않는) 일반 메서드에 넘기는 것도 안전하다.

다음 예시는 안전하게 사용하는 예이다.

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}
```

SafeVarargs 어노테이선을 다는 것은 __제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 달면 된다.__

즉, 안전하지 않으면 아애 쓰지 말라는 말이다.

## @SafeVarargs가 유일한 정답은 아니다.

아이템 28에 나온 이야기 처럼 매개변수를 List로 바꾸는 것이다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists)
    result.addAll(list);
  return result;
}

audience = flatten(List.of(friend, romans, countrymen));
```

위 코드는 매개변수만 변경하였는데 List.of 메서드를 활용하여 varargs를 안전하게 활용할 수 있다.(List.of도 SafeVarargs가 달려 있기 때문)

이 방식의 장점은 컴파일러가 에러를 찾아낼 수 있다. 이 방식을 toArray와 같이 안전하게 사용하기 힘든 상황에서 사용하기에 좋다. pickTwo 메서드를 고쳐보자.

```java
static <T> T[] pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return List.of(a, b);
    case 1: return List.of(a, c);
    case 2: return List.of(b, c);
  }
  throw new AssertionError(); // 도달 불가.
}

public static void main(String[] args) {
  List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

배열 없이 제네릭만 사용하여 안전하다. 다만 클라이언트 코드가 지저분해지고 성능상의 손해를 살짝 볼 수 있다는 점이다.