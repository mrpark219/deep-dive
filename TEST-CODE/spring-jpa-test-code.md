# Spring & JPA 기반 테스트

## 1. Spring

### 1.1 Library VS Framework

- **라이브러리는 내 코드가 주체가 되어 필요한 기능을 외부에서 가져와 사용한다**.
- **프레임워크는 이미 동작할 수 있는 환경이 구성되어 있으며, 내 코드가 프레임워크 안에서 동작하는 방식이다**.

### 1.2 Layered Architecture

- **사용자 요청 ↔ Presentation Layer ↔ Business Layer ↔ Persistence Layer ↔ DB**
- **관심사의 분리**: 각 레이어가 독립적으로 역할을 수행하도록 한다.

### 1.3 Spring 3대 개념

- **IoC(Inversion Of Control)**: 객체의 생명주기를 직접 관리하지 않고 **제3자가 관리**하는 방식이다.
- **DI(Dependency Injection)**: **객체 간의 의존성을 외부에서 주입하여 코드의 결합도를 낮추는 방법이다**.
- **AOP(Aspect Oriented Programming)**: **핵심 비즈니스 로직과 부가 기능(로깅, 트랜잭션 등)을 분리하는 기법이다**.

### 1.4 ORM(Object-Relational Mapping)

- **객체 지향 패러다임과 관계형 데이터베이스 패러다임 간의 불일치를 해결하는 기술이다**.
- 기존에는 개발자가 직접 SQL을 작성하여 데이터를 저장/조회해야 했지만, **ORM을 사용하면 반복적인 SQL 작업을 줄이고 비즈니스 로직에 집중할 수 있다**.

### 1.5 JPA(Java Persistence API)

- **Java 진영의 ORM 기술 표준이며, 인터페이스 기반이다**.
- 대표적인 구현체로 **Hibernate**를 많이 사용한다.
- **반복적인 CRUD SQL을 자동 생성 및 실행하며, 다양한 부가 기능을 제공한다**.
- SQL을 직접 작성하지 않기 때문에 **JPA가 생성하는 쿼리를 명확하게 이해하는 것이 중요하다**.
- Spring에서는 **JPA를 한 번 더 추상화한 Spring Data JPA**를 제공하며, **QueryDSL과 함께 많이 사용된다**.

### 1.6 JPA 테스트가 필요한 이유

- **JPA가 자동 생성하는 쿼리가 예상한 대로 동작하는지 보장해야 한다**.
- **미래에 다른 기술로 전환될 가능성에 대비하여 테스트가 필요하다**.

## 2. 통합 테스트(Integration Test)

- **단위 테스트만으로는 커버할 수 없는 영역을 검증하는 테스트이다**.
- **여러 모듈이 협력하는 기능을 통합적으로 검증한다**.
- 단위 테스트만으로는 **기능 전체의 신뢰성을 보장할 수 없기 때문에 통합 테스트가 필요하다**.
- **풍부한 단위 테스트와 함께, 큰 기능 단위를 검증하는 통합 테스트가 병행되어야 한다**.

## 3. Persistence Layer

- Data Access의 역할을 담당하는 레이어이다.
- 비즈니스 가공 로직이 포함되지 않고, 데이터의 CRUD에만 집중해야 한다.

### 3.1 @SpringBootTest VS @DataJpaTest

- **`@SpringBootTest`**: 스프링 애플리케이션 전체를 로드하여 테스트를 수행한다.
- **`@DataJpaTest`**: JPA 관련 빈들만 로드하여 테스트를 수행하며, 실행 속도가 `@SpringBootTest`보다 빠르다.
- **`@DataJpaTest`는 `@Transactional`을 포함하고 있어 각 테스트가 실행 후 자동으로 롤백된다**.
- **`@SpringBootTest`는 롤백되지 않으므로 이전 테스트가 다음 테스트에 영향을 줄 수 있다**.

## 3.2 Persistence Layer Test

- JPA 관련된 빈만 로딩하여 Repository를 테스트하는 코드이다.
- 판매 상태에 따라 상품을 조회하는 테스트를 수행한다.

```java
@ActiveProfiles("test")
@DataJpaTest // JPA 관련 빈들만 로딩하기 때문에 속도가 @SpringBootTest에 비해 빠르다.
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @DisplayName("원하는 판매상태를 가진 상품들을 조회한다.")
    @Test
    void findAllBySellingStatusIn() {

        // given
        Product product1 = Product.builder()
                .productNumber("001")
                .type(HANDMADE)
                .sellingStatus(SELLING)
                .name("아메리카노")
                .price(4000)
                .build();

        Product product2 = Product.builder()
                .productNumber("002")
                .type(HANDMADE)
                .sellingStatus(HOLD)
                .name("카페라떼")
                .price(4500)
                .build();

        Product product3 = Product.builder()
                .productNumber("003")
                .type(HANDMADE)
                .sellingStatus(STOP_SELLING)
                .name("팥빙수")
                .price(7000)
                .build();

        productRepository.saveAll(List.of(product1, product2, product3));

        // when
        List<Product> products = productRepository.findAllBySellingStatusIn(List.of(SELLING, HOLD));

        // then
        assertThat(products).hasSize(2)
            .extracting("productNumber", "name", "sellingStatus")
            .containsExactlyInAnyOrder(
                    tuple("001", "아메리카노", SELLING),
                    tuple("002", "카페라떼", HOLD)
            );
    }
}
```

## 4. Business Layer

- **비즈니스 로직을 구현하는 레이어이다**.
- Persistence Layer와 상호작용하며 데이터를 읽고 쓰는 행위를 통해 **비즈니스 로직을 전개한다**.
- **트랜잭션을 보장해야 하는 중요한 책임을 가진다**.
- **Service Test는 Business Layer와 Persistence Layer를 함께 통합적으로 테스트한다**.
- 서비스 테스트 코드에서 `@Transactional`을 사용하는 경우, 실제 운영 서비스 코드에서는 해당 어노테이션이 없어도 테스트 환경에서는 **있는 것처럼 동작할 수 있다**.

```java
@ActiveProfiles("test")
@SpringBootTest
class OrderServiceTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private OrderService orderService;

    @DisplayName("주문번호 리스트를 받아 주문을 생성한다.")
    @Test
    void createOrder() {

        // given
        LocalDateTime registeredDateTime = LocalDateTime.now();

        Product product1 = createProduct(HANDMADE, "001", 1000);
        Product product2 = createProduct(HANDMADE, "002", 3000);
        Product product3 = createProduct(HANDMADE, "003", 5000);

        productRepository.saveAll(List.of(product1, product2, product3));

        OrderCreateRequest request = OrderCreateRequest.builder()
                .productNumbers(List.of("001", "002"))
                .build();

        // when
        OrderResponse orderResponse = orderService.createOrder(request, registeredDateTime);

        // then
        assertThat(orderResponse.getId()).isNotNull();
        assertThat(orderResponse)
                .extracting("registeredDateTime", "totalPrice")
                .contains(registeredDateTime, 4000);

        assertThat(orderResponse.getProducts()).hasSize(2)
                .extracting("productNumber", "price")
                .containsExactlyInAnyOrder(
                        tuple("001", 1000),
                        tuple("002", 3000)
                );

    }

    private Product createProduct(ProductType type, String productNumber, int price) {
        return Product.builder()
                .type(type)
                .productNumber(productNumber)
                .price(price)
                .sellingStatus(SELLING)
                .name("메뉴 이름")
                .build();
    }
}
```

## 5. Presentation Layer

- **외부 세계의 요청을 가장 먼저 받는 계층이다**.
- 요청 파라미터에 대한 **기본적인 검증만 수행하고**, 비즈니스 로직 처리는 Service 계층에 위임한다.
- 이 레이어에서는 주로 **Controller를 테스트하며**, 테스트 시 Business Layer와 Persistence Layer는 **Mocking**하여 독립적으로 검증한다.

### 5.1 MockMvc

- **Mock(가짜) 객체를 사용해 스프링 MVC 동작을 재현할 수 있는 테스트 프레임워크다**.
- 테스트 환경에서 **가짜(Mock) 객체**를 주입하여 복잡한 의존성을 제거하고, **Controller의 요청/응답 흐름을 독립적으로 테스트**할 수 있다.
- `@WebMvcTest` 어노테이션과 함께 사용하면 **Controller만 로딩된 테스트 환경**을 구성할 수 있다.
- HTTP 요청을 흉내내고 JSON 요청/응답, 상태 코드, 헤더, 본문 등을 검증할 수 있다.

### 5.2 @MockBean

- **스프링 테스트 컨텍스트(ApplicationContext)에 Mock 객체를 등록해주는 어노테이션이다**.
- 테스트 대상 클래스가 의존하고 있는 Bean을 **가짜(Mock) 객체로 교체하여 주입할 수 있다**.
- 일반적으로 `@WebMvcTest`, `@SpringBootTest` 등과 함께 사용되며, **의존성 주입이 필요한 곳에 테스트 전용 Mock 객체를 사용하게 해준다**.
- 기존에 등록된 동일한 타입의 Bean이 있다면, **`@MockBean`이 이를 대체한다**.

```java
@WebMvcTest(controllers = ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private ProductService productService;

    @DisplayName("신규 상품을 등록한다.")
    @Test
    void createProduct() throws Exception {

        // given
        ProductCreateRequest request = ProductCreateRequest.builder()
            .type(ProductType.HANDMADE)
            .sellingStatus(ProductSellingStatus.SELLING)
            .name("아메리카노")
            .price(4000)
            .build();

        // when
        // then
        mockMvc.perform(
                post("/api/v1/products/new")
                    .content(objectMapper.writeValueAsString(request))
                    .contentType(MediaType.APPLICATION_JSON)
            )
            .andDo(print())
            .andExpect(status().isOk());
    }

    @DisplayName("판매 상품을 조회한다.")
    @Test
    void getSellingProducts() throws Exception {
        // given
        List<ProductResponse> result = List.of();
        when(productService.getSellingProducts()).thenReturn(result);

        // when
        // then
        mockMvc.perform(
                get("/api/v1/products/selling")
            )
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value("200"))
            .andExpect(jsonPath("$.status").value("OK"))
            .andExpect(jsonPath("$.message").value("OK"))
            .andExpect(jsonPath("$.data").isArray());
    }
}
```
