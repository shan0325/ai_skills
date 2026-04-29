기존 Spring Boot 프로젝트에 레이어드 아키텍처 기반의 패키지 구조를 아래와 같이 구성해 드립니다.

## 패키지 구조

```
src/
└── main/
    ├── java/
    │   └── com/
    │       └── example/
    │           └── todoapp/
    │               ├── TodoappApplication.java
    │               ├── controller/
    │               ├── service/
    │               │   └── impl/
    │               ├── repository/
    │               ├── domain/
    │               ├── dto/
    │               │   ├── request/
    │               │   └── response/
    │               ├── exception/
    │               │   └── handler/
    │               └── config/
    └── resources/
        ├── application.yaml
        ├── static/
        └── templates/
```

## 각 패키지 설명

| 패키지 | 역할 |
|---|---|
| `controller` | HTTP 요청을 받아 처리하는 REST 컨트롤러 계층 |
| `service` | 비즈니스 로직을 담당하는 서비스 계층 |
| `service/impl` | 서비스 인터페이스의 구현체 |
| `repository` | DB 접근을 담당하는 영속성 계층 (JPA Repository 등) |
| `domain` | 엔티티(Entity) 클래스 모음 |
| `dto` | 데이터 전송 객체(DTO) 모음 |
| `dto/request` | 클라이언트 → 서버 요청 DTO |
| `dto/response` | 서버 → 클라이언트 응답 DTO |
| `exception` | 커스텀 예외 클래스 모음 |
| `exception/handler` | 전역 예외 처리 핸들러 (GlobalExceptionHandler 등) |
| `config` | 애플리케이션 설정 클래스 모음 (Security, Swagger 등) |

## 레이어드 아키텍처 흐름

```
Client
  ↓ HTTP Request
Controller  (Presentation Layer)
  ↓
Service     (Business Layer)
  ↓
Repository  (Persistence Layer)
  ↓
Database
```

> **참고:** `service` 패키지에는 인터페이스를, `service/impl` 패키지에는 구현 클래스를 두는 방식으로 구성하면 의존성 역전 원칙(DIP)을 지킬 수 있습니다. 인터페이스 없이 서비스 클래스 직접 사용을 선호한다면 `impl` 패키지는 생략해도 됩니다.
