# item 48 스트림 병렬화는 주의해서 적용하라

## 병렬프로그래밍

프로세스의 개수가 점점 늘어가고 있어서 병렬 프로그래밍이 필요해져 갔다.

java7 버전의 경우는 fork-join 패키지를 통해 병렬 프로그래밍을 지원하였고 자바8부터는 parallel메서드를 통해 병렬프로그래밍을 지원한다.

하지만 안정성과 응답 가능 상태를 유지하는 것은 쉽지 않다.

예를 들어보자

```java
public class ParallelMersennePrimes {
    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .parallel()
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}

```

위 코드는 스트림을 통해서 이전에 봤던 메르센 소수를 구하는 함수인데 저 함수는 매~~~우 느리다.

그 이유는 파이프라인을 병렬화할 방법을 찾지 못해서이다.

__데이터 소스가 Stream.iterate거나 중간 연산으로 limit을 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.__

limit의 경우는 서로 병렬적으로 퍼진 스트림간에 어디까지 했는지 알 수 없기 때문.



## 효율적인 자료구조

ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int, long 범위일때 가장 좋다.

그 이유는 참조 지역성이 뛰어나기 때문이다.(하지만 실제 객체 메모리가 떨어져 저장된다면 이또한 느려진다.)



## 리듀싱 연산

리듀싱 연산은 스트림의 결과를 하나로 모아주는 연산이다.

sort와 같은 것이 들어가면 모든 것들을 모아서 수행해야 하기때문에 적당하지 않고 anyMatch, allMatch, nonMatch와 같이 조건에 맞으면 바로 반환되는 메서드들이 유리하다. (예를들어 마지막거를 반환한다거나 하는건 모든 스레드 간의 연산이 모인 상태에서 가능하다.)



## 병렬 연산의 문제

잘못 병렬화를 하면 오동작하거나 속도가 드려질 수 있다. 오동작을 안전 실패라 한다. 

병렬화로 인해서 스레드를 생성하고 그 스레드를 모으는 연산 과정에 의해서 속도에 문제가 발생할 수 있다.



## 그럼에도 불구하고..

그럼에도 불구하고 속도의 이득을 볼 수 있어 포기할 수 없다.

```java
static long pi(long n) {
  return LongStream.rangeClosed(2, n)
    .parallel()
    .mapToObj(BigInteger::valueOf)
    .filter(i -> i.isProbablePrime(50))
    .count();
}
```

코어에 수에 맞도록 속도가 줄어든다. 값이 크면 클수록 더 극적이다.