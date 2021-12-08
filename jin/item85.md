# item 85 자바 직렬화의 대안을 찾으라

## 자바 직렬화는 위험하다

- 자바의 직렬화와 역직렬화에 의해서 발생한 보안 공격들이 존재하였음. 
- 널리 쓰이는 서드파티 라이브러리에서 직렬화 가능 타입들을 연구하여 역질렬화 과정에서 호출되어 잠재적으로 위험한 동작을 수행하는 메서드들이 존재하며 이를 __가젯__이라 한다.
- 역직렬화 폭탄의 예는 아래 코드와 같다.

```java
    static byte[] bomb() {
        Set<Object> root = new HashSet<>();
        Set<Object> s1 = root;
        Set<Object> s2 = new HashSet<>();
        for (int i = 0; i < 100; i++) {
            Set<Object> t1 = new HashSet<>();
            Set<Object> t2 = new HashSet<>();
            t1.add("foo"); // make it not equal to t2
            s1.add(t1);
            s1.add(t2);
            s2.add(t1);
            s2.add(t2);
            s1 = t1;
            s2 = t2;
        }
        return serialize(root);
    }
```

 위 코드를 역직렬화 하면 영원히 수행하게 되며 시스템이 마비되게 된다.



## 해결방법

- 가장 좋은 방법은 아무것도 역직렬화 하지 않는 것이다. 우리가 새로운 시스템에서 자바 직렬화를 써야 할 이유는 없다.
- 레거시 시스템 때문에 직렬화를 배재할 수 없다면 __신뢰할 수 없는 데이터는 절대 역직렬화하지 않는것이다.__
- 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 확신할 수 없다면 역직렬화 필터링을 사용하자.(Java 9에 추가)
  특히 Black List 방식보다는 White List 방식을 더 추천한다.
- 화이트리스트를 자동으로 생성해주는 SWAT이라는 도구도 있으니 참고하자. 메모리를 과하게 사용하거나 객체 그래프가 너무 깊어지는 사태로부터 보호해 주지만 앞선 직렬화 폭탄은 걸러내지 못한다.



## 결론

Json이나 프로토콜 버퍼 같은 대안을 사용하자.