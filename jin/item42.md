# item 42 익명 클래스보다는 람다를 사용하라

## 함수 객체

자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용했다. 이런 인터페이스의 인스턴스 함수를 함수 객체(또는 함수형 인터페이스)라고 한다.

함수 객체를 만드는 주요 수단은 익명 클래스를 사용하였지만 이는 구형 방식이다. 아래 코드를 확인해보자

```java
Collections.sort(words, new Comparator<String>() {
            public int compare(String s1, String s2) {
                return Integer.compare(s1.length(), s2.length());
            }
        });
```

하지만 코드가 너무 길어지게 되어 함수형 프로그래밍에는 적합하지 않다. 이애 함수 객체는 람다식을 사용해 표현할 수 있게 되었다.

```java
Collection.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length());
```

컴파일러가 알아서 타입을 추론해주며 __타입을 명시해야 코드가 더 명확할 때만 제외하고는, 람다의 모든 매개변수 타입은 생략하자.__

## 더 짧게 줄이기

람다 자리에 비교자 생성 메서드와 메서드 참조를 통해 더 간결하게 줄일 수 있다.

```java
Collection.sort(words, comparingInt(String::length));
```

list 자체에 포함된 sort를 사용하면 더 짧아진다.

```java
words.sort(comparingInt(String::length));
```



## 기존 코드 최적화 하기

이전에 enum class를 이용해 연산자를 정의한 경우가 있었다.

```java
public enum Operation {
    PLUS  ("+") {
      public double apply(double x, double y) { return x + y;}
    },
    MINUS ("-") {
      public double apply(double x, double y) { return x - y;}
    },
    TIMES ("*") {
      public double apply(double x, double y) { return x * y;}
    },
    DIVIDE("/") {
      public double apply(double x, double y) { return x / y;}
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() { return symbol; }

    public abstract apply(double x, double y);
}
```

람다를 이용해 다음과 같이 나타낼 수 있다.

```java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }

    // Main method from Item 34 (Page 163)
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
```



## 제약사항

람다는 이름도 없고 문서화도 못하기 때문에 코드만 보고 동작을 알 수 없다면 또는 코드의 줄수가 길어지면 람다는 쓰지 말아야 한다.

또한 함수형 인터페이스가 아닌 경우에는 익명 클래스를 써야한다.

람다는 자신을 참조할 수 없다. 람다에서 this 키워드는 바깥 인스턴스를 가르키기 떄문에 함수 객체가 자기자신을 참조해야하면 반드시 익명클래스를 사용하자.

또한 직렬화가 구현별로 다를 수 있기 때문에 직렬화는 삼가해야한다.