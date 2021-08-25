# item 28 배열보다는 리스트를 사용하라

## 배열과 제네릭 타입의 차이점

1. 배열은 공변이다.
   배열은 Sub가 Super의 하위타입이라면 Sub[]는 Super[]의 하위 타입이 된다. 즉, 변하면 같이 변한다는 이야기이다. 
   하지만, 제네릭은 불공변이기 때문에 List\<Sub> list1과 List\<Super> 는 서로 상위, 하위타입이 아니다. 이것이 왜 중요할까?
   예제를 보자.

   ```java
   Object[] objectArray = new Long[1];
   objectArray[0] = "Test Object";// ArrayStoreException을 던진다.
   ----------
   List<Object> ol = new ArrayList<Long>(); //이 코드부터 컴파일 에러가 난다.
   o1.add("Test Object");
   ```

   위 두 코드의 차이가 무엇일까?

   바로 컴파일 과정에서 에러가 나타는 것과 아닌것의 차이이다.

   코드 안정성을 따져보면 컴파일 과정에서 에러가 들어나는 편이 더 고치기 쉽다.

2. 배열은 실체화가 된다.

   배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 그래서 Long에다 String을 집어넣으려고 하면 에러가 발생하는 것이다.

   반면 제네릭은 런타임 시점에는 컴파일 타임에만 검사를 하며 런타임에는 소멸되기 때문에 알 수 없게 된다. 이 소거를 통해서 전의 레거시 코드와 제네릭 타입을 함꼐 사용할 수 있게 해주는 메커니즘으로, 자바 5가 순조롭게 제네릭으로 바뀔 수 있도록 도왔다.

이 두가지 차이점 때문에 배열과 제네릭은 어울리지 못한다. 예를들어 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없다.

List\<String> a = new List\<String>[5]; 와 같이 생성하면 컴파일 중 에러가 발생한다.

이렇게 막은 이유는 컴파일 과정에서 ClassCastException이 발생하는 일을 막겠다는 제네릭의 의미가 퇴색되기 때문이다. 된다고 가정하고 다음 예시를 보자.

```
List<String>[] stringList = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
object[0] = intList;
String s = stringList[0].get(0);
```

만약 첫번째 줄이 허용된다고 치면 컴파일 시점에는 에러가 발생하지 않고 런타임에 에러가 발생하게 된다. 즉, 우리가 원하는 형변환 예외를 컴파일 타임에 잡아낼 수 없게 된다는 이야기이다.

### 실체화 불가 타입

E, List\<E>, List\<String> 같은 타입을 실체화 불가 타입(non-reifiable type)이라 한다. 쉽게 말해, 실체화되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다. 소거 매커니즘으로 인해 실체화가 가능한 타임은 List\<?>와 같은 비한정적 와일드카드 뿐이다. 배열도 마찬가지로 비한정적 와일드 카드로 만들어 줄 수 있지만 실제로 응용되는 경우는 많지 않다.

배열을 제네릭으로 만들 수 없어 귀찮은 경우가 있는데 이것을 해결해 주는 방법은 item 33에서 설명하니 넘어간다.

### 배열에서 발생하는 문제를 List로 해결하자

해열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우는 대부분 배열인 E[] 대신 컬랙션인 List\<E>를 사용하면 해결되는 경우가 대부분이다. 코드가 조금 복잡해지고 성능이 떨어질 수 있으나 타입 안정성과 상호운용성이 좋아지게 된다.

생성자에서 컬랙션을 받아 배열로 변환하는 다음 예시를 살펴보자.

```java
public class Chooser {
  private final Object[] choiceArray;
  
  public Chooser(Collection choices) {
    choiceArray = choices.toArray();
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 변경해줘야 한다. 만약 다른 타입의 원소가 들어온다면 런타임에 형변환 오류가 발생할 위험이 존재한다. 따라서 제네릭으로 만들어 보자.

```java
public class Chooser<T> {
  private final T[] choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceArray = choices.toArray();
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray[rnd.nextInt(choiceArray.length)];
  }
}
```

이렇게 바꿔줘도 워닝이 뜬다. 왜냐면 제네릭에서는 원소의 타입 정보가 런타임에 소거되어 알 수 없게되기 때문이다. 따라서 이런 경우에는 배열로 바꾸면 모두 해결이 된다.

```java
public class Chooser<T> {
  private final ArrayList<T> choiceArray;
  
  public Chooser(Collection<T> choices) {
    choiceArray = new ArrayList<>(choices);
  }
  
  public Object choose() {
    Random rnd = ThreadLocalRandom.current();
    return choiceArray.get(rnd.nextInt(choiceArray.size()));
  }
}
```

위에서 이야기 한 것 처럼 코드가 조금 길어지고 성능상의 손실이 조금 있을 수 있지만 형변환 에러가 발생할 확률이 적어지니 가치가 있다.