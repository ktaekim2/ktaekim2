# Modular Monolithic Architecture(모듈형 모노리스)
## 개요
모듈형 모노리스는 모노리스의 장점과 마이크로서비스의 단점을 보완한 새로운 형태의 아키텍처이다.
기본적으로 모노리스 형태이지만, 각 기능별 모듈화 하여 결합도를 낮추고 응집도를 높이는 방식으로 설계한다.
각 기능은 확장성을 가지면서 마이크로서비스로 전환할 수 있는 유연성을 가진다.

| 비교 항목       | 모놀리식 아키텍처                         | 모듈형 모노리스 아키텍처                                      | 마이크로서비스                                             |
|----------------|-----------------------------------------|-------------------------------------------------------------|----------------------------------------------------------|
| 배포 방식       | 단일 코드베이스와 단일 배포 단위           | 단일 코드베이스와 단일 배포 단위                                | 서비스별 독립 배포                                         |
| 코드 구조       | 전체 코드가 밀접하게 결합                 | 독립된 모듈로 나누어 코드 구조가 명확                             | 서비스별로 분리된 코드베이스                                |
| 확장성         | 기능별 확장 어려움                        | 모듈을 마이크로서비스로 전환할 수 있는 유연성 제공                 | 개별 서비스 확장 가능                                       |
| 유지보수성      | 규모가 커질수록 복잡성 증가               | 모듈화로 인해 유지보수와 기능별 개발 관리 용이                      | 서비스 분리로 인해 관리 복잡성 증가                           |
| 성능           | 모듈 간 내부 호출로 성능 우수               | 네트워크 통신 없이 빠른 모듈 호출 가능                            | 네트워크 통신 필요 (느림)                                   |
| 장애 격리       | 장애 발생 시 전체 시스템에 영향            | 장애 발생 시 다른 모듈에 미치는 영향 최소화 가능                    | 개별 서비스 장애 격리가 가능                                |
| 트랜잭션 관리   | 단일 트랜잭션 관리로 일관성 유지 가능       | 단일 트랜잭션 관리로 데이터 일관성 쉽게 유지                       | 분산 트랜잭션 관리 필요                                     |
| 설계 복잡성     | 단순한 구조                             | 모듈 간 의존성과 경계 설정으로 구조적 복잡성 증가                    | 서비스 간 API 의존성 관리가 필요                              |
| 모듈 간 호출     | 메모리 내부에서 호출 (빠름)              | 메모리 내부에서 호출 (빠름)                                   | 네트워크 통신 필요 (느림)                                   |
| 테스트와 디버깅  | 단일 애플리케이션에서 쉽게 통합 테스트 가능 | 단일 애플리케이션에서 쉽게 통합 테스트 가능                        | 서비스별 독립적 테스트 필요, 통합 테스트 복잡                    |
| 의존성 관리     | 같은 코드베이스에서 명확히 관리            | 같은 코드베이스에서 명확히 관리                                | 서비스 간 API 의존성 필요, 변경 시 각 서비스에 영향            |
| 장기 확장성     | 초기 개발 및 유지보수 편리하나 확장에 한계  | 모듈화로 유연한 확장 가능                                    | 큰 규모의 확장에 적합, 자유로운 서비스 추가 및 관리 가능        |

## 모듈 구성 및 패키지 구조
```
com.example.common         # 공통 모듈
├── config                 # 공통 설정
├── dto                    # 공통 DTO 및 기본 엔티티
├── exception              # 공통 예외 및 예외 처리기
└── util                   # 공통 유틸리티 클래스

com.example.moduleA        # 모듈 A
├── controller             # 모듈 A의 컨트롤러
├── service                # 모듈 A의 서비스
└── repository             # 모듈 A의 레포지토리

com.example.moduleB        # 모듈 B
├── controller             # 모듈 B의 컨트롤러
├── service                # 모듈 B의 서비스
└── repository             # 모듈 B의 레포지토리
```
common 모듈 외에는 다른 모듈과 의존성을 갖지 않도록 결합도를 낮춘다.
공통 기능을 common 모듈에 배치하고, 최초 개별 모듈 단일 배포 시, common 모듈은 넥서스 배포하여 다른 모듈이 참조하도록 한다.

## 단일 ApplicationContext vs 다중 ApplicationContext
단일 `ApplicationContext`와 다중 `ApplicationContext` 구조는 모듈형 모노리스 아키텍처에서 모듈 간 결합도와 독립성을 결정하는 중요한 선택입니다. 각각의 방식에 따라 모듈 구성과 패키지 구조가 달라질 수 있으며, 성능, 확장성, 유지보수성에서도 차이가 있습니다.

### 1. 단일 `ApplicationContext`

#### 특징
- **하나의 `ApplicationContext`**에서 모든 모듈을 관리하며, 공통 설정과 빈을 전체적으로 공유합니다.
- **모듈 간 결합이 약하지만 독립적이지는 않음**: 모든 모듈이 동일한 컨텍스트 내에서 동작하므로 빈 간 호출이 비교적 자유롭습니다.
- 모든 모듈이 동일한 애플리케이션 설정 파일(`application.yml`)을 공유하며, 개별 설정은 클래스 내부에서 관리합니다.

#### 모듈 구성 및 패키지 구조
```plaintext
com.new2day
├── common                 // 공통 모듈
│   ├── config             // 공통 설정
│   └── util               // 유틸리티 클래스
├── demo                   // Demo 모듈
│   ├── DemoModuleConfig   // Demo 모듈 설정 클래스
│   └── service            // Demo 서비스
└── reservation            // Reservation 모듈
    ├── ReservationModuleConfig   // Reservation 모듈 설정 클래스
    └── service            // Reservation 서비스
```

#### 장점
- **단순성**: 하나의 `ApplicationContext`로 관리되어 설정과 빈 등록이 단순하고 일관성이 있습니다.
- **유지보수 용이**: 모듈 간 빈을 공유할 수 있으므로 빈 주입과 호출이 간편합니다.
- **낮은 리소스 사용**: 하나의 컨텍스트만 유지하므로 리소스 소모가 적고 초기화 속도가 빠릅니다.

#### 단점
- **확장성 제한**: 대규모 시스템에서 모듈이 많아지면 `ApplicationContext`가 커지면서 성능이 저하될 수 있습니다.
- **독립 배포 어려움**: 각 모듈이 독립적인 애플리케이션으로 분리되기 어렵고, 공통 설정이 강하게 묶여 있어 모듈별 설정을 별도로 관리하기 힘듭니다.

### 2. 다중 `ApplicationContext`

#### 특징
- **모듈별로 독립적인 `ApplicationContext`**를 생성하여 각 모듈이 서로 독립적으로 동작할 수 있습니다.
- 모듈 간 결합이 낮으며 **완전한 독립성을 유지**하므로, 하나의 모듈이 분리되어 다른 애플리케이션으로 전환하기가 쉽습니다.
- 각 모듈이 자체적인 설정 파일을 사용하며, 필요할 때만 모듈 간 의존성을 설정합니다.

#### 모듈 구성 및 패키지 구조
```plaintext
com.new2day
├── ModularMonolithsApplication     // 전체 애플리케이션 진입점
├── common                          // 공통 모듈
│   ├── CommonModuleConfig          // 공통 모듈 설정 클래스
│   └── util                        // 공통 유틸리티 클래스
├── demo                            // Demo 모듈
│   ├── DemoModuleConfig            // Demo 모듈 설정 클래스
│   └── service                     // Demo 서비스
└── reservation                     // Reservation 모듈
    ├── ReservationModuleConfig     // Reservation 모듈 설정 클래스
    └── service                     // Reservation 서비스
```

#### 다중 `ApplicationContext` 구현 예시
`ModularMonolithsApplication`에서 `SpringApplicationBuilder`를 사용하여 모듈마다 `ApplicationContext`를 독립적으로 생성하고, `child()`나 `sibling()`으로 각 모듈을 설정합니다.

```java
public class ModularMonolithsApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplicationBuilder()
            .sources(ModularMonolithsApplication.class)
            .child(DemoModuleConfig.class).web(WebApplicationType.NONE)
            .sibling(ReservationModuleConfig.class).web(WebApplicationType.NONE)
            .build();

        application.run(args);
    }
}
```

#### 장점
- **확장성과 독립성**: 각 모듈이 독립적인 `ApplicationContext`에서 관리되므로, 성능 저하 없이 큰 시스템으로 확장할 수 있습니다.
- **독립 배포 용이**: 각 모듈이 독립적인 애플리케이션으로 전환할 준비가 되어 있으며, 필요에 따라 분리하여 배포할 수 있습니다.
- **환경별 설정 유연성**: 각 모듈에 독립적인 설정 파일을 지정할 수 있어 환경별 설정 관리가 용이합니다.

#### 단점
- **구성의 복잡성**: 다중 `ApplicationContext`로 인해 초기 구성과 유지보수가 상대적으로 복잡해질 수 있습니다.
- **빈 공유 제한**: 모듈이 독립적으로 관리되므로 빈을 서로 공유하기 어려워, 필요한 경우 빈을 명시적으로 전달하거나 다른 방식으로 호출해야 합니다.
- **리소스 사용 증가**: 여러 개의 `ApplicationContext`를 유지하므로, 리소스 사용이 증가하고 초기화 시간이 길어질 수 있습니다.

### 어느 방향이 더 나은가?

- **규모와 확장성 필요성에 따라 선택**:
  - **중소규모 애플리케이션**에서는 단일 `ApplicationContext` 구조가 더 적합합니다. 유지보수가 쉬우며, 단순한 구조로도 충분한 성능을 낼 수 있습니다.
  - **대규모 애플리케이션**이나 **향후 모듈을 독립적으로 분리할 가능성이 높은 경우**에는 다중 `ApplicationContext` 구조가 적합합니다. 이 구조는 모듈이 독립적으로 동작할 수 있어 확장성과 배포 유연성이 뛰어납니다.

- **모듈의 독립성이 중요**하다면 다중 `ApplicationContext` 구조가 더 좋으며, 초기 설정과 리소스 사용을 절감하려면 단일 `ApplicationContext` 구조가 유리합니다.

따라서, 시스템의 장기적인 요구와 확장 가능성에 따라 적절한 구조를 선택하는 것이 중요합니다.



---

네, 맞습니다. **단일 `ApplicationContext`에서 모듈을 독립시키려면** 언급하신 작업들이 필요하며, 다중 `ApplicationContext` 방식으로 미리 설계하면 이러한 추가 작업 없이 바로 독립이 가능합니다. 다만, **완전한 독립**을 위해 몇 가지 추가적인 작업이 필요할 수 있습니다.

### 1. 단일 `ApplicationContext`에서 모듈을 독립할 때 필요한 작업

- **설정 파일 (`yml`) 복사**:
  - 기존 `application.yml`에서 독립 모듈에 필요한 설정들을 분리하여 새 프로젝트에 복사하거나, 모듈별 `yml` 파일로 변경합니다.

- **스프링 부트 스타터 클래스 작성**:
  - 기존 프로젝트에 있던 `@SpringBootApplication` 클래스와는 별도로 **새로운 `@SpringBootApplication` 진입점을 작성**해야 합니다. 이를 통해 모듈이 **독립 애플리케이션으로 실행**될 수 있습니다.

- **테스트 코드 분리 및 재구성**:
  - 독립 모듈로 분리되면 **테스트 코드도 새 프로젝트로 이동**해야 하며, 의존성이 변경된 부분을 반영해 테스트 코드를 수정해야 합니다.
  - 특히, 모듈이 독립되면서 환경 설정이나 데이터베이스 구성이 달라질 수 있으므로, 테스트 환경도 그에 맞게 새로 설정합니다.

- **의존성 분리**:
  - 독립된 모듈이 기존 프로젝트의 다른 모듈에 의존하고 있다면, 이 의존성을 정리하거나 **필요한 부분을 `common`과 같은 별도의 모듈로 추출**해야 할 수 있습니다.
  - 기존에 `allowedDependencies`로 관리하던 의존성 설정도 다시 검토하여, 독립 모듈이 필요한 의존성만 포함하도록 수정해야 합니다.

### 2. 다중 `ApplicationContext` 방식에서의 독립 모듈 전환

다중 `ApplicationContext`를 사용하면 독립 모듈로 전환하는 과정이 더 간단해집니다.

- **즉시 독립 가능**:
  - 다중 `ApplicationContext`에서는 각 모듈이 이미 독립적인 `ApplicationContext`에서 동작하므로, **스프링 부트 스타터 클래스 작성이나 `yml` 파일 분리가 불필요**합니다.
  - 각 모듈의 `ApplicationContext`가 이미 분리되어 있으므로, 모듈을 쉽게 독립 애플리케이션으로 전환할 수 있습니다.

- **추가적인 의존성 작업 최소화**:
  - 모듈 간 의존성이 독립적이므로, 특정 모듈을 외부 프로젝트로 전환할 때 의존성 분리 작업이 줄어듭니다.
  - `@ApplicationModule`의 `allowedDependencies` 설정에 따라, 모듈이 필요로 하는 다른 모듈에 대한 의존성만 설정되어 있어 **추가 작업이 최소화**됩니다.

### 추가적으로 고려할 작업

- **외부 리소스 접근 및 구성**:
  - 분리된 모듈이 **데이터베이스, 메시지 큐, 파일 시스템**과 같은 외부 리소스를 새로 구성해야 할 수 있습니다. 예를 들어, `reservation` 모듈이 기존 프로젝트와 동일한 데이터베이스를 사용할지, 새로 구성할지를 결정해야 합니다.
  - 이러한 외부 리소스의 연결 설정과 접근 권한 등을 독립된 모듈에 맞게 새로 설정합니다.

- **API와 네트워크 설정**:
  - 분리된 모듈이 기존 모듈과 통신해야 한다면, **API 호출 방식이나 네트워크 구성**을 새로 설정해야 할 수 있습니다.
  - 예를 들어, 기존 `order` 모듈과 통신하던 `reservation` 모듈이 독립된 서비스가 되면, REST API, 메시지 큐, gRPC 등으로 모듈 간 연결을 설정해야 합니다.

### 결론

- **단일 `ApplicationContext`**: 모듈을 독립시킬 때 설정 파일 복사, 스프링 부트 스타터 작성, 테스트 코드 이동 등의 작업이 필요합니다.
- **다중 `ApplicationContext`**: 설정 파일과 스타터 클래스가 이미 분리되어 있어 모듈을 독립시킬 때 추가 작업이 최소화됩니다. 

장기적으로 **독립 가능성이 높은 모듈이 많다면 다중 `ApplicationContext`** 방식을 선택하는 것이 효율적입니다.