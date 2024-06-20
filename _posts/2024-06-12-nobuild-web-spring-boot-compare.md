---
title: Spring Boot - Comparing Vue.js, Jakarta Server Faces, and Thymeleaf/HTMX
tags: [spring boot, java, jbang, vuejs, jsf, faces, thymeleaf, htmx]
thumbnail-img: "/assets/img/blog/docker-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

Creating web applications with Spring Boot can be approached using different front-end technologies. This post compares three approaches: Vue.js, JakartaServer Faces (JSF), and Thymeleaf with HTMX, based on a series of tutorials 

## Related posts 
The series includes the following posts:

1. [#nobuild Web Application Development with Spring Boot](https://www.makariev.com/blog/nobuild-web-spring-boot/)
2. [Creating a Web Application with Spring Boot and Vue.js](https://www.makariev.com/blog/nobuild-web-spring-boot-vuejs/)
3. [Building a Web Application with Spring Boot and Jakarta Server Faces](https://www.makariev.com/blog/nobuild-web-spring-boot-faces/)
4. [Creating a Web Application with Thymeleaf and HTMX
](https://www.makariev.com/blog/nobuild-web-spring-boot-thymeleaf-htmx/)

## Performance Comparison

Performance tests were performed to compare these technologies. 
We've used [https://github.com/hatoo/oha](https://github.com/hatoo/oha) as load generating tool

* Minimal Docker configuration with JSF: 
```yaml
    deploy:
      resources:
        limits:
          cpus: 1.0
          memory: 640M
        reservations:
          cpus: 0.5
          memory: 256M 
```

* Minimal Docker configuration without JSF: 
```yaml
    deploy:
      resources:
        limits:
          cpus: 1.0
          memory: 256M
        reservations:
          cpus: 0.5
          memory: 256M 
```

[![Performance Comparison!](/assets/img/blog/nobuild-web-spring-boot-compare.jpg)](/assets/img/blog/nobuild-web-spring-boot-compare.jpg)

## Spring Boot and Vue.js

[Building with Vue.js](https://www.makariev.com/blog/nobuild-web-spring-boot-vuejs/) involves using Vue CLI for scaffolding the project and creating a dynamic front-end. Vue.js excels in building SPAs (Single Page Applications), offering reactive data binding and a component-based architecture. Integration with Spring Boot is achieved via REST APIs, making this combination suitable for highly interactive applications.

### Pros:

* Modern, reactive UI
* Component-based architecture
* Good for SPAs

### Cons:

* Requires separate front-end build process
* Initial setup can be complex

### Performance Test: 
```bash 
oha -z 30s http://localhost:8080/api/persons
```
* **Requests/sec**: 1619 
* **Average response time**: 0.0308 secs

## Spring Boot and JakartaServer Faces
[Using JakartaServer Faces (JSF)](https://www.makariev.com/blog/nobuild-web-spring-boot-faces/) allows for a component-based framework that integrates directly with Java EE. JSF manages UI components and state on the server side, simplifying the connection to backend services.

### Pros:

* Rich server-side UI component model
* Simplified state management
* Direct integration with Java EE

### Cons:

* Can be heavy and slower compared to modern JS frameworks
* Less flexible for highly dynamic interfaces

### Performance Test: 
```bash
oha -z 30s http://localhost:8080/person-crud-faces
```
* **Requests/sec**: 122
* **Average response time**: 0.4097 secs

## Spring Boot, Thymeleaf, and HTMX
[Combining Thymeleaf and HTMX](https://www.makariev.com/blog/nobuild-web-spring-boot-thymeleaf-htmx/) offers a hybrid approach. Thymeleaf handles server-side rendering, while HTMX allows for dynamic content updates without full page reloads by leveraging HTML over AJAX.

## Pros:

* Simplified setup with no separate front-end build
* Server-side rendering benefits
* Enhanced interactivity with minimal JavaScript

## Cons:

* Less dynamic than a full SPA
* Limited to HTML over AJAX for interactivity

### Performance Test: 
```bash 
oha -z 30s http://localhost:8080/person-crud-htmx
```
* **Requests/sec**: 366
* **Average response time**: 0.1366 secs


## Conclusion
* **Vue.js** is best for highly interactive SPAs needing a reactive UI.
* **JSF** is suitable for server-centric applications requiring robust state management.
* **Thymeleaf/HTMX** provides a middle ground, enhancing traditional server-side rendering with modern interactivity.

Each approach has its strengths, and the best choice depends on project requirements and team expertise. For a detailed comparison and practical examples, explore the full tutorials linked in each section.

Happy coding!
