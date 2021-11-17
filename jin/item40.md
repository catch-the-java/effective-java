# item 40 @Override 애너테이션을 일관되게 사용하라

## @Override 애너테이션

자바가 기본으로 제공하는 애너테이션 중 가장 중요할 것이며 메서드에만 선언이 가능하다.

상위 메서드를 재 정의 했음을 알림. 이 애너테이션을 이용하여 여러가지 버그를 예방해준다.

### 문제 발생

다음 코드를 확인해보자.

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size()); //Set은 중복을 허용하지 않기 때문에 26이 출력될 것 같지만 260이 출력된다.
    }
}
```

위 문제가 발생한 이유는 overriding이 아닌 overloading을 해버렸기 때문이다. equals를 재정의 할 때 Object 형식으로 매개변수를 선언해야하지만 다른 것으로 선언하였다.

이것을 명확하게 하기 위해선 @Override 애너테이션을 붙여주자.

### 해결

```java
@Overide public boolean equals(Bigram b) {
  return b.first == first && b.second == second;
}
```

이렇게 애너테이션을 선언해줌으로서 컴파일러에서 바로 오류를 잡아챌 수 있도록 한다. 따라서 다음과 같이 고칠 수 있다.

```java
@Override public boolean equals(Object o) {
  if(!(o instanceof Bigram))
    return false;
  Bigram b = (Bigram) o;
  return b.first == first && b.second == second;
}
```



## 결론

상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자. 추상 메서드나 인터페이스의 메서드를 구현하는 곳에는 진짜 다 달아주는것이 좋다.