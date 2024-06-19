---
title: "#nobuild Web Application Development with Spring Boot"
tags: [spring boot, java, jbang]
thumbnail-img: "/assets/img/blog/docker-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---


Welcome to our series on **web** application development with **Spring Boot**! In this series, we will explore various modern web development frameworks and tools by implementing a **CRUD** (Create, Read, Update, Delete) application for managing a `Person` entity, **without** relying on any real **build** steps on the **front end**. Each post will guide you through building the same application using different front-end technologies, comparing their features and development experiences. We'll also demonstrate how to set up these projects using [JBang](https://www.jbang.dev/), a powerful scripting tool for Java.

## Related posts 
The series includes the following posts:

1. [#nobuild Web Application Development with Spring Boot](https://www.makariev.com/blog/nobuild-web-spring-boot/)
2. [Creating a Web Application with Spring Boot and Vue.js](https://www.makariev.com/blog/nobuild-web-spring-boot-vuejs/)
3. [Building a Web Application with Spring Boot and Jakarta Server Faces](https://www.makariev.com/blog/nobuild-web-spring-boot-faces/)
4. [Creating a Web Application with Thymeleaf and HTMX
](https://www.makariev.com/blog/nobuild-web-spring-boot-thymeleaf-htmx/)

## Repository and Setup

You can clone the [`https://github.com/dmakariev/examples`](https://github.com/dmakariev/examples){:target="_blank"} repository.
```bash
git clone https://github.com/dmakariev/examples.git
cd examples/jbang/spring-boot-compare
```
This repository provides a hands-on example of how to set up and run these projects using JBang.

Run the application
```bash
jbang springbootCompare.java
```
Open the application in the browser [http://localhost:8080](http://localhost:8080){:target="_blank"}

## Project Overview
In each post, we will implement a web application with the following features:

* A `Person` entity with fields id, firstName, lastName, and birthYear.
* An API for CRUD operations on the `Person` entity using Spring Boot and Spring Data JPA.
* Different front-end frameworks to build the user interface.
Here is the `Person` entity and repository definition for our project:

```java
@Data
@Entity
@Table(name = "person")
@NoArgsConstructor
@AllArgsConstructor
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;
    private Integer birthYear;
}
```

and the `PersonRepository`

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface PersonRepository extends JpaRepository<Person, Long> {
}
```

## 1. Creating a Web Application with Spring Boot and Vue.js
In this post, we'll start with the basics of integrating Spring Boot with Vue.js. Vue.js is a progressive JavaScript framework used for building user interfaces. We will cover setting up the project, configuring the back-end and front-end, and implementing the CRUD functionality.

Read the full post: [Creating a Web Application with Spring Boot and Vue.js](https://www.makariev.com/blog/nobuild-web-spring-boot-vuejs/)

## 2. Building a Web Application with Spring Boot and Jakarta Server Faces
Next, we'll dive into Jakarta Server Faces (JSF), a Java-based web application framework. JSF simplifies the development of user interfaces for JavaServer applications. This post will guide you through integrating JSF with Spring Boot and implementing the CRUD operations.

Read the full post: [Building a Web Application with Spring Boot and Jakarta Server Faces](https://www.makariev.com/blog/nobuild-web-spring-boot-faces/)

## 3. Creating a Web Application with Thymeleaf and HTMX
Thymeleaf is a modern server-side Java template engine for web and standalone environments. Coupled with HTMX, which allows you to access modern browser features directly from HTML, we will create a dynamic and interactive web application. This post will explain how to set up Thymeleaf and HTMX with Spring Boot.

Read the full post: [Creating a Web Application with Thymeleaf and HTMX
](https://www.makariev.com/blog/nobuild-web-spring-boot-thymeleaf-htmx/)

## 4. Comparing Web Application Development with Spring Boot Using Vue.js, Jakarta Server Faces, and Thymeleaf/HTMX
Finally, we will compare the three approaches—Vue.js, Jakarta Server Faces, and Thymeleaf/HTMX. This comparative analysis will highlight the pros and cons of each framework in the context of our CRUD application, helping you make an informed decision on which technology to use for your projects.

Read the full post: [Comparing Web Application Development with Spring Boot Using Vue.js, Jakarta Server Faces, and Thymeleaf/HTMX](https://www.makariev.com/blog/nobuild-web-spring-boot-compare/)

Stay tuned as we embark on this journey to explore different web development frameworks with Spring Boot. 

Happy coding!


