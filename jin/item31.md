# item 31 한정적 와일드카드를 사용해 API 유연성을 높이라

## 불공변

이전에 배웠던 개념을 다시 생각해보자. List\<String>과 List\<Object>는 서로 상위, 하위 타입을 가지지 않는 불공변 관계이다.

List\<Object>에 들어갈 수 있는 객체가 모두 List\<String>에 들어갈 수는 없기 때문에 하위 타입이 될 수 없다.

하지만 우리는 더 유연한 구조가 필요할 때가 있다. 어떻게 해야할까?

## 와일드 카드가 없는 Stack

```java
public class Stack<E> {
  public Stack();
  public void push(E e);
  public E pop();
  public boolean isEmpty();
}
```

여기에 이터레이터를 이용하여 값을 집어넣는 코드를 추가해본다.

```java
public void pushAll(Iterable<E> src) {
  for (E e : src) {
    push(e);
  }
}
```

하지만 이 코드에는 문제가 있는데 Stack\<Number>로 선언한 후 pushAll(intVal)을 호출한다고 가정하자(intVal은 Integer 타입이다.)

 Integer Type은 Number의 하위 타입이니 잘 작동할 것 같지만 실행해보면 작동이 되지 않는다. 그 이유는 매개변수화 타입이 불공변이기 때문이다.

이것을 상황에 대처할 수 있도록 한정적 와일드카드 타입이라는 매개변수화 타입이 존재한다.

## 한정적 매개변수화 타입

pushAll 메서드의 입력 매개변수 타입은 E의 Iterable이 아니라 E의 하위 타입의 Iterable이어야 한다. 

와일드 카드 타입을 이용해 표현한 Iterable<? extends E> 타입이 (정확히는 아니지만) 위 문장을 의미한다. 따라서 위에서 봤던 pushAll 메서드를 다음과 같이 바꾸면 잘 작동 할 것이다.

```java
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
    push(e);
} 
```

다음은 popAll메서드도 마찬가지로 살펴보자.

```java
public void popAll(Collection<E> dst) {
  while(!isEmpty()) {
    dst.add(pop());
  }
}
```

위 코드 역시 컬렉션의 원소 타입이 스택과 맞는다면 잘 작동하겠지만 역시나 Stack\<Number>를 Object용 컬렉션으로 옮긴다고 가정해보자.

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

하지만.. 당연하게도 위 코드는 첫번째 본 예제인 pushAll과 마찬가지로 에러를 띄우게 된다. 이번에는 Collection\<Object>는. Collection\<Number>의 하위타입이 아니기 때문이다. 다음과 같이 표현하면 정상적으로 작동하게 된다.

```java
public void popAll(Collection<? super E> dst) {//들어오는 타입은 무조건 E보다 부모 타입이어야 한다
  while(!isEmpty())
    dst.add(pop());
}
```

즉, __유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.__

하지만, 만약에 생산자와 소비자의 역할을 동시에 해야하는 경우는 와일드카드 타입을 사용해서는 안되는데 이는 타입을 정확히 지정해야 하는 상황이기 떄문이며 다음 공식을 외워두면 와일드카드를 사용하는데에 도움이 될 것이다.

**<u>producer-extends, consumer-super</u>**

앞 아이템에서 등장한 30-2번 코드인 union메서드를 살펴보자. 이는 생산자 역할을 하고 있기 때문에 다음과 같이 고친다.

```java
//이전
public static <E> Set<E> union(Set<E> s1, Set<E> s2);

//이후
public static <E>> Set<E> union(Set<? extends E> s1, Set<? extends E> s2);
```

위와 같이 고치면 클래스를 사용하는 사람은 와일드카드를 사용하고 있다는 자각조차 없게될 것이고 __클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 문제가 있을 가능성이 크다.__

(참고로 앞선 코드는 Java8 부터 정상적으로 작동할 것이다.)

다음은 코드 30-7인 max 메서드를 개선해보자.

```java
//이전
public static <E extends Comparable<E>> E max(List<E> list);

//이후
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

위는 PECS 공식을 두번 사용했는데 입력 매개변수는 직관적이니 설명을 넘어가고 타입 매개변수 E를 살펴보도록 하겠다.

원래 선언에서는 E가 Comparable\<E>를 확장한다고 정의했는데, 이떄 Comparable\<E>는 E 인스턴스를 소비한다. 그래서 매개변수화 타입을 한정적 와일드 타입으로 바꾸었다. __Comparable은 언제나 소비자 역할이기 때문에 보통은 Comparable\<E>보다는 Comparable<? super E>가 났다.__

근데 이렇게 까지 복잡하게 만들 필요가 있을까? 그 근거는 이제 보자.

```java
//아래 리스트는 수정전 max로 처리할 수 없다.
List<ScheduledFuture<?>> scheduledFutures = ...;
```

그 이유는 ScheduledFuture가 Comparable\<ScheduledFuture>를 구현하지 않았다. ScheduledFuture를 알아 보자면 Delayed의 하위 인터페이스이고, Delayed는 Comparable\<Delayed>를 확장하였다. 

즉, ScheduledFuture의 인스턴스는 다른 ScheduledFuture의 인스턴스 뿐만 아니라 Delayed 인스턴스와도 비교할 수 있기 때문에 max가 이 리스트를 거부하게 된다. 

그림을 확인하면 더 쉽게 이해할 수 있으니 책을 참조하자.



## 마지막 논의 내용

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드 정의시 어떤 것을 골라도 되는 경우가 있다.

예를 보자.

```java
public static<E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```

Public 메서드의 경우는 아래 방식이 더 낫다. 어차피 타입 매개변수가 한번만 등장하기 떄문에 더 간단하기 떄문이다. 기본 규칙은 __메서드 선언에 타입 매개변수가 한번만 나오면 와일드카드로 대체하라.__ 이다.

하지만 아래 코드는 딱히 도움이 되지 않는다. 황당하겠지만 List<?>로 선언되어 있어서 null이외에는 어떠한 값도 집어넣을 수 없다.

하지만 로타입을 쓰거나 형변환을 사용하지 않고 __private로 도우미 메서드를 작성해 사용할 수 있다.__

와일드카드의 실제 타입을 알아내려면 이 도우미 메서드는 제네릭 메서드여야 한다.

```java
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

private static<E> void swapHelper(List<E> list, inti , int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

이를 이용해서 공개된 API는 와일드 카드를 이용하여 표현할 수 있다.



//근데 도우미 메서드랑 1번이랑 똑같은데 효용성이 정확히 뭐지..? public API로 사용하기에는 복잡하다는 의미가 뭘까?
