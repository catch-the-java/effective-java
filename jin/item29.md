# Item 29 이왕이면 제네릭 타입으로 만들자

### 1. Object 기반의 Stack Class

이제부터 Object기반으로 만들어진 stack class를 함께 보자.

```java
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT = 16;
  
  public Stack() {
    elements = new Object[DEFAULT];
  }
  
  public void push (Object e) {
    ensureCapacity();
    elements[size++] = el
  }
  
  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // 다 쓴 객체 참조 해제로 메모리 릭 방지
    return result;
  }
  
  ...이하 생략
}
```

이 코드의 문제는 굉장히 명확하다. 바로 pop메서드를 부를때 마다 형변환을 따로 해줘야 하며 이는 타입 안정성을 해쳐 런타임 에러를 발생시킬 가능성을 증가시킨다는 것이다. 다음을 보자.

```java
public class Stack<T> {
  private T[] elements;
  private int size = 0;
  private static final int DEFAULT = 16;
  
  public Stack() {
    elements = new T[DEFAULT];
  }
  
  public void push (T e) {
    ensureCapacity();
    elements[size++] = el
  }
  
  public T pop() {
    if (size == 0)
      throw new EmptyStackException();
    T result = elements[--size];
    elements[size] = null; // 다 쓴 객체 참조 해제로 메모리 릭 방지
    return result;
  }
  
  ...이하 생략
}
```

제네릭으로 바꿔준 것 같지만 당연하게도 컴파일 에러가 한가지 발생한다.

```java
elements = new T[DEFAULT];
```

배열은 제네릭으로 만들때에 문제가 발생하기 때문이다. 

```java
elements = (T[]) new Object[DEFAULT];
```

위와 같이 변환해줄 수는 있으나 비검사 형변환이므로 타입의 안정성을 보장할 수 있기 때문에 @SuppressWarings 어노테이션으로 제네릭으로 만들어 주자.

```java
@SuppressWarnings("unchecked")
public Stack() {
  elements = (T[]) new Object[DEFAULT];
}
```

또 다른 방법은 elements의 타입을 T[]에서 Object[]로 바꿔준다.

```java
T result = (T)elements[--size];
```

T는 실체화 불가 타입이기 때문에 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다. 다만 push 메서드에서 반드시 T타입만 받을수 있도록 강제하고 있기 때문에 경고를 숨겨도 된다. 또한, 전에 배운것 처럼 범위를 최소화 시켜보자.

```java
public T pop() {
    if (size == 0)
      throw new EmptyStackException();
    @SuppressWarings("unchecked")
  	T result = (T)elements[--size];
    elements[size] = null; // 다 쓴 객체 참조 해제로 메모리 릭 방지
    return result;
  }
```

이 두가지 방식중에 일반적으로 첫번째 방식이 더 지지를 많이 얻고 있는데 일단, 코드가 짧고 가독성이 더 높다.

첫번째 방식은 배열 생성시 한 곳에서만 형변환을 해주면 되지만 두번째 방식은 배열에서 원소를 읽을 때 마다 해줘야한다.

하지만 배열의 런타임 타입이 컴파일타입과 달라 힙오염(item 32)를 일으킨다. 이것이 맘에 걸리는 프로그래머는 2번째 방식을 더 선호하곤 한다.

### 전에는 배열보다는 리스트를 추천한다며?

앞선 아이템에서 설명한것과는 상반되게 이번 아이템에서는 배열을 이용한 모습을 보여줬다. 사실 제네릭을 이용한 모든 것이 리스트를 사용하는 것이 항상 가능하지 않고 더 좋다고만도 말할 수 없으며 HashMap 과 같은 제네릭 타입은 성능을 증가시키기 위해 배열을 사용하는 경우도 있다.

### 한정적 타입 매개변수

```java
class DelayQueue<E extends Delayed> implement BlockingQueue<E>
```

위와 같이 올 수 있는 제네릭을 한정지어 받을 수도 있다. 이를 통해 DelayQueue에서 Delayed 클래스의 메서드를 형변환 없이 바로 사용할 수 있으며 형변환 예외도 신경쓰지 않을 수 있따.