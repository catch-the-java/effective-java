# 		item 65 리플렉션 보다는 인터페이스를 사용하라

## Reflection 이란?

객체를 통해 클래스의 정보를 분석해 내는 클래스 기법을 의미한다.

Class 객체가 주어지면 그 클래스의 생성자, 메서드 필드에 해당하는 Constructor, Method, Field 인스턴스를 가져올 수 있다.

그 인스턴스 들로는 그 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등을 가져올 수 있다.

생성자, 메서드, 필드 인스턴스를 이용해 각각 연결된 실제 생성자, 메서드, 필드를 조작 할 수 있다.

어떤 클래스의 어떤 객체가 가진 어떤 메서드를 호출하거나 컴파일 당시에 존재하지 않던 객체도 이용할 수 있다.

.class 파일 하나당 -> bytecode로 변환된 파일

java.lang.Class 객체를 하나씩 생성해준데요.

## 단점

- 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.
- 리플렉션을 이용하면 코드가 지저분하고 장황해진다.
- 성능이 떨어진다. 일반 메서드 호출보다 속도가 매우 느리다.

프레임워크 같은 경우에는 의존성 주입과 같은 복잡한 연산때문에 사용해야 하는 경우가 있지만 우리는 일반적으로 그럴 일이 없으니 제한적으로 사용해야 한다.

## 활용 예

1. 컴파일타임에 이용할 수 없는 클래스를 사욯애야해만 하는 프로그램은 비록 컴파일 타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수 있다.(item 64) 이런 경우에는 인스턴스 생성에만 쓰고 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해서 사용하자.

   ```java
   public static void main(String[] args) {
     // Translate the class name into a Class object
     Class<? extends Set<String>> cl = null;
     try {
       cl = (Class<? extends Set<String>>) Class.forName(args[0]); // Unchecked cast!
     } catch (ClassNotFoundException e) {
       fatalError("Class not found.");
     }
   
     // Get the constructor
     Constructor<? extends Set<String>> cons = null;
     try {
       cons = cl.getDeclaredConstructor();
     } catch (NoSuchMethodException e) {
       fatalError("No parameterless constructor");
     }
   
     // Instantiate the set
     Set<String> s = null;
     try {
       s = cons.newInstance();
     } catch (IllegalAccessException e) {
       fatalError("Constructor not accessible");
     } catch (InstantiationException e) {
       fatalError("Class not instantiable.");
     } catch (InvocationTargetException e) {
       fatalError("Constructor threw " + e.getCause());
     } catch (ClassCastException e) {
       fatalError("Class doesn't implement Set");
     }
   
     // Exercise the set
     s.addAll(Arrays.asList(args).subList(1, args.length));
     System.out.println(s);
   }
   ```

   위에서 알 수 있는 단점은 두가지 이다.

   - 런타임에 총 6가지나 되는 예외를 던질 수 있다.(단 이것은 Java7 이후에 등장한 ReflectiveOperationException을 잡도록 하여 줄일 수 있다.)
   - 클래스 이름만으로 인스턴스를 생성하기 위해 25줄이나 되는 코드를 작성해야 한다.

   아무튼 일단 객체가 만들어 지고 난 이후는 Set 인스턴스와 다르지 않다.

2. 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 떄 적합하다. 이 기법은 여러개 존재하는 외부 패키지를 다룰 때 유용하다.

