# Java 주요 개념 요약

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


## JUnit 5 - Parameterized Test
- JUnit 5의 `@ParameterizedTest`를 사용해 **다양한 입력 값을 테스트할 수 있는 방법**을 배움.
- **@ValueSource**: 단일 타입의 여러 값을 반복 테스트.
- **@CsvSource**: 여러 인자와 기대 결과를 함께 제공해 테스트.

```java
@ParameterizedTest
@CsvSource({
  "1, true",
  "2, true",
  "3, true",
  "4, false",
  "5, false"
})
void containsTest(int input, boolean expectedResult) {
  assertThat(numbers.contains(input)).isEqualTo(expectedResult);
}
```




