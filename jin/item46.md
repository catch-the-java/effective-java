# item 46 스트림에서는 부작용 없는 함수를 사용하라

## 함수형 프로그래밍

스트림을 처음봐서는 이해가 쉽지 않을 것이다. 파이프 라인을 원하는 기능으로 표현하기도 쉽지 않을 수 있으며 장점도 못느낄 수 있다. 

스트림은 또 하나의 API만이 아니라 함수형 프로그래밍에 기초한 패러다임이다. 그러므로 스트림의 장점인 표현력, 속도, 병렬성 등을 얻기 위해서는 함수형 프로그래밍 패러다임을 받아들여야 한다.

## 스트림 패러다임

스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성 하는 부분이다.

이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다. 

1. 순수 함수란 오직 입력만이 결과에 영향을 주는 함수이다.

2. 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태의 변화를 주지 않아야 한다.
3. __이 모든 것을 위해서는 함수 객체는 모두 부작용이 없어야 한다.__

예시를 보자

우선은 잘못된 예시이다.

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
	});
}
```

위 코드의 문제점은 스트림 코드를 가장한 반복코드라는 것이다. 모든 연산인 foreach에서 이루어 지는데 freq에 있는 빈도수를 수정한다. 이는 별로 좋지 않다.

```java
Map<String, Long> feq;
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

위 함수는 그룹핑을 하고 종단에 모아주며 코드도 읽기 쉬워 스트림의 장점을 잘 살리고 있다.

하지만 Java에 있는 for-each문과 비슷한 foreach 메서드가 더 편하게 보일 수 있지만 foreach는 가장 덜 스트림답고 너무 반복적이어서 병렬화를 할 수 없다.

그러므로 __foreach는 스트림의 결과를 보고할때만 사용하도록 하고 계산 하는 곳에는 사용하지 말자.__

위 코드는 collector 클래스를 사용하는데 여기서는 축소 전략을 캡슐화한 블랙박스 객체라고 생각하자. 여기서 축소는 원소들을 객체 하나에 취합해준다는 뜻이다. 수집기가 일반적으로 반환하는 객체가 Collection이기 때문에 collector라는 이름을 쓴다.

```java
List<String> topTen = freq.keySet().stream()
  .sorted(comparing(freq::get).reversed())
  .limit(10)
  .collect(toList());
```

comparing은 한정적 메서드로 키 추출 함수로 쓰인다. 입력받은 단어를 빈도표에서 찾아 그 빈도를 반환한다. 그 후 reverse로 순서를 뒤집어 준 후 10개 순서로 뽑아준다.

### 다양한 Collector 인터페이스

#### toMap

```java
Map<String, Operation> stringToEnum = Stream.of(values())
  .collect(toMap(Object::toString, e->e));
```

toMap 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다. 또한 toMap에 키 매퍼와 값 매퍼는 물론 병합 함수까지 지원할 수 있다.

이를 3개의 요소를 받아 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관짓는 맵을 만들 때 유용하다.

```java
Map<Artist, Album> toHits = albums
  .collect(toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```

maxBy는 Comparator\<T>를 받아 BinaryOperator\<T>를 돌려준다.

다음은 toMap을 이용하여 충돌이 나면 마지막 값을 취하는 수집기를 만들 수 있다.

```java
toMap(keyMapper, valueMapper,(oldVal, NewVal) -> newVal)
```

마지막으로 toMap은 네번째 인수로 맵 팩터리를 받는다. 그리하여 특정 맵(TreeMap, EnumMap  등)을 직접 지정해 줄 수 있다.

#### groupingBy

1. 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵을 반환한다. 반환된 맵에 담긴 각각의 값은 해당 카테코리에 속하는 원소를 모두 담은 리스트이다.

```java
words.collect(groupingBy(word->alphabetize(word)))
```

2. groupingBy를 이용해 리스트 외의 값을 갖는 맵을 생성하려면 분류 함수와 함께 다운 스트림 수집기도 명시해야 한다.

스트림 수집기의 역할은 해당 카테고리의 원소를 담은 스트림으로부터 값을 생성하는 일이다. 리스트가 아닌 toSet()을 이용하여 모으는것이 가장 쉽다. counting()을 다운스트림 수집기로 건내 해당 카테고리에 속하는 원소의 개수와 매핑한 값을 얻을 수 있다.

3. 다운스트림 수집기에 더해 맵 팩터리도 지정할 수 있게 해준다. 이것은 점층적 인수 목록 패턴에 어긋나며 이를 이용해 맵과 그 안에 담긴 컬렉션 타입을 모두 지정할 수 있다.(mapFactory 매개변수가 다운스트림보다 앞에 오기 때문) 예를들어 값이 TreeSet인 TreeMap을 반환할 수 있다.

#### minBy, maxBy

인수로 받은 비교자를 이용해 스트림에서 값이 가장 작은 혹은 가장 큰 원소를 찾아 반환한다. Stream인터페이스의 min과 max를 가장 살짝 일반화한 형태이며 java.util.funtion.BinaryOperator의 minBy와 maxBy 메서드가 반환하는 이진 연산자의 수집기 버전이다.

#### joining

CharSequece 인스턴스의 스트림에만 적용할 수 있다. 이 중 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다. 이를 통해 스트림 중간에 구분자를 입력해 줄 수 있다. 예를들어 csv파일처럼 중간에 ","를 넣어줄 수 있다. 인수가 3개라면 prefix, postfix, 구분자를 넣어줄 수 있다. 