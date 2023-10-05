---
title: How to Build a Spring Boot Rest Api with JBang in a Single Java File
tags: [tutorial, java, spring boot, jbang]
thumbnail-img: "/assets/img/blog/spring-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---


In the world of Java development, Spring Boot has become synonymous with creating robust, scalable, and maintainable web applications. Traditionally, building a Spring Boot application involved setting up a project with a complex directory structure, multiple configuration files, and various dependencies. However, with the advent of JBang, a lightweight scripting tool for Java, you can simplify this process and build a Spring Boot Rest Api using just a single Java file. In this blog post, we will guide you through the steps to create a Spring Boot Rest Api with JBang in a single Java file.

# What is JBang?
JBang is a command-line tool that allows you to run Java code directly from source files without the need for a complex project setup or compilation. It is particularly useful for creating lightweight scripts and simplifying the development process.

### Prerequisites
Before we dive into the development process, ensure you have JBang installed on your system. You can install it from [JBang's official website](https://www.jbang.dev/download/).

# Getting Started
Let's create a simple Spring Boot Rest service that serves a "Hello, World!" message using JBang. Follow these steps:

### Initialize a New JBang Script  
Create a new directory for your project and navigate to it using your terminal. Then, create a new JBang script file with a `.java` extension, e.g., `springbootHelloWorld.java`.
```bash
$ mkdir spring-boot-hello
$ cd spring-boot-hello
$ touch springbootHelloWorld.java
```

### Write the Spring Boot Code  
Open the `springbootHelloWorld.java` file in your favorite text editor or integrated development environment (IDE) and add the following code.

```java
//usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS org.springframework.boot:spring-boot-starter-web:3.1.4

@SpringBootApplication
@RestController
public class springbootHelloWorld {

    public static void main(String[] args) {
        SpringApplication.run(springbootHelloWorld.class, args);
    }

    @GetMapping("/")
    public String sayHi(@RequestParam(required = false, defaultValue = "World") String name) {
        return "Hello, " + name + "!";
    }
}
```
This code does the following:
* Prepares the file to be executed like a shell script
* Defines the Java version needed for the application (Java 17 is the minimum for Spring Boot 3.x, but Java 21 is the current LTS )
* Imports the necessary Spring Boot dependencies.
* Defines a Spring Boot application class.
* Defines a REST controller with a single endpoint that returns "Hello, World!".

### Run the Application
Save the file and return to your terminal. Navigate to the directory containing your springbootHelloWorld.java file and execute the following command:

```bash
$ jbang springbootHelloWorld.java
```
or
```bash 
$ sh springbootHelloWorld.java
```
if you allow executable permissions for `springbootHelloWorld.java` by executing 
```bash 
$ chmod +x springbootHelloWorld.java
```
you could even execute the application like that
```bash 
$ ./springbootHelloWorld.java
```
In all of the cases above, JBang will download the required Spring Boot dependencies and start the application. You will see output indicating that the Spring Boot application is running.

### Access the Application
Open your web browser and navigate to `http://localhost:8080`. You should see the **"Hello, World!"** message displayed in your browser.
Or if you prefer more personalized message, then navigate to `http://localhost:8080/?name=Joe`. You should see the **"Hello, Joe!"** message displayed in your browser.

# Conclusion
In this blog post, we demonstrated how to create a Spring Boot Rest Api using just a single Java file and JBang. This approach can be handy for quick prototyping, lightweight applications, or when you want to reduce the complexity of your development environment. As your application grows in complexity, you can always transition to a more traditional project structure. JBang provides a flexible and efficient way to develop Java applications without the need for heavyweight project setups.

Explore further and build more sophisticated Spring Boot applications using JBang. Happy coding!

---

[![Coffee Time!](/assets/img/blog/spring-coffee.jpg)](/assets/img/blog/spring-coffee.jpg)

You could download the source from 
[https://github.com/dmakariev/examples/tree/main/jbang](https://github.com/dmakariev/examples/tree/main/jbang)

