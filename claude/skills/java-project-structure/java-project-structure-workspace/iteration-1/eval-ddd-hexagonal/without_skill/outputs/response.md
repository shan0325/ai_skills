# Spring Boot DDD + Hexagonal Architecture 프로젝트 구조

Base Package: `kr.co.redsy.shop`
도메인: `user`, `order`
빌드 도구: Gradle

---

## 디렉토리 구조

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
    │   │   └── kr/
    │   │       └── co/
    │   │           └── redsy/
    │   │               └── shop/
    │   │                   ├── ShopApplication.java
    │   │                   │
    │   │                   ├── user/
    │   │                   │   ├── domain/
    │   │                   │   │   ├── model/
    │   │                   │   │   │   ├── User.java
    │   │                   │   │   │   ├── UserId.java
    │   │                   │   │   │   └── UserStatus.java
    │   │                   │   │   ├── repository/
    │   │                   │   │   │   └── UserRepository.java
    │   │                   │   │   └── service/
    │   │                   │   │       └── UserDomainService.java
    │   │                   │   ├── application/
    │   │                   │   │   ├── port/
    │   │                   │   │   │   ├── in/
    │   │                   │   │   │   │   ├── RegisterUserUseCase.java
    │   │                   │   │   │   │   └── GetUserUseCase.java
    │   │                   │   │   │   └── out/
    │   │                   │   │   │       ├── LoadUserPort.java
    │   │                   │   │   │       └── SaveUserPort.java
    │   │                   │   │   └── service/
    │   │                   │   │       ├── RegisterUserService.java
    │   │                   │   │       └── GetUserService.java
    │   │                   │   └── adapter/
    │   │                   │       ├── in/
    │   │                   │       │   └── web/
    │   │                   │       │       ├── UserController.java
    │   │                   │       │       ├── request/
    │   │                   │       │       │   └── RegisterUserRequest.java
    │   │                   │       │       └── response/
    │   │                   │       │           └── UserResponse.java
    │   │                   │       └── out/
    │   │                   │           └── persistence/
    │   │                   │               ├── UserPersistenceAdapter.java
    │   │                   │               ├── UserJpaRepository.java
    │   │                   │               ├── UserJpaEntity.java
    │   │                   │               └── UserMapper.java
    │   │                   │
    │   │                   └── order/
    │   │                       ├── domain/
    │   │                       │   ├── model/
    │   │                       │   │   ├── Order.java
    │   │                       │   │   ├── OrderId.java
    │   │                       │   │   ├── OrderItem.java
    │   │                       │   │   └── OrderStatus.java
    │   │                       │   ├── repository/
    │   │                       │   │   └── OrderRepository.java
    │   │                       │   └── service/
    │   │                       │       └── OrderDomainService.java
    │   │                       ├── application/
    │   │                       │   ├── port/
    │   │                       │   │   ├── in/
    │   │                       │   │   │   ├── PlaceOrderUseCase.java
    │   │                       │   │   │   └── GetOrderUseCase.java
    │   │                       │   │   └── out/
    │   │                       │   │       ├── LoadOrderPort.java
    │   │                       │   │       └── SaveOrderPort.java
    │   │                       │   └── service/
    │   │                       │       ├── PlaceOrderService.java
    │   │                       │       └── GetOrderService.java
    │   │                       └── adapter/
    │   │                           ├── in/
    │   │                           │   └── web/
    │   │                           │       ├── OrderController.java
    │   │                           │       ├── request/
    │   │                           │       │   └── PlaceOrderRequest.java
    │   │                           │       └── response/
    │   │                           │           └── OrderResponse.java
    │   │                           └── out/
    │   │                               └── persistence/
    │   │                                   ├── OrderPersistenceAdapter.java
    │   │                                   ├── OrderJpaRepository.java
    │   │                                   ├── OrderJpaEntity.java
    │   │                                   └── OrderMapper.java
    │   │
    │   └── resources/
    │       └── application.yml
    └── test/
        └── java/
            └── kr/
                └── co/
                    └── redsy/
                        └── shop/
                            ├── user/
                            │   └── application/
                            │       └── service/
                            │           └── RegisterUserServiceTest.java
                            └── order/
                                └── application/
                                    └── service/
                                        └── PlaceOrderServiceTest.java
```

---

## 레이어 구조 설명

```
[Adapter In (Web/HTTP)] → [Application Port In (UseCase)] → [Application Service]
                                                                      ↓
                                                           [Domain Model / Service]
                                                                      ↓
[Adapter Out (Persistence/JPA)] ← [Application Port Out (LoadPort/SavePort)]
```

- **Domain Layer**: 순수 비즈니스 로직. 외부 의존성 없음
- **Application Layer**: 유스케이스 정의(Port In), 외부 연동 인터페이스(Port Out), 유스케이스 구현체(Service)
- **Adapter In**: HTTP 요청을 받아 유스케이스를 호출 (Controller)
- **Adapter Out**: 유스케이스가 요구하는 Port Out을 구현 (JPA Persistence)

---

## 파일 내용

### settings.gradle

```groovy
rootProject.name = 'shop'
```

---

### build.gradle

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.5'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'kr.co.redsy'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'

    runtimeOnly 'com.h2database:h2'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

---

### src/main/resources/application.yml

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:shopdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true

server:
  port: 8080
```

---

### ShopApplication.java

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

## User 도메인

### domain/model/UserId.java

```java
package kr.co.redsy.shop.user.domain.model;

import java.util.Objects;
import java.util.UUID;

public class UserId {

    private final String value;

    private UserId(String value) {
        this.value = value;
    }

    public static UserId newId() {
        return new UserId(UUID.randomUUID().toString());
    }

    public static UserId of(String value) {
        return new UserId(value);
    }

    public String getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof UserId)) return false;
        UserId userId = (UserId) o;
        return Objects.equals(value, userId.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    @Override
    public String toString() {
        return value;
    }
}
```

---

### domain/model/UserStatus.java

```java
package kr.co.redsy.shop.user.domain.model;

public enum UserStatus {
    ACTIVE,
    INACTIVE,
    DELETED
}
```

---

### domain/model/User.java

```java
package kr.co.redsy.shop.user.domain.model;

import java.time.LocalDateTime;

public class User {

    private final UserId id;
    private String name;
    private String email;
    private UserStatus status;
    private final LocalDateTime createdAt;

    private User(UserId id, String name, String email, UserStatus status, LocalDateTime createdAt) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.status = status;
        this.createdAt = createdAt;
    }

    // 신규 유저 생성 (팩토리 메서드)
    public static User create(String name, String email) {
        return new User(
            UserId.newId(),
            name,
            email,
            UserStatus.ACTIVE,
            LocalDateTime.now()
        );
    }

    // 기존 유저 복원 (재구성)
    public static User reconstitute(UserId id, String name, String email, UserStatus status, LocalDateTime createdAt) {
        return new User(id, name, email, status, createdAt);
    }

    public void deactivate() {
        if (this.status == UserStatus.DELETED) {
            throw new IllegalStateException("이미 삭제된 유저입니다.");
        }
        this.status = UserStatus.INACTIVE;
    }

    public void delete() {
        this.status = UserStatus.DELETED;
    }

    public UserId getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public UserStatus getStatus() { return status; }
    public LocalDateTime getCreatedAt() { return createdAt; }
}
```

---

### domain/repository/UserRepository.java

```java
package kr.co.redsy.shop.user.domain.repository;

import kr.co.redsy.shop.user.domain.model.User;
import kr.co.redsy.shop.user.domain.model.UserId;

import java.util.Optional;

public interface UserRepository {
    User save(User user);
    Optional<User> findById(UserId id);
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

---

### application/port/in/RegisterUserUseCase.java

```java
package kr.co.redsy.shop.user.application.port.in;

import kr.co.redsy.shop.user.domain.model.User;

public interface RegisterUserUseCase {

    User registerUser(Command command);

    record Command(String name, String email) {
        public Command {
            if (name == null || name.isBlank()) throw new IllegalArgumentException("이름은 필수입니다.");
            if (email == null || email.isBlank()) throw new IllegalArgumentException("이메일은 필수입니다.");
        }
    }
}
```

---

### application/port/in/GetUserUseCase.java

```java
package kr.co.redsy.shop.user.application.port.in;

import kr.co.redsy.shop.user.domain.model.User;

public interface GetUserUseCase {
    User getUser(String userId);
}
```

---

### application/port/out/LoadUserPort.java

```java
package kr.co.redsy.shop.user.application.port.out;

import kr.co.redsy.shop.user.domain.model.User;
import kr.co.redsy.shop.user.domain.model.UserId;

import java.util.Optional;

public interface LoadUserPort {
    Optional<User> loadById(UserId userId);
    Optional<User> loadByEmail(String email);
}
```

---

### application/port/out/SaveUserPort.java

```java
package kr.co.redsy.shop.user.application.port.out;

import kr.co.redsy.shop.user.domain.model.User;

public interface SaveUserPort {
    User save(User user);
}
```

---

### application/service/RegisterUserService.java

```java
package kr.co.redsy.shop.user.application.service;

import kr.co.redsy.shop.user.application.port.in.RegisterUserUseCase;
import kr.co.redsy.shop.user.application.port.out.LoadUserPort;
import kr.co.redsy.shop.user.application.port.out.SaveUserPort;
import kr.co.redsy.shop.user.domain.model.User;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional
public class RegisterUserService implements RegisterUserUseCase {

    private final LoadUserPort loadUserPort;
    private final SaveUserPort saveUserPort;

    @Override
    public User registerUser(Command command) {
        loadUserPort.loadByEmail(command.email()).ifPresent(u -> {
            throw new IllegalStateException("이미 사용 중인 이메일입니다: " + command.email());
        });

        User user = User.create(command.name(), command.email());
        return saveUserPort.save(user);
    }
}
```

---

### application/service/GetUserService.java

```java
package kr.co.redsy.shop.user.application.service;

import kr.co.redsy.shop.user.application.port.in.GetUserUseCase;
import kr.co.redsy.shop.user.application.port.out.LoadUserPort;
import kr.co.redsy.shop.user.domain.model.User;
import kr.co.redsy.shop.user.domain.model.UserId;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class GetUserService implements GetUserUseCase {

    private final LoadUserPort loadUserPort;

    @Override
    public User getUser(String userId) {
        return loadUserPort.loadById(UserId.of(userId))
            .orElseThrow(() -> new IllegalArgumentException("유저를 찾을 수 없습니다: " + userId));
    }
}
```

---

### adapter/in/web/request/RegisterUserRequest.java

```java
package kr.co.redsy.shop.user.adapter.in.web.request;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import kr.co.redsy.shop.user.application.port.in.RegisterUserUseCase;

public record RegisterUserRequest(
    @NotBlank(message = "이름은 필수입니다.")
    String name,

    @NotBlank(message = "이메일은 필수입니다.")
    @Email(message = "올바른 이메일 형식이어야 합니다.")
    String email
) {
    public RegisterUserUseCase.Command toCommand() {
        return new RegisterUserUseCase.Command(name, email);
    }
}
```

---

### adapter/in/web/response/UserResponse.java

```java
package kr.co.redsy.shop.user.adapter.in.web.response;

import kr.co.redsy.shop.user.domain.model.User;

import java.time.LocalDateTime;

public record UserResponse(
    String id,
    String name,
    String email,
    String status,
    LocalDateTime createdAt
) {
    public static UserResponse from(User user) {
        return new UserResponse(
            user.getId().getValue(),
            user.getName(),
            user.getEmail(),
            user.getStatus().name(),
            user.getCreatedAt()
        );
    }
}
```

---

### adapter/in/web/UserController.java

```java
package kr.co.redsy.shop.user.adapter.in.web;

import jakarta.validation.Valid;
import kr.co.redsy.shop.user.adapter.in.web.request.RegisterUserRequest;
import kr.co.redsy.shop.user.adapter.in.web.response.UserResponse;
import kr.co.redsy.shop.user.application.port.in.GetUserUseCase;
import kr.co.redsy.shop.user.application.port.in.RegisterUserUseCase;
import kr.co.redsy.shop.user.domain.model.User;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final RegisterUserUseCase registerUserUseCase;
    private final GetUserUseCase getUserUseCase;

    @PostMapping
    public ResponseEntity<UserResponse> registerUser(@RequestBody @Valid RegisterUserRequest request) {
        User user = registerUserUseCase.registerUser(request.toCommand());
        return ResponseEntity.status(HttpStatus.CREATED).body(UserResponse.from(user));
    }

    @GetMapping("/{userId}")
    public ResponseEntity<UserResponse> getUser(@PathVariable String userId) {
        User user = getUserUseCase.getUser(userId);
        return ResponseEntity.ok(UserResponse.from(user));
    }
}
```

---

### adapter/out/persistence/UserJpaEntity.java

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import jakarta.persistence.*;
import kr.co.redsy.shop.user.domain.model.UserStatus;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserJpaEntity {

    @Id
    @Column(name = "user_id")
    private String id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status;

    @Column(nullable = false)
    private LocalDateTime createdAt;

    public UserJpaEntity(String id, String name, String email, UserStatus status, LocalDateTime createdAt) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.status = status;
        this.createdAt = createdAt;
    }
}
```

---

### adapter/out/persistence/UserJpaRepository.java

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserJpaRepository extends JpaRepository<UserJpaEntity, String> {
    Optional<UserJpaEntity> findByEmail(String email);
}
```

---

### adapter/out/persistence/UserMapper.java

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import kr.co.redsy.shop.user.domain.model.User;
import kr.co.redsy.shop.user.domain.model.UserId;
import org.springframework.stereotype.Component;

@Component
public class UserMapper {

    public UserJpaEntity toJpaEntity(User user) {
        return new UserJpaEntity(
            user.getId().getValue(),
            user.getName(),
            user.getEmail(),
            user.getStatus(),
            user.getCreatedAt()
        );
    }

    public User toDomain(UserJpaEntity entity) {
        return User.reconstitute(
            UserId.of(entity.getId()),
            entity.getName(),
            entity.getEmail(),
            entity.getStatus(),
            entity.getCreatedAt()
        );
    }
}
```

---

### adapter/out/persistence/UserPersistenceAdapter.java

```java
package kr.co.redsy.shop.user.adapter.out.persistence;

import kr.co.redsy.shop.user.application.port.out.LoadUserPort;
import kr.co.redsy.shop.user.application.port.out.SaveUserPort;
import kr.co.redsy.shop.user.domain.model.User;
import kr.co.redsy.shop.user.domain.model.UserId;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
@RequiredArgsConstructor
public class UserPersistenceAdapter implements LoadUserPort, SaveUserPort {

    private final UserJpaRepository userJpaRepository;
    private final UserMapper userMapper;

    @Override
    public Optional<User> loadById(UserId userId) {
        return userJpaRepository.findById(userId.getValue())
            .map(userMapper::toDomain);
    }

    @Override
    public Optional<User> loadByEmail(String email) {
        return userJpaRepository.findByEmail(email)
            .map(userMapper::toDomain);
    }

    @Override
    public User save(User user) {
        UserJpaEntity entity = userMapper.toJpaEntity(user);
        UserJpaEntity saved = userJpaRepository.save(entity);
        return userMapper.toDomain(saved);
    }
}
```

---

## Order 도메인

### domain/model/OrderId.java

```java
package kr.co.redsy.shop.order.domain.model;

import java.util.Objects;
import java.util.UUID;

public class OrderId {

    private final String value;

    private OrderId(String value) {
        this.value = value;
    }

    public static OrderId newId() {
        return new OrderId(UUID.randomUUID().toString());
    }

    public static OrderId of(String value) {
        return new OrderId(value);
    }

    public String getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderId)) return false;
        OrderId orderId = (OrderId) o;
        return Objects.equals(value, orderId.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    @Override
    public String toString() {
        return value;
    }
}
```

---

### domain/model/OrderStatus.java

```java
package kr.co.redsy.shop.order.domain.model;

public enum OrderStatus {
    PENDING,
    CONFIRMED,
    SHIPPED,
    DELIVERED,
    CANCELLED
}
```

---

### domain/model/OrderItem.java

```java
package kr.co.redsy.shop.order.domain.model;

import java.math.BigDecimal;

public class OrderItem {

    private final String productId;
    private final String productName;
    private final int quantity;
    private final BigDecimal unitPrice;

    public OrderItem(String productId, String productName, int quantity, BigDecimal unitPrice) {
        if (quantity <= 0) throw new IllegalArgumentException("수량은 1 이상이어야 합니다.");
        if (unitPrice.compareTo(BigDecimal.ZERO) < 0) throw new IllegalArgumentException("단가는 0 이상이어야 합니다.");
        this.productId = productId;
        this.productName = productName;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }

    public BigDecimal getTotalPrice() {
        return unitPrice.multiply(BigDecimal.valueOf(quantity));
    }

    public String getProductId() { return productId; }
    public String getProductName() { return productName; }
    public int getQuantity() { return quantity; }
    public BigDecimal getUnitPrice() { return unitPrice; }
}
```

---

### domain/model/Order.java

```java
package kr.co.redsy.shop.order.domain.model;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class Order {

    private final OrderId id;
    private final String userId;
    private final List<OrderItem> items;
    private OrderStatus status;
    private final LocalDateTime orderedAt;

    private Order(OrderId id, String userId, List<OrderItem> items, OrderStatus status, LocalDateTime orderedAt) {
        this.id = id;
        this.userId = userId;
        this.items = new ArrayList<>(items);
        this.status = status;
        this.orderedAt = orderedAt;
    }

    public static Order place(String userId, List<OrderItem> items) {
        if (items == null || items.isEmpty()) {
            throw new IllegalArgumentException("주문 항목이 비어 있습니다.");
        }
        return new Order(
            OrderId.newId(),
            userId,
            items,
            OrderStatus.PENDING,
            LocalDateTime.now()
        );
    }

    public static Order reconstitute(OrderId id, String userId, List<OrderItem> items,
                                     OrderStatus status, LocalDateTime orderedAt) {
        return new Order(id, userId, items, status, orderedAt);
    }

    public void confirm() {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException("PENDING 상태의 주문만 확정할 수 있습니다.");
        }
        this.status = OrderStatus.CONFIRMED;
    }

    public void cancel() {
        if (this.status == OrderStatus.SHIPPED || this.status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("배송 중이거나 배송 완료된 주문은 취소할 수 없습니다.");
        }
        this.status = OrderStatus.CANCELLED;
    }

    public BigDecimal getTotalAmount() {
        return items.stream()
            .map(OrderItem::getTotalPrice)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public OrderId getId() { return id; }
    public String getUserId() { return userId; }
    public List<OrderItem> getItems() { return Collections.unmodifiableList(items); }
    public OrderStatus getStatus() { return status; }
    public LocalDateTime getOrderedAt() { return orderedAt; }
}
```

---

### application/port/in/PlaceOrderUseCase.java

```java
package kr.co.redsy.shop.order.application.port.in;

import kr.co.redsy.shop.order.domain.model.Order;

import java.math.BigDecimal;
import java.util.List;

public interface PlaceOrderUseCase {

    Order placeOrder(Command command);

    record Command(String userId, List<OrderItemCommand> items) {
        public Command {
            if (userId == null || userId.isBlank()) throw new IllegalArgumentException("유저 ID는 필수입니다.");
            if (items == null || items.isEmpty()) throw new IllegalArgumentException("주문 항목은 필수입니다.");
        }
    }

    record OrderItemCommand(
        String productId,
        String productName,
        int quantity,
        BigDecimal unitPrice
    ) {}
}
```

---

### application/port/in/GetOrderUseCase.java

```java
package kr.co.redsy.shop.order.application.port.in;

import kr.co.redsy.shop.order.domain.model.Order;

public interface GetOrderUseCase {
    Order getOrder(String orderId);
}
```

---

### application/port/out/LoadOrderPort.java

```java
package kr.co.redsy.shop.order.application.port.out;

import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderId;

import java.util.Optional;

public interface LoadOrderPort {
    Optional<Order> loadById(OrderId orderId);
}
```

---

### application/port/out/SaveOrderPort.java

```java
package kr.co.redsy.shop.order.application.port.out;

import kr.co.redsy.shop.order.domain.model.Order;

public interface SaveOrderPort {
    Order save(Order order);
}
```

---

### application/service/PlaceOrderService.java

```java
package kr.co.redsy.shop.order.application.service;

import kr.co.redsy.shop.order.application.port.in.PlaceOrderUseCase;
import kr.co.redsy.shop.order.application.port.out.SaveOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderItem;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Transactional
public class PlaceOrderService implements PlaceOrderUseCase {

    private final SaveOrderPort saveOrderPort;

    @Override
    public Order placeOrder(Command command) {
        List<OrderItem> items = command.items().stream()
            .map(i -> new OrderItem(i.productId(), i.productName(), i.quantity(), i.unitPrice()))
            .collect(Collectors.toList());

        Order order = Order.place(command.userId(), items);
        return saveOrderPort.save(order);
    }
}
```

---

### application/service/GetOrderService.java

```java
package kr.co.redsy.shop.order.application.service;

import kr.co.redsy.shop.order.application.port.in.GetOrderUseCase;
import kr.co.redsy.shop.order.application.port.out.LoadOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderId;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class GetOrderService implements GetOrderUseCase {

    private final LoadOrderPort loadOrderPort;

    @Override
    public Order getOrder(String orderId) {
        return loadOrderPort.loadById(OrderId.of(orderId))
            .orElseThrow(() -> new IllegalArgumentException("주문을 찾을 수 없습니다: " + orderId));
    }
}
```

---

### adapter/in/web/request/PlaceOrderRequest.java

```java
package kr.co.redsy.shop.order.adapter.in.web.request;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.Positive;
import kr.co.redsy.shop.order.application.port.in.PlaceOrderUseCase;

import java.math.BigDecimal;
import java.util.List;

public record PlaceOrderRequest(
    @NotBlank(message = "유저 ID는 필수입니다.")
    String userId,

    @NotEmpty(message = "주문 항목은 하나 이상 필요합니다.")
    List<OrderItemRequest> items
) {
    public PlaceOrderUseCase.Command toCommand() {
        List<PlaceOrderUseCase.OrderItemCommand> itemCommands = items.stream()
            .map(i -> new PlaceOrderUseCase.OrderItemCommand(
                i.productId(), i.productName(), i.quantity(), i.unitPrice()))
            .toList();
        return new PlaceOrderUseCase.Command(userId, itemCommands);
    }

    public record OrderItemRequest(
        @NotBlank String productId,
        @NotBlank String productName,
        @Positive int quantity,
        @Positive BigDecimal unitPrice
    ) {}
}
```

---

### adapter/in/web/response/OrderResponse.java

```java
package kr.co.redsy.shop.order.adapter.in.web.response;

import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderItem;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public record OrderResponse(
    String orderId,
    String userId,
    List<OrderItemResponse> items,
    BigDecimal totalAmount,
    String status,
    LocalDateTime orderedAt
) {
    public static OrderResponse from(Order order) {
        List<OrderItemResponse> itemResponses = order.getItems().stream()
            .map(OrderItemResponse::from)
            .toList();

        return new OrderResponse(
            order.getId().getValue(),
            order.getUserId(),
            itemResponses,
            order.getTotalAmount(),
            order.getStatus().name(),
            order.getOrderedAt()
        );
    }

    public record OrderItemResponse(
        String productId,
        String productName,
        int quantity,
        BigDecimal unitPrice,
        BigDecimal totalPrice
    ) {
        public static OrderItemResponse from(OrderItem item) {
            return new OrderItemResponse(
                item.getProductId(),
                item.getProductName(),
                item.getQuantity(),
                item.getUnitPrice(),
                item.getTotalPrice()
            );
        }
    }
}
```

---

### adapter/in/web/OrderController.java

```java
package kr.co.redsy.shop.order.adapter.in.web;

import jakarta.validation.Valid;
import kr.co.redsy.shop.order.adapter.in.web.request.PlaceOrderRequest;
import kr.co.redsy.shop.order.adapter.in.web.response.OrderResponse;
import kr.co.redsy.shop.order.application.port.in.GetOrderUseCase;
import kr.co.redsy.shop.order.application.port.in.PlaceOrderUseCase;
import kr.co.redsy.shop.order.domain.model.Order;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {

    private final PlaceOrderUseCase placeOrderUseCase;
    private final GetOrderUseCase getOrderUseCase;

    @PostMapping
    public ResponseEntity<OrderResponse> placeOrder(@RequestBody @Valid PlaceOrderRequest request) {
        Order order = placeOrderUseCase.placeOrder(request.toCommand());
        return ResponseEntity.status(HttpStatus.CREATED).body(OrderResponse.from(order));
    }

    @GetMapping("/{orderId}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable String orderId) {
        Order order = getOrderUseCase.getOrder(orderId);
        return ResponseEntity.ok(OrderResponse.from(order));
    }
}
```

---

### adapter/out/persistence/OrderJpaEntity.java

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import jakarta.persistence.*;
import kr.co.redsy.shop.order.domain.model.OrderStatus;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderJpaEntity {

    @Id
    @Column(name = "order_id")
    private String id;

    @Column(nullable = false)
    private String userId;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItemJpaEntity> items = new ArrayList<>();

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;

    @Column(nullable = false)
    private LocalDateTime orderedAt;

    public OrderJpaEntity(String id, String userId, OrderStatus status, LocalDateTime orderedAt) {
        this.id = id;
        this.userId = userId;
        this.status = status;
        this.orderedAt = orderedAt;
    }

    public void addItem(OrderItemJpaEntity item) {
        items.add(item);
        item.setOrder(this);
    }
}
```

---

### adapter/out/persistence/OrderItemJpaEntity.java

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;

@Entity
@Table(name = "order_items")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderItemJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private OrderJpaEntity order;

    @Column(nullable = false)
    private String productId;

    @Column(nullable = false)
    private String productName;

    @Column(nullable = false)
    private int quantity;

    @Column(nullable = false, precision = 19, scale = 2)
    private BigDecimal unitPrice;

    public OrderItemJpaEntity(String productId, String productName, int quantity, BigDecimal unitPrice) {
        this.productId = productId;
        this.productName = productName;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
    }

    void setOrder(OrderJpaEntity order) {
        this.order = order;
    }
}
```

---

### adapter/out/persistence/OrderJpaRepository.java

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import org.springframework.data.jpa.repository.JpaRepository;

public interface OrderJpaRepository extends JpaRepository<OrderJpaEntity, String> {
}
```

---

### adapter/out/persistence/OrderMapper.java

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderId;
import kr.co.redsy.shop.order.domain.model.OrderItem;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.stream.Collectors;

@Component
public class OrderMapper {

    public OrderJpaEntity toJpaEntity(Order order) {
        OrderJpaEntity entity = new OrderJpaEntity(
            order.getId().getValue(),
            order.getUserId(),
            order.getStatus(),
            order.getOrderedAt()
        );

        order.getItems().stream()
            .map(item -> new OrderItemJpaEntity(
                item.getProductId(),
                item.getProductName(),
                item.getQuantity(),
                item.getUnitPrice()
            ))
            .forEach(entity::addItem);

        return entity;
    }

    public Order toDomain(OrderJpaEntity entity) {
        List<OrderItem> items = entity.getItems().stream()
            .map(i -> new OrderItem(
                i.getProductId(),
                i.getProductName(),
                i.getQuantity(),
                i.getUnitPrice()
            ))
            .collect(Collectors.toList());

        return Order.reconstitute(
            OrderId.of(entity.getId()),
            entity.getUserId(),
            items,
            entity.getStatus(),
            entity.getOrderedAt()
        );
    }
}
```

---

### adapter/out/persistence/OrderPersistenceAdapter.java

```java
package kr.co.redsy.shop.order.adapter.out.persistence;

import kr.co.redsy.shop.order.application.port.out.LoadOrderPort;
import kr.co.redsy.shop.order.application.port.out.SaveOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderId;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
@RequiredArgsConstructor
public class OrderPersistenceAdapter implements LoadOrderPort, SaveOrderPort {

    private final OrderJpaRepository orderJpaRepository;
    private final OrderMapper orderMapper;

    @Override
    public Optional<Order> loadById(OrderId orderId) {
        return orderJpaRepository.findById(orderId.getValue())
            .map(orderMapper::toDomain);
    }

    @Override
    public Order save(Order order) {
        OrderJpaEntity entity = orderMapper.toJpaEntity(order);
        OrderJpaEntity saved = orderJpaRepository.save(entity);
        return orderMapper.toDomain(saved);
    }
}
```

---

## 테스트 예시

### RegisterUserServiceTest.java

```java
package kr.co.redsy.shop.user.application.service;

import kr.co.redsy.shop.user.application.port.in.RegisterUserUseCase;
import kr.co.redsy.shop.user.application.port.out.LoadUserPort;
import kr.co.redsy.shop.user.application.port.out.SaveUserPort;
import kr.co.redsy.shop.user.domain.model.User;
import kr.co.redsy.shop.user.domain.model.UserStatus;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verify;

@ExtendWith(MockitoExtension.class)
class RegisterUserServiceTest {

    @Mock
    private LoadUserPort loadUserPort;

    @Mock
    private SaveUserPort saveUserPort;

    @InjectMocks
    private RegisterUserService registerUserService;

    @Test
    @DisplayName("새로운 이메일로 유저 등록에 성공한다")
    void registerUser_success() {
        // given
        RegisterUserUseCase.Command command = new RegisterUserUseCase.Command("홍길동", "hong@example.com");
        given(loadUserPort.loadByEmail("hong@example.com")).willReturn(Optional.empty());
        given(saveUserPort.save(any(User.class))).willAnswer(inv -> inv.getArgument(0));

        // when
        User result = registerUserService.registerUser(command);

        // then
        assertThat(result.getName()).isEqualTo("홍길동");
        assertThat(result.getEmail()).isEqualTo("hong@example.com");
        assertThat(result.getStatus()).isEqualTo(UserStatus.ACTIVE);
        verify(saveUserPort).save(any(User.class));
    }

    @Test
    @DisplayName("이미 존재하는 이메일로 유저 등록 시 예외가 발생한다")
    void registerUser_duplicateEmail_throwsException() {
        // given
        RegisterUserUseCase.Command command = new RegisterUserUseCase.Command("홍길동", "hong@example.com");
        User existingUser = User.create("기존유저", "hong@example.com");
        given(loadUserPort.loadByEmail("hong@example.com")).willReturn(Optional.of(existingUser));

        // when & then
        assertThatThrownBy(() -> registerUserService.registerUser(command))
            .isInstanceOf(IllegalStateException.class)
            .hasMessageContaining("이미 사용 중인 이메일");
    }
}
```

---

### PlaceOrderServiceTest.java

```java
package kr.co.redsy.shop.order.application.service;

import kr.co.redsy.shop.order.application.port.in.PlaceOrderUseCase;
import kr.co.redsy.shop.order.application.port.out.SaveOrderPort;
import kr.co.redsy.shop.order.domain.model.Order;
import kr.co.redsy.shop.order.domain.model.OrderStatus;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verify;

@ExtendWith(MockitoExtension.class)
class PlaceOrderServiceTest {

    @Mock
    private SaveOrderPort saveOrderPort;

    @InjectMocks
    private PlaceOrderService placeOrderService;

    @Test
    @DisplayName("주문 항목이 있으면 주문이 정상적으로 생성된다")
    void placeOrder_success() {
        // given
        List<PlaceOrderUseCase.OrderItemCommand> items = List.of(
            new PlaceOrderUseCase.OrderItemCommand("prod-1", "노트북", 1, BigDecimal.valueOf(1_200_000))
        );
        PlaceOrderUseCase.Command command = new PlaceOrderUseCase.Command("user-123", items);
        given(saveOrderPort.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));

        // when
        Order result = placeOrderService.placeOrder(command);

        // then
        assertThat(result.getUserId()).isEqualTo("user-123");
        assertThat(result.getStatus()).isEqualTo(OrderStatus.PENDING);
        assertThat(result.getItems()).hasSize(1);
        assertThat(result.getTotalAmount()).isEqualByComparingTo(BigDecimal.valueOf(1_200_000));
        verify(saveOrderPort).save(any(Order.class));
    }

    @Test
    @DisplayName("주문 항목이 없으면 예외가 발생한다")
    void placeOrder_emptyItems_throwsException() {
        // given
        PlaceOrderUseCase.Command command = new PlaceOrderUseCase.Command("user-123", List.of());

        // when & then
        // Command record 자체가 검증하므로 생성 시점에 예외 발생
        assertThatThrownBy(() -> new PlaceOrderUseCase.Command("user-123", List.of()))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessageContaining("주문 항목은 필수입니다");
    }
}
```

---

## API 호출 예시

### 유저 등록

```http
POST /api/users
Content-Type: application/json

{
  "name": "홍길동",
  "email": "hong@example.com"
}
```

응답:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "홍길동",
  "email": "hong@example.com",
  "status": "ACTIVE",
  "createdAt": "2026-04-29T10:00:00"
}
```

---

### 주문 생성

```http
POST /api/orders
Content-Type: application/json

{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "items": [
    {
      "productId": "prod-001",
      "productName": "노트북",
      "quantity": 1,
      "unitPrice": 1200000
    },
    {
      "productId": "prod-002",
      "productName": "마우스",
      "quantity": 2,
      "unitPrice": 30000
    }
  ]
}
```

응답:
```json
{
  "orderId": "660e8400-e29b-41d4-a716-446655441111",
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "items": [
    {
      "productId": "prod-001",
      "productName": "노트북",
      "quantity": 1,
      "unitPrice": 1200000,
      "totalPrice": 1200000
    },
    {
      "productId": "prod-002",
      "productName": "마우스",
      "quantity": 2,
      "unitPrice": 30000,
      "totalPrice": 60000
    }
  ],
  "totalAmount": 1260000,
  "status": "PENDING",
  "orderedAt": "2026-04-29T10:05:00"
}
```

---

## 핵심 설계 원칙 요약

| 원칙 | 적용 내용 |
|------|-----------|
| **의존성 역전** | Application Layer가 Port 인터페이스만 알고, Adapter가 Port를 구현 |
| **도메인 순수성** | Domain Layer는 Spring, JPA 등 프레임워크 의존성 없음 |
| **팩토리 메서드** | `User.create()`, `Order.place()` - 생성 의도를 명확히 표현 |
| **reconstitute** | DB에서 복원할 때 `reconstitute()` 메서드로 구분 |
| **Command 객체** | UseCase Input을 record로 캡슐화하여 유효성 검증 포함 |
| **Unmodifiable Collection** | `Order.getItems()`는 외부 수정 불가 |
