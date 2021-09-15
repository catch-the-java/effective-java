# item 38 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴보다 우수하다.

하지만 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다. 사실 열거 타입을 확장하는 건 좋지 않은 생각이다.

확장할 수 있는 열거타입이 어울리는 쓰임이 한가지 있는데 바로 연산코드(operation code 혹은 opcode)이다. 가끔은 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가해야하는 경우가 있다.

열거 타입으로 이 효과를 낼 수 있는데, 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하면 된다.

```java
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
  PLUS("+") {
    public double apply(double x, double y) { return x+y;}
  },
  MINUS("-") {
    public double apply(double x, double y) { return x-y;}
  },
  TIMES("*") {
    public double apply(double x, double y) { return x*y;}
  },
  MINUS("/") {
    public double apply(double x, double y) { return x/y;}
  }
}
```

BasicOperation은 확장할 수 없지만 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다. 이렇게 하면 Operation을 구현한 또 다른 열거타입을 정의해 BasicOperation을 대체할 수 있다. 아래 코드를 보자.

```java
public enum ExtendedOperation implements Operation {
  EXP("^") {
    public double apply(double x, double y) {
      return Math.pow("^");
    }
  },
  REMAINDER("%") {
    public double apply(double x, double y) {
      return x%y;
    }
  }
  
  private final String symbol;
  
  ExtendedOperation(String symbol) {
    this.symbol = symbol;
  }
  
  @Override
  public String toString() {
    return symbol;
  }
}
```

ExtendedOperation의 모든 연산자를 모두 테스트하는 코드를 보자.

```java
public static void main(string[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
  for(Operation op : opEnumType.getEnumConstants()) {
    System.out.println("%f %s %f = %f%n"x, op, y, op.apply(x,y));
  }
}
```

<T extends Enum\<T> & Operation>은 Enum\<T>의 요소이면서 Operation의 하위 타입이여 한다.

또는 Class 객체 대신 한정적 와일드카드 타입인 Collection<? Extends Operation>을 넘기는 방법이다.

```java
public static void main(string[] args) {
  double x = Double.parseDouble(args[0]);
  double y = Double.parseDouble(args[1]);
  test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, double x, double y) {
  for(Operation op : opEnumType.getEnumConstants()) {
    System.out.println("%f %s %f = %f%n"x, op, y, op.apply(x,y));
  }
}
```

인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 사소한 문제가 있는데 열거 타입 끼리 구현을 상속할 수 없다는 점이다.

중복량이 많은 경우는 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 중복을 없앨 수 있을 것이다.

