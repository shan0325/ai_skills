# Java Project Structure — Layered Architecture

## Step 1: 프로젝트 정보 확인

사용자 입력 및 기존 프로젝트 파일(`build.gradle`, `settings.gradle`)에서 추론한 정보입니다.

| 항목 | 값 |
|------|-----|
| Base package | `com.example.todoapp` |
| 프로젝트명 | `todoapp` |
| 빌드 툴 | Gradle |
| Java 버전 | 21 |
| 아키텍처 | Layered Architecture |
| 예시 파일 | 생성 안 함 |

---

## Step 2: 아키텍처 선택

사용자가 **Layered Architecture**를 선택했습니다.

---

## Step 3: 구조 제안

아래 디렉토리 트리를 생성합니다.

```
src/main/java/com/example/todoapp/
├── controller/          # @RestController — HTTP 요청/응답 처리
├── service/             # @Service — 비즈니스 로직 인터페이스
│   └── impl/            # Service 구현체
├── repository/          # @Repository / JpaRepository
├── domain/
│   └── entity/          # @Entity — JPA 엔티티
├── dto/
│   ├── request/         # 요청 DTO (record 또는 class)
│   └── response/        # 응답 DTO
└── config/              # @Configuration
```

---

## Step 4: 파일 생성

예시 파일은 생성하지 않습니다. 각 디렉토리에 `.gitkeep` 파일을 추가하여 Git에서 빈 디렉토리를 추적합니다.

### 생성할 디렉토리 및 `.gitkeep` 파일 목록

```
src/main/java/com/example/todoapp/controller/.gitkeep
src/main/java/com/example/todoapp/service/.gitkeep
src/main/java/com/example/todoapp/service/impl/.gitkeep
src/main/java/com/example/todoapp/repository/.gitkeep
src/main/java/com/example/todoapp/domain/entity/.gitkeep
src/main/java/com/example/todoapp/dto/request/.gitkeep
src/main/java/com/example/todoapp/dto/response/.gitkeep
src/main/java/com/example/todoapp/config/.gitkeep
```

### 패키지 구조 설명

| 패키지 | 역할 |
|--------|------|
| `com.example.todoapp.controller` | `@RestController` — HTTP 요청/응답 처리 |
| `com.example.todoapp.service` | `@Service` — 비즈니스 로직 인터페이스 정의 |
| `com.example.todoapp.service.impl` | Service 인터페이스 구현체 |
| `com.example.todoapp.repository` | `@Repository` / `JpaRepository` 확장 인터페이스 |
| `com.example.todoapp.domain.entity` | `@Entity` — JPA 엔티티 클래스 |
| `com.example.todoapp.dto.request` | 요청 DTO (`record` 또는 `class`) |
| `com.example.todoapp.dto.response` | 응답 DTO |
| `com.example.todoapp.config` | `@Configuration` — Spring 설정 클래스 |

---

생성이 완료되었습니다. 총 **8개 디렉토리**와 각 디렉토리의 `.gitkeep` 파일이 생성되었습니다.
