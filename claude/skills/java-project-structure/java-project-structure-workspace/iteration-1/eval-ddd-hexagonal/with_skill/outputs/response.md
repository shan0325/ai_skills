# Spring Boot 프로젝트 구조 — DDD + Hexagonal Architecture

## 프로젝트 정보

| 항목 | 값 |
|------|-----|
| Base Package | `kr.co.redsy.shop` |
| 프로젝트명 | `shop` |
| 빌드 툴 | Gradle |
| Java 버전 | 21 |
| 도메인 | `user`, `order` |
| 아키텍처 | DDD + Hexagonal (Ports & Adapters) |

---

## 디렉토리 트리

```
shop/
├── build.gradle
├── settings.gradle
├── gradlew
├── gradlew.bat
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
└── src/
    ├── main/
    │   ├── java/
    │   │   └── kr/co/redsy/shop/
    │   │       ├── ShopApplication.java
    │   │       │
    │   │       ├── user/
    │   │       │   ├── domain/
    │   │       │   │   ├── model/
    │   │       │   │   │   └── User.java
    │   │       │   │   ├── service/
    │   │       │   │   │   └── .gitkeep
    │   │       │   │   └── event/
    │   │       │   │       └── .gitkeep
    │   │       │   ├── application/
    │   │       │   │   ├── port/
    │   │       │   │   │   ├── in/
    │   │       │   │   │   │   ├── CreateUserUseCase.java
    │   │       │   │   │   │   └── GetUserUseCase.java
    │   │       │   │   │   └── out/
    │   │       │   │   │       ├── SaveUserPort.java
    │   │       │   │   │       └── LoadUserPort.java
    │   │       │   │   └── service/
    │   │       │   │       └── CreateUserService.java
    │   │       │   └── adapter/
    │   │       │       ├── in/
    │   │       │       │   └── web/
    │   │       │       │       ├── UserController.java
    │   │       │       │       ├── request/
    │   │       │       │       │   └── CreateUserRequest.java
    │   │       │       │       └── response/
    │   │       │       │           └── UserResponse.java
    │   │       │       └── out/
    │   │       │           └── persistence/
    │   │       │               ├── UserPersistenceAdapter.java
    │   │       │               ├── UserJpaEntity.java
    │   │       │               └── UserJpaRepository.java
    │   │       │
    │   │       ├── order/
    │   │       │   ├── domain/
    │   │       │   │   ├── model/
    │   │       │   │   │   ├── Order.java
    │   │       │   │   │   └── OrderItem.java
    │   │       │   │   ├── service/
    │   │       │   │   │   └── .gitkeep
    │   │       │   │   └── event/
    │   │       │   │       └── OrderCreatedEvent.java
    │   │       │   ├── application/
    │   │       │   │   ├── port/
    │   │       │   │   │   ├── in/
    │   │       │   │   │   │   ├── CreateOrderUseCase.java
    │   │       │   │   │   │   └── GetOrderUseCase.java
    │   │       │   │   │   └── out/
    │   │       │   │   │       ├── SaveOrderPort.java
    │   │       │   │   │       └── LoadOrderPort.java
    │   │       │   │   └── service/
    │   │       │   │       └── CreateOrderService.java
    │   │       │   └── adapter/
    │   │       │       ├── in/
    │   │       │       │   └── web/
    │   │       │       │       ├── OrderController.java
    │   │       │       │       ├── request/
    │   │       │       │       │   └── CreateOrderRequest.java
    │   │       │       │       └── response/
    │   │       │       │           └── OrderResponse.java
    │   │       │       └── out/
    │   │       │           └── persistence/
    │   │       │               ├── OrderPersistenceAdapter.java
    │   │       │               ├── OrderJpaEntity.java
    │   │       │               └── OrderJpaRepository.java
    │   │       │
    │   │       └── common/
    │   │           ├── exception/
    │   │           │   ├── GlobalExceptionHandler.java
    │   │           │   └── BusinessException.java
    │   │           └── config/
    │   │               └── JpaConfig.java
    │   └── resources/
    │       └── application.yaml
    └── test/
        └── java/
            └── kr/co/redsy/shop/
                ├── user/
                │   └── application/
                │       └── service/
                │           └── CreateUserServiceTest.java
                └── order/
                    └── application/
                        └── service/
                            └── CreateOrderServiceTest.java
```

---

## Gradle 설정 파일

### `settings.gradle`

```groovy
rootProject.name = 'shop'
```

### `build.gradle`

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '4.0.6'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'kr.co.redsy'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

---

## 예시 파일 내용

### 진입점

---

#### `src/main/java/kr/co/redsy/shop/ShopApplication.java`

```java
package kr.co.redsy.shop;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ShopApplication {
    public static void main(String[] args) {
        SpringApplication.run(ShopApplication.class, args);
    }
}
```

---

## USER 도메인

### Domain Layer

#### `src/main/java/kr/co/redsy/shop/user/domain/model/User.java`

```java
package kr.co.redsy.shop.user.domain.model;

/**
 * User Aggregate Root.
 * 프레임워크(JPA, Spring) 의존 없음 — 순수 도메인 객체.
 */
public class User {

    private final Long id;
    private String name;
    private String email;

    // 신규 생성용 (id 없음)
    public User(String name, String email) {
        this.id = null;
        this.name = name;
        this.email = email;
    }

    // 조회·복원용 (id 있음)
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public Long getId()    { return id; }
    public String getName()  { return name; }
    public String getEmail() { return email; }
}
```

---

### Application Layer — Inbound Ports

#### `src/main/java/kr/co/redsy/shop/user/application/port/in/CreateUserUseCase.java`

```java
package kr.co.redsy.shop.user.application.port.in;

public interface CreateUserUseCase {

    Long createUser(CreateUserCommand command);

    record CreateUserCommand(String name, String email) {}
}
```

#### `src/main/java/kr/co/redsy/shop/user/application/port/in/GetUserUseCase.java`

```java
package kr.co.redsy.shop.user.application.port.in;

import kr.co.redsy.shop.user.domain.model.User;

public interface GetUserUseCase {

    User getUser(Long userId);
}
```

---

### Application Layer — Outbound Ports

#### `src/main/java/kr/co/redsy/shop/user/application/port/out/SaveUserPort.java`

```java
package kr.co.redsy.shop.user.application.port.out;

import kr.co.redsy.shop.user.domain.model.User;

public interface SaveUserPort {
    User save(User user);
}
```

#### `src/main/java/kr/co/redsy/shop/user/application/port/out/LoadUserPort.java`

```java
package kr.co.redsy.shop.user.application.port.out;

import kr.co.redsy.shop.user.domain.model.User;
import java.util.Optional;

public interface LoadUserPort {
    Optional<User> findById(Long id);
}
```

---

### Application Layer — Service (Use Case 구현)

#### `src/main/java/kr/co/redsy/shop/user/application/service/CreateUserService.java`

```java
package kr.co.redsy.shop.user.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import kr.co.redsy.shop.user.application.port.in.CreateUserUseCase;
import kr.co.redsy.shop.user.application.port.out.SaveUserPort;
import kr.co.redsy.shop.user.domain.model.User;

@Service
@RequiredArgsConstructor
@Transactional
class CreateUserService implements CreateUserUseCase {

    private final SaveUserPort saveUserPort;

    @Override
    public Long createUser(CreateUserCommand command) {
        User user = new User(command.name(), command.email());
        User saved = saveUserPort.save(user);
        return saved.getId();
    }
}
```

---

### Adapter — In (Web)

#### `src/main/java/kr/co/redsy/shop/user/adapter/in/web/UserController.java`

```java
package kr.co.redsy.shop.user.adapter.in.web;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import kr.co.redsy.shop.user.adapter.in.web.request.CreateUserRequest;
import kr.co.redsy.shop.user.adapter.in.web.response.UserResponse;
import kr.co.redsy.shop.user.application.port.in.CreateUserUseCase;
import kr.co.redsy.shop.user.application.port.in.GetUserUseCase;
import kr.co.redsy.shop.user.domain.model.User;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final CreateUserUseCase createUserUseCase;
    private final GetUserUseCase getUserUseCase;

    @PostMapping
    public ResponseEntity<Long> createUser(@RequestBody CreateUserRequest request) {
        Long userId = createUserUseCase.createUser(
                new CreateUserUseCase.CreateUserCommand(request.name(), request.email()));
        return ResponseEntity.ok(userId);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getUser(@PathVariable Long id) {
        User user = getUserUseCase.getUser(id);
        return ResponseEntity.ok(new UserResponse(user.getId(), user.getName(), user.getEmail()));
    }
}
```

#### `src/main/java/kr/co/redsy/shop/user/adapter/in/web/request/CreateUserRequest.java`

```java
package kr.co.redsy.shop.user.adapter.in.web.request;

public record CreateUserRequest(String name, String email) {}
```

#### `src/main/java/kr/co/redsy/shop/user/adapter/in/web/response/UserResponse.java`

```java
package kr.co.redsy.shop.user.adapter.in.web.response;

public record UserResponse(Long id, String name, String email) {}
```

---

### Adapter — Out (Persistence)

#### `src/main/java/kr/co/redsy/shop/user/adapter/out/persistence/UserJpaEntity.java`

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import jakarta.persistence.*;
import lombok.*;

/**
 * JPA 전용 엔티티 (@Entity는 인프라 계층에만 위치).
 * 도메인 모델 User와 별도로 관리한다.
 */
@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class UserJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    UserJpaEntity(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

#### `src/main/java/kr/co/redsy/shop/user/adapter/out/persistence/UserJpaRepository.java`

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import org.springframework.data.jpa.repository.JpaRepository;

interface UserJpaRepository extends JpaRepository<UserJpaEntity, Long> {
}
```

#### `src/main/java/kr/co/redsy/shop/user/adapter/out/persistence/UserPersistenceAdapter.java`

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import kr.co.redsy.shop.user.application.port.out.LoadUserPort;
import kr.co.redsy.shop.user.application.port.out.SaveUserPort;
import kr.co.redsy.shop.user.domain.model.User;

import java.util.Optional;

@Component
@RequiredArgsConstructor
class UserPersistenceAdapter implements SaveUserPort, LoadUserPort {

    private final UserJpaRepository userJpaRepository;

    @Override
    public User save(User user) {
        UserJpaEntity entity = new UserJpaEntity(user.getName(), user.getEmail());
        UserJpaEntity saved = userJpaRepository.save(entity);
        return new User(saved.getId(), saved.getName(), saved.getEmail());
    }

    @Override
    public Optional<User> findById(Long id) {
        return userJpaRepository.findById(id)
                .map(e -> new User(e.getId(), e.getName(), e.getEmail()));
    }
}
```

---

## ORDER 도메인

### Domain Layer

#### `src/main/java/kr/co/redsy/shop/order/domain/model/Order.java`

```java
package kr.co.redsy.shop.order.domain.model;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * Order Aggregate Root.
 * 프레임워크 의존 없음.
 */
public class Order {

    private final Long id;
    private final Long userId;
    private final List<OrderItem> items;
    private OrderStatus status;

    public enum OrderStatus { PENDING, CONFIRMED, CANCELLED }

    // 신규 생성용
    public Order(Long userId, List<OrderItem> items) {
        this.id = null;
        this.userId = userId;
        this.items = new ArrayList<>(items);
        this.status = OrderStatus.PENDING;
    }

    // 복원용
    public Order(Long id, Long userId, List<OrderItem> items, OrderStatus status) {
        this.id = id;
        this.userId = userId;
        this.items = new ArrayList<>(items);
        this.status = status;
    }

    public Long getId()                    { return id; }
    public Long getUserId()                { return userId; }
    public List<OrderItem> getItems()      { return Collections.unmodifiableList(items); }
    public OrderStatus getStatus()         { return status; }
}
```

#### `src/main/java/kr/co/redsy/shop/order/domain/model/OrderItem.java`

```java
package kr.co.redsy.shop.order.domain.model;

/**
 * Order Value Object.
 */
public record OrderItem(Long productId, int quantity, long price) {}
```

#### `src/main/java/kr/co/redsy/shop/order/domain/event/OrderCreatedEvent.java`

```java
package kr.co.redsy.shop.order.domain.event;

/**
 * 주문 생성 도메인 이벤트.
 * 외부로 발행하거나 다른 도메인 서비스가 구독할 수 있다.
 */
public record OrderCreatedEvent(Long orderId, Long userId) {}
```

---

### Application Layer — Inbound Ports

#### `src/main/java/kr/co/redsy/shop/order/application/port/in/CreateOrderUseCase.java`

```java
package kr.co.redsy.shop.order.application.port.in;

import java.util.List;

public interface CreateOrderUseCase {

    Long createOrder(CreateOrderCommand command);

    record CreateOrderCommand(Long userId, List<OrderItemCommand> items) {
        public record OrderItemCommand(Long productId, int quantity, long price) {}
    }
}
```

#### `src/main/java/kr/co/redsy/shop/order/application/port/in/GetOrderUseCase.java`

```java
package kr.co.redsy.shop.order.application.port.in;

import kr.co.redsy.shop.order.domain.model.Order;

public interface GetOrderUseCase {

    Order getOrder(Long orderId);
}
```

---

### Application Layer — Outbound Ports

#### `src/main/java/kr/co/redsy/shop/order/application/port/out/SaveOrderPort.java`

```java
package kr.co.redsy.shop.order.application.port.out;

import kr.co.redsy.shop.order.domain.model.Order;

public interface SaveOrderPort {
    Order save(Order order);
}
```

#### `src/main/java/kr/co/redsy/shop/order/application/port/out/LoadOrderPort.java`

```java
package kr.co.redsy.shop.order.application.port.out;

import kr.co.redsy.shop.order.domain.model.Order;
import java.util.Optional;

public interface LoadOrderPort {
    Optional<Order> findById(Long orderId);
}
```

---

### Application Layer — Service (Use Case 구현)

#### `src/main/java/kr/co/redsy/shop/order/application/service/CreateOrderService.java`

```java
package kr.co.redsy.shop.order.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import kr.co.redsy.shop.order.application.port.in.CreateOrderUseCase;
import kr.co.redsy.shop.order.application.port.out.SaveOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderItem;

import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional
class CreateOrderService implements CreateOrderUseCase {

    private final SaveOrderPort saveOrderPort;

    @Override
    public Long createOrder(CreateOrderCommand command) {
        List<OrderItem> items = command.items().stream()
                .map(i -> new OrderItem(i.productId(), i.quantity(), i.price()))
                .toList();
        Order order = new Order(command.userId(), items);
        Order saved = saveOrderPort.save(order);
        return saved.getId();
    }
}
```

---

### Adapter — In (Web)

#### `src/main/java/kr/co/redsy/shop/order/adapter/in/web/OrderController.java`

```java
package kr.co.redsy.shop.order.adapter.in.web;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import kr.co.redsy.shop.order.adapter.in.web.request.CreateOrderRequest;
import kr.co.redsy.shop.order.adapter.in.web.response.OrderResponse;
import kr.co.redsy.shop.order.application.port.in.CreateOrderUseCase;
import kr.co.redsy.shop.order.application.port.in.GetOrderUseCase;
import kr.co.redsy.shop.order.domain.model.Order;

import java.util.List;

@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {

    private final CreateOrderUseCase createOrderUseCase;
    private final GetOrderUseCase getOrderUseCase;

    @PostMapping
    public ResponseEntity<Long> createOrder(@RequestBody CreateOrderRequest request) {
        List<CreateOrderUseCase.CreateOrderCommand.OrderItemCommand> itemCmds = request.items().stream()
                .map(i -> new CreateOrderUseCase.CreateOrderCommand.OrderItemCommand(
                        i.productId(), i.quantity(), i.price()))
                .toList();
        Long orderId = createOrderUseCase.createOrder(
                new CreateOrderUseCase.CreateOrderCommand(request.userId(), itemCmds));
        return ResponseEntity.ok(orderId);
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
        Order order = getOrderUseCase.getOrder(id);
        return ResponseEntity.ok(OrderResponse.from(order));
    }
}
```

#### `src/main/java/kr/co/redsy/shop/order/adapter/in/web/request/CreateOrderRequest.java`

```java
package kr.co.redsy.shop.order.adapter.in.web.request;

import java.util.List;

public record CreateOrderRequest(Long userId, List<OrderItemRequest> items) {
    public record OrderItemRequest(Long productId, int quantity, long price) {}
}
```

#### `src/main/java/kr/co/redsy/shop/order/adapter/in/web/response/OrderResponse.java`

```java
package kr.co.redsy.shop.order.adapter.in.web.response;

import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderItem;

import java.util.List;

public record OrderResponse(Long id, Long userId, String status, List<OrderItemResponse> items) {

    public record OrderItemResponse(Long productId, int quantity, long price) {}

    public static OrderResponse from(Order order) {
        List<OrderItemResponse> itemResponses = order.getItems().stream()
                .map(i -> new OrderItemResponse(i.productId(), i.quantity(), i.price()))
                .toList();
        return new OrderResponse(order.getId(), order.getUserId(), order.getStatus().name(), itemResponses);
    }
}
```

---

### Adapter — Out (Persistence)

#### `src/main/java/kr/co/redsy/shop/order/adapter/out/persistence/OrderJpaEntity.java`

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import jakarta.persistence.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class OrderJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long userId;

    private String status;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "order_id")
    private List<OrderItemJpaEntity> items = new ArrayList<>();

    OrderJpaEntity(Long userId, String status) {
        this.userId = userId;
        this.status = status;
    }

    void addItem(OrderItemJpaEntity item) {
        this.items.add(item);
    }
}
```

#### `src/main/java/kr/co/redsy/shop/order/adapter/out/persistence/OrderJpaRepository.java`

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import org.springframework.data.jpa.repository.JpaRepository;

interface OrderJpaRepository extends JpaRepository<OrderJpaEntity, Long> {
}
```

#### `src/main/java/kr/co/redsy/shop/order/adapter/out/persistence/OrderPersistenceAdapter.java`

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import kr.co.redsy.shop.order.application.port.out.LoadOrderPort;
import kr.co.redsy.shop.order.application.port.out.SaveOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderItem;

import java.util.List;
import java.util.Optional;

@Component
@RequiredArgsConstructor
class OrderPersistenceAdapter implements SaveOrderPort, LoadOrderPort {

    private final OrderJpaRepository orderJpaRepository;

    @Override
    public Order save(Order order) {
        OrderJpaEntity entity = new OrderJpaEntity(order.getUserId(), order.getStatus().name());
        order.getItems().forEach(i -> entity.addItem(
                new OrderItemJpaEntity(i.productId(), i.quantity(), i.price())));
        OrderJpaEntity saved = orderJpaRepository.save(entity);
        return toDomain(saved);
    }

    @Override
    public Optional<Order> findById(Long orderId) {
        return orderJpaRepository.findById(orderId).map(this::toDomain);
    }

    private Order toDomain(OrderJpaEntity e) {
        List<OrderItem> items = e.getItems().stream()
                .map(i -> new OrderItem(i.getProductId(), i.getQuantity(), i.getPrice()))
                .toList();
        return new Order(e.getId(), e.getUserId(), items, Order.OrderStatus.valueOf(e.getStatus()));
    }
}
```

---

## Common

### `src/main/java/kr/co/redsy/shop/common/exception/BusinessException.java`

```java
package kr.co.redsy.shop.common.exception;

public class BusinessException extends RuntimeException {

    public BusinessException(String message) {
        super(message);
    }
}
```

### `src/main/java/kr/co/redsy/shop/common/exception/GlobalExceptionHandler.java`

```java
package kr.co.redsy.shop.common.exception;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        return ResponseEntity.badRequest().body(new ErrorResponse(e.getMessage()));
    }

    public record ErrorResponse(String message) {}
}
```

### `src/main/java/kr/co/redsy/shop/common/config/JpaConfig.java`

```java
package kr.co.redsy.shop.common.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@Configuration
@EnableJpaRepositories(basePackages = "kr.co.redsy.shop")
public class JpaConfig {
}
```

---

## 테스트 예시 파일

### `src/test/java/kr/co/redsy/shop/user/application/service/CreateUserServiceTest.java`

```java
package kr.co.redsy.shop.user.application.service;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import kr.co.redsy.shop.user.application.port.in.CreateUserUseCase;
import kr.co.redsy.shop.user.application.port.out.SaveUserPort;
import kr.co.redsy.shop.user.domain.model.User;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;

@ExtendWith(MockitoExtension.class)
class CreateUserServiceTest {

    @Mock
    SaveUserPort saveUserPort;

    @InjectMocks
    CreateUserService createUserService;

    @Test
    void createUser_success() {
        // given
        given(saveUserPort.save(any())).willReturn(new User(1L, "홍길동", "hong@example.com"));

        // when
        Long userId = createUserService.createUser(
                new CreateUserUseCase.CreateUserCommand("홍길동", "hong@example.com"));

        // then
        assertThat(userId).isEqualTo(1L);
    }
}
```

### `src/test/java/kr/co/redsy/shop/order/application/service/CreateOrderServiceTest.java`

```java
package kr.co.redsy.shop.order.application.service;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import kr.co.redsy.shop.order.application.port.in.CreateOrderUseCase;
import kr.co.redsy.shop.order.application.port.out.SaveOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderItem;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;

@ExtendWith(MockitoExtension.class)
class CreateOrderServiceTest {

    @Mock
    SaveOrderPort saveOrderPort;

    @InjectMocks
    CreateOrderService createOrderService;

    @Test
    void createOrder_success() {
        // given
        Order fakeOrder = new Order(1L, 10L, List.of(new OrderItem(100L, 2, 5000L)), Order.OrderStatus.PENDING);
        given(saveOrderPort.save(any())).willReturn(fakeOrder);

        // when
        Long orderId = createOrderService.createOrder(
                new CreateOrderUseCase.CreateOrderCommand(10L,
                        List.of(new CreateOrderUseCase.CreateOrderCommand.OrderItemCommand(100L, 2, 5000L))));

        // then
        assertThat(orderId).isEqualTo(1L);
    }
}
```

---

## 아키텍처 핵심 원칙 요약

| 계층 | 위치 | Spring 의존 | JPA 의존 |
|------|------|:-----------:|:--------:|
| Domain Model | `{domain}/domain/model/` | X | X |
| Domain Service | `{domain}/domain/service/` | X | X |
| Domain Event | `{domain}/domain/event/` | X | X |
| Inbound Port (Use Case) | `{domain}/application/port/in/` | X | X |
| Outbound Port | `{domain}/application/port/out/` | X | X |
| Application Service | `{domain}/application/service/` | O (`@Service`) | X |
| Web Adapter | `{domain}/adapter/in/web/` | O (`@RestController`) | X |
| Persistence Adapter | `{domain}/adapter/out/persistence/` | O (`@Component`) | O (`@Entity`) |

> **의존성 방향**: Adapter → Application (Port) → Domain
> Domain은 아무것도 모른다. Port만 바라본다.

---

## 생성된 파일 목록 요약

### User 도메인 (14개)
- `user/domain/model/User.java`
- `user/application/port/in/CreateUserUseCase.java`
- `user/application/port/in/GetUserUseCase.java`
- `user/application/port/out/SaveUserPort.java`
- `user/application/port/out/LoadUserPort.java`
- `user/application/service/CreateUserService.java`
- `user/adapter/in/web/UserController.java`
- `user/adapter/in/web/request/CreateUserRequest.java`
- `user/adapter/in/web/response/UserResponse.java`
- `user/adapter/out/persistence/UserJpaEntity.java`
- `user/adapter/out/persistence/UserJpaRepository.java`
- `user/adapter/out/persistence/UserPersistenceAdapter.java`
- `user/domain/service/.gitkeep`
- `user/domain/event/.gitkeep`

### Order 도메인 (15개)
- `order/domain/model/Order.java`
- `order/domain/model/OrderItem.java`
- `order/domain/event/OrderCreatedEvent.java`
- `order/application/port/in/CreateOrderUseCase.java`
- `order/application/port/in/GetOrderUseCase.java`
- `order/application/port/out/SaveOrderPort.java`
- `order/application/port/out/LoadOrderPort.java`
- `order/application/service/CreateOrderService.java`
- `order/adapter/in/web/OrderController.java`
- `order/adapter/in/web/request/CreateOrderRequest.java`
- `order/adapter/in/web/response/OrderResponse.java`
- `order/adapter/out/persistence/OrderJpaEntity.java`
- `order/adapter/out/persistence/OrderJpaRepository.java`
- `order/adapter/out/persistence/OrderPersistenceAdapter.java`
- `order/domain/service/.gitkeep`

### Common (3개)
- `common/exception/BusinessException.java`
- `common/exception/GlobalExceptionHandler.java`
- `common/config/JpaConfig.java`

### 진입점 및 설정 (3개)
- `ShopApplication.java`
- `build.gradle`
- `settings.gradle`

### 테스트 (2개)
- `user/application/service/CreateUserServiceTest.java`
- `order/application/service/CreateOrderServiceTest.java`

**총 37개 파일** (`.gitkeep` 포함)
