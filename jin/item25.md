# item 25 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러개 선언하더라도 컴파일러에선 문제가 되지 않는다.

하지만 문제가 되는 것을 알아보자.

Dessert.java 파일에 정의 되어 있다고 가정해보자.

```java
class Utensil {
  static final String NAME = "pot";
}

class Dessert {
  static final String NAME = "pie";
}
```

Utensil.java 파일에 정의 되어 있다고 가정해보자.

```java
class Utensil {
  static final String NAME = "pot";
}

class Dessert {
  static final String NAME = "pie";
}
```

```java
public class Main {
  public static void main(String[] args) {
    System.out.println(Utensil.NAME + Dessert.NAME);
  }
}
```

이러면 Java Main.java Dessert.java 명령으로 컴파일을 한다면 컴파일 오류가 난다.

하지만 컴파일할때 파일을 어떻게 넣느냐에 따라서 출력이 달라질 여지가 존재한다.

따라서, 톱레벨 클래스들은 별도의 소스 파일로 분리해야한다.

