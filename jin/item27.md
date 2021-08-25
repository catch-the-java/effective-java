# Item 27 비검사 경고를 제거하라

제네릭을 사용하기 시작하면 굉장히 많은 컴파일러 경고를 보게 된다.

1. 비검사 형변환 경고
2. 비검사 메서드 호출 경고
3. 비검사 매개변수화 가변인수 타입경고
4. 비검사 변환 경고
5. 등등..

다음과 같이 잘못 코드를 작성했다고 가정해보자.

```java
Set<Lark> exaltation = new HashSet();
```

이렇게 코드를 작성하면 구현체인 HashSet에 매개변수 타입을 선언해주지 않아서 unchecked warning이 나타나게 될 것이다. 이 문제는 <>를 추가해주면 해결할 수 있다. 코드를 작성하며 이러한 unchecked waring을 최대한 제거하는 방향으로 코드를 작성해주면 타입 안정성이 높아질 수 있다. 

*__하지만, 경고를 제거할 수 없지만 타입 안정성이 확보되어 있다면 @SuppressWarnings("unchecked") 어노테이션으로 해당 사항을 없애주자.__*

다만 반드시 안정성이 확보되어 있는 경우에만 제거해줘야 한다. 그렇지 않으면 다른사람이 코드를 봤을때 코드를 잘못 작성하여 ClassCastException을 발생시킬 가능성이 있으며 보안상 문제가 될 수도 있다.

그러므로 *__꼭 필요한 경우에만 가장 적은 범위로 @SuppressWarnings 어노테이션을 달아주자.__*

@SuppressWaring 어노테이션은 한줄이 넘는 메서드나 생성자에 달린 어노테이션을 발견한다면 지역변수로 옮겨주자. 예를 보자.

```java
public <T> T[] toArray(T[] a) {
  if (a.length < size)
    return (T[]) Arrays.copyOf(elements, size, a.getClass());
  System.arraycopy(elements, 0, a, 0 ,size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

위 코드는 ArrayList를 컴파일하면 이 메서드에서 다음 경고가 발생하게 된다.

ArrayList.java:305: warning: [unchecked] unchecked cast

​	return (T[]) Arrays.copyOf(elements, size, a.getClass());

required : T[]

Found: Object[]

어노테이션은 선언에만 달 수 있어 return 에는 달 수 없기 떄문에 범위를 최대한 줄이기위해 지역변수를 하나 더 추가해준다.

```java
public <T> T[] toArray(T[] a) {
  if(a.length < size) {
    @SuppressWarning("unchecked") 
    T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }
  ...
}
```

T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());이 부분에서 경고가 뜨지만 매개변수 타입과 생성한 배열의 타입이 모두 T[]이기 때문에 올바른 변환이다.

그리고 마지막으로 *__@SuppressWarning 어노테이션을 추가할 때에는 반드시 경고를 무시해도 되는 이유를 주석으로 반드시 달아놓자.__*

