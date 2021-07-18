# 인스턴스화를 막으려거든 private 생성자를 사용하라
정적멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 게 아니다. 
-  하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다.
  - public으로
- 추상 클래스로 만들어지는 것은 인스턴스화를 막을 수 없다.
- **private 생성자**를 추가하여 클래스의 인스턴스화를 막자 
- **인스턴스 방지용**이라는 주석을 추가해주면 좋다.

- 실제 Utils 폴더에서 작성했었던 코드
```java
public class ResponseMessage {
  // 인스턴스 방지용
	private ResponseMessage() { }

	//s.f
	public static final String SUCCESS = "Success";
	public static final String FAIL = "fail";

```
- 해당 링크 : https://github.com/yougahee/auth-server/blob/master/auth_server/src/main/java/com/gaga/auth_server/utils/ResponseMessage.java