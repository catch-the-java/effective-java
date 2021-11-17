# item 55 옵셔널 반환은 신중히 하라

## Java8 이전 특정 메서드에서 값을 반환할 수 없을 때

- 예외를 던진다.
- null을 반환한다.

두 가지 방법에서 생기는 허점

- 예외는 진짜 예외적인 상황에서만 사용해야하며 예외를 생성할 때 스택 추적 전체를 캡처하는 비용이 만만치 않다.
- null을 반환할 수 있는 메서드를 호출할 때는 null에 대한 처리를 추가해줘야 한다.
- null처리를 하지 않을 시 어디에선가 NullPointException이 발생할 가능성이 있다.

## Java8 이후 생긴 새로운 선택지

Optional을 이용하여 처리한다.

- Optional\<T>는 null이 아닌 T타입 참조를 하나 담거나, 혹은 하나도 담지 않는다.
- 아무것도 담지 않은 옵셔널을 "비었다"라고 이야기 하며 담은 옵셔널을 "비지 않았다"라고 한다.
- 옵셔널은 원소 1개를 가질 수 있는 "불변" 컬랙션이다.

이는 예외보다 간단하며 null보다 유연하다.

## 신경써줘야 할 점

Optional은 팩터리를 이용하여 옵셔널을 생성해준다.

- 빈값의 경우는 Optional.empty()를 사용하고 비지 않았을 경우 Optional.of()를 사용한다.

- Optional.of의 경우 null을 집어넣으면 NullPointException을 던지기 때문에 null이 올 수 있는 경우에는 Optional.nullalbe()을 사용하자.

- Optional을 반환하는 메서드의 경우 절대로 null을 반환하지 않도록 한다.
- null과 예외 대신 옵셔널 반환을 선택해야 하는 경우는 반환값이 없음을 API 사용자에게 명확히 알려주기 위함이다.

## 클라이언트가 값을 받지 못했을 경우

메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을때 행동을 선택해야한다.

- 기본값을 정해둔다.

  ```
  String listWordInLexicon = max(words).orElse("단어 없음..");
  ```

- 원하는 예외를 던진다.

  ```java
  Toy myToy = max(toys).orElseThrow(RuntimeException::new);
  ```

- 항상 값이 채워져 있다고 가정한다.

  ```java
  Toy myToy = max(toys).get();
  ```

- 기본값 설정이 비용이 아주 커서 부담이 되는 경우에는 Supplier\<T>를 인수로 받는 orElseGet을 사용한다.
  참고 : orElse는 null이 반환되지 않는 경우에도 불리기 때문에 만약에 함수를 넣는 경우에는 조심히 사용해야한다. orElseGet은 null이 아닌 경우에만 불리며 Supplier를 사용한다.

- 이러한 것들을 모두 사용하여도 원하는 동작을 구현하기 힘들다면 ifPresent함수를 이용해 분기 처리 한다.

## Optional Stream

Optional을 사용한다면 옵셔널들을 Stream\<Optional\<T>>로 받아서, 그중 채워진 옵셔널들에서 값을 뽑아 Stream\<T>에 건네 담아 처리하는 경우가 있다.

```java
//Optional에 값이 있으면 Optional에서 값을 꺼내 스트림에 매핑한다.
streamOfOptionals.filter(Optional::isPresent).map(Optional::get);
```

Java9 부터 Optional에 stream() 메서드가 추가되었다. 이 메서드는 Optional을 Stream으로 변환해주는 어댑터이다.

옵셔널에 값이 있으면 스트림을 반환해주고, 없으면 빈 스트림으로 변환해준다.

## 주의할점

- 컬랙션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
  (컨테이너 자체로 처리할 수 있기 때문)
- 결과가 없을 수 있을 경우 메서드 타입을 T 대신 Optional\<T>를 반환한다고 생각하면 된다.
- Optional도 새로 할당하고 초기화해야 하는 객체이기 때문에 비용이 든다. 따라서 성능이 크리티컬한 경우는 사용하지 않는다.
- Integer, Boolean과 같이 박싱된 객체를 위한 OptionalInt와 같은 것이 존재하기 때문에 박싱된 기본 타입을 반환하는 옵셔널은 지양한다.
  (단 Bollean, Character 등과 같이 덜 중요한 기본 타입용은 예외일 수 있다.)
- 옵셔널을 Key값으로 사용하는 경우는 없어야 한다.(Key값이 존재할 수도 없을수도 있기 때문) 

