# DDD 학습 가이드

> 이 문서는 향후 DDD(Domain-Driven Design)를 학습할 때 참고하기 위해 작성되었습니다.

## 1. 이 문서의 목적

### 현재 상황

- 10주 Kotlin + Spring Boot 멘토링 과정 진행 중
- TDD와 설계 역량 강화가 현재 교육의 핵심 목표
- DDD에 대한 개념이 아직 충분하지 않은 상태

### DDD를 당장 도입하지 않는 이유

팀원이 DDD 방식으로 구현한 코드를 보고 "저렇게 해야 하나?"라는 생각이 들었지만, 다음 이유로 당장 도입하지 않기로 결정했습니다:

1. **개념 이해 부족**: "왜 이렇게 하는지" 모르고 구조만 따라하면 문제 발생 시 해결하기 어려움
2. **학습 우선순위**: TDD, 테스트 작성법, 레이어드 아키텍처 등 기반 역량이 DDD보다 선행되어야 함
3. **비용-효과**: DDD 완전 분리 구조는 매핑 복잡성이 높아 학습 효율이 떨어질 수 있음

### 이 문서의 활용 방법

- 교육과정 완료 후 DDD 학습 시작 시 참고
- 학습 로드맵 및 추천 자료 확인
- 실무 적용 시 고려사항 검토

---

## 2. DDD 핵심 개념 요약

### Domain Model vs Persistence Model

**Domain Model (도메인 모델)**

- 비즈니스 로직을 담은 순수한 객체
- 기술적 관심사(DB, 프레임워크)에서 자유로움
- "주문은 취소될 수 있다", "재고가 부족하면 주문 불가" 등 비즈니스 규칙 표현

**Persistence Model (영속성 모델)**

- 데이터베이스와의 매핑을 담당
- JPA `@Entity`, `@Column` 등 ORM 어노테이션 포함
- 테이블 구조에 맞춘 설계

**분리 여부에 대한 논쟁**

| 관점    | 주장                      | 출처                        |
|-------|-------------------------|---------------------------|
| 분리 찬성 | 도메인 순수성 유지, ORM 교체 용이   | Eric Evans, Vaughn Vernon |
| 분리 반대 | 매핑 비용 과다, ORM 기능 재구현 필요 | Vladimir Khorikov         |

> "별도의 persistence model은 기본적으로 권장하지 않습니다. ORM을 사용하는 것이 처음부터 재구현하는 것보다 낫습니다." - Vladimir Khorikov

### Persistence Ignorance 원칙

도메인 모델이 영속성 메커니즘을 알지 못해야 한다는 원칙입니다.

```kotlin
// Persistence Ignorance 준수 (순수 도메인)
class Order(
    val id: OrderId,
    private val items: MutableList<OrderItem>
) {
    fun cancel() {
        ...
    }  // 비즈니스 로직만
}

// Persistence Ignorance 위반 (JPA 침투)
@Entity
class Order(
    @Id @GeneratedValue
    val id: Long,
    @OneToMany(cascade = [CascadeType.ALL])
    val items: List<OrderItem>
) {
    fun cancel() {
        ...
    }
}
```

### Bounded Context (경계 컨텍스트)

같은 개념(예: "사용자")이 다른 맥락에서 다른 의미를 가질 수 있음을 인정합니다.

```
[주문 컨텍스트]           [배송 컨텍스트]
   Customer                Customer
   - 주문 내역             - 배송 주소
   - 결제 정보             - 수령 희망 시간
```

멀티모듈 프로젝트에서 "하나의 원천 데이터를 각 서비스에서 어떻게 다룰까?"라는 고민의 해답 중 하나입니다.

### Aggregate, Entity, Value Object

| 개념                 | 설명                                  | 예시                          |
|--------------------|-------------------------------------|-----------------------------|
| **Aggregate**      | 일관성 경계를 공유하는 객체 군집                  | 주문(Order) + 주문항목(OrderItem) |
| **Aggregate Root** | Aggregate의 진입점, 외부에서 접근 가능한 유일한 엔티티 | Order                       |
| **Entity**         | 식별자로 구분되는 객체, 상태 변경 가능              | User, Product               |
| **Value Object**   | 값으로 동등성 비교, 불변                      | Money, Address, Email       |

```kotlin
// Aggregate Root
class Order(val id: OrderId) {
    private val items = mutableListOf<OrderItem>()  // Aggregate 내부

    fun addItem(product: ProductId, quantity: Int) {
        items.add(OrderItem(product, quantity))
    }
}

// Value Object
data class Money(val amount: BigDecimal, val currency: Currency) {
    operator fun plus(other: Money): Money {
        require(currency == other.currency)
        return Money(amount + other.amount, currency)
    }
}
```

### Repository Pattern

도메인 관점에서 컬렉션처럼 Aggregate에 접근하는 인터페이스입니다.

```kotlin
// 도메인 레이어 - 인터페이스만 정의
interface OrderRepository {
    fun findById(id: OrderId): Order?
    fun save(order: Order)
    fun delete(order: Order)
}

// 인프라 레이어 - 구현 제공
class OrderRepositoryImpl(
    private val jpaRepository: OrderJpaRepository,
    private val mapper: OrderMapper
) : OrderRepository {
    override fun findById(id: OrderId): Order? {
        return jpaRepository.findByIdOrNull(id.value)?.let { mapper.toDomain(it) }
    }
}
```

---

## 3. 현재 프로젝트 구조와 DDD

### 현재 구조: Rich Domain Model

```
apps/commerce-api/
├── domain/
│   └── example/
│       ├── ExampleModel.kt      # @Entity + 비즈니스 로직
│       ├── ExampleRepository.kt # 인터페이스
│       └── ExampleService.kt
├── infrastructure/
│   └── example/
│       ├── ExampleJpaRepository.kt
│       └── ExampleRepositoryImpl.kt
└── ...

modules/jpa/
└── domain/
    └── BaseEntity.kt  # @MappedSuperclass
```

**특징:**

- `ExampleModel`이 `@Entity`이면서 비즈니스 로직(`update()`, `guard()`)도 포함
- JPA 어노테이션이 도메인에 침투해 있음
- 하지만 Repository 인터페이스/구현 분리로 Hexagonal Architecture 적용

**장점:**

- 단순함, JPA의 모든 기능(lazy loading, dirty checking) 활용 가능
- 매핑 코드 불필요
- 학습 곡선이 낮음

**단점:**

- Persistence Ignorance 원칙 위반
- ORM 교체 시 도메인 레이어 수정 필요

### DDD 완전 분리 구조

```
modules/jpa/
├── entity/
│   ├── BaseJpaEntity.kt
│   └── example/
│       └── ExampleJpaEntity.kt     # 순수 JPA @Entity

apps/commerce-api/
├── domain/
│   └── example/
│       ├── Example.kt              # 순수 도메인 (POJO)
│       ├── ExampleId.kt            # Value Object
│       ├── ExampleRepository.kt    # 인터페이스
│       └── ExampleService.kt
├── infrastructure/
│   └── example/
│       ├── ExampleJpaRepository.kt
│       ├── ExampleRepositoryImpl.kt
│       └── ExampleMapper.kt        # Entity ↔ Domain 매핑
└── ...
```

**특징:**

- 도메인 모델에 JPA 어노테이션 없음
- 영속성 모델(Entity)과 도메인 모델(Domain) 완전 분리
- Mapper가 변환 담당

**장점:**

- Persistence Ignorance 완전 준수
- 도메인 모델 테스트가 매우 쉬움 (순수 POJO)
- ORM 교체 시 인프라 레이어만 수정

**단점:**

- 매핑 코드 복잡성 증가 (1:N, N:M 관계에서 기하급수적)
- ORM의 편의 기능을 직접 구현해야 할 수 있음
- 학습 및 유지보수 비용 증가

### 두 구조 비교

| 관점        | Rich Domain Model | 완전 분리 |
|-----------|-------------------|-------|
| 순수성       | 낮음                | 높음    |
| 복잡성       | 낮음                | 높음    |
| 테스트 용이성   | 중간                | 높음    |
| ORM 기능 활용 | 완전                | 제한적   |
| 학습 곡선     | 낮음                | 높음    |
| 실무 적용 빈도  | 매우 높음             | 선택적   |

---

## 4. DDD 학습 로드맵

### Phase 1: 현재 교육과정 집중 (Now)

**목표:** TDD와 설계 역량이 DDD의 기반

```
현재 학습 중:
├── TDD (Red → Green → Refactor)
├── 레이어드 아키텍처
├── 테스트 작성법 (Unit, Integration, E2E)
├── Repository 패턴
└── Hexagonal Architecture 기초
```

**왜 중요한가:**

- DDD의 Repository 패턴은 이미 프로젝트에 적용되어 있음
- TDD는 도메인 모델 설계의 핵심 도구
- 레이어 분리 개념이 Bounded Context 이해의 기반

### Phase 2: DDD 이론 학습

**추천 순서:**

1. **입문** (1-2주)
    - [Martin Fowler - Domain Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html) (블로그)
    - [Wikipedia - Domain-driven design](https://en.wikipedia.org/wiki/Domain-driven_design)

2. **기초** (2-4주)
    - [DEV.to - Domain-Driven Design by Eric Evans 요약](https://dev.to/alco/domain-driven-design-by-eric-evans-30g8)
    - [Microsoft - DDD Oriented Microservice](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)

3. **심화** (4-8주)
    - 📘 "도메인 주도 설계" - Eric Evans (Blue Book)
    - 📘 "도메인 주도 설계 구현" - Vaughn Vernon (Red Book)

### Phase 3: 실습 프로젝트 적용

**단계별 적용:**

1. **Value Object부터 시작**
   ```kotlin
   // 기존: primitive 타입
   fun createOrder(userId: Long, amount: BigDecimal)

   // 개선: Value Object
   fun createOrder(userId: UserId, amount: Money)
   ```

2. **Entity에 비즈니스 로직 집중**
   ```kotlin
   // 기존: Service에서 검증
   class OrderService {
       fun cancel(order: Order) {
           if (order.status != PENDING) throw ...
           order.status = CANCELLED
       }
   }

   // 개선: Entity가 책임
   class Order {
       fun cancel() {
           require(status == PENDING) { "취소 불가 상태" }
           status = CANCELLED
       }
   }
   ```

3. **Aggregate 경계 정의**

4. **(선택) Domain/Persistence 분리**

### Phase 4: 심화 학습

- CQRS (Command Query Responsibility Segregation)
- Event Sourcing
- Domain Events
- Saga Pattern

---

## 5. 실무 적용 시 고려사항

### 언제 DDD가 필요한가?

**적합한 경우:**

- 복잡한 비즈니스 로직이 있는 도메인
- 도메인 전문가와 협업이 필요한 프로젝트
- 장기적으로 유지보수될 시스템
- 여러 Bounded Context가 존재하는 대규모 시스템

**적합하지 않은 경우:**

- 단순 CRUD 애플리케이션
- 짧은 생명주기의 프로젝트
- 팀에 DDD 경험이 전혀 없는 경우
- 학습 시간이 부족한 상황

### 언제 완전 분리가 과한가?

Vladimir Khorikov에 따르면, **대부분의 경우 완전 분리는 과합니다:**

> "A fully-fledged persistence model is too costly to implement"

**분리가 필요한 예외 상황:**

1. ORM을 사용할 수 없는 경우
2. 레거시 DB 스키마를 제어할 수 없는 경우 (Anti-corruption layer)
3. MongoDB 같은 문서 DB에서 스키마 버전 관리가 필요한 경우

### MapStruct 활용법

Entity ↔ Domain 매핑 시 보일러플레이트 코드를 줄여줍니다.

**build.gradle.kts 설정:**

```kotlin
plugins {
    kotlin("kapt")
}

dependencies {
    implementation("org.mapstruct:mapstruct:1.5.5.Final")
    kapt("org.mapstruct:mapstruct-processor:1.5.5.Final")
}
```

**Mapper 정의:**

```kotlin
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
interface OrderMapper {
    fun toDomain(entity: OrderJpaEntity): Order
    fun toEntity(domain: Order): OrderJpaEntity

    @Mapping(target = "items", source = "orderItems")
    fun toDomainWithItems(entity: OrderJpaEntity): Order
}
```

**주의사항:**

- 복잡한 Aggregate는 수동 매핑이 더 명확할 수 있음
- 양방향 매핑 시 순환 참조 주의
- Kotlin data class와의 호환성 확인 필요

---

## 6. 참고 자료 (Sources)

### DDD 이론 및 원칙

- [Enterprise Craftsmanship - Domain Model vs Persistence Model](https://enterprisecraftsmanship.com/posts/having-the-domain-model-separate-from-the-persistence-model/)
- [Vladimir Khorikov - When do you need a persistence model?](https://khorikov.org/posts/2020-04-20-when-do-you-need-persistence-model/)
- [Microsoft - DDD Oriented Microservice](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice)
- [Martin Fowler - Domain Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html)

### Hexagonal Architecture & Spring Boot

- [Vaadin - DDD and Hexagonal Architecture](https://vaadin.com/blog/ddd-part-3-domain-driven-design-and-the-hexagonal-architecture)
- [Baeldung - Hexagonal Architecture, DDD, and Spring](https://www.baeldung.com/hexagonal-architecture-ddd-spring)
- [Baeldung - Persisting DDD Aggregates](https://www.baeldung.com/spring-persisting-ddd-aggregates)

### JPA & Persistence 패턴

- [ORM Anti-patterns - Persistence vs Domain Model](https://www.mehdi-khalili.com/orm-anti-patterns-part-4-persistence-domain-model)
- [DEV.to - Building Aggregates with Spring Data](https://dev.to/peholmst/building-aggregates-with-spring-data-2iig)
- [GitHub - domain-driven-hexagon](https://github.com/Sairyss/domain-driven-hexagon)

### 멀티모듈 & Bounded Context

- [DZone - DDD Spring Boot Multi-Module Maven Project](https://dzone.com/articles/ddd-spring-boot-multi-module-maven-project)
- [Baeldung - DDD Bounded Contexts and Java Modules](https://www.baeldung.com/java-modules-ddd-bounded-contexts)
- [DEV.to - Modeling Shared Entities Across Bounded Contexts](https://dev.to/aws-builders/modeling-shared-entities-across-bounded-contexts-in-domain-driven-design-5hih)

### 도구 (MapStruct)

- [MapStruct with Kotlin - Comprehensive Guide](https://medium.com/hprog99/mapstruct-with-kotlin-and-spring-boot-a-comprehensive-guide-1b2eb0d1e2a0)
- [MapStruct Official Reference Guide](https://mapstruct.org/documentation/stable/reference/html/)

### 추가 학습 자료

- [Wikipedia - Domain-driven design](https://en.wikipedia.org/wiki/Domain-driven_design)
- [DEV.to - Domain-Driven Design by Eric Evans](https://dev.to/alco/domain-driven-design-by-eric-evans-30g8)
- [Vaughn Vernon 공식 사이트](https://vaughnvernon.com/)
