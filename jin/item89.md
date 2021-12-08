# item 89 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라
## 싱글턴 파괴
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
}
```
이 클래스 선언에 implements Serializable을 추가하는 순간 싱글턴이 아니게 된다.

(item 87과 item 88에서 설명한 기본 직렬화와 readObject를 제공하더라도 마찬가지이다.)

## readResolve
readResolve 역직렬화를 하려는 객체에 적절히 잘 정의 해놓았다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고 기존에 만들어 놓은 원래 객체가 반환된다.

(새로 생성된 객체는 가비지 컬랙션의 대상이 된다.)
```java
private Object readResolve() {
    return INSTANCE;
}
```
위 코드를 선언해놓으면 새로 생성된 객체는 무시하고 원래 객체를 반환한다.

인스턴스는 역직렬화 될 필요가 없기 떄문에 transient로 선언되어야 한다.

## 도둑 클래스
싱글턴 클래스 중 transient로 표현되지 않은 필드가 존재한다면 그 싱글턴의 참조를 훔쳐올 수 있다.

1. readResolve 메서드와 인스턴스 필드 하나를 포함한 도둑(stealer)클래스를 작성한다.
2. stealer 클래스의 필드는 '숨겨져야 할' 직렬화된 인스턴스를 참조하는 역할을 한다. 이를 통해 싱글턴의 비 휘발성 필드를 이 도둑의 인스턴스로 교체한다.
3. 이제부터 싱글턴이 역직렬화 될 때 도둑의 readResolve가 먼저 호출되게 되게 된다. 이때, 역직렬화 진행중인 싱글턴 인스턴스를 필드로 가지게 된다.
4. 도둑의 인스턴스는 싱글턴 인스턴스를 복사하여 readResolve 메서드가 끝나도 계속 싱글턴 객체를 참조할 수 있다.
5. 그 후 도둑이 숨긴 transient가 아닌 필드의 원래 타입에 맞는 값을 반환한다.

다음은 예제이다.
```java
//원래 클래스
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}

    private String[] favoriteSongs = {"밤편지", "Wake up"};

    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}

//도둑 클래스
public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
        //resolve 되기 전의 Elvis 인스턴스의 참조를 저장한다.
        impersonator = payload;

        // favoriteSongs 필드에 맞는 타입의 객체를 반환한다.
        return new String[] {"What a Fool"};
    }

    private static final long serialVersionUID = 0;
}
```
이후 수작업으로 만든 스트림을 이용해 2개의 싱글턴 객체를 만든다.
```java
public class ElvisImpersonator {

    // 진짜 Elvis 인스턴스로는 만들어질 수 없는 바이트 스트림! 
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05, 0x45, 0x6c, 0x76, 0x69, 0x73, 
        (byte)0x84, (byte)0xe6, (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
        ...
    };

    public static void main(String[] args) {
        // ElvisStealer.impersonator를 초기화한 다음,
        // 진짜 Elvis(즉, Elvis.INSTANCE)를 반환한다.
        Elvis elvis = (Elvis) deserialize(serializedForm); 
        Elvis impersonator = ElvisStealer.impersonator;

        elvis.printFavorites();
        impersonator.printFavorites(); 
    }
}
```
다음과 같이 출력된다.
```json
["밤편지", "Wake up"]
["What a Fool"]
```

## 해결
1. favoriteSongs 필드를 transient 선언하여 이 문제를 해결한다.
2. __이넘으로 선언하여 선언된 상수 말고는 객체가 존재하지 않음을 보장한다.__

2번의 한계는 AccessibleObject.setAccessible를 막지 못한다.

```java
public enum Elvis {
    INSTANCE;
    
    private String[] favoriteSongs = {"밤편지", "Wake up"};

    public void printFavorites() {
        ...
    }
}
```

## readResolve에서의 접근자
readResolve 메서드는 접근자가 매우 중요하다.

1. final 클래스에서 라면 readResolve 메서드는 private여야 한다.(다만 하위에서 사용할 수 없다.)
2. 우리가 아는 접근제어자와 내용이 같긴한데 protected나 public 이면서 하위클래스에서 재정의 하지 않으면 ClassCastException을 일으킬 수 있다.