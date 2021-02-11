# Spring Actuator

> An actuator is a manufacturing term that refers to a mechanical device for moving or controlling something. Actuators can generate a large amount of motion from a small change.


## 엔드 포인트
- [Spring Boot 1.X](https://docs.spring.io/spring-boot/docs/1.5.x/reference/html/production-ready-endpoints.html) 에서는 기본으로 노출 되어 있는 엔드포인트가 꽤 많음
- [Spring Boot 2.X](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints) 부터는 두 개의 엔드포인트만 기본 노출되도록 변경 (`/actuator/info`, `/actuator/health`)


### 헬스체크
> [2.8. Health Information](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health)

- actuator 를 통해 헬스체크 엔드포인트 노출시 주의가 필요.
- [2.8.1. Auto-configured HealthIndicators](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health-indicators) 를 보면 레디스, 데이터소스 등의 헬스인디케이터가 존재 
- 별도의 actuator 설정이 없다면 이 헬스인디케이터의 결과를 종합하여 최종 `status`를 결정함. (아래 `management.endpoint.health.show-components` 활성화시 응답 참고)
- 그런데 이런 상황을 가정해보자. 
  - 애플리케이션에 redis 에 문제가 생겼을때를 대비한 fallback 로직이 구현되어 있고, (즉, redis 에 문제가 생겨도 애플리케이션 정상 작동)
  - 해당 애플리케이션이 동작하는 서버는 LB 뒤에 있음.
  - LB 는 주기적으로 인스턴스들의 헬스체크를 하고 실패한다면 LB 에서 제외.
  - 레디스에 일시적으로 문제가 생겼고 actuator 기본설정으로 헬스체크시 실패하게 됨.
  - 이로 인해 해당 서버는 LB 에서 제외되고 새로운 서버 투입.
  - 하지만 레디스가 여전히 같은 상태라면 위 상황의 반복.
  - `org.springframework.boot.actuate.health.HealthEndpointSupport.getCompositeHealth` 참고
- 따라서 헬스체크 실패 결과에 따른 일을 정확히 파악하고, 경우에 따라 헬스체크 엔드포인트를 직접 정의하는 것이 나을 수 있음.
- 액추에이터의 기본 헬스체크 구현을 변경하려면 `HealthIndicator` 를 [구현](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#writing-custom-healthindicators) 하거나 [헬스그룹](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health-groups) 을 커스텀.




#### 기본 응답
```
{
  "status": "UP"
}
```
#### `management.endpoint.health.show-details` 활성화
```
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "H2",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 499963174912,
        "free": 155270021120,
        "threshold": 10485760,
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    },
    "redis": {
      "status": "UP",
      "details": {
        "version": "2.8.19"
      }
    }
  }
}

```

#### `management.endpoint.health.show-components` 활성화
```
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "diskSpace": {
      "status": "UP"
    },
    "ping": {
      "status": "UP"
    },
    "redis": {
      "status": "UP"
    }
  }
}
```

## 참고
- [Spring Boot Actuator: Production-ready Features](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints)
