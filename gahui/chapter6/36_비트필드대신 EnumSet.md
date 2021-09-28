# 36. 비트 필드 대신 EnumSet을 사용하라

## 비트 필드
- 비트별 OR을 사용하여 여러 상수를 하나의 집합으로 모아 만들어진 집합
- 장점
    - 집합 연산을 효율적으로 수행할 수 있다. 
- 단점
    - 정수 열거 상수의 단점
    - 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다.
    - 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다.
    - 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다.


</br>

## EnumSet
- Set 인터페이스를 완벽히 구현
- 타입안전하고 어떤 Set 구현체와도 함께 사용 가능하다.

```java
public class EnumSetTest {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    public void applyStyles(Set<Style> styleSet) { }
}

//client 코드
public static void main(String[] args) {
        EnumSetTest enumSetTest = new EnumSetTest();
        enumSetTest.applyStyles(EnumSet.of(Style.BOLD, Style.UNDERLINE));
    }

```
- applyStyles에서 Set을 받는 이유
    - 모든 클라이언트가 EnumSet을 건낼 것으로 예상되어도 이왕이면 __인터페이스__ 로 받는 것이 좋다.(아이템 64)
    - 혹시나 클라이언트가 다른 Set 구현체를 넘기더라도 처리할 수 있다.
    


## 기타
1. of 메소드
    - 파라미터 E를 포함하는 EnumSet을 반환
```java
public static <E extends Enum<E>> EnumSet<E> of(E e) {
        EnumSet<E> result = noneOf(e.getDeclaringClass());
        result.add(e);
        return result;
    }
```