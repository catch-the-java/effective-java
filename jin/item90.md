# item 90 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
## 직렬화의 위험성
Serializable을 구현하기로 한 순간 생성자 이외에 인스턴스를 생산할 방법이 생겨버린다.
버그나 보안 문제가 발생할 수 있는데 이를 줄여줄 방법으로 "직렬화 프록시 패턴"이 있다.

## 직렬화 프록시 패턴
1. 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private static으로 선언한다. 이 중첩 클래스가 바로 바깥 클래스의 __직렬화 프록시__ 이다.
2. 이 중첩 클래스는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다.
3. 이 클래스는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다.(일관성 검사나 방어적 복사도 포함하지 않는다.)
4. 바깥 클래스와 직렬화 프록시 모드 Serializable을 구현한다고 선언해야한다.

이전에 직렬화 하려고 구현한 Period 예시를 보자.
```java
import java.util.*;
import java.io.*;

// 방어적 복사를 사용하는 불변 클래스
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
    }

    public Date start () { return new Date(start.getTime()); }

    public Date end () { return new Date(end.getTime()); }

    public String toString() { return start + " - " + end; }


    // 코드 90-1 Period 클래스용 직렬화 프록시 (479쪽)
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private static final long serialVersionUID =
                234098243823485285L; // 아무 값이나 상관없다. (아이템 87)
    }

    // 직렬화 프록시 패턴용 writeReplace 메서드 (480쪽)
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // 직렬화 프록시 패턴용 readObject 메서드 (480쪽)
    private void readObject(ObjectInputStream stream)
            throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
```

## 직렬화 프록시 패턴의 장점
1. 방어적 복사처럼, 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부필드 탈취 공격을 프록시 수준에서 차단해준다.
2. 앞에서 나온 두 방식과는 다르게 필드를 final로 선언해도 되므로 Period 클래스를 진정한 불변으로 만들 수 있다.
3. 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 동한다.

### 실제 직렬화 프록시의 예
EnumSet의 예시를 보면 이 클래스는 public 생성자 없이 정적 팩터리들만 제공한다.

클라이언트 입장에서는 이 팩터리들이 EnumSet 인스턴스를 반환하는 걸로 보이지만 OpenJDK를 보면 열거 타입의 크기에 따라 두 하위 클래스 중 하나의 인스턴스만 반환한다.

동작은 다음과 같이 한다.

1. 열거타입의 원소가 64개 이하면 RegularEnumSet을 사용한다.
2. 그보다 크면 JumboEnumSet을 사용한다.

만약 원소 64개짜리 열거 타입을 가진 EnumSet을 직렬화한 다음 원소 5개를 추가하려고 역직렬화 하려면 JumboEnumSet로 역직렬화 된다.

아래 코드를 보면 실제로 그렇게 작동한다.
```java
private static class SerializationProxy <E extends Enum<E>> implements Serializable {

    // 이 EnumSet의 원소 타입
    private final Class<E> elementType;
    // 이 EnumSet 안의 원소들
    private final Enum<?>[] elements;

    SerializationProxy(EnumSet<E> set) { 
        elementType = set.elementType;
        elements = set.toArray(new Enum<?>[0]); 
    }

    private Object readResolve() {
        EnumSet<E> result = EnumSet.noneOf(elementType); 

        for (Enum<?> e : elements)
            result.add((E)e); 
            
        return result;
    }

    private static final long serialVersionUID = 362491234563181265L;
}
```

## 직렬화 프록시 패턴의 한계
1. 클라이언트가 멋대로 확장할 수 있는 클래스에 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에 적용할 수 없다.
3. 안정성에 따른 대가로 속도가 느리다.