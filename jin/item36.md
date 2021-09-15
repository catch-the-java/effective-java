# item36 비트 필드 대신 EnumSet을 사용하라

## 이전 비트 필드 열거 상수 패턴

열거한 값들이 주로 집합으로 사용될 경우, 이전에는 서로 다른 2의 거듭제곱 꼴로 나타냈다.

```java
public class Text {
  public static final int STYLE_BOLD = 1<<0;
  public static final int STYLE_ITALIC = 1<<1;
  public static final int STYLE_UNDERLINE = 1<<2;
  public static final int STYLE_STRIKETHROUGH = 1<<3;
  
  public void applyStyles(int styles) {...}
}
```

위와 같이 나타내면 |(or)로 여러 상수를 하나의 집합으로 모을 수 있으며 이렇게 만들어진 집합을 비트 필드(bit field)라 한다. 이를 이용하면 합집합, 교집합과 같은 집합 연산을 효율적으로 수행할 수 있다. 하지만 정수 열거 상수의 단점을 그대로 지니며 다음과 같은 추가적인 문제를 가진다.

1. 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.
3. 최대 몇 비트가 필요한지를 API 작성시 미리 예측하고 자료형을 결졍해야한다.(한번 결정하면 API가 변경되어야 하는데 그것은 쉽지 않다.)



## EnumSet 클래스

비트 필드 열거 상수 패턴을 대체할 수 있는 대안이 추가되었는데 바로 EnumSet 이다.

1. Set 인터페이스를 완벽히 구현한다.
2. 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.
3. EnumSet의 내부는 비트 벡터로 구현되었다.

원소가 총 64개 이하라면 즉, 대부분의 경우에는 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드와 비슷한 성능을 낼 수 있다.

(removeAll과 retainAll과 같은 대량 작업은 비트를 효율적으로 처리할 수 있는 산술 연산자를 이용하여 구현하였다.)

위의 예제를 EnumSet을 이용하여 수정하였다.

```java
public class Test {
  public enum Sytle {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}
  
  public void applyStyles(Set<Style> styles) {...}
}

Text text = new Text();
text.applyStyles(EnumSet.of(Style.BOLD, STYLE.ITALIC));
```

applyStyles가 EnumSet\<Style>가 아닌 Set\<Style>을 받은 이유는 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이기 때문이다.

(클라이언트가 다른 Set을 넘겨도 처리할 수 있기 때문)

