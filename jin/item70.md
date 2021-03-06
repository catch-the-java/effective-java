# item 70 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

## Java의 throwable 타입

1. 검사예외
2. 런타임 예외,
3. 에러

__호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용하라.__

검사 예외를 던지면 호출자가 그 예외를 catch로 잡아 처리하거나 더 바깥으로 전파하도록 강제하게 된다.

(이를 통해 메서드에 선언된 검사예제는 그 메서드를 호출 했을때 발생 가능한 결과라는 것을 알려준다.)



비검사 예외는 throwable 두가지로 바로 런타임 예외와 에러이다. 이 두가지는 예외를 프로그램에서 잡을 필요가 없거나 잡지 말아야한다.



## 런타임 예외

프로그래밍 오류를 나타낼 떄는 런타임 예외를 사용하자. 런타임예외는 대부분은 전제조건을 만족하지 못했을 떄 발생한다.

(전재 조건은 클라이언트가 해당 API의 명세를 지키지 않았을 떄 일어나는 것을 의미한다.)

예외가 한가지 발생했을때 발생했을때 복구할 수 있는 오류인지 아닌지 판단하기 쉽지 않으며 이는 API의 설계자의 판단에 달렸다.

복구가 가능하다면 검사예외, 불가능 하다면 런타임 예외를 던진다.



## 에러

에러는 보통 JVM이 자원부족, 불변식 깨짐 등 더이상 수행을 계속할 수 없는 상황을 나타낼 때 사용한다. 

(일반적으로 업계에 널리 퍼진 규약이다.)

따라서, Error를 상속받아서 하위 클래스를 구현하지는 말자.

그러므로 __우리가 구현하는 모든 비검사 예외는 RuntimeException을 상속받아서 구현해야한다.__



## RuntimeException을 상속받지 않은 일반적인 검사 예외

이로울게 없으니 절대로 사용하지 말자. API 사용자만 햇갈린다.