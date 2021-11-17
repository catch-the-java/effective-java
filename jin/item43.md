# item 43 람다보다는 메서드 참조를 사용하라

## 람다의 장점이 뭐였지?

람다가 익명함수보다 나은 점 중 가장 큰 특징은 간결함이다. 앞서서 본것과 같이 메서드 참조를 이용하면 더 코드를 간결하게 만들 수 있다.

```java
map.merge(key, 1, (count, incr) -> count + incr);
//메서드 참조를 통해 간결하게 만들기
map.merge(key, 1, Integer::sum)
```

하지만 매개변수 이름을 통해 다른 프로그래머가 코드를 더 읽기 쉽게 해주는 가이드가 될 수 있으므로 잘 선택 해야한다.

## 메서드 참조의 장점

람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다. 하지만 메서드 참조를 통해 코드를 더 짧게 만들 수 있기 때문에 람다로 구현한 것이 너무 길어진다면 메서드 참조를 통해 간단하게 만들어주며 메서드 이름을 통해 메서드가 기능을 잘 드러내는 이름을 지어줄 수 있다.

IDE는 아마도 메서드 참조로 바꾸는 것을 추천할 것이지만 무조건 좋은 것은 아니다. 다음 예시를 보자.

```java
service.excute(GoshThisClassNameIsHumongous::action);
//람다로 대체하면
service.execute(()->action());
```

메서드 참조가 더 길고 읽기도 힘들다.

메서드 참조를 하는 유형은 다섯 가지이다. 다음과 같이 표로 정리된다.

---

| 메서드 참조 유형   | 예                     | 같은 기능을 하는 람다                                  |
| ------------------ | ---------------------- | ------------------------------------------------------ |
| 정적               | Integer::parseInt      | str -> Integer.parseInt(str)                           |
| 한정적(인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now()<br />t -> then.isAfter(t) |
| 비한정적(인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                               |
| 클래스 생성자      | TreeMap<K,V>::new      | () -> new TreeMap<K,V> ()                              |
| 배열 생성자        | int[]::new             | len -> new int[len]                                    |

이 중 생성자 참조는 팩터리 객체로 사용된다.