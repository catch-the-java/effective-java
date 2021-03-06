# item 62 다른 타입이 적절하다면 문자열 사용을 피해라

## String

문자열은 텍스트를 표현하기 위해 설계되었고 굉장히 잘 동작하지만, 의도하지 않는 용도로 쓰이는 경우가 왕왕 존재한다.

## 문제점

1. 문자열은 다른 값 타입을 대신하기에 적합하지 않다.

   예를들어 int 타입(수치형 타입)으로 받을수 있는 값을 string으로 받는다면 후에 연산이 필요할 때 이를 다시 변환 시켜줘야 하는 수고스러움과 비용을 감당해야 한다. 혹은 true/false의 질문에 대답해야하는 경우라면 열거 타입이나 boolean을 이용해야 한다.

2. 문자열은 열거타입을 대신하기에 적합하지 않다. 해당 이유에 대해서는 item 34에서 설명하였다.

3. 문자열은 혼합타입을 나타내기에 적절하지 않다.
   예를들어 \":"으로 구분되는 혼합 데이터가 있다고 가정했을때, 구분자를 제외한 부분에서 \":"가 들어간다면 파싱하는데에 비용이 추가로 들고 compareTo, equals 등과 같은 메서드를 제공할 수 없어 String에 있는 메서드를 이용해야한다.

4. 문자열은 권한을 표현하기에 적합하지 않다.
   권한을 간혹 문자열로 표현하는 경우가 있는데 이 문자열 키가 전역 이름공간에 공유되기 때문에(string contant pool) 문제가 생길 수 있다.

## 결론

문자열 말고 더 적합한 타입이 있다면 그것을 사용하자. Enum, 기본 타입, 혼합 타입 등