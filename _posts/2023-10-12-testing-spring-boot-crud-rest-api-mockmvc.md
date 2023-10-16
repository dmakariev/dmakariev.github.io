---
title: Testing Spring Boot CRUD REST APIs with MockMvc - Practical Example
tags: [spring boot, mockmvc, junit]
thumbnail-img: "/assets/img/blog/spring-coffee-3.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---


Testing is a crucial part of any application development process, especially when building RESTful APIs with Spring Boot. In this blog post, we'll walk you through a practical example of testing CRUD (Create, Read, Update, Delete) REST APIs for a Person entity using Spring Data JPA with H2 as the database. We'll create a Maven project from scratch using Spring Initializr, write JUnit 5 tests, and demonstrate how to test the PersonController using MockMvc. Let's get started!


## Prerequisites
If you don't already have Maven installed, you can download it from the official Maven website [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi) or through SDKMAN [https://sdkman.io/sdks#maven](https://sdkman.io/sdks#maven)

You can clone the `https://github.com/dmakariev/examples` repository.
```bash
git clone https://github.com/dmakariev/examples.git
cd examples/spring-boot/crud-mockmvc
```

## Creating a Maven Project 
First, let's create a new Spring Boot project with Spring Initializr using the terminal. 
1. Open your terminal and navigate to the directory where you want to create your project.
2. Run the following command to generate a new Maven project:
```shell
curl https://start.spring.io/starter.tgz -d packaging=jar \
-d dependencies=data-jpa,web,h2,lombok \
-d baseDir=crud-mockmvc -d artifactId=crud-mockmvc \
-d name=crud-mockmvc -d type=maven-project \
-d groupId=com.makariev.examples.spring | tar -xzvf -
```
3. This command will download the Spring Initializr project and extract it into a directory named crud-mockmvc.
4. Next, open the project in your favorite IDE (such as NetBeans, IntelliJ IDEA, VSCode or Eclipse) to proceed.

## Creating the Entity: `Person`

We'll create a simple `Person` entity with `firstName`, `lastName`, and `birthYear` fields.

```java
package com.makariev.examples.spring.crudmockmvc.person;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data // generates getters, setters, equals and hashCode and constructor
@NoArgsConstructor //generates default constructor
@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String firstName;
    private String lastName;
    private int birthYear;

    public Person(String firstName, String lastName, int birthYear) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.birthYear = birthYear;
    }

}
```

## Creating the Repository: `PersonRepository`

```java
package com.makariev.examples.spring.crudmockmvc.person;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PersonRepository extends JpaRepository<Person, Long> {

}
```

## Implementing the CRUD REST API

Now, let's create a RESTful CRUD API to manage `Person` entities using the `PersonController`. The controller will handle HTTP requests to perform operations like creating, retrieving, updating, and deleting `Person` objects.

```java
package com.makariev.examples.spring.crudmockmvc.person;

import java.util.Optional;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

@RequiredArgsConstructor // generates constructor for all 'final' fields 
@RestController
@RequestMapping("/api/persons")
public class PersonController {

    private final PersonRepository personRepository;

    @GetMapping
    public Page<Person> findAll(Pageable pageable) {
        return personRepository.findAll(pageable);
    }

    @GetMapping("{id}")
    public Optional<Person> findById(@PathVariable("id") Long id) {
        return personRepository.findById(id);
    }

    @PostMapping
    @ResponseStatus( HttpStatus.CREATED )
    public Person create(@RequestBody Person person) {
        return personRepository.save(person);
    }

    @PutMapping("{id}")
    public Person updateById(@PathVariable("id") Long id, @RequestBody Person person) {
        var loaded = personRepository.findById(id).orElseThrow();
        loaded.setFirstName(person.getFirstName());
        loaded.setLastName(person.getLastName());
        loaded.setBirthYear(person.getBirthYear());
        return personRepository.save(loaded);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus( HttpStatus.NO_CONTENT )
    public void deleteById(@PathVariable("id") Long id) {
        personRepository.deleteById(id);
    }
}

```

## Run the Application 
Return to your terminal. Navigate to the directory containing your project.
Execute 
```shell
./mvnw spring-boot:run
```

## Access the Application from Terminal/CLI
Open your web browser and navigate to `http://localhost:8080/hi`. You should see the **"Hello, World!"** message displayed in your browser.
Or if you prefer more personalized message, then navigate to `http://localhost:8080/hi?name=Joe`. You should see the **"Hello, Joe!"** message displayed in your browser.

### Using the Rest Api with Curl 

To create a new person, use the POST method with the person data as a JSON body:
```bash
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"Katherine", "lastName":"Johnson", "birthYear":1919}' \
http://localhost:8080/api/persons
```

To get a list of all persons, use the GET method:
```bash
curl -X GET http://localhost:8080/api/persons
```

To get a specific person by id, use the GET method with the id as a path variable:
```bash
curl -X GET http://localhost:8080/api/persons/1
```

To update an existing person by id, use the PUT method with the person data as a JSON body:
```bash
curl -X PUT -H "Content-Type: application/json" \
-d '{"firstName":"Katherine", "lastName":"Johnson", "birthYear":1918}' \
http://localhost:8080/api/persons/1
```

### Using the Rest Api with HTTPIE
you could download alternative Terminal/CLI client from here [https://httpie.io/cli](https://httpie.io/cli) 

To create a new person, use the POST method with the person data as a JSON body:
```bash
http POST http://localhost:8080/api/persons firstName=Alice lastName=Smith birthYear=1996
```

To get a list of all persons, use the GET method:
```bash
http GET http://localhost:8080/api/persons
```

To get a specific person by id, use the GET method with the id as a path variable:
```bash
http GET http://localhost:8080/api/persons/1
```

To update an existing person by id, use the PUT method with the person data as a JSON body:
```bash
http PUT http://localhost:8080/api/persons/1 firstName=Bob lastName=Jones birthYear=1990
```

To delete an existing person by id, use the DELETE method with the id as a path variable:
```bash
http DELETE http://localhost:8080/api/persons/1
```


## Writing JUnit Tests with MockMvc

We'll create JUnit 5 tests for the `PersonController` using `MockMvc`, which allows us to simulate HTTP requests and validate the responses.

Let's create a test class `PersonControllerTest` in the `src/test/java/com/makariev/examples/spring/crudmockmvc/person` directory:

```java
package com.makariev.examples.spring.crudmockmvc.person;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.assertj.core.api.AssertionsForClassTypes.assertThat;
import static org.hamcrest.CoreMatchers.is;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.delete;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class PersonControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private PersonRepository personRepository;

    // Test method for creating a new Person
    @Test
    void shouldCreateNewPerson() throws Exception {
        // Create a new Person instance
        final Person person = new Person();
        person.setFirstName("John");
        person.setLastName("Doe");
        person.setBirthYear(1980);

        // Perform a POST request to create the Person
        mockMvc.perform(post("/api/persons")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(person)))
                .andDo(print())
                .andExpect(status().isCreated());

        // Verify that the Person was created in the database
        assertThat(personRepository.count()).isEqualTo(1);

        //clean the database
        personRepository.deleteAll();
    }

    // Test method for retrieving a Person by ID
    @Test
    void shouldRetrievePersonById() throws Exception {
        // Create a new Person and save it in the database
        final Person savedPerson = personRepository.save(new Person("Alice", "Smith", 1990));

        // Perform a GET request to retrieve the Person by ID
        mockMvc.perform(get("/api/persons/{id}", savedPerson.getId()))
                .andExpect(status().isOk())
                .andDo(print())
                .andExpect(jsonPath("$.firstName", is(savedPerson.getFirstName())))
                .andExpect(jsonPath("$.lastName", is(savedPerson.getLastName())))
                .andExpect(jsonPath("$.birthYear", is(savedPerson.getBirthYear())));

        //clean the database
        personRepository.delete(savedPerson);
    }

    // Test method for updating a Person
    @Test
    void shouldUpdatePerson() throws Exception {
        // Create a new Person and save it in the database
        final Person savedPerson = personRepository.save(new Person("Bob", "Johnson", 1985));

        // Update the Person's information
        savedPerson.setFirstName("UpdatedFirstName");
        savedPerson.setLastName("UpdatedLastName");

        // Perform a PUT request to update the Person
        mockMvc.perform(put("/api/persons/{id}", savedPerson.getId())
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(savedPerson)))
                .andDo(print())
                .andExpect(status().isOk());

        // Verify that the Person's information was updated in the database
        final Person updatedPerson = personRepository.findById(savedPerson.getId()).orElse(null);
        assertThat(updatedPerson).isNotNull();
        assertThat(updatedPerson.getFirstName()).isEqualTo("UpdatedFirstName");
        assertThat(updatedPerson.getLastName()).isEqualTo("UpdatedLastName");

        //clean the database
        personRepository.delete(savedPerson);
    }

    // Test method for deleting a Person
    @Test
    void shouldDeletePerson() throws Exception {
        // Create a new Person and save it in the database
        final Person savedPerson = personRepository.save(new Person("Eve", "Williams", 2000));

        // Perform a DELETE request to delete the Person by ID
        mockMvc.perform(delete("/api/persons/{id}", savedPerson.getId()))
                .andDo(print())
                .andExpect(status().isNoContent());

        // Verify that the Person was deleted from the database
        assertThat(personRepository.existsById(savedPerson.getId())).isFalse();
    }
}

```

### Running the Test
To run the test, execute the following command in the project's root directory:
```shell
./mvnw test
```

JUnit 5 and AssertJ will execute the test, and you should see output indicating whether the test passed or failed.

# Conclusion

In this blog post, we've demonstrated how to create a Spring Boot project for testing CRUD REST APIs using MockMvc. We've set up a `Person` entity, implemented the CRUD operations in the `PersonController`, and written JUnit tests in `PersonControllerTest.java` to validate the API endpoints. Proper testing ensures the reliability and correctness of your RESTful services.

---

[![Coffee Time!](/assets/img/blog/spring-coffee-3.jpg)](/assets/img/blog/spring-coffee-3.jpg)

Happy coding!



