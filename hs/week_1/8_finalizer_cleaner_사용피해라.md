# 아이템8. finalizer와 cleaner 사용을 피하라

## finalizer & cleaner 란?
- 자바는 객체 소멸자는 2가지 (finalizer과 그 대안인 cleaner)을 제공한다.
- 하지만 기본적으로는 사용하면 안된다.
- 문제점은 아래 참조.

<br/>

- c++ destructor과 다르게 java에서는 객체를 회수하는 역할을 가비지 컬렉터가 담당.

<br/>

## 문제점

- __finalizer, cleaner는 실행되기까지 얼마나 걸릴지 알 수 없다.__
  - __심지어 수행 여부도 보장안한다.__
  - 즉, 제때 실행되어야 하는 작업은 할 수 없어서 문제 생길 수 있다.
  - 예를들어
    - 파일 닫기를 finalize, cleaner하면 처리를 게을리해서 오류 생길 수 있다. 시스템에 열 수 있는 파일 개수에는 한계가 있기 때문.
    - 클래스에 finalizer를 달아두면 인스턴스의 자원 회수가 늦어질 수도 있음. -> out of memory 위험
    - 상태를 영구적으로 수정하는 작업에서는 절대 finalizer, cleaner에 의존해서는 안된다.
      - ex) 데이터베이스 같은 공유 자원의 영구 락 해제

<br/>

- finalize, cleaner는 심각한 성능, 보안문제 동반한다.
  - finalizer는 가비지 컬렉터의 효율을 떨어 뜨린다.


## 대안

- autocloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고나면 close 메서드를 호출한다.


## 쓰임새

- finalizer, cleaner의 쓰임새는 두가지있다.  

1. 자원의 소유자가
