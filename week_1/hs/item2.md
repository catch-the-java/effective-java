# 아이템1. 생성자에 매개변수가 많다면 빌더를 고려하라.

정적 팩터리와 생성자는 모두 선택적인 매개변수가 많을때 적절히 대응하기 어렵다.

## 해결책 1. 생성자

- 예를들어
    - 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수 2개까지 받는 생성자... 선택적 매개변수 모두 받는 생성자 등등 모두 생성해야한다.
- 단점
    - 작성하고, 읽기 어려움
    - 만약 사용자가 설정하길 원치 않는 매개변수까지 포함되어 있어도 어쩔수없이 값을 설정해줘야함.
    - 등등
    

## 해결책 2. 자바빈즈 패턴(JavaBeans pattern)


매개변수가 없는 생성자로 객체를 만든 후, setter 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.
```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8); //(1)
cocaCola.setCalories(100);
cocaCola.setSodium(35);
```


- 장점
  - 앞선 생성자의 단점 해결. 인스턴스를 만들기 쉽고, 생성자 방식보단 읽기 쉬운 코드 
- 단점
  - 객체 하나를 만들려면 메서드를 여러개 호출해야함
  - 객체 완성되기전 자바빈이 중간에 사용되는 경우 안정적이지 않은 상태로 사용될 여지가 있다.
      - 예를들어 (1)에서 객체 사용하는 경우
  - getter, setter가 있어서 불변 클래스를 만들지 못함.
  - (쓰레드 간에 공유 가능한 상태가 있으니까) 스레드 안정성 얻으려면 추가적인 수고(locking)필요하다.
  

## 해결책 3. 빌더 패턴 (Builder pattern)

신축적인 (Telescoping, 여기선 필수적인 매개변수와 부가적인 매개변수 조합으로 여러 생성자를 만들 수 있다는 것을 의미하는 단어로 쓰인듯)   
생성자의 안정성과 자바빈을 사용할때 얻을 수 있었던 가독성을 모두 취할 수 있는 대안이 있다

1. 필요한 객체를 직접 생성자(or 정적 팩터리)를 호출해 빌더 객체를 얻는다.   
2. 빌더 객체가 제공하는 setter로 선택 매개변수를 설정한다.
    - setter 메서드는 자신(this)을 반환하기 때문에 연쇄적 호출 가능
3. build 메서드를 호출해서 필요한 객체를 얻는다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```


- 빌더패턴은 **유효성 확인 할수 있다.**
  - (최대한 일찍 발견하려면) 각 빌더 생성자와 메서드에서 수행
  - build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식 검사.

- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋다.
- 상당히 유연하다.
  - 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수 있다.

- 단점 
  - 빌더 생성 비용이 크지는 않지만 성능에 민감하면 문제가될 수도 있다.
  - (무시해도될듯) 점층적 생성자 패턴보다는 코드가 장황해져서 매개변수가 4개 이상은 되어야 값어치함
    - but 시간이 지날수록 매개변수는 증가하는걸 고려하자.

- 예제코드 참조
  - https://github.com/greekZorba/java-design-patterns/tree/master/builder

### Lombok을 이용하여 builder 구현

앞서 builder를 모두 구하면 힘드니 Lombok라이브러리의 @Builder어노테이션을 활용하자.
- builder패턴 적용 클래스
```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
    @Builder(builderMethodName = "travelCheckListBuilder")
    @ToString
    public class TravelCheckList {

        private Long id;
        private String passport;
        private String flightTicket;
        private String creditCard;
        private String internationalDriverLicense;
        private String travelerInsurance;

        public static TravelCheckListBuilder builder(Long id) {
            if(id == null) {
                throw new IllegalArgumentException("필수 파라미터 누락");
            }
            return travelCheckListBuilder().id(id);
        }
    }
```
- 확인용 
```java
public class MainClass {

        public static void main(String[] args) {
            // 빌더패턴을 통해 어떤 필드에 어떤 값을 넣어주는지 명확히 눈으로 확인할 수 있다!
            TravelCheckList travelCheckList = TravelCheckList.builder(145L)
                    .passport("M12345")
                    .flightTicket("Paris flight ticket")
                    .creditCard("Shinhan card")
                    .internationalDriverLicense("1235-5345")
                    .travelerInsurance("Samsung insurance")
                    .build();

            System.out.println("빌더 패턴 적용하기 : " + travelCheckList.toString());

        }

       // 결과
       // 빌더 패턴 적용하기 : TravelCheckList(id=1, passport=M12345, flightTicket=Paris flight ticket, creditCard=Shinhan card, internationalDriverLicense=1235-5345, travelerInsurance=Samsung insurance)
    }
```


---

# 참조 
- Lombok builder 예제
  - https://zorba91.tistory.com/298
- Lombok 사용법
  - https://github.com/cheese10yun/blog-sample/tree/master/lombok
- Lombok에서 builder 안전하게 사용법
  - https://www.popit.kr/builder-%EA%B8%B0%EB%B0%98%EC%9C%BC%EB%A1%9C-%EA%B0%9D%EC%B2%B4%EB%A5%BC-%EC%95%88%EC%A0%84%ED%95%98%EA%B2%8C-%EC%83%9D%EC%84%B1%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95/

