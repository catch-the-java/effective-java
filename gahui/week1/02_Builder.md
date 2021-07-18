# Builder 
매개변수가 많다면 Builder를 사용하자. 

</br>

# 인스턴스 생성 방법
## 1. 생성자로 매개변수 작성하기 
```java
NutritionFacts cocaCola = new NutritionFacts(240,8,100,0,35,27)
```
- 이렇게 인스턴스를 만든다면, 어떤 변수의 값이 몇인지 쉽게 알 수 있을까?
- 아마, 어떤 값인지 클래스를 들어가보면서 확인해야할 것이다.
- 코드를 읽기가 굉장히 힘들어진다. 

</br>


## 2. 자바빈즈 패턴
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
```
- 빈 생성자로 만들어, setter을 통해 매개변수의 값들을 넣는다. 
- 생성자로 매개변수를 작성하는 것보다 읽기 쉬워졌지만, 일관성이 무너지며 클래스를 불변으로 만들 수 없다. 

</br>

## 3. 빌더패턴
```java
private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}


NutritionFacts cocaCola = new Builder(240, 8)
				.calories(100)
				.sodium(35)
				.carbohydrate(27)
				.build();
```



## 장단점
- **장점**
    1. 상당히 유연
        - 빌더 하나로 여러 객체를 순회하면서 만들 수 있음
        - 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있음
    2. 매개변수가 많을수록 관리하기 쉬움
        - 보통, API는 시간이 지날수록 매개변수가 많아지는 경향이 있음
        - 애초에 적더라도, API에서는 빌더로 만들어보자 !
- **단점**
    1. 객체를 만들기 위해서는 빌더부터 만들어야 함
    2. 생성 비용이 크지는 않지만, 성능에 민감한 상황에서는 문제가 될 수도 있음 
    3. 매개변수가 적을 때는 유용하지 않음