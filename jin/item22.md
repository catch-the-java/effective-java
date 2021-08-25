# Item 22 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 즉, 자신의 인스턴스로 어떤 역할을 할지를 클라이언트에 얘기해주는 것이다.

즉, 인터페이스는 오로지 이 용도로만 사용해야 한다. 하지만 이것을 지키지 않는 상수 인터페이스 라는 것이 있다. 바로 메서드 없이 static final 필드로만 가득찬 인터페이스이다. 

이런 식으로 사용하고 싶다면 차라리 상수 유틸리티 클래스를 이용하여 나타낸다. 그 형식은 다음과 같다.

```java
public class PhysicalConstants {
  pirvate PhysicalConstants() {}
  
  public static final double AVOGADROS_NUMBER = 6.022_140e23;
  ...
}
```

만약 상수를 빈번히 사용한다면 static import를 사용하여 클래스의 이름을 생략할 수 있다.