# item 34. int 상수대신 열거 타입을 사용하라

## 열거 타입이란?
- 열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외값은 허용하지 않는 타입

<br/>

## 정수 열거 패턴
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
### 단점
#### 타입 안전하지 않고, 표현력 좋지 않다.
- 동등연산자 비교해도 아무런 경고 메시지 출력하지 않는다. => 타입 안전 x
  - `APPLE_FUJI == ORANGE_NAVEL;`
- 별도 namespace를 지원하지 않아서 어쩔 수 없이 접두어를 써서 이름 충돌 방지한다.
  - `APPLE_` , `ORANGE_`
#### 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽다.
- 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다.
- 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.
#### 정수 상수는 문자열로 출력하기가 다소 까다롭다.
```java
public static final int APPLE_PIPPIN = 1;
System.out.println(APPLE_PIPPIN); // 문자 APPLE_PIPPIN가 아닌 1이 출력된다.
```

<br/>

## 문자열 열거 패턴
```java
// 내가 생각한 예시
public static final String APPLE = "APPLE";
public static final String GRAPE = "GRAPE";
public static final String ORANGE = "ORANGE";
```
### 단점
- 정수 열거 패턴보다 더 나쁨
- 필드명 대신 string 상수 값을 클라이언트 코드에 하드 코딩 => 문자열에 오타가 있어도 컴파일러는 확인 못함 => 런타임 버그
- 문자열 비교에 따른 성능저하
####

<br/>

## 열거 타입
```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
```
### 장점
#### 열거 타입은 실제로는 클래스다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개, 생성자 제공 x
- 따라서 열거 타입으로 만들어진 인스턴스들은 딱 하나씩만 존재
- 싱글턴은 원소가 하나뿐인 열거 타입, 거꾸로 열거 타입은 싱글턴을 일반화한 형태
#### 열거 타입은 컴파일타임 타입 안정성 제공
- 열거타입을 매개변수로 받는 메서드를 선언하면 다른 타입 값을 넘기려고 하면 컴파일 오류
#### 열거 타입에는 각자의 namespace가 있어서 이름이 같은 상수도 평화롭게 공존한다.
- 열거 타입에 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 안해도 된다.
- 정수 열거 패턴과 달리 상수 값이 클라이언트로 컴파일되어 각인되지 않기 때문이다. (공개되는 것은 오직 필드 이름뿐)
#### 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
#### 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스 구현 가능
#### Object메서드, Comparable, Serializable들을 잘 구현해놓음

<br/>

## 열거타입 예시
```java
enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6);
    // ...

    private final double mass; // 질량
    private final double radius; // 반지름
    private final double surfaceGravity; // 표면중력

    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장
- 불변이므로 모든 필드는 final
