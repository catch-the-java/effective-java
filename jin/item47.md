# item 47 반환 타입으로는 스트림보다 컬렉션이 낫다

## Stream은 Iteration을 지원하지 않는다.

자바7까지는 일련의 원소를 반환하기 위해 컬렉션 인터페이스, Iterable, 배열을 사용했지만 스트림이 등장한 후 스트림을 사용해야 한다.

하지만 스트림은 반복(iteration)을 지원하지 않기 때문에 스트림과 반복을 적절하게 잘 조합해 사용해야 한다.

foreach로 스트림을 반복할 수 없는 이유는 Stream이 Iterable을 확장하지 않아서이다.

다음 코드는 동작할것 같지만 오류를 뱉어내며 해결 책도 좋은 방법이 아니다.

(Method는 저 위치에 올 수 없다.)

```java
for (ProcessHandle ph : ProcessHandle.allProcess()::iterator) {
  
}

for (ProcessHandle ph : (Iterable<ProcessHandle>)ProcessHandle.allProcess()::iterator) {
  
}
```

어댑터 매서드를 사용해보자.

```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  
}

for(ProcessHandle p : iterableOf(ProcessHandle.addProcesses())) {
  
}
```

Iterable만 반환하는 API를 스트림 파이프라인으로 처리하고 싶다면 다음과 같이 어댑터를 구현할 수 있다.

```java
public static <E> Stream<E> streamOf(IOterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```



## Collection 인터페이스

Collection은 Iterable의 하위 타임이고 stream 메서드 또한 동시에 제공한다.

__따라서, 원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 사용하는 것이 최선이다.__

만약 원소의 개수가 적은 시퀀스를 반환하는 경우라면 표준 컬랙션에 담고 큰 경우에는 전용 컬랙션을 구현한다.

멱집합의 경우 2^n개 만큼 생기기 때문에 원소의 개수가 매우 크다.

```java
public class PowerSet {
    // Returns the power set of an input set as custom collection (Page 218)
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException("Set too big " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                return 1 << src.size(); // 2 to the power srcSize
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }

```



## 더 구현이 쉬운 쪽을 선택

이렇게 모든 것을 구현할 수도 있지만 구현이 쉬운 쪽을 선택할 수도 있다.(다만 입력 리스트의 크기가 너무 크다면 메모리를 너무 많이 차지하게 된다.)

부분리스트 중 프리픽스와 서브픽스를 반환하는 함수를 구현해보자. 

```java
public class SubLists {
    // Returns a stream of all the sublists of its input list (Page 219)
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }

    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
}
```



## 결론

위와 같이 하면 반복이 더 편한 상황에서도 사용자는 강제로 Stream을 쓰거나 스트림을 Iterable로 변경해주는 어댑터를 사용해야 한다.

이는 코드가 읽기 더 불편하며 속도도 느리다.

만약 컬랙션을 반환하는 것이 불가능 하다면 스트림과 Iterable 중 더 자연스러운 것을 골라야 한다.

