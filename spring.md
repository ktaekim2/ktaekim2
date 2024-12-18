# 스프링 프레임워크
## Bean
Bean이란, 스프링이라는 프레임워크에서 '스프링이 대신 관리해주는 객체'이다. 스프링이 알아서 만들고, 보관하고, 필요할 때 꺼내 쓰도록 준비해놓는 "물건"같은 것이다.  
우리가 Bean으로 등록한 객체는 스프링이 보관해 두었다가 우리가 필요할 때 그 객체(Bean)를 꺼내준다.

## 싱글톤 vs 프로토타입
싱글톤은 인스턴스가 하나이고, 프로토타입은 인스턴스가 여러개이다.  
@Bean, @Component, @Service 등 스프링 빈과 컴포넌트들은 기본적으로 싱글톤이다.  
DTO 등 일반적인 클래스들은 new 키워드로 새로운 객체를 만들어 다중 인스턴스가 된다.  

### 싱글톤 빈 특징
싱글톤 빈은 인스턴스 하나를 공유하지만 매서드 호출 시점에 전달되는 파라미터는 독립적이다.  
싱글톤 빈은 상태(필드)가 아닌 동작(매서드)에 집중하는 구조.  

하지만, 상태를 가지는 빈의 경우 문제가 될 수있다.

```java
@Service
public class LoginService {
    private String currentUser; // 상태를 저장

    public void login(String username) {
        this.currentUser = username; // 상태 저장
    }

    public String getCurrentUser() {
        return this.currentUser;
    }
}
```
login() 가 currentUser를 덮어쓰기 때문에, 모든 요청에 같은 currentUser가 공유되어 동시성 문제가 발생.  
싱글톤은 기본적으로 무상태(stateless)여야 한다. 입력값을 처리하여 결과를 반환하는 방식으로 사용.  
상태가 필요하다면, 매서드 내부 지역 변수나 파라미터를 사용해 상태를 관리해야한다.  

