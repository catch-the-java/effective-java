# item 57 지역변수의 범위를 최소화 하라

item 15인 클래스와 멤버의 점근 권한을 최소화하라와 취지가 비슷하다.

지역 변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성이 낮아진다.

## 지역변수를 선언하는 올바른 방법(권장)

- 지역 변수의 범위를 줄이는 가장 강력한 기법은 가장 처음에 쓰일 때 선언하기다.

- 거의 모든 지역변수는 선언과 동시에 초기화해야한다.
  단 try-catch 구문은 여기서 예외인다 변수를 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록안에서 초기화해야한다. 그렇지 않으면 예외가 블록을 넘어서 메서드에 전파될 가능성이 존재한다.

- 반복변수의 값이 반복문이 종료된 후에도 사용되야 하는 것이 아니라면 while문 대신 for문을 사용한다.
  예시:

  ```java
  //while문이 문제가 생기는 상황
  Iterator<Element> i = c.iterator();
  while(i.hasNext()) {
    doSomething(i.next());
  }
  
  ...
  Iterator<Element> i2 = c2.iterator();
  
  while(i.hasNext()) {//버그
    doSomething(i2.next());
  }
  ```

  위 코드만 봤을때 오타로 인해서 c2는 순회하지 않아 c2가 비어있는 것처럼 착각할 수 있다.(i의 값이 유지되기 때문에)

  ```java
  for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
  }
  
  for(Iterator<Element> i2 = c2.iterator(); i.hasNext();) {// 컴파일오류
    Element e = i2.next();
  }
  ```

  아래는 while과 같은 코드이지만 i를 찾을 수 없다는 컴파일 오류를 사전에 뱉어주게 된다.

  for문의 유효변수 범위는 블록안으로 제한되기 때문이다.

  (추가로 for문에 while문 보다 짧아 가독성이 좋다.)

- 메서드를 작게 유지하고 한가지 기능에만 집중하도록 한다.

