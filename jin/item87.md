# item 87 커스텀 직렬화 형태를 고려해보라

앞서 계속 설명한 것 처럼 Serializable을 구현하고 기본 직렬화 형태를 사용하면 다음 릴리즈에 버리려한 현재 구현에 계속 묶인다.



## 먼저 고민해보고 괜찮다고 판단될 떄만 기본 직렬화 형태를 사용하라

- 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야함
- 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.

```java
//적절한 경우
public class Name implements Serializable {
  private final String lastName;
  
  private final String firstName;
  
  private final String middleName
}
```

성명은 논리적으로 이름, 성 중간이름이라는 3개의 문자열로 구성되며 위 코드는 모두 만족함.

책의 예제는 private도 모두 문서화 주석이 달려있는데 이는 직렬화 될 클래스는 공개 API라는 것이다.

- 기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.
- 다음은 직렬화에 적합하지 않다.

```java
public final class StringList implements Serializable {
  private int size = 0;
  private Entry head = null;
  
  private static class Entry implements Serializable {
    String data;
    Entry next;
    Entry previous;
  }
}
```

위 코드는 객체의 물리적 표현과 논리적 표현의 차이가 크다.



## 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용할 때 발생하는 문제

- 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
  내부 구현이 변경되어도 직렬화 관련 코드는 더 이상 지울 수 없는 상태가 된다.
- 너무 많은 공간을 차지할 수 있다.
  위 예시의 경우 연결 정보 까지 모두 직렬화 하지만 이부분은 직렬화 할 필요가 없는 부분이다.
- 시간이 너무 많이 걸릴수 있다.
  위 예시에서 모든 연결리스트를 타고 들어가서 직렬화 해야한다.
- 스택 오버플로를 일으킬 수 있다.
  위 예시가 그래프가 길게 늘어져 있다면 이를 직렬화 하느라 메모리의 stack 영역 제한을 넘어갈 수 있다.



## 위 예시를 합리적으로 바꿔보자

```java
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    // No longer Serializable!
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    public final void add(String s) {  }


    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }
}
```

단순히 리스트를 포함한 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하면 된다. 위는 그것을 코드로 나타낸 것이다.

WriteObject와 ReadObject가 직렬화 형태를 처리한다. (Transient 한정자는 직렬화를 하지 않음을 의미)

...? 뭔 내용인지 이해를 못하겠음..