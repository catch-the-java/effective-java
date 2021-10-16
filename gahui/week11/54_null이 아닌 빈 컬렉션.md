## 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라


### 반환 값이 Null인 예시
- cheese가 없으면 null을 반환
- 작성하면 안되는 이유
    - null을 반환하면, 클라이언트는 null상황을 처리하는 코드를 추가로 작성해야 한다.
    - 써주지 않으면, 오류 발생 가능성
```java
private final List<Cheese> cheesesInStock = new ArrayList<>();
    
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock);
}
```

</br>
</br>

### null 반환이 낫다????
> 틀린 주장이다.
1. 빈 컬렉션은 성능 저하의 주범이다? -> 성능차이는 신경 쓸 수준이 못된다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

```java
//빈 컬렉션을 반환하는 올바른 예
 public List<Cheese> getCheesesInStock() {
        return new ArrayList<>(cheesesInStock);
    }
```

</br>
</br>

### 빈 컬렉션 반환
- 사용 패턴에 따라서 성능을 떨어뜨릴 수 있다.
    - 불변 컬렉션을 반환 (ex. Collections.emptyList, Collection.emptySet, Collection.emptyMap)

- 최적화 : 불변 컬렉션 반환
    - 최적화에 해당하니 꼭 필요할 때만 사용하자.
```java
public List<Cheese> getCheese() {
        return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
    }
```

## 빈 배열 반환
> 절대 null을 반환하지 말고 길이가 0인 배열을 반환하자.

- 예시 : 길이가 0일수도 있는 배열 반환
```java
public Cheese[] getCheeseArray() {
        return cheesesInStock.toArray(new Cheese[0]);
    }
```

- 최적화 : 빈 배열을 매번 새로 할당하지 않는다.
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeseArray() {
        return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
    }
```


</br>
</br>




