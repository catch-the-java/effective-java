### Item 75. 예외의 상세 메세지에 실패 관련 정보를 담으라

### 스택 추적
- Java에서 예외가 발생했을 때, 스택 추적 정보를 자동으로 출력한다.
- 스택추적은 예외 객체의 toString 메서드를 호출해 얻는 문자열로, 보통은 예외의 클래스 이름 뒤에 상세 메시지가 붙는다.


</br>
</br>


### 실패 순간을 포착하려면 발생한 예외에 관여된 모든 매개변수와 필드의 값을 실패 메시지에 담아야 한다.
- Ex)
    - IndexOutOfBoundsException이 났을 때, 상세 메세지에는 범위의 최솟값, 최댓값, 그리고 범위를 벗어난 인덱스의 값을 담아야 한다.
    - 실패에 관한 많은 것을 알려주게 하라
        - 무엇을 고쳐야 할지 분석하는 데 큰 도움이 된다.
    - 보안에 문제가 있는 것은 보여주지 않도록 하자!
    ```java
    public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
        super(String.format("최솟값: %d, 최댓값 : %d, 인덱스 : %d", lowerBound, upperBound, index));
        this.lowerBound = lowerBound;
        this.uppderBound = upperBound;
        this.index = index;
    }
    ```