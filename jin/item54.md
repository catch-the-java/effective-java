# item 54 null이 아닌, 빈 컬렉션이나 배열을 반환하라

다음 코드는 재고가 없다면 Null을 반환하는 메서드이다.

```java
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? null
    : new ArrayList<>(cheesesInStock);
}
```

이 코드는 사실 좋은 코드는 아닌데 null이라고 특별 취급해줄 필요가 없기 때문이다. null을 리턴 해준다면 null을 처리해주기 위한 코드를 별도로 넣어줘야 하기 때문이다. 이러한 방어 코드를 추가적으로 작성해줘야 하기 때문에 실수로 빼먹는다면 nullpointexception을 발생시킬 수 있다.

하지만, null을 반환하는 것이 빈 컨테이너를 반환하는 것보다 할당에 대한 오버헤드가 줄어 더 좋다고 생각 하는 사람이 있지만 이는 두가지에서 틀린 주장이다.

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는한 이 성능 저하는 신경쓸 정도의 성능 차이가 발생하지 않는다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다. 다음은 빈 컬랙션을 반환하는 전형적인 코드로 대부분의 상황에서는 이렇게 하면 된다.

```java
public List<Cheese> getCheeses() {
  return new ArrayList<>(cheesesInStock);
}
```

이도 사용 방법에 따라 성능 저하를 발생시킬 수 있지만 이럴때에는 빈 불변 컬랙션을 반환한다. Collections.emptyList, Collections.emptySet과 같은 메서드가 그러하다. 이는 성능 최적화를 위한 것이며 반드시 성능 체크를 하기 바란다.

```java
public List<Cheese> getCheeses() {
  return cheesesInStock.isEmpty() ? Collections.emptyList()
    :new ArrayList<>(cheesesInStock);
}
```

배열도 마찬가지로 빈배열을 반환해주자.

```java
//배열도 마찬가지로 빈 배열을 반환하자.
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(new Cheese[0]);
}
```

이 방식이 성능을 떨어트릴 것 같다면 미리 배열을 만들어 두고 그 배열을 반환한다.(길이 0배열은 모두 불변이기 때문)

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```

하지만 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 건 추천하지 않는다.

```java
return cheesesInStock.toArray(new Cheese[cheesesInStock.size()]);
```

