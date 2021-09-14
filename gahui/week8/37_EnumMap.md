# 37. ordinal 인덱싱 대신 EnumMap을 사용하라

## 배열/리스트에서 ordinal메서드를 통해 인덱스 가져오는 것이 좋지 않은 이유.
1. 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야한다. (컴파일이 깔끔하지 않다)
2. 정수 값을 정확히 사용한다는 것을 직접 보증해야 한다.



## EnumMap
- 열거 타입 상수를 Key로 매핑하여 사용.
```java
Map<LifeCycle, Set<Plant>> plantsByLifeCycle1 = new EnumMap<>(Plant.LifeCycle.class);
        
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle1.put(lc, new HashSet<>());
for(Plant p : garden)
    plantsByLifeCycle1.get(p.lifeCycle).add(p);

System.out.println(plantsByLifeCycle);
```
- 내부에서 배열을 사용하기 때문에 성능이 좋다.
- 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 높이고 유지보수하기 쉽다.


</br>

## EnumMap과 Stream을 사용하는 경우
1. Stream 사용 - 1
    - EnumMap이 아닌 고유한 맵 구현체를 사용했기에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.
```java
System.out.println(Arrays.stream(garden)
                    .collect(groupingBy(p -> p.lifeCycle));
```

2. Stream 사용 - 2
    - 