# item 52 다중정의는 신중히 사용하라

### 다중정의가 혼동을 일으킨다?!

다음은 컬랙션을 집합, 리스트, 그 외로 구분하고자 만든 프로그램이다.

```java
package effectivejava.chapter8.item52;
import java.util.*;
import java.math.*;

// Broken! - What does this program print?  (Page 238)
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }

    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}

```

제대로 출력할 것 같지만 "그 외"만 세번 출력한다. 오버로딩된 함수 중 어떤 메서드를 호출할 지는 컴파일타임에 정해지기 때문이다.
(위 코드는 런타임에는 타입이 다양해지지만 컴파일타임에는 for문은 모두 컬랙션이다.)

우리가 착각을 하는 이유는 오버라이드는 동적으로 선택되고, 오버로딩은 정적으로 선택되기 때문이다.

다음 코드를 확인해보자.

```java
import java.math.BigInteger;
import java.util.*;

class Wine {
  String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
  @Override String name() { return "샴페인"; }
}

public class Override {
  public static void main(String[] args) {
    List<Wine> wineList = List.of(new Wine(), new SparkingWine(), new Champagne());
    
    for (Wine wine : wineList)
      System.out.println(wine.name());
  }
}

```

위 코드는 우리가 원하는데로 동작을 한다.(포도주, 발포성 포도주, 샴페인 순)

첫번째로 본 코드는 런타임에 동작하기를 원했으나 컴파일타임에 결정되기 때문에 우리가 원하는데로 동작하지 않았다. 이를 해결하기 위해서는 다음과 같이 고친다.

```java
public static String classify(Collection<?> c) {
  return c instanceOf Set ? "집합" : 
				 c instanceOf List ? "리스트" : "그 외";
}
```

다중 정의가 어떻게 호출될지 예상을 못한다면 API 사용자들이 문제의 원인을 찾느라 고생할 것이다. 

__따라서 다중정의가 혼란을 야기하는 상황은 피해야한다.__

사실 가장 좋은 방법은 다중정의 함수를 안만드는 것이 좋다. 다중정의 하는 대신 메서드의 이름을 다르게 지어주자.

예를들어 ObjectOutputStream 클래스는 타입별로 write 메서드가 이름이 모두 다르게 선언되어 있다.

### 생성자

생성자는 이름을 다르게 지을 수 없으니 반드시 다중정의가 된다. 하지만 정적 팩터리 메서드를 이용해 대신할 수 있다.

또한 생성자는 재정의 할 수 없기때문에 다중정의와 재정의가 혼용될 일은 없다. 그럼에도 같은 수의 매개변수를 받는 경우가 있는데 그런 경우에 대한 대책을 알아보자.

- 매개변수 수가 같은 다중정의 메서드가 많더라도 그 중 어느 것이 주어진 매개변수 집합을 처리할지가 명확히 구분된다면 햇갈일 일이 없을 것이다.
  (근본적으로 다르다는 것은 두 타입의 값을 서로 어느 쪽으로든 형 변환할 수 없다는 뜻이다.)

### Java 5 이후

하지만 자바 5로 넘어와 오토박싱이 생긴 이후 이것만으로는 해결되지 않는다.

```java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```

위 코드에서는 양수가 모드 지워지기를 바라겠지만 set은 Object를 지우고 list는 int로 받은 index를 지워 동작이 다르게 되는 것이다. 따라서 명시적으로 (Integer)i 와 같이 바꿔줘야 원하는 대로 동작한다.

### Java 8 도입이후 

람다와 메서드 참조로 더 복잡해졌다. 이는 부정확한 메서드 참조 같은 인수 표현식은 목표 타입이 선택되기 전에는 그 의미가 정해지지 않기 때문에 적용성 테스트가 무시되기 때문이다. 이는 컴파일러 제작자들을 위한 설명이니 알아듣지 못해도 괜찮고 다음것만 기억하자.

__다중정의된 메서드(혹은 생성자)들이 함수형 인터페이스를 인수로 받을때, 비록 서로 다른 함수형 인터페이스라도 인수의 위치가 같으면 혼란이 생기므로, 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다.__

## 결론

다중정의를 할 때 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야한다. 이것이 불가능하다면, 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중 정의 메서드가 모두 동일하게 동작하도록 만들어야 한다.