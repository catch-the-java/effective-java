# item50 적시에 방어적 복사본을 만들라

## 클라이언트를 믿지 말자

C++과 같은 네이티브 언어에 비해서 비교적 안전하지만 Java도 모든것을 막을수는 없다.

클라이언트가 불변식을 깨려고 난리를 친다고 생각하자.

```java
public class Attacks {
    public static void main(String[] args) {
        // Attack the internals of a Period instance  (Page 232)
        Date start = new Date();
        Date end = new Date();
        Period p = new Period(start, end);
        end.setYear(78);  
        System.out.println(p);

        start = new Date();
        end = new Date();
        p = new Period(start, end);
        p.end().setYear(78);  //Date가 가변이기 때문에 불변식이 깨짐
        System.out.println(p);
    }
}

```

이를 해결할 방법들이 생겼지만(LocalTime, Instant 등) 이전 API들은 이를 사용할 수 없다. 이를 막아보자



## 매개변수의 방어적 복사본

매개변수의 방어적 복사본을 만들어보자. 다음과 같이 생성자를 수정해보자.

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start the beginning of the period
     * @param  end the end of the period; must not precede start
     * @throws IllegalArgumentException if start is after end
     * @throws NullPointerException if start or end is null
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end); //여기서 미리 막아준다.
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }

    public String toString() {
        return start + " - " + end;
    }

}

```

하지만 매개변수가 제3자에 의해 확장될 가능성이 있다면 clone을 사용하면 안된다.

다음은 두번째 공격을 가정해보자.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);//내부를 바꾼다.
```

이를 해결하기 위해서 다음과 같이 방어적 복사본을 반환한다.

```java
public Date start() {
  return new Date(start.getTime());
}

public Date end() {
  return new Date(end.getTime());
}
```

다만 생성자와 달리 메서드에서는 방어적 복사에 clone()을 사용해도 된다. 하지만 인스턴스를 복사하는 데에는 일반적으로 생성자나 정적팩터리 메서드를 사용하는 편이 낫다.

