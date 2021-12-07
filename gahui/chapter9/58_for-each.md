## 58. 전통적인 for문보다는 for-each문을 사용하라

</br>

### for문과 비교했을 때, for-each문 장점
- 명료, 유연, 버그를 예방해준다.
- 성능 저하도 없다.
- 가능한 모든 곳에서 for문이 아닌 for-each 문을 사용하자.


</br>
</br>

### for문을 사용했을 때, 문제가 되는 점
- 우리가 진짜 필요한 것이 "원소"뿐 일 때, for문의 인덱스 변수는 코드를 지저분하게 할 뿐이다. 
- 요소 종류가 많아지면, 오류가 생길 가능성이 높다. 


</br>
</br>

### for-each문의 장점
- 반복자와 인덱스를 사용하지 않으니 코드가 깔끔하고 오류가 날 일도 없다.
- 잘못된 예
    ```java
    enum Suit {CLUB, DIAMOND, HEART, SPADE}
    enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}
    
    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();
        for(Iterator<Suit> i = suits.iterator(); i.hasNext();)
            for(Iterator<Rank> j = ranks.iterator(); j.hasNext();)
                deck.add(new Card(i.next(), j.next()));
    }
    ```
    - i.next()가 계속 불려, 원하는 결과를 저장하지 못한다. 
    - 숫자가 바닥나면 NoSuchElementException 발생
- 오류를 해결한 코드
    ```java
    for (Suit suit : suits)
        for (Rank rank : ranks)
            deck.add(new Card(suit, rank));
    ```

</br>
</br>

### for-each문을 사용할 수 없는 경우
- 파괴적인 필터링
    - 컬렉션을 순회하면서 선택된 원소를 제거해야 할 때
    - 자바 8부터 removeIf메서드를 사용할 수 있다.
- 변형
    - 순회하면서 원소 값의 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
- 병렬반복
    - 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.


</br>
</br>

### Iterable
- for-each 문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.
- Iterable을 직접 구현하기는 까다롭지만, 원소들의 묶음을 표현하는 타입을 작성해야 한다면 Iterable을 구현하는 쪽으로 고민해보자.