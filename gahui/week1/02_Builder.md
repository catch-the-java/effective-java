# Builder 
매개변수가 많다면 Builder를 사용하자. 

## 예시
```java
NutritionFacts cocaCola = new NutritionFacts(240,8,100,0,35,27)
```
- 이렇게 인스턴스를 만든다면, 어떤 변수의 값이 몇인지 쉽게 알 수 있을까?
- 아마, 어떤 값인지 클래스를 들어가보면서 확인해야할 것이다.
- 코드를 읽기가 굉장히 힘들어진다. 


## Builder 