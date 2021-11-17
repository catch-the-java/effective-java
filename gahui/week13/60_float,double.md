## 60. 정확한 답이 필요하다면 float와 double은 피하라

### float와 double
- float와 doulbe은 공학 계산용으로 설계되었다.
- 정확한 결과가 필요할 때는 사용하면 안된다. 
- 특히, 금융 관련 계산과는 절대 맞지 않는다. 
- ex) 금융 계산에 부동 소수 사용한 경우
    ```java
    double funds = 1.00;
    int itemsBought = 0;
    for (double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }

    결과 
    3개 구입
    잔돈(달러):0.3999999999999999
    ```

</br>
</br>

### 금융 계산에는 BigDecimal, int, long을 사용해야 한다. 
    ```java
    public static void main(String[] args) {
        final BigDecimal TEN_CENTS = new BigDecimal(".10");
        
        int itemsBought = 0;
        BigDecimal funds = new BigDecimal("1.00");
        for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
            funds = funds.subtract(price);
            itemsBought++;
        }

        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
    ```

- BigDecimal의 단점
    - 기본 타입보다 쓰기가 훨씬 불편하고 훨씬 느리다.
    - 대안으로 int 혹은 long 타입을 쓸 수도 있다.
    ```java
    public static void main(String[] args) {
        int itemsBought = 0;
        int funds = 100;
        for(int price = 10; funds >= price; price += 10) {
            funds -= price;
            itemsBought++;
        }
        
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
    ```

</br>
</br>