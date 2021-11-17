# item 53 가변인수는 신중히 사용하여라

가변인수 메서드는 명시한 타입의 인수르르 0개이상 받을 수 있다.

동작 방식은 매개변수를 받으면 받은 매개변수를 해당 타입의 배열로 변화시켜 건내준다.

만약 인수가 한개 이상이어야 한다면 다음과 같이 작성해야 한다.

```java
    // The WRONG way to use varargs to pass one or more arguments! (Page 245)
    static int min(int... args) {
        if (args.length == 0)
            throw new IllegalArgumentException("Too few arguments");
        int min = args[0];
        for (int i = 1; i < args.length; i++)
            if (args[i] < min)
                min = args[i];
        return min;
    }

```

하지만 해당 코드는 인수하 0개라면 런타임에러를 발생시키고 코드도 더럽다. 따라서 컴파일 시점에 알아챌 수 있도록 다음과 같이 바꾸자.

```java
    static int min(int firstArg, int... remainingArgs) {
        int min = firstArg;
        for (int arg : remainingArgs)
            if (arg < min)
                min = arg;
        return min;
    }
```

이러면 컴파일 시점에 인수를 반드시 한개 이상 받도록 할 수 있다.

### 성능에 민감한 상황

성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있는데 가변인수는 메서드가 호출될 때 마다 배열을 할당하고 초기화하기 때문이다. 하지만 이러한 것도 매개변수를 다음과 같이 선언해주어 확률에 기대어 해결하는 것이다.

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, a2) {}
public void foo(int a1, a2, a3) {}
public void foo(int a1, a2, a3, int... rest) {}
```

EnumSet의 정적 팩터리도 이 기법을 이용해 열거 타입 집합 생성 비용을 최소화 한다.