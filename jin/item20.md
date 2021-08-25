## item 20 추상 클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상클래스 이렇게 두가지이다. 이전에는 인터페이스에는 디폴트 메서드를 지원하지 않았지만 자바 8에 오면서 제공하기 시작하엿따. 이 둘의 가장 큰 차이점은 추상 클래스가 정의한 타입을 구현하는 클래스는 반드시 하위 클래스가 되어야 한다는 점이다.

하지만 인터페이스가 선언한 메서드를 모두 정의하고 규약을 잘 지킨 클래스라면 어떤 클래스를 상속하더라도 같은 타입으로 취급된다.

1. __인터페이스는 믹스인 정의에 안성맞춤이다.__ 믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택정 행위를 제공한다고 선언하는 효과를 준다. 
2. __인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.__ 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있다. 현실 세계에서는 상속과 같은 계층 구조로 나타낼 수 없는 구조가 있어서 그러한 구조를 효과적으로 표현 할 수 있다. (Ex : sing a song writer)
3. __인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.__ 타입을 추상 클래스로 정의해두면 그 타입에 기능을 추가하는 방법은 상속 뿐이다. 상속해서 만든 클래스는 래퍼클래스보다 활용도가 떨어지고 깨지기는 더 쉽다. 만약에 구현 방식이 명백한게 있다면 디폴트 메서드를 이용하여 미리 정의해 줄 수 있다.

인터페이스와 추상 골격 구현 클래스를 함께 제공하는 식으로 인터페이스와 추상클래스의 장점을 모두 취하는 방법도 있다. 인터페이스로는 타입을 정의하고 필요하다면 디폴트 메서드 몇개까지도 구현한다. 그리고 골격 구현 클래스는 나머지 메서드 가지 구현한다. 템플릿 메서드 패턴을 이용하여 구현할 수 있다.

### **Template Method 패턴**

간단하게 이야기하자면 공통 부분을 추상클래스로 선언해놓고 추상클래스를 상속받아 자식 클래스에 맞도록 구현하는 것이다.

간단한 예시를 보자. 우선 추상클래스를 구현해보겠다.

```
public abstract class Person {
  public void goToOutside() {
    breath();
    if(hasPhone) {
      System.out.println("핸드폰을 들고 나갑니다.");
    }
  }

  abstract void breath();

  boolean hasPhone() {
    return false;
  }
}
```

인간은 모두 숨을 쉬기 때문에 breath라는 공통 메서드를 가지기 때문에 breath 메서드 앞에 abstract가 붙어있고 핸드폰은 없을 수 있기 때문에 hasPhone앞에는 abstract가 없다. 여기서 abstract가 없는 메서드는 꼭 상속받은 클래스에서 구현하지 않아도 되며 이것을 훅 메소드라고 부른다.

만약 핸드폰이 없는 사람이라고 한다면

```
public class NoPhonePerson extends Person{
  @Override
  void breath() {
    System.out.println("편안히 숨을 쉰다.");
  }
}
```

핸드폰이 있는 사람이라면

```
public class YesPhonePerson extends Person{
  @Override
  void breath() {
    System.out.println("빠르게 숨을 쉰다.");
  }

  @Override
  boolean hasPhone() {
    return true;
  }
}
```

그래서 두 클래스가 공통으로 가지고 있으며 호출가능한 goToOutside를 출력하게되면 각각 다음과 같이 출력된다.

NoPhonePerson

"편안히 숨을 쉰다."

YesPhonePerson

"빠르게 숨을 쉰다."

"핸드폰을 들고 나갑니다."

따라서, 템플릿 메서드 클래스를 통해 상속받은 클래스들의 원형을 비슷하게 유지하고 훅 메서드를 통해서 하위 클래스에게 구현의 선택권을 넘겨주는 식으로 구현할 수 있는 장점이 있습니다.
