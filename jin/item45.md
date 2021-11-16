# item 45 스트림은 주의해서 사용하라

## 스트림

스트림 API는 다량의 데이터 처리 작업을 돕고자 Java8에 추가되었다. 이 API가 추상 개념 중 핵심은 두가지이다.

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
2. 스트림 파이프라인은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

### 스트림 원소의 근원

- 어디로부터든 올 수 있다. 컬랙션, 배열, 파일, 정규표현식 패턴 매처, 난수 생성기, 등

### 스트림 안의 데이터 원소

- 객체 참조나 기본 타입 값이다. 기본 타입 값으론 int, long, double 세가지를 지원한다.

### 스트림 파이프 라인의 과정

소스 스트림 -> 중간연산(하나 이상)-> 종단연산

중간 연산은 스트림을 어떤 방식으로든 변환한다. 변환 전과 후는 같을 수도 다를 수 도 있다.

종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다. 원소를 정렬해 컬랙션에 담거나, 특정 원소를 선택하거나 모든 원소를 출력한다.

### 지연평가(Lazy Evaluation)

스트림 파이프 라인은 지연 평가가 된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터는 계산에 쓰이지 않는다.

이 지연 평가를 이용해 무한 스트림을 다룰 수 있게 해준다. 또한, 종단 연산이 없다면 아무일도 하지 않는 것과 같으니 빼먹지 않도록 하자.

### 플루언트 API

스트림은 연쇄를 지원하는 플루언트 API로 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 즉, 파이프라인 여러개를 연결해 표현식 하나로 만들 수 있다. 파이프라인은 순차적으로 수행되며 Parallel 메서드를 이용하여 병렬적으로 처리가 가능하다.

### 스트림을 잘 사용하기

스트림을 잘 사용하면 코드가 짧고 깔끔해지지만 잘못 쓰면 오히려 읽기 힘들어 진다. 잘 쓰는 방법을 알아보자.

다음 코드는 스트림을 사용하지 않고 짠 코드이다.

```java
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
```

computeIfAbsent를 이용해 쉽게 맵핑을 해주는 모습을 볼 수 있지만 다소 길다.

다음은 스트림을 과도하게 쓴 예시이다.

```java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
```

딱봐도 읽기 힘들다. 만약 스트림에 익숙하지 않다면 더욱 읽기 쉽지 않다. 스트림안에 스트림이 들어가 읽기 쉽지 않다.

```java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```

위 코드는 try-with-resource 블록에서 사전 파일을 열고 스트림으로 만들어 준다. 이 스트림에서 중간 연산은 없고 종단에서 map 메서드를 이용하여 모아준다. 함수와 함께 사용하여 더 깔끔해진 모습을 볼 수 있다.

alphabetize함수도 스트림을 이용할 수 있지만 오히려 읽기만 힘들어진다. 다음 코드를 보면

```java
"Hello World".chars().forEach(System.out::println);
```

각 알파벳이 나올것 같지만 72101108...을 출력한다. chars는 아스키 코드 값을 리턴하기 때문이다.

쓰고 싶으면 형변환을 명시적으로 써야하는데 char형은 웬만하면 쓰지 말도록 하자.

### 스트림을 꼭 사용해야하는 경우는?

스트림을 처음 배우면 모든 반복문을 스트림으로 바꿀 수 있을 것 같지만 천천히 해야한다. 오히려 코드 가독성이나 유지보수성에서 손해를 복 가능성이 있기 때문에 __기존 코드는 스트림을 사용하도록 리팩토링을 하되, 새 코드가 더 나아 보일 때만 반영하도록 하자__

### 함수 객체로는 할 수 없지만 코드 블록으로는 할 수 있는 일들과 스트림에 어울리는 작업들

함수 객체로 할 수 없는 일

1. 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다. 하지만 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있고 지역변수 수정은 불가능하다.
2. 코드 블록에서는 return이나 break, continue와 같은 것으로 블록을 제어할 수 있지만 람다는 할 수 있는게 없다.

스트림에 어울리는 작업

1. 원소들의 시퀀스를 일관되게 변경한다.
2. 원소들의 시퀀스를 필터링한다.
3. 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.
4. 원소들의 시퀀스를 컬렉션에 모은다.
5. 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

스트림은 또한 한 데이터가 여러단계를 거치지만 동시에 접근해야 하는 경우는 불가능하다.(다음 스트림으로 넘어가면 값을 잃기 때문)

매핑 객체가 필요한 단계가 여러 곳이라면 코드 양도 많고 지저분해져 스트림을 쓰는 이유가 없어진다.

메르센 소수에 대한 예제는 무한 스트림에 대한 예제이다. 메스센 소수가 무엇인지는 설명하지 않고 코드만 봅시다.

```java
public static void main() {
  primes().map(p-> TWO.pow(p.intValueExact()).substract(ONE))
    .filter(mersenne->mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}
```

우리가 원하는대로 잘 구현되어있다. 그리고 p를 띄워주고 싶다면 foreach로 하면 될 것 같지만 첫번째 중간 연산에서만 p가 나타나기 때문에 단순히는 못나타 낸다. 하지만 비트만 세면 되기 때문에 종단 연산을 다음과 같이 바꾸면 출력이 쉽게 가능하다.

```java
.forEach(mp -> System.out.println(mp.bitlenth() + ": " + mp));
```

## 결론

스트림과 반복문 중 어떤 것을 써야할 지 애매한 경우들이 왕왕 존재한다. 이런 경우에는 사람마다 다르기 때문에 성향에 잘 맞게 사용하자.