# Class

## 중첩 클래스 (Nested Class)
- **정의**: 다른 클래스 안에 선언된 클래스
- **종류**:
    - **정적 중첩 클래스**: `static` 키워드를 사용하여 외부 클래스의 인스턴스와 독립적으로 존재.
    - **비정적 중첩 클래스**: 외부 클래스의 인스턴스에 종속되어 함께 존재.
- **사용 예시**:
```java
public class OuterClass {
  static class StaticNestedClass {
      void display() {
          System.out.println("Static Nested Class");
      }
  }

  class InnerClass {
      void display() {
          System.out.println("Non-static Inner Class");
      }
  }
}

```
## 비공개 클래스 (Non-public Class)
- 정의: public 키워드 없이 선언된 클래스. 같은 패키지 내에서만 접근 가능.
- 사용 목적: 외부에서 직접 접근할 필요가 없는 클래스에 대해 접근을 제한하고, 코드의 캡슐화를 강화.

```java
public class MainClass {
    private HelperClass helper;
}

// 비공개 클래스
class HelperClass {
    void assist() {
        System.out.println("Helper class method");
    }
}
```

## 최신 트렌드

1. **람다와 함수형 프로그래밍 활용 증가**
    - Java 8 이후로 **람다 표현식과 함수형 인터페이스**가 도입되면서, 간단한 일회성 로직에서는 익명 중첩 클래스 대신 **람다 표현식**을 사용하는 추세입니다.
    - 예를 들어, `Runnable`을 익명 클래스와 람다 표현식으로 각각 구현할 수 있습니다:

      ```java
      // 익명 클래스
      Runnable runnable = new Runnable() {
          @Override
          public void run() {
              System.out.println("Running");
          }
      };
 
      // 람다 표현식
      Runnable runnable = () -> System.out.println("Running");
      ```

    - 이처럼 간결한 코드 작성이 가능해졌으며, 코드 가독성과 유지보수성이 크게 향상되었습니다.

2. **모듈 시스템을 통한 캡슐화 강화** (Java 9+)
    - Java 9에서는 **모듈 시스템**이 도입되어, 클래스와 패키지의 접근 제어를 더 세밀하게 설정할 수 있게 되었습니다. 이를 통해 프로젝트의 캡슐화와 의존성을 효과적으로 관리할 수 있습니다.
    - 예를 들어, `module-info.java` 파일을 통해 필요한 패키지들만 외부에 공개할 수 있습니다:

      ```java
      // module-info.java
      module com.example.app {
          exports com.example.app.utils; // 특정 패키지만 외부에 공개
      }
      ```

    - 이렇게 하면 외부에서 `com.example.app.utils` 패키지에 접근할 수 있지만, 나머지 패키지는 모듈 내부에 감출 수 있습니다. 이는 의도하지 않은 접근을 제한하고, 모듈 간의 명확한 경계를 설정하는 데 유용합니다.

3. **코드 단순화**
    - 현대적인 코드 작성에서는 중첩 클래스를 남발하지 않고, 가독성과 유지보수성을 고려하여 필요한 경우에만 사용하는 것이 트렌드입니다.
    - 특히, **큰 프로젝트**에서는 중첩 클래스와 비공개 클래스를 많이 사용하면 코드가 복잡해지고, 유지보수가 어려워질 수 있습니다. 따라서 관련 클래스는 가능한 한 **독립된 파일**로 분리해 관리하는 방향으로 발전하고 있습니다.
    - 예를 들어, `Order`와 `OrderItem` 클래스가 밀접하게 관련되어 있다면 중첩 클래스로 작성할 수도 있지만, 유지보수성과 코드 가독성을 위해 별도 파일로 관리하는 것이 권장됩니다.


## 객체 지향 코드 vs 절차적 코드

이 문장은 **객체 지향 프로그래밍(OOP)**과 절차적 프로그래밍의 성격 차이를 설명하는 것입니다. 두 패러다임은 서로 다른 방식으로 문제를 해결하므로, 어떤 변경은 객체 지향에서 쉽고 절차적 코드에서는 어려울 수 있으며, 그 반대 상황도 발생할 수 있습니다.

예시로 설명

1. 절차적 코드에서 어려운 변경이 객체 지향에서 쉬운 경우

   •	예를 들어, 다양한 종류의 결제 방식을 처리해야 하는 상황을 생각해봅시다. 절차적 프로그래밍에서는 결제 방식을 처리하는 processPayment 함수가 있고, 여기에서 조건문(if-else나 switch)으로 결제 유형을 구분하여 처리합니다.
```java
// 절차적 코드 예시
public void processPayment(String paymentType, double amount) {
if (paymentType.equals("creditCard")) {
// 신용 카드 결제 처리
} else if (paymentType.equals("paypal")) {
// 페이팔 결제 처리
}
// 새로운 결제 방식이 추가되면 여기에 또 추가
}
```
	•	여기서 새로운 결제 방식(예: 암호화폐 결제)이 추가되면 processPayment 함수에 else if 조건을 추가해야 합니다. 코드가 점점 길어지고 복잡해지면서 유지보수가 어려워집니다.
	•	반면에 객체 지향 코드에서는 다형성을 사용하여 각 결제 방식을 클래스로 분리하고, 인터페이스를 통해 공통 동작을 정의할 수 있습니다.
```java
// 객체 지향 코드 예시
public interface Payment {
void process(double amount);
}

public class CreditCardPayment implements Payment {
@Override
public void process(double amount) {
// 신용 카드 결제 처리
}
}

public class PaypalPayment implements Payment {
@Override
public void process(double amount) {
// 페이팔 결제 처리
}
}

// 새로운 결제 방식 추가 시 새로운 클래스 생성만 필요
public class CryptoPayment implements Payment {
@Override
public void process(double amount) {
// 암호화폐 결제 처리
}
}
```
	•	이제 새로운 결제 방식이 필요할 때마다 Payment 인터페이스를 구현하는 클래스를 하나만 추가하면 되므로 변경이 쉽습니다. 따라서 절차적 코드에서 어려운 변경이 객체 지향 코드에서는 쉬워집니다.

2. 객체 지향 코드에서 어려운 변경이 절차적 코드에서 쉬운 경우

   •	반대로, 간단한 작업이 필요할 때 절차적 코드가 더 직관적이고 빠르게 구현될 수 있습니다. 예를 들어, 1회성으로 실행되는 단순한 작업(파일 읽기 및 특정 데이터 가공) 같은 경우, 모든 동작을 한 함수 안에 절차적으로 작성할 수 있습니다.

```java
// 절차적 코드 예시
public void readFileAndProcess() {
// 파일 읽기
// 데이터 가공
// 결과 출력
}
```

	•	이 경우에 객체 지향으로 접근하면, FileReader, DataProcessor, ResultPrinter와 같은 여러 클래스를 정의해야 할 수 있습니다. 클래스와 메서드를 세분화하는 것이 오히려 복잡해지고 개발 속도가 느려질 수 있습니다.
	•	절차적 코드에서는 단순히 함수 하나로 작업을 완료할 수 있으므로 객체 지향으로 작성하는 것이 비효율적일 수 있습니다. 따라서, 객체 지향에서 어려운 변경이 절차적 코드에서 쉽게 해결될 수 있습니다.

요약

	•	OOP: 확장성, 재사용성이 필요한 구조에서는 변경이 쉽습니다. 하지만, 단순 작업에서는 불필요하게 복잡할 수 있습니다.
	•	절차적 프로그래밍: 단순 작업이나 일회성 작업에서는 효율적입니다. 하지만 확장성이 요구될 때는 유지보수가 어려워질 수 있습니다.

이렇게 서로 다른 방식이 서로의 약점을 보완하므로, 상황에 맞는 방식을 선택하는 것이 중요합니다.
