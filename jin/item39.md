# Item39 명명 패턴보다 애너테이션을 이용하라

전통적인 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다. 예를들어 Junit3 까지는 테스트메서드의 이름은 test로 시작해야했다.

(그래서 메서드 명에 오타가 발생하면 테스트를 안하고 넘어가는 경우들이 좀 있었다.)

따라서 명명 패턴에는 단점들이 존재한다.

1. 위에서 쓴것 처럼 메서드에 오타가 발생하는 경우에는 인식을 못할 수 있다.
2. 올바른 프로그램 요소에서만 사용되리라 보증할 방법이 없다.(TestSafety라는 메서드를 개발자는 테스트가 진행될 것이라 생각되겠지만 시작이 대문자여서 그냥 넘어간다.)
3. 매개변수로 전달할 마땅한 방법이 없다는 것이다.
   특정 예외를 던져야만 성공하는 테스트가 있다고 해보자. 기대하는 예외 타입을 테스트에 매개변수로 전달해야 하는 상황이 있을 때 예외의 이름을 테스트에 덧붙이는 방법도 있지만, 보기도 나쁘고 깨지기도 쉽다.

이 모든 명명 패턴의 단점을 해결해주는 것이 바로 애너테이션이라는 개념이다.

## Annotation(어노테이션 또는 에너테이션)

Test라는 이름의 에터네이션을 정의한다고 가정해보자. 자동으로 수행되는 간단한 테스트 에너테이션으로 예외가 발생하면 해당 테스트를 실패로 처리한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

보는 것과 같이 Test 에너테이션에도 추가적인 에너테이션이 붙어있다. 이처럼 에너테이션에 달리는 에너테이션을 *__메타에너테이션(meta-annotation)__*이라 한다.

@Retention에너테이션은 런타임동안에도 @Test 에너테이션이 유지되어야 한다는 뜻이다. 한편, @Target(ElementType.METHOD)는 메서드 선언에만 사용되어야 한다고 알려주는 것이다. 만약 더 강제적으로 제약을 하고 싶다면 애너테이션 처리기를 직접 구현해야한다. 관련은 javax.annotation.processing API 문서를 참조하도록 하자.

@Test 에너테이션을 실제로 적용시킨 예시를 보자.

```java
public class Sample {
  @Test public static void m1() {}//성공해야한다.
  public static void m2() {}
  @Test public static void m3() {//실패해야한다.
    throw new RuntimeException("실패");
  }
 	public static void m4() {}
  @Test public void m5() {} //정적 메서드가 아니다. 즉, 잘못 사용되었다.
  public static void m6() {}
  @Test public static void m7() {
    throw new RuntimeException("실패");
  }
  public static void m8() {}
}
```

결과만 얘기하자면 총 4개의 테스트 메서드 중 1개는 성공 2개는 실패 1개는 잘못 사용했다. 그리고 나머지 4개의 메서드는 테스트 도구가 무시할 것이다.

@Test 애너테이션이 Sample 클래스의 의미에 직접적인 영향을 주지는 않는다. 에너테이션에 관심 있는 프로그램에게 추가 정보를 제공할 뿐이다. 다음의 코드를 보자.

```java
public class RunTests {
  public static void main(String[] args) throws Exception {
    int tests = 0;
    int passed = 0;
    Class<?> testClass = Class.forName(args[0]);
    for (Method m : testClass.getDeclareMethods()) {
      if(m.isAnnotationPresent(Test.class)) {
        tests++;
        try {
          m.invoke(null);
          passed++;
        } catch (InvocationTargetException wrappedExc) {
          Throwable exc = wrappedExc.getCause();
          System.out.println(m + " 실패: " + exc);
        } catch (Exception exc) {
          System.out.println("잘못 사용한 @Test: " + m);
        }
      }
    }
  }
  System.out.println("성공: %d, 실패\: %d%n", passed, tests - passed);
}
```

테스트 러너의 동작 순서는 명령줄로부터 클래스의 이름을 받아, 그 클래스에서 @Test 애너테이션이 달린 메서드를 차례로 호출한다. 

isAnnotationPresent 함수가 에너테이션의 존재 여부를 판단하는 메서드이다. 테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 InvocationTargetException으로 감싸서 다시 던진다. 이 프로그램은 InnovationTargetException을 잡아 원래 담긴 예외를 getCause로 출력해준다.

다음은 특정 예외를 던져야만 성공하는 테스트를 지원해보도록 해보자.

```java
@Retation(RetationPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable> value();
}
```

이 에너테이션의 매개변수 타입은 Class\<? extends Throwable>이다. "Throwable"을 확장한 클래스의 Class 객체"라는 뜻이며, 따라서 모든 예외 타입을 다 수용한다.

```java
public class Sample2 {
  @ExceptionTest(ArithmeticException.class)
  public static void m1() { //성공해야한다.
    int i=0;
    i = i/i;
  }
  
  @ExceptionTest(ArithmeticException.class)
  public static void m2() {//다른 예외가 발생하여 실패해야함
    int[] a = new int[0];
    int i = a[1];
  }
  
  @ExceptionTesT(ArithmeticException.class)
  public static void m3() {} //예외가 발생하지 않음
}
```

실행하는 코드를 보자.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " failed: " + exc);
                } catch (Exception exc) {
                    System.out.println("Invalid @Test: " + m);
                }
            }

            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("Test %s failed: no exception%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf(
                                "Test %s failed: expected %s, got %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("Invalid @ExceptionTest: " + m);
                }
            }
        }

        System.out.printf("Passed: %d, Failed: %d%n",
                passed, tests - passed);
    }
}

```

@Test 에너테이션용 코드와 비슷해 보이지만 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는 데 사용한다.

이 예외 테스트 예에서 한 걸음 더 들어가, 예외를 여러 개 명시하고 그 중 하나가 발생하면 성공하게 만들어 보자. 매개변수의 타입을 Class 객체의 배열로 수정해보자.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
  Class<? extends Throwable>[] values();
}
```

```java
@ExceptionTest({IndexOutOfBoundException.class,
               NullPointerException.class})
public static void doublyBad() {
  List<String> list = new ArrayList<>();
  list.addAll(5,null);
}
```

```java
if (m.isAnnotationPresent(ExceptionTest.class)) {
    tests++;
    try {
      m.invoke(null);
      System.out.printf("Test %s failed: no exception%n", m);
    } catch (InvocationTargetException wrappedEx) {
      Throwable exc = wrappedEx.getCause();
      int oldPassed = passed;
      for(Class<? extends Throwable>[] exeTypes : excTypes) {
        if(excType.inInstance(exc)) {
          passed++;
          break;
        }
      }
      if(passed == oldPassed)
        system.out.printf("테스트 %s 실패: %s %n", m, exc);
  }
```

앞선 코드보다 꽤나 직관적이다.



자바8에서 여러개의 값을 받는 에너테이션을 다른 방식으로 만들 수 있다. 배열 배개변수를 사용하는 대신 애너테이션에 @Repeatable 메타애너테이션을 다는 방식이다. @Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다. 

주의할 점이 있다.

1. Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
2. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
3. 컨테이너 애너테이션 타입에는 적절한 보전 정책과 적용 대상을 명시해야한다. 그렇지 않으면 컴파일되지 않는다.

```java
//반복가능한 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
  Class<? extends Throwable> values();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
  ExceptionTest[] values();
}
```

애너테이션을 적용해보자.

```java
@ExceptionTest(IndexOutOfBoundException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {...}
```

반복 가능 애너테이션은 처리할 때도 주의를 요하는데, 반복 가능 애너테이션을 여러개 달면 하나만 달았을 때와 구분하기 위해 해당 '컨테이너' 타입 애너테이션이 적용된다.

getAnnotationsByType 메서드는 이 둘을 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져오지만, isAnnotationPresent 메서드는 둘을 명확히 구분한다. 따라서 반복 가능 애너테이션이 달렸는지 검사한다면 "그렇지 않다"라고 대답한다.(컨테이너가 달렸기 때문)

이제 RunTests를 ExceptionTest의 반복 가능 버전을 사용하도록 수정해보자.

```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " failed: " + exc);
                } catch (Exception exc) {
                    System.out.println("INVALID @Test: " + m);
                }
            }

            // Processing repeatable annotations (Page 187)
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("Test %s failed: no exception%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("Test %s failed: %s %n", m, exc);
                }
            }
        }
        System.out.printf("Passed: %d, Failed: %d%n",
                          passed, tests - passed);
    }
}

```

반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달 때의 코드 가독성을 높였다.

애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이요는 없으며 자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야 한다.