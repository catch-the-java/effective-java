# item 73 추상화 수준에 맞는 예외를 던져라

## 추상화 수준에 맞는 예외

수행하려는 일과 관련 없어 보이는 예외가 튀어나오는 경우는 메서드가 저수준의 예외를 처리하지 않고 바깥으로 전파하는 경우이다.

또한 내부 구현 방식을 밖으로 드러내어 윗 레벨 API를 오염시킨다.

이를 피하려면 __상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다. 이를 예외 번역이라 한다.__

```java
try {
	...
} catch (LowerLevelException e) {
  throw new HigherLevelException(...);
}
```



## 예외 연쇄

저수준 예외가 디버깅에 도움이 돈다면 예외 연쇄를 사용하면 좋은데 이는 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식이다.

이는 Throwable 의 getCause 메서드를 통해 보내어 필요하면 꺼내볼 수 있다.

```java
try {
  ...
} catch (LowerLevelException cause) {
  throw new HigherLevelException(cause);
}
```



대부분의 표준 예외는 예외용 생성자를 갖추고 있으며, 그렇지 않은 예외라도 Throwable의 initCause 메서드를 이용해 원인을 직접 못박을 수 있다.



## 남용은 금지!

무턱대고 예외를 전파하는 것보다는 좋은 방법이지만 그렇다고 남용해서는 안된다.

저수준 메서드가 반드시 성공하게 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.



## 차선책

아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법이 있다.

이는 적절히 로깅을 활용하여 기록해 두는 것이 좋다.