---
name: java-project-structure
description: Java 프로젝트의 패키지 구조와 디렉토리 레이아웃을 설계하고 실제로 생성해주는 스킬. 사용자가 아키텍처 패턴(Layered, DDD, Hexagonal/Clean Architecture, Multi-module)을 선택하면 구조를 제안하고 확인 후 파일을 생성한다. "프로젝트 구조", "패키지 구조", "아키텍처 구조", "DDD", "헥사고날", "클린 아키텍처", "멀티모듈", "레이어드", "계층형 구조", "Spring Boot 구조"처럼 Java 프로젝트 설계를 언급하는 모든 상황에서 이 스킬을 사용한다. 새 프로젝트 시작 시뿐만 아니라 기존 프로젝트에 구조를 추가하거나 리팩터링할 때도 사용한다.
---

# Java Project Structure Generator

## 이 스킬이 하는 일

사용자가 원하는 아키텍처 패턴을 선택하면:
1. 프로젝트 기본 정보를 수집한다
2. 아키텍처 선택지를 제시하고 (복수 선택 가능)
3. 선택에 맞는 디렉토리 트리를 제안한다
4. 확인 후 실제 파일/디렉토리를 생성한다
5. 예시 파일 생성 여부도 선택하게 한다

---

## Step 1: 프로젝트 정보 수집

다음 정보를 묻거나 현재 프로젝트에서 추론한다:

| 항목 | 예시 | 추론 방법 |
|------|------|-----------|
| Base package | `kr.co.redsy.myapp` | `build.gradle`의 `group` + artifact |
| 프로젝트명 | `myapp` | `settings.gradle`의 `rootProject.name` |
| 빌드 툴 | Gradle / Maven | `build.gradle` 또는 `pom.xml` 존재 여부 |
| Java 버전 | 21 (기본값) | `build.gradle`의 `languageVersion` |
| 도메인명 | `user`, `order`, `product` | DDD/Hexagonal 선택 시만 필요. 없으면 `sample`로 대체 |

기존 프로젝트라면 파일을 먼저 읽어서 추론하고, 확인만 받는다.

---

## Step 2: 아키텍처 선택

아래 옵션을 보여주고 사용자가 번호로 선택하게 한다. **복수 선택 가능**.

```
어떤 아키텍처 패턴을 사용하시겠습니까? (번호 입력, 복수 선택 가능)

[1] Layered Architecture        — Controller → Service → Repository 계층형 구조
                                  단순 CRUD, 소규모 팀에 적합
[2] DDD (Domain-Driven Design)  — 도메인별로 패키지를 분리하는 구조
                                  복잡한 비즈니스 도메인, 중대형 팀에 적합
[3] Hexagonal Architecture      — Port & Adapter (Clean Architecture)
                                  인프라 교체 가능성이 높거나 테스트 중심 개발 시 적합
[4] Multi-module                — Gradle/Maven 멀티모듈 프로젝트
                                  독립 배포, 팀별 소유권 분리가 필요할 때 적합
[5] Package by Layer (JPA in Domain) — 레이어가 최상위, 도메인 안에 JPA까지 허용
                                  실용적인 중간 단계: 순수 헥사고날보단 간단하고
                                  단순 레이어드보단 도메인 중심적

조합 예시: "1" (레이어드), "2,3" (DDD+헥사고날), "4,2,3" (멀티모듈+DDD+헥사고날)
```

선택이 들어오면 **Step 3**으로 바로 진행한다.

---

## Step 3: 구조 제안

선택된 아키텍처에 맞는 디렉토리 트리를 보여준다. 아래 **Architecture Patterns** 섹션의 정확한 구조를 사용한다.

트리를 보여준 후 다음 두 가지를 확인한다:
1. "이 구조로 진행할까요? 수정할 부분이 있으면 말씀해 주세요."
2. "예시 파일(빈 클래스/인터페이스 스켈레톤)도 함께 생성할까요?"

---

## Step 4: 파일 생성

확인을 받으면:

1. **디렉토리 생성**: 모든 패키지 디렉토리를 생성하고 각 디렉토리에 `.gitkeep` 파일을 추가한다.
2. **Java 파일 패키지 선언**: 생성하는 모든 `.java` 파일에 경로에 맞는 정확한 `package` 선언을 포함한다.
3. **예시 파일**: 요청한 경우 **Example File Templates** 섹션을 참고해 스켈레톤 클래스를 생성한다.
4. **Multi-module**: `settings.gradle`(또는 `pom.xml`)과 각 모듈의 `build.gradle`을 생성/수정한다.
5. **기존 파일 보호**: 이미 존재하는 파일은 덮어쓰기 전에 반드시 확인을 받는다.

생성 완료 후 생성된 파일 목록을 간단히 요약해준다.

---

## Architecture Patterns

### 1. Layered Architecture

```
src/main/java/{basePackage}/
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

### 2. DDD (Domain-Driven Design)

도메인별로 패키지를 분리한다. `{domain}`은 실제 도메인명(user, order 등)으로 치환.

```
src/main/java/{basePackage}/
├── {domain}/
│   ├── domain/
│   │   ├── model/           # Aggregate Root, Entity, Value Object
│   │   ├── repository/      # Repository 인터페이스 (도메인 계층 정의)
│   │   ├── service/         # Domain Service (순수 비즈니스 규칙)
│   │   └── event/           # Domain Event
│   ├── application/
│   │   └── service/         # Application Service (유스케이스 조율, 트랜잭션)
│   ├── infrastructure/
│   │   └── persistence/     # Repository 구현체 (JPA, QueryDSL 등)
│   └── presentation/
│       ├── api/             # @RestController
│       └── dto/             # Request/Response DTO
└── common/
    ├── exception/           # 공통 예외 (GlobalExceptionHandler 등)
    └── config/              # 공통 설정
```

### 3. Hexagonal Architecture (Ports & Adapters)

핵심 원칙: Application Core(domain + application)는 프레임워크나 DB를 모른다. 외부 세계는 Port를 통해서만 연결된다.

```
src/main/java/{basePackage}/
├── {domain}/
│   ├── domain/
│   │   ├── model/           # 순수 도메인 모델 (프레임워크 의존 없음)
│   │   └── exception/       # 도메인 예외
│   ├── application/
│   │   ├── port/
│   │   │   ├── in/          # Inbound Port — Use Case 인터페이스 (입력 경계)
│   │   │   └── out/         # Outbound Port — Repository/외부서비스 인터페이스 (출력 경계)
│   │   └── service/         # Use Case 구현 (Port In 구현, Port Out 사용)
│   └── adapter/
│       ├── in/
│       │   └── web/         # Web Adapter — @RestController, Port In 호출
│       └── out/
│           └── persistence/ # Persistence Adapter — Port Out 구현, JPA 사용
└── config/                  # Spring Bean 등록, 설정
```

### 5. Package by Layer (JPA in Domain)

레이어가 최상위 패키지이고 그 안에 도메인이 위치한다. 헥사고날과 달리 domain 패키지 안에 JPA Entity와 JpaRepository를 허용한다. infrastructure는 외부 연동(외부 API 호출, 메시징 등)이 생길 때만 추가한다.

```
src/main/java/{basePackage}/
├── domain/
│   └── {domain}/
│       ├── model/               # 도메인 모델 (순수 Java)
│       ├── entity/              # JPA 엔티티 (@Entity)
│       └── repository/
│           ├── {Domain}Repository.java      # 도메인 인터페이스
│           ├── {Domain}JpaRepository.java   # Spring Data JPA (package-private)
│           └── {Domain}RepositoryImpl.java  # 구현체 (package-private)
├── application/
│   └── {domain}/
│       └── service/             # Application Service (유스케이스, 트랜잭션)
├── presentation/
│   └── {domain}/
│       ├── api/                 # @RestController
│       └── dto/                 # Request/Response DTO (record)
├── infrastructure/              # 외부 연동이 생길 때만 추가 (외부 API, 메시징 등)
└── common/
    ├── exception/
    └── config/
```

### 4. Multi-module (Gradle 기준)

모듈별로 독립적인 `build.gradle`을 가지며, 의존 방향은 단방향이다.

```
{project-root}/
├── settings.gradle              # 모든 모듈 등록
├── build.gradle                 # 공통 플러그인/의존성 설정
├── {name}-domain/               # 순수 도메인 (외부 의존성 최소화)
│   ├── build.gradle
│   └── src/main/java/
├── {name}-application/          # 유스케이스, Application Service
│   ├── build.gradle             # depends on: domain
│   └── src/main/java/
├── {name}-infrastructure/       # DB, 외부 API, 메시징
│   ├── build.gradle             # depends on: domain, application
│   └── src/main/java/
└── {name}-api/                  # HTTP API, Spring Boot main 진입점
    ├── build.gradle             # depends on: application, infrastructure
    └── src/main/java/
```

각 모듈 내부는 Layered, DDD, Hexagonal 중 하나를 따를 수 있다.

---

## Combining Patterns

### DDD + Hexagonal (가장 권장되는 조합)

DDD의 도메인별 패키지 분리 + Hexagonal의 Port/Adapter 구조를 결합한다.

```
src/main/java/{basePackage}/
├── {domain}/
│   ├── domain/
│   │   ├── model/           # Aggregate Root, Entity, Value Object
│   │   ├── service/         # Domain Service
│   │   └── event/           # Domain Event
│   ├── application/
│   │   ├── port/
│   │   │   ├── in/          # Use Case 인터페이스
│   │   │   └── out/         # Output Port 인터페이스 (Repository, 외부서비스)
│   │   └── service/         # Application Service (Use Case 구현)
│   └── adapter/
│       ├── in/
│       │   └── web/         # @RestController + Request/Response DTO
│       └── out/
│           └── persistence/ # JPA 구현체 + JPA Entity (@Entity는 여기)
└── common/
    ├── exception/
    └── config/
```

### Multi-module + Layered

```
{project-root}/
├── settings.gradle
├── build.gradle
├── {name}-domain/
│   └── src/main/java/{basePackage}/
│       └── {domain}/entity/         # @Entity
├── {name}-application/
│   └── src/main/java/{basePackage}/
│       ├── {domain}/service/
│       └── {domain}/repository/     # JpaRepository 인터페이스
├── {name}-infrastructure/
│   └── src/main/java/{basePackage}/
│       └── {domain}/repository/impl/  # QueryDSL 등 구현
└── {name}-api/
    └── src/main/java/{basePackage}/
        └── {domain}/controller/
```

### Multi-module + DDD + Hexagonal

각 모듈이 아키텍처 계층 하나를 담당한다.

```
{project-root}/
├── settings.gradle
├── build.gradle
├── {name}-domain/               # domain 계층 (의존성 0)
│   └── src/main/java/{basePackage}/{domain}/domain/
├── {name}-application/          # application 계층 (port/in, port/out, service)
│   └── src/main/java/{basePackage}/{domain}/application/
├── {name}-infrastructure/       # adapter/out (JPA, 외부 API)
│   └── src/main/java/{basePackage}/{domain}/adapter/out/
└── {name}-api/                  # adapter/in (Web) + Spring Boot main
    └── src/main/java/{basePackage}/{domain}/adapter/in/
```

---

## Example File Templates

예시 파일 생성 시 아래 템플릿을 도메인명과 패키지에 맞게 치환해 생성한다.
모든 파일은 비즈니스 로직 없이 구조만 보여주는 최소한의 스켈레톤이다.

### Layered 예시 (도메인: User)

**`controller/UserController.java`**
```java
package {basePackage}.controller;

import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;
}
```

**`service/UserService.java`**
```java
package {basePackage}.service;

public interface UserService {
}
```

**`service/impl/UserServiceImpl.java`**
```java
package {basePackage}.service.impl;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
}
```

**`repository/UserRepository.java`**
```java
package {basePackage}.repository;

import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
}
```

**`domain/entity/User.java`**
```java
package {basePackage}.domain.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

---

### Hexagonal 예시 (도메인: User)

**`user/application/port/in/CreateUserUseCase.java`**
```java
package {basePackage}.user.application.port.in;

public interface CreateUserUseCase {
    Long createUser(CreateUserCommand command);

    record CreateUserCommand(String name, String email) {}
}
```

**`user/application/port/out/SaveUserPort.java`**
```java
package {basePackage}.user.application.port.out;

import {basePackage}.user.domain.model.User;

public interface SaveUserPort {
    User save(User user);
}
```

**`user/application/service/CreateUserService.java`**
```java
package {basePackage}.user.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import {basePackage}.user.application.port.in.CreateUserUseCase;
import {basePackage}.user.application.port.out.SaveUserPort;

@Service
@RequiredArgsConstructor
class CreateUserService implements CreateUserUseCase {
    private final SaveUserPort saveUserPort;

    @Override
    public Long createUser(CreateUserCommand command) {
        // TODO: implement
        return null;
    }
}
```

**`user/adapter/in/web/UserController.java`**
```java
package {basePackage}.user.adapter.in.web;

import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import {basePackage}.user.application.port.in.CreateUserUseCase;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    private final CreateUserUseCase createUserUseCase;
}
```

**`user/adapter/out/persistence/UserPersistenceAdapter.java`**
```java
package {basePackage}.user.adapter.out.persistence;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import {basePackage}.user.application.port.out.SaveUserPort;
import {basePackage}.user.domain.model.User;

@Component
@RequiredArgsConstructor
class UserPersistenceAdapter implements SaveUserPort {
    private final UserJpaRepository userJpaRepository;

    @Override
    public User save(User user) {
        // TODO: map domain → JPA entity, save, map back
        return null;
    }
}
```

**`user/domain/model/User.java`** (프레임워크 의존 없음)
```java
package {basePackage}.user.domain.model;

public class User {
    private final Long id;
    private final String name;
    private final String email;

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    // getters...
}
```

---

### Package by Layer (JPA in Domain) 예시 (도메인: User)

**`domain/user/model/User.java`**
```java
package {basePackage}.domain.user.model;

import lombok.*;

@Getter
public class User {
    private final Long id;
    private final String name;
    private final String email;

    private User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public static User create(String name, String email) {
        return new User(null, name, email);
    }

    public static User reconstitute(Long id, String name, String email) {
        return new User(id, name, email);
    }
}
```

**`domain/user/entity/UserJpaEntity.java`**
```java
package {basePackage}.domain.user.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    public UserJpaEntity(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

**`domain/user/repository/UserRepository.java`** — 도메인 인터페이스
```java
package {basePackage}.domain.user.repository;

import {basePackage}.domain.user.model.User;
import java.util.Optional;

public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
}
```

**`domain/user/repository/UserJpaRepository.java`**
```java
package {basePackage}.domain.user.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import {basePackage}.domain.user.entity.UserJpaEntity;

interface UserJpaRepository extends JpaRepository<UserJpaEntity, Long> {
}
```

**`domain/user/repository/UserRepositoryImpl.java`**
```java
package {basePackage}.domain.user.repository;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;
import {basePackage}.domain.user.entity.UserJpaEntity;
import {basePackage}.domain.user.model.User;
import java.util.Optional;

@Repository
@RequiredArgsConstructor
class UserRepositoryImpl implements UserRepository {

    private final UserJpaRepository userJpaRepository;

    @Override
    public User save(User user) {
        UserJpaEntity entity = new UserJpaEntity(user.getName(), user.getEmail());
        UserJpaEntity saved = userJpaRepository.save(entity);
        return User.reconstitute(saved.getId(), saved.getName(), saved.getEmail());
    }

    @Override
    public Optional<User> findById(Long id) {
        return userJpaRepository.findById(id)
                .map(e -> User.reconstitute(e.getId(), e.getName(), e.getEmail()));
    }
}
```

**`application/user/service/UserApplicationService.java`**
```java
package {basePackage}.application.user.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import {basePackage}.domain.user.model.User;
import {basePackage}.domain.user.repository.UserRepository;

@Service
@RequiredArgsConstructor
@Transactional
public class UserApplicationService {

    private final UserRepository userRepository;

    public Long createUser(String name, String email) {
        User user = User.create(name, email);
        User saved = userRepository.save(user);
        return saved.getId();
    }
}
```

**`presentation/user/dto/CreateUserRequest.java`**
```java
package {basePackage}.presentation.user.dto;

public record CreateUserRequest(String name, String email) {
}
```

**`presentation/user/api/UserController.java`**
```java
package {basePackage}.presentation.user.api;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import {basePackage}.application.user.service.UserApplicationService;
import {basePackage}.presentation.user.dto.CreateUserRequest;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserApplicationService userApplicationService;

    @PostMapping
    public ResponseEntity<Long> createUser(@RequestBody CreateUserRequest request) {
        Long id = userApplicationService.createUser(request.name(), request.email());
        return ResponseEntity.ok(id);
    }
}
```

---

### DDD 예시 (도메인: User)

**`user/domain/model/User.java`** — Aggregate Root
```java
package {basePackage}.user.domain.model;

public class User {
    private Long id;
    private String name;
    private String email;
}
```

**`user/domain/repository/UserRepository.java`** — 도메인 계층 인터페이스
```java
package {basePackage}.user.domain.repository;

import {basePackage}.user.domain.model.User;
import java.util.Optional;

public interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
}
```

**`user/application/service/UserApplicationService.java`**
```java
package {basePackage}.user.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import {basePackage}.user.domain.repository.UserRepository;

@Service
@RequiredArgsConstructor
@Transactional
public class UserApplicationService {
    private final UserRepository userRepository;
}
```

**`user/presentation/api/UserController.java`**
```java
package {basePackage}.user.presentation.api;

import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
}
```

---

## Gradle Multi-module 설정 예시

### `settings.gradle`
```groovy
rootProject.name = '{name}'
include '{name}-domain'
include '{name}-application'
include '{name}-infrastructure'
include '{name}-api'
```

### 루트 `build.gradle`
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '4.0.6' apply false
    id 'io.spring.dependency-management' version '1.1.7' apply false
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'

    group = '{group}'
    version = '0.0.1-SNAPSHOT'

    java {
        toolchain {
            languageVersion = JavaLanguageVersion.of(21)
        }
    }

    repositories {
        mavenCentral()
    }
}
```

### `{name}-api/build.gradle` (진입점 모듈)
```groovy
plugins {
    id 'org.springframework.boot'
}

dependencies {
    implementation project(':{name}-application')
    implementation project(':{name}-infrastructure')
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

### `{name}-domain/build.gradle`
```groovy
dependencies {
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```

### `{name}-application/build.gradle`
```groovy
dependencies {
    implementation project(':{name}-domain')
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```

### `{name}-infrastructure/build.gradle`
```groovy
dependencies {
    implementation project(':{name}-domain')
    implementation project(':{name}-application')
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
}
```

---

## 아키텍처 선택 가이드

사용자가 어떤 걸 골라야 할지 모를 때 간단히 안내한다:

| 상황 | 추천 |
|------|------|
| 간단한 CRUD, 소규모 팀 | Layered |
| 비즈니스 로직이 복잡하고 도메인이 많음 | DDD |
| 인프라 교체 가능성 높음, 테스트 중시 | Hexagonal |
| 팀별 독립 개발/배포 필요 | Multi-module |
| 현실적으로 가장 많이 쓰이는 구조 | DDD + Hexagonal |
| 실용적 중간 단계, JPA 허용 | Package by Layer (JPA in Domain) |
| 대규모, 완전한 분리 | Multi-module + DDD + Hexagonal |
