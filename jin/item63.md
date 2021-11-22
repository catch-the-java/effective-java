# item 63 문자열 연결은 느리니 주의하라

## +연산을 통한 연결

\+ 연산은 문자열을 연결하기에 굉장히 쉬운 방법이지만 문자열 연결 연산자는 n^2의 시간이 걸리게 된다.

그 이유는 두가지 문자열을 모두 복사해서 새로운 메모리 공간에 넣어줘야하기 때문이다.

## 해결 방법

성능을 포기하고 싶지 않다면 StringBuilder를 사용하여라.(동시성이 필요하다면 StringBuffer)