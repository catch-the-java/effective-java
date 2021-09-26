# 40. @override_애너테이션을 일관되게 사용하라 

## @Override 애너테이션
1. 메서드 선언 시에만, 사용 가능하다.
2. 상위타입 메서드를 재정의했음을 알 수 있다.
3. @Override 애너테이션을 작성함으로서 버그를 예방할 수 있다.


</br>

## 예시 - @Override를 쓰지 않았을 때, 실수할 수 할 수 있는 부분

``` java
public class Biagram {
    private final char first;
    private final char second;

    Biagram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Biagram biagram) {
        return biagram.first == this.first && biagram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Biagram> set = new HashSet<>();

        for(int i=0; i<10; i++) {
            for(char j = 'a'; j <= 'z'; j++) {
                set.add(new Biagram(j,j));
            }
        }

        System.out.println("Set의 사이즈 :  "+ set.size());
    }
}
```

- 원하는 결과 : 26, 실제 결과 : Set의 사이즈 --> '260'
- 왜 이런 결과가 나왔을까?
    - equals는 재정의가 아닌 ___다중정의___
    - 재정의가 되지 않았기 때문에, set에서 Biagram객체를 넣을 때, "객체 식별성"만 확인하여 set에 값을 넣었기 때문이다.

</br>

## equals를 제대로 재정의하기 위해서는?
1. 매개변수 type을 Object로 받아야한다. 
2. HashCode를 재정의해주어야 한다.

```java
    // 변경 후 결과 : size -> 26
    @Override
    public boolean equals(Object o) {
        if(!(o instanceof Biagram))
            return false;
        Biagram biagram = (Biagram) o;
        return biagram.first == this.first && biagram.second == this.second;
    }
```


</br>

## 결론

### 상위 클래스의 메서드를 재정의하는 경우, 꼭! ___@Override 애너테이션___ 을 쓰자
- 단, 상위 클래스의 추상 메서드를 재정의할 때는 안달아도 됨.(단다고 해로울 것도 없음)
    - 이유 : 구현되지 않은 추상 메서드가 있다면, 컴파일러가 알려주기 때문. 
- 재정의하는 모든 메서드에 작성해주는 것이 좋음.