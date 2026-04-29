# Java 프로젝트 구조: Multi-module + DDD + Hexagonal

## 프로젝트 기본 정보

| 항목 | 값 |
|------|-----|
| 프로젝트명 | `commerce` |
| Base Package | `kr.co.myshop` |
| 도메인 | `product`, `payment` |
| 빌드 툴 | Gradle 멀티모듈 |
| Java 버전 | 21 |
| 아키텍처 | Multi-module + DDD + Hexagonal |

---

## 전체 디렉토리 트리

```
commerce/
├── settings.gradle
├── build.gradle
│
├── commerce-domain/                          # 순수 도메인 계층 (외부 의존성 없음)
│   ├── build.gradle
│   └── src/
│       └── main/
│           └── java/
│               └── kr/co/myshop/
│                   ├── product/
│                   │   └── domain/
│                   │       ├── model/
│                   │       │   ├── Product.java
│                   │       │   ├── ProductId.java
│                   │       │   └── Money.java
│                   │       ├── service/
│                   │       │   └── ProductDomainService.java
│                   │       └── event/
│                   │           └── ProductCreatedEvent.java
│                   └── payment/
│                       └── domain/
│                           ├── model/
│                           │   ├── Payment.java
│                           │   ├── PaymentId.java
│                           │   └── PaymentStatus.java
│                           ├── service/
│                           │   └── PaymentDomainService.java
│                           └── event/
│                               └── PaymentCompletedEvent.java
│
├── commerce-application/                     # 유스케이스 계층 (Port 정의 + Application Service)
│   ├── build.gradle
│   └── src/
│       └── main/
│           └── java/
│               └── kr/co/myshop/
│                   ├── product/
│                   │   └── application/
│                   │       ├── port/
│                   │       │   ├── in/
│                   │       │   │   ├── CreateProductUseCase.java
│                   │       │   │   └── GetProductUseCase.java
│                   │       │   └── out/
│                   │       │       ├── SaveProductPort.java
│                   │       │       └── LoadProductPort.java
│                   │       └── service/
│                   │           ├── CreateProductService.java
│                   │           └── GetProductService.java
│                   └── payment/
│                       └── application/
│                           ├── port/
│                           │   ├── in/
│                           │   │   ├── ProcessPaymentUseCase.java
│                           │   │   └── GetPaymentUseCase.java
│                           │   └── out/
│                           │       ├── SavePaymentPort.java
│                           │       ├── LoadPaymentPort.java
│                           │       └── PaymentGatewayPort.java
│                           └── service/
│                               ├── ProcessPaymentService.java
│                               └── GetPaymentService.java
│
├── commerce-infrastructure/                  # adapter/out (JPA, 외부 API)
│   ├── build.gradle
│   └── src/
│       └── main/
│           └── java/
│               └── kr/co/myshop/
│                   ├── product/
│                   │   └── adapter/
│                   │       └── out/
│                   │           └── persistence/
│                   │               ├── ProductPersistenceAdapter.java
│                   │               ├── ProductJpaEntity.java
│                   │               └── ProductJpaRepository.java
│                   └── payment/
│                       └── adapter/
│                           └── out/
│                               ├── persistence/
│                               │   ├── PaymentPersistenceAdapter.java
│                               │   ├── PaymentJpaEntity.java
│                               │   └── PaymentJpaRepository.java
│                               └── external/
│                                   └── ExternalPaymentGatewayAdapter.java
│
└── commerce-api/                             # adapter/in (Web) + Spring Boot 진입점
    ├── build.gradle
    └── src/
        └── main/
            └── java/
                └── kr/co/myshop/
                    ├── CommerceApplication.java
                    ├── config/
                    │   └── WebConfig.java
                    ├── common/
                    │   └── exception/
                    │       └── GlobalExceptionHandler.java
                    ├── product/
                    │   └── adapter/
                    │       └── in/
                    │           └── web/
                    │               ├── ProductController.java
                    │               ├── request/
                    │               │   └── CreateProductRequest.java
                    │               └── response/
                    │                   └── ProductResponse.java
                    └── payment/
                        └── adapter/
                            └── in/
                                └── web/
                                    ├── PaymentController.java
                                    ├── request/
                                    │   └── ProcessPaymentRequest.java
                                    └── response/
                                        └── PaymentResponse.java
```

---

## Gradle 설정 파일

### `settings.gradle`

```groovy
rootProject.name = 'commerce'
include 'commerce-domain'
include 'commerce-application'
include 'commerce-infrastructure'
include 'commerce-api'
```

---

### 루트 `build.gradle`

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.1' apply false
    id 'io.spring.dependency-management' version '1.1.7' apply false
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'

    group = 'kr.co.myshop'
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
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
}
```

---

### `commerce-domain/build.gradle`

```groovy
// 도메인 모듈: 순수 Java, 외부 프레임워크 의존 없음
dependencies {
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```

---

### `commerce-application/build.gradle`

```groovy
dependencies {
    implementation project(':commerce-domain')
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    // Spring의 @Service, @Transactional 사용을 위해 최소 의존성 추가
    implementation 'org.springframework:spring-context'
    implementation 'org.springframework:spring-tx'
}
```

---

### `commerce-infrastructure/build.gradle`

```groovy
plugins {
    id 'io.spring.dependency-management'
}

dependencies {
    implementation project(':commerce-domain')
    implementation project(':commerce-application')
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    // 실제 운영 DB 사용 시 아래 주석 해제
    // runtimeOnly 'org.postgresql:postgresql'
}
```

---

### `commerce-api/build.gradle` (Spring Boot 진입점)

```groovy
plugins {
    id 'org.springframework.boot'
}

dependencies {
    implementation project(':commerce-application')
    implementation project(':commerce-infrastructure')
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
}
```

---

## 예시 파일 (commerce-domain 모듈)

### `commerce-domain` — Product 도메인

**`kr/co/myshop/product/domain/model/Product.java`**

```java
package kr.co.myshop.product.domain.model;

/**
 * Product Aggregate Root
 * 프레임워크 의존성 없는 순수 도메인 모델
 */
public class Product {

    private final ProductId id;
    private String name;
    private Money price;
    private int stockQuantity;

    public Product(ProductId id, String name, Money price, int stockQuantity) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stockQuantity = stockQuantity;
    }

    public void decreaseStock(int quantity) {
        if (this.stockQuantity < quantity) {
            throw new IllegalStateException("재고가 부족합니다.");
        }
        this.stockQuantity -= quantity;
    }

    public ProductId getId() { return id; }
    public String getName() { return name; }
    public Money getPrice() { return price; }
    public int getStockQuantity() { return stockQuantity; }
}
```

---

**`kr/co/myshop/product/domain/model/ProductId.java`**

```java
package kr.co.myshop.product.domain.model;

/**
 * Product ID Value Object
 */
public record ProductId(Long value) {

    public ProductId {
        if (value == null || value <= 0) {
            throw new IllegalArgumentException("ProductId는 양수여야 합니다.");
        }
    }
}
```

---

**`kr/co/myshop/product/domain/model/Money.java`**

```java
package kr.co.myshop.product.domain.model;

import java.math.BigDecimal;

/**
 * Money Value Object
 */
public record Money(BigDecimal amount) {

    public Money {
        if (amount == null || amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("금액은 0 이상이어야 합니다.");
        }
    }

    public Money add(Money other) {
        return new Money(this.amount.add(other.amount));
    }

    public Money multiply(int quantity) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(quantity)));
    }
}
```

---

**`kr/co/myshop/product/domain/service/ProductDomainService.java`**

```java
package kr.co.myshop.product.domain.service;

import kr.co.myshop.product.domain.model.Product;

/**
 * Product Domain Service
 * 단일 Aggregate로 처리하기 어려운 도메인 규칙을 담당
 */
public class ProductDomainService {

    public boolean isAvailableForPurchase(Product product, int requestedQuantity) {
        return product.getStockQuantity() >= requestedQuantity;
    }
}
```

---

**`kr/co/myshop/product/domain/event/ProductCreatedEvent.java`**

```java
package kr.co.myshop.product.domain.event;

import kr.co.myshop.product.domain.model.ProductId;

/**
 * Product 생성 Domain Event
 */
public record ProductCreatedEvent(ProductId productId, String productName) {
}
```

---

### `commerce-domain` — Payment 도메인

**`kr/co/myshop/payment/domain/model/Payment.java`**

```java
package kr.co.myshop.payment.domain.model;

import kr.co.myshop.product.domain.model.Money;

/**
 * Payment Aggregate Root
 */
public class Payment {

    private final PaymentId id;
    private final Long orderId;
    private final Money amount;
    private PaymentStatus status;

    public Payment(PaymentId id, Long orderId, Money amount) {
        this.id = id;
        this.orderId = orderId;
        this.amount = amount;
        this.status = PaymentStatus.PENDING;
    }

    public void complete() {
        if (this.status != PaymentStatus.PENDING) {
            throw new IllegalStateException("PENDING 상태에서만 완료 처리할 수 있습니다.");
        }
        this.status = PaymentStatus.COMPLETED;
    }

    public void fail() {
        this.status = PaymentStatus.FAILED;
    }

    public PaymentId getId() { return id; }
    public Long getOrderId() { return orderId; }
    public Money getAmount() { return amount; }
    public PaymentStatus getStatus() { return status; }
}
```

---

**`kr/co/myshop/payment/domain/model/PaymentId.java`**

```java
package kr.co.myshop.payment.domain.model;

/**
 * Payment ID Value Object
 */
public record PaymentId(Long value) {

    public PaymentId {
        if (value == null || value <= 0) {
            throw new IllegalArgumentException("PaymentId는 양수여야 합니다.");
        }
    }
}
```

---

**`kr/co/myshop/payment/domain/model/PaymentStatus.java`**

```java
package kr.co.myshop.payment.domain.model;

/**
 * Payment 상태 Value Object (Enum)
 */
public enum PaymentStatus {
    PENDING,
    COMPLETED,
    FAILED,
    CANCELLED
}
```

---

**`kr/co/myshop/payment/domain/event/PaymentCompletedEvent.java`**

```java
package kr.co.myshop.payment.domain.event;

import kr.co.myshop.payment.domain.model.PaymentId;

/**
 * Payment 완료 Domain Event
 */
public record PaymentCompletedEvent(PaymentId paymentId, Long orderId) {
}
```

---

## 예시 파일 (commerce-application 모듈)

### `commerce-application` — Product 포트 & 서비스

**`kr/co/myshop/product/application/port/in/CreateProductUseCase.java`**

```java
package kr.co.myshop.product.application.port.in;

import java.math.BigDecimal;

/**
 * Inbound Port — 상품 생성 유스케이스 인터페이스
 */
public interface CreateProductUseCase {

    Long createProduct(CreateProductCommand command);

    record CreateProductCommand(
            String name,
            BigDecimal price,
            int stockQuantity
    ) {}
}
```

---

**`kr/co/myshop/product/application/port/in/GetProductUseCase.java`**

```java
package kr.co.myshop.product.application.port.in;

import kr.co.myshop.product.domain.model.Product;

/**
 * Inbound Port — 상품 조회 유스케이스 인터페이스
 */
public interface GetProductUseCase {

    Product getProduct(Long productId);
}
```

---

**`kr/co/myshop/product/application/port/out/SaveProductPort.java`**

```java
package kr.co.myshop.product.application.port.out;

import kr.co.myshop.product.domain.model.Product;

/**
 * Outbound Port — 상품 저장 (infrastructure 구현)
 */
public interface SaveProductPort {
    Product save(Product product);
}
```

---

**`kr/co/myshop/product/application/port/out/LoadProductPort.java`**

```java
package kr.co.myshop.product.application.port.out;

import kr.co.myshop.product.domain.model.Product;
import java.util.Optional;

/**
 * Outbound Port — 상품 조회 (infrastructure 구현)
 */
public interface LoadProductPort {
    Optional<Product> loadById(Long id);
}
```

---

**`kr/co/myshop/product/application/service/CreateProductService.java`**

```java
package kr.co.myshop.product.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import kr.co.myshop.product.application.port.in.CreateProductUseCase;
import kr.co.myshop.product.application.port.out.SaveProductPort;
import kr.co.myshop.product.domain.model.Money;
import kr.co.myshop.product.domain.model.Product;
import kr.co.myshop.product.domain.model.ProductId;

import java.math.BigDecimal;

/**
 * Application Service — CreateProductUseCase 구현체
 * Port In 을 구현하고, Port Out 을 사용한다.
 */
@Service
@RequiredArgsConstructor
@Transactional
class CreateProductService implements CreateProductUseCase {

    private final SaveProductPort saveProductPort;

    @Override
    public Long createProduct(CreateProductCommand command) {
        Product product = new Product(
                new ProductId(System.currentTimeMillis()), // 실제로는 ID 생성 전략 별도 적용
                command.name(),
                new Money(command.price()),
                command.stockQuantity()
        );
        Product saved = saveProductPort.save(product);
        return saved.getId().value();
    }
}
```

---

**`kr/co/myshop/product/application/service/GetProductService.java`**

```java
package kr.co.myshop.product.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import kr.co.myshop.product.application.port.in.GetProductUseCase;
import kr.co.myshop.product.application.port.out.LoadProductPort;
import kr.co.myshop.product.domain.model.Product;

/**
 * Application Service — GetProductUseCase 구현체
 */
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
class GetProductService implements GetProductUseCase {

    private final LoadProductPort loadProductPort;

    @Override
    public Product getProduct(Long productId) {
        return loadProductPort.loadById(productId)
                .orElseThrow(() -> new IllegalArgumentException("상품을 찾을 수 없습니다. id=" + productId));
    }
}
```

---

### `commerce-application` — Payment 포트 & 서비스

**`kr/co/myshop/payment/application/port/in/ProcessPaymentUseCase.java`**

```java
package kr.co.myshop.payment.application.port.in;

import java.math.BigDecimal;

/**
 * Inbound Port — 결제 처리 유스케이스 인터페이스
 */
public interface ProcessPaymentUseCase {

    Long processPayment(ProcessPaymentCommand command);

    record ProcessPaymentCommand(
            Long orderId,
            BigDecimal amount
    ) {}
}
```

---

**`kr/co/myshop/payment/application/port/out/SavePaymentPort.java`**

```java
package kr.co.myshop.payment.application.port.out;

import kr.co.myshop.payment.domain.model.Payment;

/**
 * Outbound Port — 결제 저장 (infrastructure 구현)
 */
public interface SavePaymentPort {
    Payment save(Payment payment);
}
```

---

**`kr/co/myshop/payment/application/port/out/PaymentGatewayPort.java`**

```java
package kr.co.myshop.payment.application.port.out;

import kr.co.myshop.product.domain.model.Money;

/**
 * Outbound Port — 외부 결제 게이트웨이 연동 (infrastructure 구현)
 */
public interface PaymentGatewayPort {
    boolean pay(Long orderId, Money amount);
}
```

---

**`kr/co/myshop/payment/application/service/ProcessPaymentService.java`**

```java
package kr.co.myshop.payment.application.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import kr.co.myshop.payment.application.port.in.ProcessPaymentUseCase;
import kr.co.myshop.payment.application.port.out.PaymentGatewayPort;
import kr.co.myshop.payment.application.port.out.SavePaymentPort;
import kr.co.myshop.payment.domain.model.Money;
import kr.co.myshop.payment.domain.model.Payment;
import kr.co.myshop.payment.domain.model.PaymentId;

/**
 * Application Service — ProcessPaymentUseCase 구현체
 */
@Service
@RequiredArgsConstructor
@Transactional
class ProcessPaymentService implements ProcessPaymentUseCase {

    private final SavePaymentPort savePaymentPort;
    private final PaymentGatewayPort paymentGatewayPort;

    @Override
    public Long processPayment(ProcessPaymentCommand command) {
        Money amount = new Money(command.amount());
        Payment payment = new Payment(
                new PaymentId(System.currentTimeMillis()),
                command.orderId(),
                amount
        );

        boolean success = paymentGatewayPort.pay(command.orderId(), amount);
        if (success) {
            payment.complete();
        } else {
            payment.fail();
        }

        Payment saved = savePaymentPort.save(payment);
        return saved.getId().value();
    }
}
```

---

## 예시 파일 (commerce-infrastructure 모듈)

### `commerce-infrastructure` — Product 영속성 어댑터

**`kr/co/myshop/product/adapter/out/persistence/ProductJpaEntity.java`**

```java
package kr.co.myshop.product.adapter.out.persistence;

import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;

/**
 * Product JPA Entity
 * infrastructure 계층에만 존재 — 도메인 모델과 분리
 */
@Entity
@Table(name = "products")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class ProductJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, precision = 19, scale = 4)
    private BigDecimal price;

    @Column(nullable = false)
    private int stockQuantity;

    ProductJpaEntity(Long id, String name, BigDecimal price, int stockQuantity) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stockQuantity = stockQuantity;
    }
}
```

---

**`kr/co/myshop/product/adapter/out/persistence/ProductJpaRepository.java`**

```java
package kr.co.myshop.product.adapter.out.persistence;

import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Spring Data JPA Repository
 * package-private — 외부에서 직접 접근 불가
 */
interface ProductJpaRepository extends JpaRepository<ProductJpaEntity, Long> {
}
```

---

**`kr/co/myshop/product/adapter/out/persistence/ProductPersistenceAdapter.java`**

```java
package kr.co.myshop.product.adapter.out.persistence;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import kr.co.myshop.product.application.port.out.LoadProductPort;
import kr.co.myshop.product.application.port.out.SaveProductPort;
import kr.co.myshop.product.domain.model.Money;
import kr.co.myshop.product.domain.model.Product;
import kr.co.myshop.product.domain.model.ProductId;

import java.util.Optional;

/**
 * Persistence Adapter — SaveProductPort, LoadProductPort 구현
 * 도메인 모델 <-> JPA Entity 매핑 담당
 */
@Component
@RequiredArgsConstructor
public class ProductPersistenceAdapter implements SaveProductPort, LoadProductPort {

    private final ProductJpaRepository productJpaRepository;

    @Override
    public Product save(Product product) {
        ProductJpaEntity entity = mapToJpaEntity(product);
        ProductJpaEntity saved = productJpaRepository.save(entity);
        return mapToDomainModel(saved);
    }

    @Override
    public Optional<Product> loadById(Long id) {
        return productJpaRepository.findById(id)
                .map(this::mapToDomainModel);
    }

    private ProductJpaEntity mapToJpaEntity(Product product) {
        return new ProductJpaEntity(
                product.getId() != null ? product.getId().value() : null,
                product.getName(),
                product.getPrice().amount(),
                product.getStockQuantity()
        );
    }

    private Product mapToDomainModel(ProductJpaEntity entity) {
        return new Product(
                new ProductId(entity.getId()),
                entity.getName(),
                new Money(entity.getPrice()),
                entity.getStockQuantity()
        );
    }
}
```

---

### `commerce-infrastructure` — Payment 외부 게이트웨이 어댑터

**`kr/co/myshop/payment/adapter/out/persistence/PaymentJpaEntity.java`**

```java
package kr.co.myshop.payment.adapter.out.persistence;

import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
import kr.co.myshop.payment.domain.model.PaymentStatus;

import java.math.BigDecimal;

/**
 * Payment JPA Entity
 */
@Entity
@Table(name = "payments")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
class PaymentJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private Long orderId;

    @Column(nullable = false, precision = 19, scale = 4)
    private BigDecimal amount;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private PaymentStatus status;

    PaymentJpaEntity(Long id, Long orderId, BigDecimal amount, PaymentStatus status) {
        this.id = id;
        this.orderId = orderId;
        this.amount = amount;
        this.status = status;
    }
}
```

---

**`kr/co/myshop/payment/adapter/out/persistence/PaymentPersistenceAdapter.java`**

```java
package kr.co.myshop.payment.adapter.out.persistence;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import kr.co.myshop.payment.application.port.out.SavePaymentPort;
import kr.co.myshop.payment.domain.model.Payment;
import kr.co.myshop.payment.domain.model.PaymentId;
import kr.co.myshop.product.domain.model.Money;

/**
 * Persistence Adapter — SavePaymentPort 구현
 */
@Component
@RequiredArgsConstructor
public class PaymentPersistenceAdapter implements SavePaymentPort {

    private final PaymentJpaRepository paymentJpaRepository;

    @Override
    public Payment save(Payment payment) {
        PaymentJpaEntity entity = mapToJpaEntity(payment);
        PaymentJpaEntity saved = paymentJpaRepository.save(entity);
        return mapToDomainModel(saved);
    }

    private PaymentJpaEntity mapToJpaEntity(Payment payment) {
        return new PaymentJpaEntity(
                payment.getId() != null ? payment.getId().value() : null,
                payment.getOrderId(),
                payment.getAmount().amount(),
                payment.getStatus()
        );
    }

    private Payment mapToDomainModel(PaymentJpaEntity entity) {
        Payment payment = new Payment(
                new PaymentId(entity.getId()),
                entity.getOrderId(),
                new Money(entity.getAmount())
        );
        // 상태 복원
        if (entity.getStatus() == kr.co.myshop.payment.domain.model.PaymentStatus.COMPLETED) {
            payment.complete();
        }
        return payment;
    }
}
```

---

**`kr/co/myshop/payment/adapter/out/external/ExternalPaymentGatewayAdapter.java`**

```java
package kr.co.myshop.payment.adapter.out.external;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import kr.co.myshop.payment.application.port.out.PaymentGatewayPort;
import kr.co.myshop.product.domain.model.Money;

/**
 * External Payment Gateway Adapter — PaymentGatewayPort 구현
 * 실제 외부 결제 서비스(PG사 등)와의 연동 담당
 */
@Slf4j
@Component
public class ExternalPaymentGatewayAdapter implements PaymentGatewayPort {

    @Override
    public boolean pay(Long orderId, Money amount) {
        log.info("외부 결제 게이트웨이 호출 — orderId={}, amount={}", orderId, amount.amount());
        // TODO: 실제 외부 결제 API 연동 구현
        return true;
    }
}
```

---

## 예시 파일 (commerce-api 모듈)

### Spring Boot 진입점

**`kr/co/myshop/CommerceApplication.java`**

```java
package kr.co.myshop;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class CommerceApplication {

    public static void main(String[] args) {
        SpringApplication.run(CommerceApplication.class, args);
    }
}
```

---

### Product Web 어댑터

**`kr/co/myshop/product/adapter/in/web/ProductController.java`**

```java
package kr.co.myshop.product.adapter.in.web;

import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import kr.co.myshop.product.application.port.in.CreateProductUseCase;
import kr.co.myshop.product.application.port.in.GetProductUseCase;
import kr.co.myshop.product.domain.model.Product;

import java.net.URI;

/**
 * Web Adapter — Product HTTP API
 * Inbound Port(UseCase)만 알고, 도메인/인프라는 모른다.
 */
@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {

    private final CreateProductUseCase createProductUseCase;
    private final GetProductUseCase getProductUseCase;

    @PostMapping
    public ResponseEntity<Void> createProduct(@RequestBody @Valid CreateProductRequest request) {
        Long productId = createProductUseCase.createProduct(request.toCommand());
        return ResponseEntity.created(URI.create("/api/products/" + productId)).build();
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProduct(@PathVariable Long id) {
        Product product = getProductUseCase.getProduct(id);
        return ResponseEntity.ok(ProductResponse.from(product));
    }
}
```

---

**`kr/co/myshop/product/adapter/in/web/request/CreateProductRequest.java`**

```java
package kr.co.myshop.product.adapter.in.web.request;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import kr.co.myshop.product.application.port.in.CreateProductUseCase;

import java.math.BigDecimal;

/**
 * 상품 생성 Request DTO
 */
public record CreateProductRequest(
        @NotBlank String name,
        @NotNull @Positive BigDecimal price,
        @Positive int stockQuantity
) {
    public CreateProductUseCase.CreateProductCommand toCommand() {
        return new CreateProductUseCase.CreateProductCommand(name, price, stockQuantity);
    }
}
```

---

**`kr/co/myshop/product/adapter/in/web/response/ProductResponse.java`**

```java
package kr.co.myshop.product.adapter.in.web.response;

import kr.co.myshop.product.domain.model.Product;

import java.math.BigDecimal;

/**
 * 상품 Response DTO
 */
public record ProductResponse(
        Long id,
        String name,
        BigDecimal price,
        int stockQuantity
) {
    public static ProductResponse from(Product product) {
        return new ProductResponse(
                product.getId().value(),
                product.getName(),
                product.getPrice().amount(),
                product.getStockQuantity()
        );
    }
}
```

---

### Payment Web 어댑터

**`kr/co/myshop/payment/adapter/in/web/PaymentController.java`**

```java
package kr.co.myshop.payment.adapter.in.web;

import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import kr.co.myshop.payment.application.port.in.ProcessPaymentUseCase;

import java.net.URI;

/**
 * Web Adapter — Payment HTTP API
 */
@RestController
@RequestMapping("/api/payments")
@RequiredArgsConstructor
public class PaymentController {

    private final ProcessPaymentUseCase processPaymentUseCase;

    @PostMapping
    public ResponseEntity<Void> processPayment(@RequestBody @Valid ProcessPaymentRequest request) {
        Long paymentId = processPaymentUseCase.processPayment(request.toCommand());
        return ResponseEntity.created(URI.create("/api/payments/" + paymentId)).build();
    }
}
```

---

**`kr/co/myshop/payment/adapter/in/web/request/ProcessPaymentRequest.java`**

```java
package kr.co.myshop.payment.adapter.in.web.request;

import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import kr.co.myshop.payment.application.port.in.ProcessPaymentUseCase;

import java.math.BigDecimal;

/**
 * 결제 처리 Request DTO
 */
public record ProcessPaymentRequest(
        @NotNull Long orderId,
        @NotNull @Positive BigDecimal amount
) {
    public ProcessPaymentUseCase.ProcessPaymentCommand toCommand() {
        return new ProcessPaymentUseCase.ProcessPaymentCommand(orderId, amount);
    }
}
```

---

### 공통 예외 처리

**`kr/co/myshop/common/exception/GlobalExceptionHandler.java`**

```java
package kr.co.myshop.common.exception;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.Map;

/**
 * 전역 예외 처리기
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleIllegalArgument(IllegalArgumentException e) {
        log.warn("잘못된 요청: {}", e.getMessage());
        return ResponseEntity.badRequest()
                .body(Map.of("message", e.getMessage()));
    }

    @ExceptionHandler(IllegalStateException.class)
    public ResponseEntity<Map<String, String>> handleIllegalState(IllegalStateException e) {
        log.warn("비즈니스 규칙 위반: {}", e.getMessage());
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(Map.of("message", e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidation(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(fe -> fe.getField() + ": " + fe.getDefaultMessage())
                .findFirst()
                .orElse("입력값 검증 오류");
        return ResponseEntity.badRequest()
                .body(Map.of("message", message));
    }
}
```

---

## 아키텍처 의존성 흐름 요약

```
[commerce-api]          (Web Adapter — adapter/in/web)
       |
       | depends on
       v
[commerce-application]  (Port In/Out 정의 + Application Service)
       |
       | depends on
       v
[commerce-domain]       (순수 도메인 모델, 의존성 없음)
       ^
       |
       | depends on
[commerce-infrastructure] (Persistence Adapter — adapter/out/persistence, external)
       |
       | depends on
[commerce-api]          (Spring Boot 진입점 — infrastructure를 조립)
```

### 의존성 규칙 한 줄 정리

```
API -> Application -> Domain
Infrastructure -> Domain + Application
API -> Infrastructure  (Spring Boot AutoConfiguration 조립)
```

- `commerce-domain`: 아무것도 의존하지 않는다. 순수 Java.
- `commerce-application`: `domain`만 의존한다. Port(인터페이스)를 정의하고 UseCase를 구현한다.
- `commerce-infrastructure`: `domain` + `application`을 의존한다. Port Out 인터페이스를 구현한다.
- `commerce-api`: `application` + `infrastructure`를 의존한다. Port In 인터페이스를 호출한다.

---

## 생성된 파일 목록 요약

### settings.gradle, build.gradle (루트 + 각 모듈)
- `settings.gradle`
- `build.gradle` (루트)
- `commerce-domain/build.gradle`
- `commerce-application/build.gradle`
- `commerce-infrastructure/build.gradle`
- `commerce-api/build.gradle`

### commerce-domain
- `product/domain/model/Product.java`
- `product/domain/model/ProductId.java`
- `product/domain/model/Money.java`
- `product/domain/service/ProductDomainService.java`
- `product/domain/event/ProductCreatedEvent.java`
- `payment/domain/model/Payment.java`
- `payment/domain/model/PaymentId.java`
- `payment/domain/model/PaymentStatus.java`
- `payment/domain/service/PaymentDomainService.java`
- `payment/domain/event/PaymentCompletedEvent.java`

### commerce-application
- `product/application/port/in/CreateProductUseCase.java`
- `product/application/port/in/GetProductUseCase.java`
- `product/application/port/out/SaveProductPort.java`
- `product/application/port/out/LoadProductPort.java`
- `product/application/service/CreateProductService.java`
- `product/application/service/GetProductService.java`
- `payment/application/port/in/ProcessPaymentUseCase.java`
- `payment/application/port/in/GetPaymentUseCase.java`
- `payment/application/port/out/SavePaymentPort.java`
- `payment/application/port/out/LoadPaymentPort.java`
- `payment/application/port/out/PaymentGatewayPort.java`
- `payment/application/service/ProcessPaymentService.java`
- `payment/application/service/GetPaymentService.java`

### commerce-infrastructure
- `product/adapter/out/persistence/ProductJpaEntity.java`
- `product/adapter/out/persistence/ProductJpaRepository.java`
- `product/adapter/out/persistence/ProductPersistenceAdapter.java`
- `payment/adapter/out/persistence/PaymentJpaEntity.java`
- `payment/adapter/out/persistence/PaymentJpaRepository.java`
- `payment/adapter/out/persistence/PaymentPersistenceAdapter.java`
- `payment/adapter/out/external/ExternalPaymentGatewayAdapter.java`

### commerce-api
- `CommerceApplication.java`
- `common/exception/GlobalExceptionHandler.java`
- `config/WebConfig.java`
- `product/adapter/in/web/ProductController.java`
- `product/adapter/in/web/request/CreateProductRequest.java`
- `product/adapter/in/web/response/ProductResponse.java`
- `payment/adapter/in/web/PaymentController.java`
- `payment/adapter/in/web/request/ProcessPaymentRequest.java`
- `payment/adapter/in/web/response/PaymentResponse.java`
