# item 61 박싱된 기본 타입보다는 기본 타입을 사용하라.

## 자바의 데이터 타입

1. 기본 타입 : int, long, float, double. boolean 등
2. 참조 타입 : String, List 등
3. 각 기본 타입에 대응되는 참조 타입 : Integer, Long, Double, Boolean 등

## Auto boxing

primitive type을 그에 상응하는 wrapper class로 자동 변환 시켜 주는 것을 의미한다.

이는 자바 complier가 자동으로 형 변환 시켜준다.

## 두가지의 차이점

1. 기본 타입은 값만 가지고 있지만 Wrapper type은 값에 더해 식별성 이라는 속성을 가진다. 따라서 같은 Integer라도 서로 다른 객체로 인식될 수 있다.
2. 기본 타입은 언제나 유효하지만 wrapper type은 null을 가질 수 있다.
3. 기본 타입이 wrapper type보다 메모리 면에서 효율적이다.

이러한 차이점을 생각하지 않고 개발을 하면 진짜 큰 문제를 일으킬 수 있다.

## 발생하는 문제점

1. Integer의 값을 비교해 정렬하는 예시를 보자.

   ```java
   Comparator<Integer> naturalOrder = (i, j) -> (i < j)?-1:(i==j ? 0 : 1);
   ```

   잘 될것 같지만 Integer(42)와 Integer(42) (두개는 다른 객체)가 있을때 첫번째 조건은 잘 통과하지만 i==j는 다르다고 판단하게 된다.

   (다른 객체이기 때문에)

   따라서, wrapper type은 \"==" 연산자를 사용하면 오류가 발생한다. 이를 해결하기 위해서는 comparator를 직접 구현해줘야 한다.

2. Nullpointer Exception이 발생할 수 있다.
3. 기본 타입과 wrapper type을 혼동하여 사용하면 자동으로 unboxing, boxing이 반복적으로 일어나 성능이 굉장히 느려질 수 있다.

## 그럼에도 불구하고 쓰는 이유는?

- 컬렉션의 원소, 키, 값으로 쓴다. 컬랙션은 primitive type을 담을 수 없기 때문이다.
  매개변수화 타입이나 매개변수화 메서드의 타입 매개 변수로는 박싱된 기본 타입을 써야한다.
- 리플렉션(item 65)를 통해 메서드를 호출할 때도 wrapper type을 사용해야 한다.



