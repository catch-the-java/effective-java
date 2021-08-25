# item 23 태그 달린 클래스보다는 클래스 계층구조를 활용하라

이 아이템에선 바로 예시를 보자.

```java
class Figure {
  enum Shape {RECTANGLE, CIRCLE};
  
  final Shape shape;
  
  double length;
  double width;
  
  double radius;
  
  Figure(double radius) {
    shape = Shape.CIRCLE;
    this.radius = radius;
  }
  
  Figure(double length, double width) {
    shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }
  
  double area() {
    switch(shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
        
      ...
    }
  }
}
```

객체지향적으로 보면 좋지 안은 코드이다. 우선 클래스 안에 열거클래스, 태그필드, switch문과 같은 쓸 때 없는 코드가 많다. 코드를 보면 CIRCLE에서만 사용되는 필드와 RECTANGLE에서만 사용되는 필드가 중복으로 들어가 있기 때문에 사용하지 않는 엉뚱한 필드를 초기와 하더라도 런타임에 그 문제가 나타나게 된다. 또한 switch문에서 새로운 Shape이 추가될 때마다 수정이 되야한다는 문제이다. 그리고 Figure라는 클래스 이름만으로는 정확히 어떤 클래스인지 알기 쉽지 않으며 이렇게 태그를 달아서 표현하는 방식은 *__클래스 계층구조를 어설프레 흉내낸 아류일 뿐이다. __* 굉장히 비효율 적인 방식이다. 이제 이 클래스를 서브타입핑 클래스를 이용해 계층구조로 바꿔보도록 하겠다.

```java
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
	final double radius;
  
  Circle(double radius) {this.radius = radius;}
  
  @Override double area() {return Math.PI * (radius * radius);}
}

class Rectangle extends Figure {
  final double length;
  final double width;
  
  Rectangle(double length, double width) {
    this.length = length;
    this.width = width;
  }
  
  @Override double area() {return length * width;}
}
```

이런 식으로 계층식으로 만들면 공통된 메서드인 area를 공통화 할 수 있다. 또한 위 예제처럼 필요없는 필드를 초기화 할 필요도 없어지며 switch 문을 실수로 고치지 않아(추가하지 않아) 생기는 런타임 에러를 사전에 방지해 줄 수 있다. 그리고 Class 이름만으로 어떠한 역할을 하는지 쉽게 알 수 있다. 그리고 타입간의 자연스러운 계층 관계를 반영 할 수 있다. 예를들어 정사각형을 표현해야 한다면 다음과 같이 쉽게 나타낼 수 있다.

```java
class Sqaure extends Rectangle {
  Square(double side) {
    super(side, side);
  }
}
```

