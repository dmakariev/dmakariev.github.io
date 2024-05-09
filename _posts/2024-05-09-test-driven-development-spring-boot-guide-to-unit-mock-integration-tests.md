---
title: "Test-Driven Development with Spring Boot: Guide to Unit, Mock, and Integration Tests"
tags: [tutorial, java, spring boot, mockmvc, junit]
thumbnail-img: "/assets/img/blog/spring-coffee-4.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

In the previous [blog post](https://www.makariev.com/blog/advanced-spring-boot-structure-clean-architecture-modulith/), we explored advanced **Spring Boot** architecture, delving into **clean architecture** and modularity. This follow-up post will focus on implementing **Test-Driven Development (TDD)** in Spring Boot using the Inventory entity as a **practical example**. We'll clarify the differences between unit, mock, and integration testing while providing illustrative code snippets.

* toc
{:toc}

# Prerequisites
Before we dive into the development process, ensure you have:
1. Java 21 installed on your system. You can install using [sdkman](https://sdkman.io/install), and select Java 21 [https://sdkman.io/usage](https://sdkman.io/usage)
2. `Docker` and `Docker Compose` installed for setting up the local environment.

You can clone the `https://github.com/dmakariev/examples` repository.
```bash
git clone https://github.com/dmakariev/examples.git
cd examples/spring-boot/bookstore
```

# Test-Driven Development Overview
Test-Driven Development is a software development methodology where tests are written before writing the functional code. The TDD cycle involves:

1. **Write a Test**: Create a test that defines the desired behavior.
2. **Run the Test**: Ensure the new test fails since the feature isn't implemented yet.
3. **Write the Code**: Develop code that will make the test pass.
4. **Run the Test Again**: Verify that the test now passes.
5. **Refactor**: Refine and optimize the code while ensuring the test still passes.

# Types of Tests
In Spring Boot, three primary testing approaches stand out:

* **Unit Tests**: Validate individual components in isolation.
* **Mock Tests**: Simulate complex behavior by replacing dependencies with mock objects.
* **Integration Tests**: Ensure that different components work together seamlessly.

# Setting Up the Environment
The folder structure for the tests is based on a focused approach, limiting our scope to the Inventory entity. The files relevant to our discussion are organized as follows:

```bash
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── makariev
    │               └── examples
    │                   └── spring
    │                       └── bookstore
    │                           ├── inventory
    │                           │   ├── Inventory.java
    │                           │   ├── InventoryController.java
    │                           │   ├── InventoryRepository.java
    │                           │   ├── InventoryService.java
    │                           │   ├── StockAddedEvent.java
    │                           │   └── StockRemovedEvent.java
    │                           └── product
    │                               ├── Author.java
    │                               ├── AuthorRepository.java
    │                               ├── Book.java
    │                               └── BookRepository.java
    └── test
        ├── java
        │   └── com
        │       └── makariev
        │           └── examples
        │               └── spring
        │                   └── bookstore
        │                       └── inventory
        │                           ├── InventoryControllerIT.java
        │                           ├── InventoryControllerTest.java
        │                           ├── InventoryRepositoryTest.java
        │                           └── InventoryServiceTest.java
        └── resources
            ├── application-integration-test.properties
            └── application-test.properties

```

# Testing Configurations
## Unit and Mock Tests
Files with `Test` in their names (e.g., `InventoryControllerTest.java`) utilize an **H2 in-memory database**, as specified in `application-test.properties`:
```
spring.jpa.hibernate.ddl-auto=create-drop
spring.sql.init.mode=never
spring.autoconfigure.exclude=org.springframework.boot.actuate.autoconfigure.tracing.zipkin.ZipkinAutoConfiguration
```

## Integration Tests
Files with `IT` in their names (e.g., `InventoryControllerIT.java`) use **PostgreSQL** via **Testcontainers**, configured in `application-integration-test.properties`:
```
spring.jpa.hibernate.ddl-auto=create-drop
spring.sql.init.mode=never
spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver
spring.datasource.url=jdbc:tc:postgresql:16://localhost/bookstore?TC_DAEMON=true
spring.autoconfigure.exclude=org.springframework.boot.actuate.autoconfigure.tracing.zipkin.ZipkinAutoConfiguration
```

The Maven Failsafe Plugin is used to execute these tests:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <configuration>
        <includes>
            <include>**/*IT.java</include>
        </includes>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

# Testing the Inventory Entity
## Unit Testing with `InventoryServiceTest`
In this example, we use **Mockito** to mock `InventoryRepository` and `ApplicationEventPublisher`, testing isolated business logic in the `InventoryService` class.
Here, we ensure the `addStock` method functions correctly while also checking the emitted events.
```java
@ExtendWith(MockitoExtension.class)
public class InventoryServiceTest {
    @Mock
    private InventoryRepository inventoryRepository;
    @Mock
    private ApplicationEventPublisher applicationEventPublisher;
    @InjectMocks
    private InventoryService inventoryService;
    ...
    @Test
    void addStock_ShouldIncreaseInventoryAndPublishEvent() {
        given(inventoryRepository.findById(1L)).willReturn(Optional.of(inventory));
        given(inventoryRepository.save(inventory)).willReturn(inventory);
        ...
        verify(applicationEventPublisher, times(1)).publishEvent(new StockAddedEvent(1L, 10));
    }
}

```

## Mock Testing with `InventoryControllerTest`
In mock testing, we isolate the controller by using **@WebMvcTest** and mocking the `InventoryService`.
This testing setup enables simulating controller behavior with the mocked service. This allows us to focus on controller logic without involving actual database interactions:
```java
@ExtendWith(SpringExtension.class)
@WebMvcTest(InventoryController.class)
public class InventoryControllerTest {
    @Autowired
    private MockMvc mockMvc;
    @MockBean
    private InventoryService inventoryService;
    ...
    @Test
    void getAllInventories_ShouldReturnAllInventories() throws Exception {
        final List<Inventory> inventories = Arrays.asList(inventory);
        given(inventoryService.findAll()).willReturn(inventories);
        ...
    }
}

```

## Repository Testing with `InventoryRepositoryTest`
`@DataJpaTest` sets up a test environment for repository logic in a lightweight manner.
Here, we directly test the `InventoryRepository` to ensure data correctness.
This ensures that JPA transactions and entity management are functioning as expected:

```java
@DataJpaTest
public class InventoryRepositoryTest {
    @Autowired
    private TestEntityManager entityManager;
    @Autowired
    private InventoryRepository inventoryRepository;
    ...
    @Test
    public void whenSaveInventory_thenFindById() {
        // Create and persist the entities
        final Author author = new Author("Jane Austen");
        entityManager.persist(author);
        ...
        // Verify persistence and retrieval
        final Inventory found = inventoryRepository.findById(inventory.getId()).orElse(null);
        assertThat(found).isNotNull();
    }
}

```

## Integration Testing with `InventoryControllerIT`
Integration tests in Spring Boot combine various components to validate their interactions, achieved with `@SpringBootTest`.
This example validates that the inventory data persists correctly across different layers.
Our integration tests ensure that the application components work correctly together. These tests run with a real database setup, replicating a production-like environment:
```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class InventoryControllerIT {
    @Autowired
    private MockMvc mockMvc;
    @Autowired
    private InventoryRepository inventoryRepository;
    ...
    @Test
    void getAllInventories_ShouldReturnAllInventories() throws Exception {
        mockMvc.perform(get("/api/inventory"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray())
            .andExpect(jsonPath("$[0].quantity").value(100));
    }
}

```


# Conclusion
Embracing **TDD** in **Spring Boot** not only enforces a **disciplined** approach to development but also ensures that each component is rigorously tested, resulting in a more robust and error-resistant application. Each type of test serves a unique purpose, together providing a **comprehensive coverage** across all layers of the application.

---

[![Coffee Time!](/assets/img/blog/spring-coffee-4.jpg)](/assets/img/blog/spring-coffee-4.jpg)

---

 Happy coding!
