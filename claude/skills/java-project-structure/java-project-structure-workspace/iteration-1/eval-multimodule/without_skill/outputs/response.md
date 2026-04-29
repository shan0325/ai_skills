# Commerce 멀티모듈 + DDD + 헥사고날 아키텍처 프로젝트 구조

## 프로젝트 개요

- **프로젝트명**: commerce
- **Base Package**: kr.co.myshop
- **도메인**: product, payment
- **빌드 도구**: Gradle 멀티모듈
- **아키텍처**: DDD + Hexagonal Architecture (Ports & Adapters)

---

## 헥사고날 아키텍처 개념

```
                    ┌─────────────────────────────┐
  REST/gRPC ──────► │     Inbound Adapter         │
  Kafka Consumer ──►│  (Driving / Primary Side)   │
                    └──────────┬──────────────────┘
                               │ calls
                    ┌──────────▼──────────────────┐
                    │      Application Layer       │
                    │   (Use Case / Port 정의)      │
                    └──────────┬──────────────────┘
                               │ calls
                    ┌──────────▼──────────────────┐
                    │        Domain Layer          │
                    │   (Entity, Value Object,     │
                    │    Domain Service)           │
                    └──────────┬──────────────────┘
                               │ implemented by
                    ┌──────────▼──────────────────┐
  DB / External ◄──│     Outbound Adapter         │
  API / Kafka ◄────│  (Driven / Secondary Side)   │
                    └─────────────────────────────┘
```

---

## 전체 디렉토리 구조

```
commerce/
├── settings.gradle
├── build.gradle                          # 루트 빌드 파일
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
│
├── commerce-common/                      # 공통 모듈 (유틸, 예외, 공통 Value Object 등)
│   ├── build.gradle
│   └── src/main/java/kr/co/myshop/common/
│       ├── exception/
│       │   ├── BusinessException.java
│       │   └── ErrorCode.java
│       └── vo/
│           └── Money.java
│
├── commerce-product/                     # Product 도메인 모듈
│   ├── build.gradle
│   └── src/
│       ├── main/java/kr/co/myshop/product/
│       │   ├── domain/                   # 도메인 레이어 (순수 Java, 의존성 없음)
│       │   │   ├── model/
│       │   │   │   ├── Product.java
│       │   │   │   ├── ProductId.java
│       │   │   │   └── ProductStatus.java
│       │   │   └── service/
│       │   │       └── ProductDomainService.java
│       │   │
│       │   ├── application/              # 애플리케이션 레이어 (Use Case + Ports)
│       │   │   ├── port/
│       │   │   │   ├── in/               # Inbound Port (Use Case 인터페이스)
│       │   │   │   │   ├── RegisterProductUseCase.java
│       │   │   │   │   ├── GetProductUseCase.java
│       │   │   │   │   └── UpdateProductStockUseCase.java
│       │   │   │   └── out/              # Outbound Port (Repository/외부 인터페이스)
│       │   │   │       ├── SaveProductPort.java
│       │   │   │       ├── LoadProductPort.java
│       │   │   │       └── PublishProductEventPort.java
│       │   │   ├── service/              # Use Case 구현체
│       │   │   │   ├── RegisterProductService.java
│       │   │   │   ├── GetProductService.java
│       │   │   │   └── UpdateProductStockService.java
│       │   │   └── dto/
│       │   │       ├── RegisterProductCommand.java
│       │   │       └── ProductResult.java
│       │   │
│       │   └── adapter/                  # 어댑터 레이어
│       │       ├── in/
│       │       │   ├── web/              # REST Controller (Inbound Adapter)
│       │       │   │   ├── ProductController.java
│       │       │   │   ├── request/
│       │       │   │   │   └── RegisterProductRequest.java
│       │       │   │   └── response/
│       │       │   │       └── ProductResponse.java
│       │       │   └── messaging/        # Kafka Consumer 등 (Inbound Adapter)
│       │       │       └── ProductEventConsumer.java
│       │       └── out/
│       │           ├── persistence/      # JPA Repository (Outbound Adapter)
│       │           │   ├── ProductPersistenceAdapter.java
│       │           │   ├── ProductJpaRepository.java
│       │           │   └── entity/
│       │           │       └── ProductJpaEntity.java
│       │           └── messaging/        # Kafka Producer (Outbound Adapter)
│       │               └── ProductEventPublisherAdapter.java
│       └── test/java/kr/co/myshop/product/
│           ├── domain/
│           │   └── ProductTest.java
│           └── application/
│               └── RegisterProductServiceTest.java
│
├── commerce-payment/                     # Payment 도메인 모듈
│   ├── build.gradle
│   └── src/
│       ├── main/java/kr/co/myshop/payment/
│       │   ├── domain/
│       │   │   ├── model/
│       │   │   │   ├── Payment.java
│       │   │   │   ├── PaymentId.java
│       │   │   │   └── PaymentStatus.java
│       │   │   └── service/
│       │   │       └── PaymentDomainService.java
│       │   ├── application/
│       │   │   ├── port/
│       │   │   │   ├── in/
│       │   │   │   │   ├── ProcessPaymentUseCase.java
│       │   │   │   │   └── GetPaymentUseCase.java
│       │   │   │   └── out/
│       │   │   │       ├── SavePaymentPort.java
│       │   │   │       ├── LoadPaymentPort.java
│       │   │   │       └── PaymentGatewayPort.java
│       │   │   ├── service/
│       │   │   │   ├── ProcessPaymentService.java
│       │   │   │   └── GetPaymentService.java
│       │   │   └── dto/
│       │   │       ├── ProcessPaymentCommand.java
│       │   │       └── PaymentResult.java
│       │   └── adapter/
│       │       ├── in/
│       │       │   └── web/
│       │       │       ├── PaymentController.java
│       │       │       ├── request/
│       │       │       │   └── ProcessPaymentRequest.java
│       │       │       └── response/
│       │       │           └── PaymentResponse.java
│       │       └── out/
│       │           ├── persistence/
│       │           │   ├── PaymentPersistenceAdapter.java
│       │           │   ├── PaymentJpaRepository.java
│       │           │   └── entity/
│       │           │       └── PaymentJpaEntity.java
│       │           └── external/
│       │               └── PaymentGatewayAdapter.java
│       └── test/java/kr/co/myshop/payment/
│           └── application/
│               └── ProcessPaymentServiceTest.java
│
└── commerce-bootstrap/                   # 스프링부트 실행 모듈 (Main 클래스, 설정)
    ├── build.gradle
    └── src/main/
        ├── java/kr/co/myshop/
        │   └── CommerceApplication.java
        └── resources/
            └── application.yml
```

---

## Gradle 설정

### settings.gradle

```groovy
rootProject.name = 'commerce'

include(
    'commerce-common',
    'commerce-product',
    'commerce-payment',
    'commerce-bootstrap'
)
```

### build.gradle (루트)

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.0' apply false
    id 'io.spring.dependency-management' version '1.1.5' apply false
}

allprojects {
    group = 'kr.co.myshop'
    version = '0.0.1-SNAPSHOT'

    repositories {
        mavenCentral()
    }
}

subprojects {
    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'

    java {
        sourceCompatibility = JavaVersion.VERSION_21
        targetCompatibility = JavaVersion.VERSION_21
    }

    dependencyManagement {
        imports {
            mavenBom "org.springframework.boot:spring-boot-dependencies:3.3.0"
        }
    }

    dependencies {
        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'

        testImplementation 'org.springframework.boot:spring-boot-starter-test'
        testImplementation 'org.junit.jupiter:junit-jupiter'
    }

    test {
        useJUnitPlatform()
    }
}
```

### commerce-common/build.gradle

```groovy
dependencies {
    // 공통 모듈은 최소 의존성
}
```

### commerce-product/build.gradle

```groovy
dependencies {
    implementation project(':commerce-common')

    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.kafka:spring-kafka'
}
```

### commerce-payment/build.gradle

```groovy
dependencies {
    implementation project(':commerce-common')

    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

### commerce-bootstrap/build.gradle

```groovy
plugins {
    id 'org.springframework.boot'
}

dependencies {
    implementation project(':commerce-common')
    implementation project(':commerce-product')
    implementation project(':commerce-payment')

    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.mysql:mysql-connector-j'
}
```

---

## 예시 파일 내용 (Product 도메인)

### 1. 도메인 모델 - Product.java

```java
package kr.co.myshop.product.domain.model;

import kr.co.myshop.common.exception.BusinessException;
import kr.co.myshop.common.exception.ErrorCode;
import kr.co.myshop.common.vo.Money;
import lombok.Getter;

@Getter
public class Product {

    private final ProductId id;
    private String name;
    private Money price;
    private int stock;
    private ProductStatus status;

    // 정적 팩토리 메서드 - 신규 생성
    public static Product create(String name, Money price, int stock) {
        if (stock < 0) {
            throw new BusinessException(ErrorCode.INVALID_STOCK);
        }
        Product product = new Product();
        product.name = name;
        product.price = price;
        product.stock = stock;
        product.status = ProductStatus.ACTIVE;
        return product;
    }

    // 재고 차감 (도메인 핵심 비즈니스 로직)
    public void decreaseStock(int quantity) {
        if (this.stock < quantity) {
            throw new BusinessException(ErrorCode.INSUFFICIENT_STOCK);
        }
        this.stock -= quantity;
    }

    // 재고 증가
    public void increaseStock(int quantity) {
        if (quantity <= 0) {
            throw new BusinessException(ErrorCode.INVALID_QUANTITY);
        }
        this.stock += quantity;
    }

    public void deactivate() {
        this.status = ProductStatus.INACTIVE;
    }

    private Product() {}

    // 재구성용 (영속성에서 복원할 때)
    public static Product reconstruct(ProductId id, String name, Money price,
                                      int stock, ProductStatus status) {
        Product product = new Product();
        product.id = id;
        product.name = name;
        product.price = price;
        product.stock = stock;
        product.status = status;
        return product;
    }

    private ProductId id;
}
```

### 2. Value Object - ProductId.java

```java
package kr.co.myshop.product.domain.model;

import java.util.Objects;

public record ProductId(Long value) {

    public ProductId {
        Objects.requireNonNull(value, "ProductId must not be null");
    }

    public static ProductId of(Long value) {
        return new ProductId(value);
    }
}
```

### 3. 공통 Value Object - Money.java

```java
package kr.co.myshop.common.vo;

import java.math.BigDecimal;
import java.util.Objects;

public record Money(BigDecimal amount) {

    public Money {
        Objects.requireNonNull(amount, "amount must not be null");
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Money cannot be negative");
        }
    }

    public static Money of(long amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public static Money of(BigDecimal amount) {
        return new Money(amount);
    }

    public Money add(Money other) {
        return new Money(this.amount.add(other.amount));
    }

    public Money multiply(int multiplier) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(multiplier)));
    }

    public boolean isGreaterThan(Money other) {
        return this.amount.compareTo(other.amount) > 0;
    }
}
```

### 4. Inbound Port - RegisterProductUseCase.java

```java
package kr.co.myshop.product.application.port.in;

import kr.co.myshop.product.application.dto.RegisterProductCommand;
import kr.co.myshop.product.application.dto.ProductResult;

public interface RegisterProductUseCase {
    ProductResult register(RegisterProductCommand command);
}
```

### 5. Command DTO - RegisterProductCommand.java

```java
package kr.co.myshop.product.application.dto;

import kr.co.myshop.common.vo.Money;
import lombok.Builder;

@Builder
public record RegisterProductCommand(
        String name,
        Money price,
        int stock
) {
    public static RegisterProductCommand of(String name, Money price, int stock) {
        return new RegisterProductCommand(name, price, stock);
    }
}
```

### 6. Outbound Port - SaveProductPort.java

```java
package kr.co.myshop.product.application.port.out;

import kr.co.myshop.product.domain.model.Product;

public interface SaveProductPort {
    Product save(Product product);
}
```

### 7. Outbound Port - LoadProductPort.java

```java
package kr.co.myshop.product.application.port.out;

import kr.co.myshop.product.domain.model.Product;
import kr.co.myshop.product.domain.model.ProductId;

import java.util.Optional;

public interface LoadProductPort {
    Optional<Product> findById(ProductId productId);
}
```

### 8. Application Service (Use Case 구현) - RegisterProductService.java

```java
package kr.co.myshop.product.application.service;

import kr.co.myshop.product.application.dto.ProductResult;
import kr.co.myshop.product.application.dto.RegisterProductCommand;
import kr.co.myshop.product.application.port.in.RegisterProductUseCase;
import kr.co.myshop.product.application.port.out.SaveProductPort;
import kr.co.myshop.product.domain.model.Product;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional
public class RegisterProductService implements RegisterProductUseCase {

    private final SaveProductPort saveProductPort;

    @Override
    public ProductResult register(RegisterProductCommand command) {
        // 1. 도메인 객체 생성 (도메인 로직은 도메인 레이어에서)
        Product product = Product.create(
                command.name(),
                command.price(),
                command.stock()
        );

        // 2. 영속성 저장 (Outbound Port 호출)
        Product savedProduct = saveProductPort.save(product);

        // 3. 결과 반환
        return ProductResult.from(savedProduct);
    }
}
```

### 9. Result DTO - ProductResult.java

```java
package kr.co.myshop.product.application.dto;

import kr.co.myshop.product.domain.model.Product;
import kr.co.myshop.product.domain.model.ProductStatus;
import kr.co.myshop.common.vo.Money;
import lombok.Builder;

@Builder
public record ProductResult(
        Long id,
        String name,
        Money price,
        int stock,
        ProductStatus status
) {
    public static ProductResult from(Product product) {
        return ProductResult.builder()
                .id(product.getId().value())
                .name(product.getName())
                .price(product.getPrice())
                .stock(product.getStock())
                .status(product.getStatus())
                .build();
    }
}
```

### 10. REST Controller (Inbound Adapter) - ProductController.java

```java
package kr.co.myshop.product.adapter.in.web;

import kr.co.myshop.common.vo.Money;
import kr.co.myshop.product.adapter.in.web.request.RegisterProductRequest;
import kr.co.myshop.product.adapter.in.web.response.ProductResponse;
import kr.co.myshop.product.application.dto.ProductResult;
import kr.co.myshop.product.application.dto.RegisterProductCommand;
import kr.co.myshop.product.application.port.in.GetProductUseCase;
import kr.co.myshop.product.application.port.in.RegisterProductUseCase;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/products")
@RequiredArgsConstructor
public class ProductController {

    private final RegisterProductUseCase registerProductUseCase;
    private final GetProductUseCase getProductUseCase;

    @PostMapping
    public ResponseEntity<ProductResponse> register(@RequestBody RegisterProductRequest request) {
        RegisterProductCommand command = RegisterProductCommand.of(
                request.name(),
                Money.of(request.price()),
                request.stock()
        );
        ProductResult result = registerProductUseCase.register(command);
        return ResponseEntity.ok(ProductResponse.from(result));
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProduct(@PathVariable Long id) {
        ProductResult result = getProductUseCase.getById(id);
        return ResponseEntity.ok(ProductResponse.from(result));
    }
}
```

### 11. REST Request - RegisterProductRequest.java

```java
package kr.co.myshop.product.adapter.in.web.request;

import java.math.BigDecimal;

public record RegisterProductRequest(
        String name,
        BigDecimal price,
        int stock
) {}
```

### 12. REST Response - ProductResponse.java

```java
package kr.co.myshop.product.adapter.in.web.response;

import kr.co.myshop.product.application.dto.ProductResult;
import kr.co.myshop.product.domain.model.ProductStatus;
import lombok.Builder;

import java.math.BigDecimal;

@Builder
public record ProductResponse(
        Long id,
        String name,
        BigDecimal price,
        int stock,
        ProductStatus status
) {
    public static ProductResponse from(ProductResult result) {
        return ProductResponse.builder()
                .id(result.id())
                .name(result.name())
                .price(result.price().amount())
                .stock(result.stock())
                .status(result.status())
                .build();
    }
}
```

### 13. JPA Entity - ProductJpaEntity.java

```java
package kr.co.myshop.product.adapter.out.persistence.entity;

import jakarta.persistence.*;
import lombok.*;

import java.math.BigDecimal;

@Entity
@Table(name = "products")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
public class ProductJpaEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, precision = 19, scale = 2)
    private BigDecimal price;

    @Column(nullable = false)
    private int stock;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private String status;
}
```

### 14. Persistence Adapter (Outbound Adapter) - ProductPersistenceAdapter.java

```java
package kr.co.myshop.product.adapter.out.persistence;

import kr.co.myshop.common.vo.Money;
import kr.co.myshop.product.adapter.out.persistence.entity.ProductJpaEntity;
import kr.co.myshop.product.application.port.out.LoadProductPort;
import kr.co.myshop.product.application.port.out.SaveProductPort;
import kr.co.myshop.product.domain.model.Product;
import kr.co.myshop.product.domain.model.ProductId;
import kr.co.myshop.product.domain.model.ProductStatus;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
@RequiredArgsConstructor
public class ProductPersistenceAdapter implements SaveProductPort, LoadProductPort {

    private final ProductJpaRepository productJpaRepository;

    @Override
    public Product save(Product product) {
        ProductJpaEntity entity = toEntity(product);
        ProductJpaEntity saved = productJpaRepository.save(entity);
        return toDomain(saved);
    }

    @Override
    public Optional<Product> findById(ProductId productId) {
        return productJpaRepository.findById(productId.value())
                .map(this::toDomain);
    }

    private ProductJpaEntity toEntity(Product product) {
        return ProductJpaEntity.builder()
                .id(product.getId() != null ? product.getId().value() : null)
                .name(product.getName())
                .price(product.getPrice().amount())
                .stock(product.getStock())
                .status(product.getStatus().name())
                .build();
    }

    private Product toDomain(ProductJpaEntity entity) {
        return Product.reconstruct(
                ProductId.of(entity.getId()),
                entity.getName(),
                Money.of(entity.getPrice()),
                entity.getStock(),
                ProductStatus.valueOf(entity.getStatus())
        );
    }
}
```

### 15. JPA Repository - ProductJpaRepository.java

```java
package kr.co.myshop.product.adapter.out.persistence;

import kr.co.myshop.product.adapter.out.persistence.entity.ProductJpaEntity;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProductJpaRepository extends JpaRepository<ProductJpaEntity, Long> {
}
```

---

## 예시 파일 내용 (Payment 도메인)

### 1. 도메인 모델 - Payment.java

```java
package kr.co.myshop.payment.domain.model;

import kr.co.myshop.common.exception.BusinessException;
import kr.co.myshop.common.exception.ErrorCode;
import kr.co.myshop.common.vo.Money;
import lombok.Getter;

@Getter
public class Payment {

    private PaymentId id;
    private Long orderId;
    private Money amount;
    private PaymentStatus status;
    private String pgTransactionId;

    public static Payment create(Long orderId, Money amount) {
        Payment payment = new Payment();
        payment.orderId = orderId;
        payment.amount = amount;
        payment.status = PaymentStatus.PENDING;
        return payment;
    }

    public void complete(String pgTransactionId) {
        if (this.status != PaymentStatus.PENDING) {
            throw new BusinessException(ErrorCode.INVALID_PAYMENT_STATUS);
        }
        this.status = PaymentStatus.COMPLETED;
        this.pgTransactionId = pgTransactionId;
    }

    public void fail() {
        if (this.status != PaymentStatus.PENDING) {
            throw new BusinessException(ErrorCode.INVALID_PAYMENT_STATUS);
        }
        this.status = PaymentStatus.FAILED;
    }

    public void cancel() {
        if (this.status != PaymentStatus.COMPLETED) {
            throw new BusinessException(ErrorCode.CANNOT_CANCEL_PAYMENT);
        }
        this.status = PaymentStatus.CANCELLED;
    }

    private Payment() {}

    public static Payment reconstruct(PaymentId id, Long orderId, Money amount,
                                      PaymentStatus status, String pgTransactionId) {
        Payment payment = new Payment();
        payment.id = id;
        payment.orderId = orderId;
        payment.amount = amount;
        payment.status = status;
        payment.pgTransactionId = pgTransactionId;
        return payment;
    }
}
```

### 2. Outbound Port - PaymentGatewayPort.java

```java
package kr.co.myshop.payment.application.port.out;

import kr.co.myshop.common.vo.Money;

public interface PaymentGatewayPort {
    PaymentGatewayResult charge(String orderId, Money amount);

    record PaymentGatewayResult(
            boolean success,
            String transactionId,
            String errorMessage
    ) {}
}
```

### 3. Application Service - ProcessPaymentService.java

```java
package kr.co.myshop.payment.application.service;

import kr.co.myshop.payment.application.dto.PaymentResult;
import kr.co.myshop.payment.application.dto.ProcessPaymentCommand;
import kr.co.myshop.payment.application.port.in.ProcessPaymentUseCase;
import kr.co.myshop.payment.application.port.out.PaymentGatewayPort;
import kr.co.myshop.payment.application.port.out.SavePaymentPort;
import kr.co.myshop.payment.domain.model.Payment;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional
public class ProcessPaymentService implements ProcessPaymentUseCase {

    private final SavePaymentPort savePaymentPort;
    private final PaymentGatewayPort paymentGatewayPort;

    @Override
    public PaymentResult process(ProcessPaymentCommand command) {
        // 1. 결제 도메인 객체 생성
        Payment payment = Payment.create(command.orderId(), command.amount());

        // 2. PG사 결제 요청 (Outbound Port 호출)
        PaymentGatewayPort.PaymentGatewayResult gatewayResult =
                paymentGatewayPort.charge(command.orderId().toString(), command.amount());

        // 3. 결과에 따라 도메인 상태 변경
        if (gatewayResult.success()) {
            payment.complete(gatewayResult.transactionId());
        } else {
            payment.fail();
        }

        // 4. 저장
        Payment savedPayment = savePaymentPort.save(payment);

        return PaymentResult.from(savedPayment);
    }
}
```

---

## 공통 예외 처리

### ErrorCode.java

```java
package kr.co.myshop.common.exception;

import lombok.Getter;
import org.springframework.http.HttpStatus;

@Getter
public enum ErrorCode {

    // Product
    PRODUCT_NOT_FOUND(HttpStatus.NOT_FOUND, "P001", "상품을 찾을 수 없습니다."),
    INSUFFICIENT_STOCK(HttpStatus.BAD_REQUEST, "P002", "재고가 부족합니다."),
    INVALID_STOCK(HttpStatus.BAD_REQUEST, "P003", "재고는 0 이상이어야 합니다."),
    INVALID_QUANTITY(HttpStatus.BAD_REQUEST, "P004", "수량은 1 이상이어야 합니다."),

    // Payment
    PAYMENT_NOT_FOUND(HttpStatus.NOT_FOUND, "PAY001", "결제 정보를 찾을 수 없습니다."),
    INVALID_PAYMENT_STATUS(HttpStatus.BAD_REQUEST, "PAY002", "유효하지 않은 결제 상태입니다."),
    CANNOT_CANCEL_PAYMENT(HttpStatus.BAD_REQUEST, "PAY003", "완료된 결제만 취소할 수 있습니다.");

    private final HttpStatus httpStatus;
    private final String code;
    private final String message;

    ErrorCode(HttpStatus httpStatus, String code, String message) {
        this.httpStatus = httpStatus;
        this.code = code;
        this.message = message;
    }
}
```

### BusinessException.java

```java
package kr.co.myshop.common.exception;

import lombok.Getter;

@Getter
public class BusinessException extends RuntimeException {

    private final ErrorCode errorCode;

    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
}
```

---

## Bootstrap 모듈

### CommerceApplication.java

```java
package kr.co.myshop;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication(scanBasePackages = "kr.co.myshop")
public class CommerceApplication {
    public static void main(String[] args) {
        SpringApplication.run(CommerceApplication.class, args);
    }
}
```

### application.yml

```yaml
spring:
  application:
    name: commerce

  datasource:
    url: jdbc:mysql://localhost:3306/commerce?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:password}
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect

server:
  port: 8080
```

---

## 테스트 예시

### RegisterProductServiceTest.java

```java
package kr.co.myshop.product.application;

import kr.co.myshop.common.vo.Money;
import kr.co.myshop.product.application.dto.ProductResult;
import kr.co.myshop.product.application.dto.RegisterProductCommand;
import kr.co.myshop.product.application.port.out.SaveProductPort;
import kr.co.myshop.product.application.service.RegisterProductService;
import kr.co.myshop.product.domain.model.Product;
import kr.co.myshop.product.domain.model.ProductId;
import kr.co.myshop.product.domain.model.ProductStatus;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;

class RegisterProductServiceTest {

    private SaveProductPort saveProductPort;
    private RegisterProductService registerProductService;

    @BeforeEach
    void setUp() {
        saveProductPort = Mockito.mock(SaveProductPort.class);
        registerProductService = new RegisterProductService(saveProductPort);
    }

    @Test
    @DisplayName("상품을 정상적으로 등록한다")
    void registerProduct_success() {
        // given
        RegisterProductCommand command = RegisterProductCommand.of(
                "테스트 상품", Money.of(10000L), 100
        );

        Product savedProduct = Product.reconstruct(
                ProductId.of(1L), "테스트 상품",
                Money.of(10000L), 100, ProductStatus.ACTIVE
        );
        given(saveProductPort.save(any(Product.class))).willReturn(savedProduct);

        // when
        ProductResult result = registerProductService.register(command);

        // then
        assertThat(result.id()).isEqualTo(1L);
        assertThat(result.name()).isEqualTo("테스트 상품");
        assertThat(result.stock()).isEqualTo(100);
        assertThat(result.status()).isEqualTo(ProductStatus.ACTIVE);
    }
}
```

---

## 아키텍처 원칙 요약

| 레이어 | 위치 | 역할 | 의존 방향 |
|--------|------|------|-----------|
| Domain | `domain/model`, `domain/service` | 핵심 비즈니스 로직 | 외부 의존 없음 |
| Application | `application/port`, `application/service` | Use Case 정의 및 조율 | Domain에만 의존 |
| Adapter (In) | `adapter/in/web`, `adapter/in/messaging` | 외부 요청 수신 | Application Port에 의존 |
| Adapter (Out) | `adapter/out/persistence`, `adapter/out/external` | 외부 시스템 호출 | Application Port 구현 |

### 핵심 규칙
- **Domain 레이어**는 Spring, JPA 등 어떤 프레임워크도 import하지 않는다
- **Application Service**는 Port 인터페이스를 통해서만 외부와 통신한다
- **Adapter**는 Port를 구현하거나 Port를 호출한다 (양방향)
- **모듈 간 도메인 의존**은 금지한다 (product ↔ payment 직접 참조 X, 이벤트로 통신)
- **Bootstrap 모듈**만 모든 모듈에 의존하며, 실행 진입점 역할만 한다
