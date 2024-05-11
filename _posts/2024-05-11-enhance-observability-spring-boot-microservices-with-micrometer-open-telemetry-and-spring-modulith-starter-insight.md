---
title: "Enhancing Observability in Spring Boot Microservices with Micrometer, OpenTelemetry, and Spring Modulith Starter Insight"
tags: [tutorial, java, spring boot, mockmvc, junit]
thumbnail-img: "/assets/img/blog/spring-coffee-2s.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

In the previous [post](https://www.makariev.com/blog/advanced-spring-boot-structure-clean-architecture-modulith/), we explored the structure of a **Spring Boot** application using **advanced** modular **architecture**. This time, let's shift gears to discuss how to enhance **observability** in Spring Boot **microservices** and **applications**. Specifically, we'll delve into how integrating Micrometer, OpenTelemetry, and the `Spring Modulith Starter Insight` can offer comprehensive **performance insights** across your application's various layers.
* toc
{:toc}

# Prerequisites
Before diving into the implementation, make sure you have the following set up:
1. **Java 21**: Ensure Java 21 is installed using [SDKMAN](https://sdkman.io/install).
2. **Docker & Docker Compose**: Required for setting up the local environment.
3. **Repository Clone**: Clone the [examples](https://github.com/dmakariev/examples) repository to your local environment.

```bash
git clone https://github.com/dmakariev/examples.git
cd examples/spring-boot/bookstore
```

# Basic Observability in Spring Boot
A Spring Boot microservice by default exposes limited performance data at the RestController level. While useful, this data is often insufficient for a comprehensive understanding of the interactions between controllers, services, repositories, event publishers, and listeners.
[![basic!](/assets/img/blog/zipkin-basic.png)](/assets/img/blog/zipkin-basic.png)

# Integrating Advanced Observability Tools
To gain deeper insights, you can enhance observability by integrating the following dependencies:

## Spring Modulith Starter Insight
```xml
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-starter-insight</artifactId>
    <scope>runtime</scope>
</dependency>
```

## Micrometer Tracing Bridge OpenTelemetry
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
```

## OpenTelemetry Exporter Zipkin
```xml
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-starter-insight</artifactId>
    <scope>runtime</scope>
</dependency>
```

## Application Configuration
Update your `application.properties` file to enable tracing:

```properties
management.tracing.sampling.probability=1.0
management.tracing.enabled=true
management.zipkin.tracing.endpoint=http://localhost:9411/api/v2/spans
```

After integrating Micrometer, OpenTelemetry, and the `Spring Modulith Starter Insight`, the screenshot below shows the detailed performance data captured, including interactions between controllers, services, and other components. This **enhanced trace**, visualized using Zipkin, highlights the **robust observability capabilities** now at our disposal.
[![enhanced!](/assets/img/blog/zipkin-enhanced.png)](/assets/img/blog/zipkin-enhanced.png)

# Running the Application
Start the application with Zipkin enabled
```bash
docker compose up --build
``` 

Alternatively, you can run only the backend in a separate window
```bash
docker compose up --build backend
```

# Cyclic Dependency Example
## CyclicDependencyController
We've implemented a new `CyclicDependencyController` in the `com.makariev.examples.spring.bookstore.product` package. This controller introduces a **cyclic dependency** between the `product` and `inventory` packages, deliberately triggering modular structure verification errors.

```java
@RestController
@RequestMapping("/api/cycle")
public class CyclicDependencyController {

    private final InventoryService inventoryService;

    public CyclicDependencyController(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }

    @GetMapping
    public ResponseEntity<List<Inventory>> getAllInventories() {
        final List<Inventory> inventories = inventoryService.findAll();
        return ResponseEntity.ok(inventories);
    }
}

```
## Modularity Tests
Enable the `ModularityTests` class to verify the modular structure

```java
public class ModularityTests {

    private final ApplicationModules modules = ApplicationModules.of(BookstoreApplication.class);

    @Test
    void verifiesModularStructure() {
        modules.verify();
    }

    @Test
    void createModuleDocumentation() {
        new Documenter(modules).writeDocumentation();
    }
}
```

When these tests are run, you'll encounter errors due to the cyclic dependency:
```
[ERROR] Errors: 
[ERROR]   ModularityTests.verifiesModularStructure:17 » Violations - Cycle detected: Slice inventory ->
...
```

# Conclusion
By adding these observability tools to your Spring Boot microservices or applications, you can obtain valuable performance insights, even across complex structures like those encountered with cyclic dependencies. The **Spring Modulith Starter Insight**, in particular, offers a modular and flexible approach to structured monitoring, irrespective of the architecture conventions in place.

Be sure to test your application's modularity with `ModularityTests` and leverage the visualized traces provided by Zipkin to fully comprehend interactions across your system's various layers.

---

[![Coffee Time!](/assets/img/blog/spring-coffee-2s.jpg)](/assets/img/blog/spring-coffee-2s.jpg)

---

 Happy coding!
