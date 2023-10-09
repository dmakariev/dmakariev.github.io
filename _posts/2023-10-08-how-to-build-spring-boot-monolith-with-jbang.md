---
title: How to Build a Spring Boot Monolith with JBang
tags: [tutorial, java, spring boot, jbang, docker, vue]
thumbnail-img: "/assets/img/blog/spring-monolith.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---


In this blog post, we will embark on an exciting journey of building a full-fledged Spring Boot monolith application implementing CRUD (Create, Read, Update, Delete) operations on a "Person" entity. We'll leverage JPA for data persistence, Swagger for API documentation, Postgres as our database, and Vue.js 3 for the front-end. All of this will be achieved using the simplicity of JBang in a single Java file!

If you are interested in building a Spring Boot monolith with JBang, you might also want to check out our previous article on [How to Build a Spring Boot Rest Api with JBang in a Single Java File](https://www.makariev.com/blog/how-to-build-spring-boot-rest-api-with-jbang-in-single-java-file/). In that post, we showed you how to use JBang to create a simple Rest web service that exposes a "Hello, World!" endpoint. We also explained how to use JBang features such as dependencies and scripts to simplify the development and execution of your Java application. 

# Prerequisites
Before we dive into the development process, ensure you have:
1. read the previous article [How to Build a Spring Boot Rest Api with JBang in a Single Java File](https://www.makariev.com/blog/how-to-build-spring-boot-rest-api-with-jbang-in-single-java-file/)
2. JBang installed on your system. You can install it from [JBang's official website](https://www.jbang.dev/download/).
3. Docker and Docker Compose installed for setting up the Postgres database.

You can clone the `https://github.com/dmakariev/examples` repository.
```bash
git clone https://github.com/dmakariev/examples.git
cd examples/jbang/spring-boot-jpa-vue
```

# Getting Started
Let's create the files for the Spring Boot Monolith. 
Follow these steps:

### Initialize a New Directory  
Create a new directory for your project and navigate to it using your terminal. Then, create :
* an empty JBang script file with a `.java` extension, e.g., `springbootJpaVue.java`.
* an empty file with `.html` extension for the Vue.js UI app, e.g., `index-fetch.html`.
* an empty `Dockerfile`
* an empty Docker Compose file `compose.yaml`


```bash
$ mkdir spring-boot-jpa-vue
$ cd spring-boot-jpa-vue
$ touch springbootJpaVue.java
$ touch index-fetch.html
$ touch Dockerfile
$ touch compose.yaml
```

### Write the Spring Boot Code  
Open the `springbootJpaVue.java` file in your favorite text editor or integrated development environment (IDE) and add the following code.

```java
//usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 21
//DEPS org.springframework.boot:spring-boot-dependencies:3.1.4@pom
//DEPS org.springframework.boot:spring-boot-starter-web
//DEPS org.springframework.boot:spring-boot-starter-data-jpa
//DEPS org.springframework.boot:spring-boot-starter-actuator
//DEPS com.h2database:h2
//DEPS org.postgresql:postgresql
//DEPS org.projectlombok:lombok
//DEPS org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0
//JAVA_OPTIONS -Dserver.port=8080
//JAVA_OPTIONS -Dspring.datasource.url=jdbc:h2:mem:person-db;MODE=PostgreSQL;
//JAVA_OPTIONS -Dspring.h2.console.enabled=true -Dspring.h2.console.settings.web-allow-others=true
//JAVA_OPTIONS -Dmanagement.endpoints.web.exposure.include=health,env,loggers
//FILES META-INF/resources/index.html=index-fetch.html
package com.makariev.examples.jbang;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.List;
import java.util.Optional;

import jakarta.persistence.Entity;
import jakarta.persistence.Table;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.RequiredArgsConstructor;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;


@SpringBootApplication
public class springbootJpaVue {

    public static void main(String[] args) {
        SpringApplication.run(springbootJpaVue.class, args);
    }

}

@Component
@RequiredArgsConstructor
class InitialRecords {

    private final PersonRepository personRepository;

    @EventListener(ApplicationReadyEvent.class)
    public void exercise() {

        if (personRepository.count() > 0) {
            return;
        }
        List.of(
                new Person(1L, "Ada", "Lovelace", 1815),
                new Person(2L, "Niklaus", "Wirth", 1934),
                new Person(3L, "Donald", "Knuth", 1938),
                new Person(4L, "Edsger", "Dijkstra", 1930),
                new Person(5L, "Grace", "Hopper", 1906),
                new Person(6L, "John", "Backus", 1924)
        ).forEach(personRepository::save);
    }
}

@RestController
class HiController {

    @GetMapping("/hi")
    public String sayHi(@RequestParam(required = false, defaultValue = "World") String name) {
        return "Hello, " + name + "!";
    }
}

@RestController
@RequestMapping("/api/persons")
@RequiredArgsConstructor
class PersonController {

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
    public void deleteById(@PathVariable("id") Long id) {
        personRepository.deleteById(id);
    }
}

@Data
@Entity
@Table(name = "person")
@NoArgsConstructor
@AllArgsConstructor
class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;
    private int birthYear;
}

interface PersonRepository extends JpaRepository<Person, Long> {
}

```

### Write the Vue.js Code  
Open the `index-fetch.html` file in your favorite text editor or integrated development environment (IDE) and add the following code.
{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html>
    <head>
        <title>Person CRUD Application</title>
        <style>
            body {
                font-family: -apple-system,"Segoe UI",Helvetica,Arial,sans-serif;
                margin: 0;
                padding: 0;
                background-color: #f5f5f5;
            }

            #app {
                max-width: 800px;
                margin: 0 auto;
                padding: 20px;
                background-color: #fff;
                box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
            }

            h1 {
                text-align: center;
                margin-bottom: 20px;
            }

            ul {
                list-style: none;
                padding: 0;
            }

            li {
                border: 1px solid #ddd;
                padding: 10px;
                margin: 10px 0;
                background-color: #fff;
                display: flex;
                justify-content: space-between;
                align-items: center;
            }

            button {
                padding: 5px 10px;
                background-color: #007bff;
                color: #fff;
                border: none;
                cursor: pointer;
            }

            form {
                display: flex;
                flex-direction: column;
            }

            input {
                margin: 5px 0;
                padding: 5px;
                border: 1px solid #ddd;
            }

            button[type="submit"] {
                background-color: #28a745;
            }

            .modal {
                position: fixed;
                z-index: 1;
                left: 0;
                top: 0;
                width: 100%;
                height: 100%;
                background-color: rgba(0, 0, 0, 0.4);
                justify-content: center;
                align-items: center;
            }

            .modal-content {
                background-color: #fff;
                border: 1px solid #ddd;
                padding: 20px;
                width: 70%;
                margin: 10% auto;
            }

            .pagination {
                display: flex;
                justify-content: center;
                margin-top: 20px;
            }

            .page-item {
                margin: 0 5px;
                cursor: pointer;
            }

            .page-item.active {
                font-weight: bold;
            }
        </style>
        <script src="https://cdn.jsdelivr.net/npm/vue@3.3.4/dist/vue.global.prod.js"></script>
    </head>
    <body>
        <div id="app">
            <h1>Person CRUD Application</h1>
            <button @click="showPersonModal(null)">Add Person</button>
            <ul>
                <li v-for="person in persons" :key="person.id">
                    {{ person.firstName }} {{ person.lastName }} ({{ person.birthYear }} year of birth)
                    <button @click="showPersonModal(person)">Edit</button>
                    <button @click="deletePerson(person.id)">Delete</button>
                </li>
            </ul>
            <div class="pagination">
                <span v-for="page in totalPages" :key="page" @click="changePage(page)" class="page-item" :class="{ active: currentPage === page }">{{ page }}</span>
            </div>

            <!-- Modal -->
            <div class="modal" v-if="modalVisible">
                <div class="modal-content">
                    <h2>{{ editMode ? 'Edit' : 'Add' }} Person</h2>
                    <form @submit.prevent="savePerson">
                        <input type="text" v-model="formData.firstName" placeholder="First Name" required>
                        <input type="text" v-model="formData.lastName" placeholder="Last Name" required>
                        <input type="number" v-model="formData.birthYear" placeholder="Year of birth" required>
                        <button type="submit">{{ editMode ? 'Update' : 'Add' }}</button>
                        <button @click="closeModal">Cancel</button>
                    </form>
                </div>
            </div>
        </div>        

        <script>
            const {createApp, ref, computed} = Vue;

            createApp({
                data() {
                    return {
                        persons: [],
                        modalVisible: false,
                        editMode: false,
                        formData: {
                            firstName: '',
                            lastName: '',
                            age: ''
                        },
                        editedPersonId: null,
                        pageSize: 5,
                        currentPage: 1,
                        totalPages: 1
                    };
                },
                methods: {
                    getAllPersons(page) {
                        fetch(`/api/persons?page=${page - 1}&size=${this.pageSize}`)
                                .then(response => response.json())
                                .then(data => {
                                    this.persons = data.content;
                                    this.totalPages = data.totalPages;
                                })
                                .catch(error => {
                                    console.error('Error fetching persons:', error);
                                });
                    },
                    showPersonModal(person) {
                        this.editMode = !!person;
                        this.modalVisible = true;
                        if (person) {
                            this.editedPersonId = person.id;
                            this.formData = {...person};
                        } else {
                            this.resetForm();
                        }
                    },
                    savePerson() {
                        if (this.editMode) {
                            fetch(`/api/persons/${this.editedPersonId}`, {
                                method: 'PUT',
                                headers: {
                                    'Content-Type': 'application/json'
                                },
                                body: JSON.stringify(this.formData)
                            })
                                    .then(() => {
                                        this.getAllPersons(this.currentPage);
                                        this.closeModal();
                                    })
                                    .catch(error => {
                                        console.error('Error updating person:', error);
                                    });
                        } else {
                            fetch('/api/persons', {
                                method: 'POST',
                                headers: {
                                    'Content-Type': 'application/json'
                                },
                                body: JSON.stringify(this.formData)
                            })
                                    .then(() => {
                                        this.getAllPersons(this.currentPage);
                                        this.closeModal();
                                    })
                                    .catch(error => {
                                        console.error('Error adding person:', error);
                                    });
                        }
                    },
                    deletePerson(personId) {
                        fetch(`/api/persons/${personId}`, {
                            method: 'DELETE'
                        })
                                .then(() => {
                                    this.getAllPersons(this.currentPage);
                                })
                                .catch(error => {
                                    console.error('Error deleting person:', error);
                                });
                    },
                    closeModal() {
                        this.modalVisible = false;
                        this.editMode = false;
                        this.resetForm();
                    },
                    resetForm() {
                        this.formData = {
                            firstName: '',
                            lastName: '',
                            age: ''
                        };
                        this.editedPersonId = null;
                    },
                    changePage(page) {
                        this.currentPage = page;
                        this.getAllPersons(page);
                    }
                },
                mounted() {
                    this.getAllPersons(this.currentPage);
                }
            }).mount('#app');
        </script>
    </body>
</html>
{% endraw %}
{% endhighlight %}

### Write the Dockerfile 
Open the `Dockerfile` file in your favorite text editor and add the following code.
```docker
FROM public.ecr.aws/docker/library/amazoncorretto:21-alpine AS build

RUN apk --no-cache add bash
RUN apk --no-cache add curl
RUN mkdir /app
WORKDIR /app
COPY . /app

RUN curl -Ls https://sh.jbang.dev | bash -s - export portable springbootJpaVue.java

FROM public.ecr.aws/docker/library/amazoncorretto:21-alpine
RUN mkdir /app/
RUN mkdir /app/lib
COPY --from=build /app/springbootJpaVue.jar /app/springbootJpaVue.jar
COPY --from=build /app/lib/* /app/lib/
WORKDIR /app

ENTRYPOINT ["java","-jar","springbootJpaVue.jar"]
```

### Write the Docker Compose file
Open the `compose.yaml` file in your favorite text editor and add the following code.
```yaml
services:
  backend:
    build: .
    ports:
      - 8080:8088
    environment:
      - SERVER_PORT=8088
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/example
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=pass-example
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
    networks:
      - spring-postgres
  db:
    image: postgres
    restart: always
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - spring-postgres
    environment:
      - POSTGRES_DB=example
      - POSTGRES_PASSWORD=pass-example
    expose:
      - 5432
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin_not_used@user.com
      PGADMIN_DEFAULT_PASSWORD: admin_not_used
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    volumes:
       - pgadmin:/var/lib/pgadmin
    ports:
      - "5050:80"
    networks:
      - spring-postgres
    restart: always      
volumes:
  db-data:
  pgadmin:
networks:
  spring-postgres:
```


# Run the Application
We have created the Spring Boot Monolith application. It consists of exactly two source files and two configuration files for docker. 
* `springbootJpaVue.java` is the backend, implemented as Spring Boot Java application, it contains also some default values
* `index-fetch.html` is the frontend, implemented with Vue.js as [standalone script](https://vuejs.org/guide/extras/ways-of-using-vue.html#standalone-script)

The way the two files are related is with this JBang directive  
```java
//FILES META-INF/resources/index.html=index-fetch.html
```

The application has a single jpa entity `Person` that could be stored in a database.

Return to your terminal. Navigate to the directory containing your `springbootJpaVue.java`

The application could be configured to run with one of two databases 
*  H2 Database in memory 
```bash
$ jbang -Dspring.datasource.url=jdbc:h2:mem:person-db \
 springbootJpaVue.java
```
*  H2 Database filesystem - database data is stored in file 
```bash
$ jbang -Dspring.datasource.url=jdbc:h2:file:./person-db-data \
-Dspring.jpa.hibernate.ddl-auto=update \ 
springbootJpaVue.java
```
* Postgres, it needs localhost instance of Postgres 
```bash
$ jbang -Dspring.datasource.url=jdbc:postgresql://localhost:5432/example \
-Dspring.datasource.username=postgres \
-Dspring.datasource.password=postgres \
-Dspring.jpa.hibernate.ddl-auto=update springbootJpaVue.java
```

 to run it with default settings file and execute any the following commands:

```bash
$ jbang springbootJpaVue.java
```
```bash 
$ sh springbootJpaVue.java
```
if you allow executable permissions for `springbootJpaVue.java` by executing 
```bash 
$ chmod +x springbootJpaVue.java
```
you could even execute the application like that
```bash 
$ ./springbootJpaVue.java
```
you could build a `fatJar` 
```bash 
$ jbang export fatjar springbootJpaVue.java
```
and then run it as 
```bash
$ jbang springbootJpaVue-fatjar.jar
```
or like normal java application 
```bash
$ java -jar springbootJpaVue-fatjar.jar
```
you could create a `portable` jar file with `./lib` folder containing all dependencies
```bash
$ jbang export portable springbootJpaVue.java
```
and then run it as 
```bash
$ jbang springbootJpaVue.jar
```
or like normal java application 
```bash
$ java -jar springbootJpaVue.jar
```
docker compose
```bash 
$ docker compose up
```
In all of the cases above, JBang will download the required Spring Boot dependencies and start the application. You will see output indicating that the Spring Boot application is running.

# Access the Application
 
You could access the UI at `http://localhost:8080/`
[![Person CRUD list](/assets/img/blog/person-crud-list.png)](/assets/img/blog/person-crud-list.png)

[![Person CRUD Create](/assets/img/blog/person-crud-create.png)](/assets/img/blog/person-crud-create.png)

[![Person CRUD Update](/assets/img/blog/person-crud-update.png)](/assets/img/blog/person-crud-update.png)

---

The application exposes several additional features
* the H2 Console Application, that lets you access a SQL database using a browser interface. You could access it at `http://localhost:8080/h2-console`
[![H2 Console](/assets/img/blog/person-crud-h2-console.png)](/assets/img/blog/person-crud-h2-console.png)

---

* OpenAPI definition. You could access it at `http://localhost:8080/v3/api-docs` 
[![OpenAPI](/assets/img/blog/person-crud-openapi.png)](/assets/img/blog/person-crud-openapi.png)

---

* the Swagger UI. You could access it at `http://localhost:8080/swagger-ui/index.html` 
[![OpenAPI](/assets/img/blog/person-crud-swagger-ui.png)](/assets/img/blog/person-crud-swagger-ui.png)

---

* the Spring Boot actuator endpoints. You could access it at `http://localhost:8080/actuator`  
[![actuator](/assets/img/blog/person-crud-actuator.png)](/assets/img/blog/person-crud-actuator.png)

---

* When executed with docker compose, the application provides access to web version of PgAdmin, that lets you access a SQL database using a browser interface.
You could access it at `http://localhost:5050/`
[![PgAdmin](/assets/img/blog/person-crud-pgadmin.png)](/assets/img/blog/person-crud-pgadmin.png)

[![PgAdmin prop](/assets/img/blog/person-crud-pgadmin-prop.png)](/assets/img/blog/person-crud-pgadmin-prop.png)

---

### Access the Application from Terminal/CLI
Open your web browser and navigate to `http://localhost:8080/hi`. You should see the **"Hello, World!"** message displayed in your browser.
Or if you prefer more personalized message, then navigate to `http://localhost:8080/hi?name=Joe`. You should see the **"Hello, Joe!"** message displayed in your browser.

### Using the Rest Api with Curl 

To create a new person, use the POST method with the person data as a JSON body:
```bash
$ curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"Katherine", "lastName":"Johnson", "birthYear":1919}' \
http://localhost:8080/api/persons
```

To get a list of all persons, use the GET method:
```bash
$ curl -X GET http://localhost:8080/api/persons
```

To get a specific person by id, use the GET method with the id as a path variable:
```bash
$ curl -X GET http://localhost:8080/api/persons/1
```

To update an existing person by id, use the PUT method with the person data as a JSON body:
```bash
$ curl -X PUT -H "Content-Type: application/json" \
-d '{"firstName":"Katherine", "lastName":"Johnson", "birthYear":1918}' \
http://localhost:8080/api/persons/1
```

### Using the Rest Api with HTTPIE
you could download alternative Terminal/CLI client from here [https://httpie.io/cli](https://httpie.io/cli) 

To create a new person, use the POST method with the person data as a JSON body:
```bash
$ http POST http://localhost:8080/api/persons firstName=Alice lastName=Smith birthYear=1996
```

To get a list of all persons, use the GET method:
```bash
$ http GET http://localhost:8080/api/persons
```

To get a specific person by id, use the GET method with the id as a path variable:
```bash
$ http GET http://localhost:8080/api/persons/1
```

To update an existing person by id, use the PUT method with the person data as a JSON body:
```bash
$ http PUT http://localhost:8080/api/persons/1 firstName=Bob lastName=Jones birthYear=1990
```

To delete an existing person by id, use the DELETE method with the id as a path variable:
```bash
$ http DELETE http://localhost:8080/api/persons/1
```


# Details About the Implementation 
To enable JPA, which is the Java/Jakarta Persistence API, we need 
```java
//DEPS org.springframework.boot:spring-boot-starter-data-jpa:3.1.4
```
we also need a database, so we will add dependency for H2 Database
the section becomes
```java
//DEPS org.springframework.boot:spring-boot-starter-web:3.1.4
//DEPS org.springframework.boot:spring-boot-starter-data-jpa:3.1.4
//DEPS com.h2database:h2:2.2.224
```
to minimize the boilerplate code, we'll add also Lombok
```java
//DEPS org.springframework.boot:spring-boot-starter-web:3.1.4
//DEPS org.springframework.boot:spring-boot-starter-data-jpa:3.1.4
//DEPS com.h2database:h2:2.2.224
//DEPS org.projectlombok:lombok:1.18.30
``` 
JBang supports importing of `.pom` files, 
let's change the dependencies to 
```java
//DEPS org.springframework.boot:spring-boot-dependencies:3.1.4@pom
//DEPS org.springframework.boot:spring-boot-starter-web
//DEPS org.springframework.boot:spring-boot-starter-data-jpa
//DEPS com.h2database:h2
//DEPS org.projectlombok:lombok
```
as you can see the dependency versions are removed, the Spring Boot version is defined only once. 

### Persistence : `Person` Entity and Repository
This is the JPA entity and the data repository 
```java
@Data
@Entity
@Table(name = "person")
@NoArgsConstructor
@AllArgsConstructor
class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;
    private int birthYear;
}

interface PersonRepository extends JpaRepository<Person, Long> {
}
```

### Rest Api:  `PersonController`
This is the rest controller 
```java
@RestController
@RequestMapping("/api/persons")
@RequiredArgsConstructor
class PersonController {

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
    public void deleteById(@PathVariable("id") Long id) {
        personRepository.deleteById(id);
    }
}
``` 

### OpenAPI Support and Enable Swagger UI
We are using the `springdoc` project. To enable it, all we have to do is add the following dependency 
```java
//DEPS org.springdoc:springdoc-openapi-starter-webmvc-ui:2.2.0
```
after restarting the application, you are going to get the **Swagger UI**  at the following URL `http://localhost:8080/swagger-ui/index.html`

### Enable H2 Console Application
The H2 Console Application lets you access a SQL database using a browser interface.
To activate it we need to add the following configuration right after the dependency section 
```java
//JAVA_OPTIONS -Dspring.h2.console.enabled=true
//JAVA_OPTIONS -Dspring.h2.console.settings.web-allow-others=true
//JAVA_OPTIONS -Dspring.datasource.url=jdbc:h2:mem:person-db;MODE=PostgreSQL;
```


# Conclusion
In this blog post, we demonstrated how to create a Spring Boot Monolith using just a single Java file for the backend, single HTML file for the frontend and JBang. This approach can be handy for quick prototyping, lightweight applications, or when you want to reduce the complexity of your development environment. As your application grows in complexity, you can always transition to a more traditional project structure. JBang provides a flexible and efficient way to develop Java applications without the need for heavyweight project setups.

---

[![Coffee Time!](/assets/img/blog/spring-monolith.jpg)](/assets/img/blog/spring-monolith.jpg)

---

Explore further and build even more sophisticated Spring Boot applications using JBang. Happy coding!
