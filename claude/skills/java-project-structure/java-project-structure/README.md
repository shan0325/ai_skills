# java-project-structure

Claude Code 스킬 — Java/Spring Boot 프로젝트의 패키지 구조를 대화형으로 설계하고 실제 파일까지 생성해주는 스킬입니다.

## 설치

`SKILL.md` 파일을 Claude Code 스킬 디렉토리에 배치합니다.

```
~/.claude/skills/java-project-structure/SKILL.md
```

## 사용법

프로젝트 디렉토리에서 Claude Code를 열고 아래와 같이 말하면 스킬이 자동으로 활성화됩니다.

```
프로젝트 구조 만들어줘
DDD로 패키지 구조 잡아줘
헥사고날 아키텍처로 설정해줘
지금 구조를 Package by Layer로 바꿔줘
```

---

## 동작 방식

### Step 1 — 프로젝트 정보 수집

`build.gradle` / `pom.xml` / `settings.gradle`을 읽어 아래 정보를 자동으로 추론합니다. 기존 프로젝트라면 확인만 받고 바로 진행합니다.

| 항목 | 예시 |
|------|------|
| Base package | `kr.co.example.myapp` |
| 프로젝트명 | `myapp` |
| 빌드 툴 | Gradle / Maven |
| Java 버전 | 21 |
| 도메인명 | `user`, `order`, `product` |

### Step 2 — 아키텍처 선택

번호로 선택하며, **복수 조합**이 가능합니다.

```
[1] Layered Architecture
[2] DDD (Domain-Driven Design)
[3] Hexagonal Architecture (Ports & Adapters)
[4] Multi-module
[5] Package by Layer (JPA in Domain)
```

### Step 3 — 구조 제안

선택한 패턴의 디렉토리 트리를 먼저 보여줍니다. 수정 요청을 받은 뒤 확정합니다.

### Step 4 — 파일 생성

- 모든 패키지 디렉토리 + `.gitkeep` 생성
- 각 `.java` 파일에 정확한 `package` 선언 포함
- 예시 파일(스켈레톤 클래스) 생성 여부를 선택 가능
- **기존 파일은 덮어쓰기 전 반드시 확인**

---

## 지원 아키텍처 패턴

### 1. Layered Architecture

가장 단순한 계층형 구조. 소규모 팀, 단순 CRUD에 적합.

```
src/main/java/{basePackage}/
├── controller/
├── service/
│   └── impl/
├── repository/
├── domain/
│   └── entity/
└── dto/
    ├── request/
    └── response/
```

### 2. DDD (Domain-Driven Design)

도메인이 최상위 패키지. 복잡한 비즈니스 도메인, 중대형 팀에 적합.

```
src/main/java/{basePackage}/
├── {domain}/
│   ├── domain/
│   │   ├── model/
│   │   ├── repository/
│   │   └── service/
│   ├── application/
│   │   └── service/
│   ├── infrastructure/
│   │   └── persistence/
│   └── presentation/
│       ├── api/
│       └── dto/
└── common/
```

### 3. Hexagonal Architecture (Ports & Adapters)

도메인이 프레임워크를 전혀 모름. 인프라 교체 가능성이 높거나 테스트 중심 개발에 적합.

```
src/main/java/{basePackage}/
├── {domain}/
│   ├── domain/
│   │   └── model/          # 순수 Java, 프레임워크 의존 없음
│   ├── application/
│   │   ├── port/
│   │   │   ├── in/          # UseCase 인터페이스
│   │   │   └── out/         # Repository/외부서비스 인터페이스
│   │   └── service/
│   └── adapter/
│       ├── in/web/          # @RestController
│       └── out/persistence/ # JPA 구현체
└── config/
```

### 4. Multi-module

Gradle/Maven 멀티모듈. 팀별 독립 개발·배포가 필요할 때 적합.

```
{project-root}/
├── {name}-domain/
├── {name}-application/
├── {name}-infrastructure/
└── {name}-api/
```

각 모듈 내부는 Layered / DDD / Hexagonal 중 하나를 따를 수 있습니다.

### 5. Package by Layer (JPA in Domain)

레이어가 최상위, 도메인 안에 JPA Entity와 JpaRepository를 허용하는 실용적인 구조.
순수 헥사고날보다 간단하고, 단순 레이어드보다 도메인 중심적.

```
src/main/java/{basePackage}/
├── domain/
│   └── {domain}/
│       ├── model/               # 도메인 모델 (순수 Java)
│       ├── entity/              # @Entity
│       └── repository/
│           ├── {Domain}Repository.java      # 도메인 인터페이스 (public)
│           ├── {Domain}JpaRepository.java   # Spring Data JPA (package-private)
│           └── {Domain}RepositoryImpl.java  # 구현체 (package-private)
├── application/
│   └── {domain}/service/
├── presentation/
│   └── {domain}/
│       ├── api/
│       └── dto/
├── infrastructure/              # 외부 API·메시징 등이 생길 때만 추가
└── common/
```

**핵심 설계 의도**
- `JpaRepository`와 `RepositoryImpl`은 package-private으로 선언해 외부 접근 차단
- `infrastructure`는 진짜 외부 연동(HTTP 클라이언트, 메시지 브로커 등)을 위해 비워둠

---

## 패턴 조합

| 조합 | 특징 |
|------|------|
| DDD + Hexagonal | 가장 많이 쓰이는 실전 구조 |
| Multi-module + Layered | 모듈 분리 + 단순 계층 |
| Multi-module + DDD + Hexagonal | 대규모, 완전한 계층·도메인 분리 |

---

## 아키텍처 선택 가이드

| 상황 | 추천 |
|------|------|
| 간단한 CRUD, 소규모 팀 | Layered |
| 비즈니스 로직이 복잡하고 도메인이 많음 | DDD |
| 인프라 교체 가능성 높음, 테스트 중시 | Hexagonal |
| 팀별 독립 개발/배포 필요 | Multi-module |
| 현실적으로 가장 많이 쓰이는 구조 | DDD + Hexagonal |
| 실용적 중간 단계, JPA 허용 | Package by Layer (JPA in Domain) |
| 대규모, 완전한 분리 | Multi-module + DDD + Hexagonal |

---

## 파일 구성

```
java-project-structure/
├── SKILL.md    # 스킬 본문 (Claude Code가 읽는 파일)
└── README.md   # 이 파일
```
