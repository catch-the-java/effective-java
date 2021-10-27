## 56. 공개 API 요소에는 항상 문서화 주석을 작성하라
> 여러분의 API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.

</br>

### '자바독' 유틸리티
- 문서화 주석을 API문서로 변환


### API 문서 작성시 주의할 점
1. 기본 생성자에는 문서화 주석을 달 방법이 없다.
    - 공개 클래스에는 절대 기본 생성자를 사용하면 안된다.

2. 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 문서화 주석을 달아야 할 것이다.
    - 공개 API처럼 친절하게 설명하지 않아도

3. 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야한다.
    - what을 기술하자.

4. 클라이언트가 해당 메서드를 호출하기 위한 전제조건도 나열해야한다.

5. 메서드가 성공적으로 수행된 후 만족해야 하는 사후조건도 모두 나열해야한다.

6. 부작용도 문서화하자.
    - 시스템에 어떤 변화를 가져오는지 

7. @param, @return, @throws 태그를 작성하자.

</br>
</br>

### 규칙 및 관례
1. @throws 태그는 if로 시작해 해당 예외를 던지는 조건을 설명한다.
2. 관례상 @param, @return, @throws 태그의 설명에는 마침표를 붙이지 않는다.
3. 문서화 주석에 HTML태그 사용
    - 자바독 유틸리티가 문서화 주석을 HTML로 변환하므로 최종 HTML 문서에 반영된다.
4. {@code}태그
    - 효과1 : 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
    - 효과2 : 태그로 감싼 내용에 포함된 HTML요소나 다른 자바독 태그를 무시한다. 
5. this list 
    - this는 호출된 메서드가 자리하는 객체를 가리킨다.

- 예시
    ```java
    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= size()})
     */
    E get(int index);
    ```


</br>
</br>

### 자기사용 패턴 : @implSpec
- 올바로 재정의하는 방법을 알려줘야 한다.
- 메서드와 하위 클래스 사이의 계약을 설명.
- 하위 클래스들이 그 메서드를 상속하거나 super키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지 인지시키자. 
- 자바 11까지도 자바독 명령줄에 -tag "implSpec:a:Implementation Requirements:" 스위치를 켜야 한다.
- 예시
    ```java
    
    ```


</br>
</br>

### API설명에 <,>,& 등의 HTML 메타문자를 포함
-  {@literal}태그로 감싸기
    - 이 태그는 HTML 마크업이나 자바독 태그를 무시하게 해준다.


### 문서화 주석의 문장
- 첫 번째 문장은 해당 요소의 요약 설명으로 간주한다.
    - 요약 설명은 반드시 대상의 기능을 고유하게 기술해야 한다.
    - 헷갈리지 않으려면 한 클래스 안에서 요약 설명이 똑같은 멤버가 둘 이상이면 안된다. 
- 



</br>
</br>

### 