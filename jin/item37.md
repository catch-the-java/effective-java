# item 37 ordinal 인덱싱 대신 EnumMap을 사용하라

간혹 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다.

다음 코드를 보자.

```java
class Plant {
  enum LifeCycle {ANNUAL, PERNNIAL, BIENNIAL}
  
  final String name;
  final LifeCycle lifeCycle;
  
  Plant(String name, LifeCycle lifeCycle) {
    this.name = name;
    this.lifeCycle = lifeCycle;
  }
  
  @Override
  public String toString() {
    return name;
  }
}
```

식물들을 배열 하나로 관리하고, 이들을 생애주기별로 묶는 예시를 보자.

```java
Set<Plant> plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

for(int i = 0; i < plantByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>(); //각 배열 인덱스에 Set을 할당

for(Plant p : garden)
  plantByLifeCycle[p.lifeCycle.ordinal()].add(p);//ordinal 메서드로 값을 인덱스 값을 얻음

for(int i = 0; i<plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n",Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

위 코드의 문제가 존재한다.

1. 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야하고 컴파일되지 않음
2. 배열은 각 인덱스의 의미를 모르기 때문에 출력 결과에 직접 레이블을 달아야한다.
3. (가장 심각) 정확한 정숫값을 사용한다는 것을 개발자가 보증해야한다. 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다.

이를 한번에 해결할 수 있는 방법은 EnumMap을 이용한다.



## EnumMap

열거타입을 키로 사용하도록 설계한 아주 빠른 Map의 구현체이다.

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lc : Plant.LifeCycle.values()) plantsByLifeCycle.put(lc, new HashSet<>());

for (Plant p : garden)
  plantsByLifeCycle.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```

위 코드는 성능은 비슷하지만 안전하지 않은 형변환이나 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공한다.

내부 구현은 배열로 되어 있으며 앞서서 ordinal 메서드를 설명할때 EnumMap을 언급하였다.

EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로 런타입 제네릭 타입 정보를 제공한다.

스트림을 사용하면 코드가 더 짧아진다.

```java
System.out.println(Arrays.stream(garden).collect(groupingBy(p->p.lifeCycle)));
```

이 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간적인 이점이 사라진다. 

이것을 다음과 같이 EnumSet을 사용하는 것으로 명시할 수 있다.

```java
System.out.println(Arrays.stream(garden)
                   .collect(groupingBy(p->p.lifeCycle,
                                      ()->new EnumMap<>(LifeCycle.class), toSet())));
```

스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다.

EnumMap은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만든다. 하지만 스트림 버전에는 해당 생애주기에 속하는 식물이 있을 때만 만든다.

예를들어 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개 만들고(LifeCycle 열거 타입의 요소가 3개니까) 스트림은 2개만 만든다.



두 열거타입을 매핑하기 위해 ordinal을 사용하는 경우를 보자.

```java
public enum Phase {
  SOILD, LIQUID, GAS;
  
  public enum Transaction {
    MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
    
    private static final Transaction[][] TRANSITIONS = {
      {null, MELT, SUBLIME},
      {FREEZE, null, BOIL},
      {DEPOSIT, CONDENSE, null}
    };
    
    public static Transition from(Phase from, Phase to) {
      return TRANSITIONS[from.ordinal()][to.oridinal()];
    }
  }
}
```

이 예제는 컴파일러는 oridinal과 배열 인덱스 사이의 관계를 알 수 없어 문제가 발생할 수 있다. 예를들어 Phase 열거타입의 순서가 수정됐을때 TRANSITIONS 배열도 같이 바뀌지 않으면 문제가 생길 수 있다. 또한 상태가 늘어나면 늘어날수록 TRANSITIONS 배열의 크기도 제곱으로 늘어난다.

즉, 이러지 말고 Enum을 쓰자.

```java
public enum Phase {
  SOLID, LIQUID, GAS;
  
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOILD);
  }
  
  private final Phase from;
  private final Phase to;
  
  Transition(Phase from ,Pahse to) {
    this.from = from;
    this.to = to;
  }
  
  private static final Map<Phase, Map<Phase, Transition>> m = 
    Stream.of(values()).collect(groupingBy(t -> t.from, 
                                           ()->new EnumMap<>(Phase.class),
                                          toMap(t->t.to, t->t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));
  
  public static Transition from(Phase from, Phase to) {
    return m.get(from).get(to);
  }
}
```

자 이 코드에 Plazma라는 상태를 추가해보자. IONIZE는 가스에서 -> 플라즈마 DEIONIZE는 플라즈마 -> 가스상태로 전이하는 것을 의미한다.

이것을 수정하는 것은 어렵지 않다.

```java
public enum Phase {
  SOLID, LIQUID, GAS, PLAZMA;//플라즈마 추가
  
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOILD),
    IONIZE(GAS, PLASMA), DEIONZE(PLASMA, GAS);//두가지 상태 전이를 추가
  }
```

EnumMap을 통하여 간단하게 표현이 가능하다.