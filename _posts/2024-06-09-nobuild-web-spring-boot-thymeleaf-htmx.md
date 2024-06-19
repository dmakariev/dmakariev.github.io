---
title: Creating a Web Application with Thymeleaf and HTMX
tags: [spring boot, java, jbang, thymeleaf, htmx]
thumbnail-img: "/assets/img/blog/docker-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

In this blog post, we will create a web application using Spring Boot, Thymeleaf, and HTMX. We will build a CRUD (Create, Read, Update, Delete) application for managing a list of persons. This post will guide you through the code and explain each part to help you understand the functionality.

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
## Setting Up the Project
Start by creating a Spring Boot project and add the necessary dependencies for Thymeleaf. You can use the following dependencies in your `pom.xml`:
```xml
<dependencies>
    <!-- Other dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <!-- Other dependencies -->
</dependencies>
```

## Creating the Controller
Our controller, `HtmxPersonController`, will handle the CRUD operations and return the appropriate views. Here’s the complete code for the controller:

{% highlight java %}
package com.makariev.examples.jbang;

import org.springframework.web.bind.annotation.*;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.PageRequest;

@Controller
@RequestMapping("/person-crud-htmx")
@RequiredArgsConstructor
public class HtmxPersonController {

    private final PersonRepository personRepository;

    @GetMapping
    public String getPersonsPage(Model model) {
        Pageable pageable = PageRequest.of(0, 5);
        Page<Person> personPage = personRepository.findAll(pageable);

        model.addAttribute("persons", personPage.getContent());
        model.addAttribute("totalPages", personPage.getTotalPages());
        model.addAttribute("currentPage", 0);
        model.addAttribute("size", 5);

        return "person-crud-htmx";
    }

    @GetMapping("/htmx/list")
    public String findAll(@RequestParam(name = "page", defaultValue = "0") int page,
                          @RequestParam(name = "size", defaultValue = "5") int size, Model model) {
        Pageable pageable = PageRequest.of(page, size);
        Page<Person> personPage = personRepository.findAll(pageable);

        model.addAttribute("persons", personPage.getContent());
        model.addAttribute("totalPages", personPage.getTotalPages());
        model.addAttribute("currentPage", page);
        model.addAttribute("size", size);

        return "person-crud-htmx :: personRows";
    }

    @GetMapping("/htmx/pagination")
    public String getPagination(@RequestParam(name = "page", defaultValue = "0") int page,
                                @RequestParam(name = "size", defaultValue = "5") int size, Model model) {
        Pageable pageable = PageRequest.of(page, size);
        Page<Person> personPage = personRepository.findAll(pageable);

        model.addAttribute("totalPages", personPage.getTotalPages());
        model.addAttribute("currentPage", page);
        model.addAttribute("size", size);

        return "person-crud-htmx :: pagination";
    }

    @GetMapping("/htmx/form")
    public String showPersonForm(@RequestParam(name = "id", required = false) Long id, @RequestParam(name = "page", defaultValue = "0") int page, Model model) {
        Person person = id != null
                ? personRepository.findById(id).orElse(new Person())
                : new Person();

        model.addAttribute("person", person);
        model.addAttribute("editMode", id != null);
        model.addAttribute("currentPage", page);

        return "person-crud-htmx :: personForm";
    }

    @PostMapping("/htmx/create")
    public String createPerson(@ModelAttribute Person person,
                               @RequestParam(name = "page", defaultValue = "0") int page, Model model) {
        personRepository.save(person);
        return findAll(page, 5, model);
    }

    @PostMapping("/htmx/update")
    public String updatePerson(@ModelAttribute Person person,
                               @RequestParam(name = "page", defaultValue = "0") int page, Model model) {
        Person existingPerson = personRepository.findById(person.getId())
                .orElseThrow();
        existingPerson.setFirstName(person.getFirstName());
        existingPerson.setLastName(person.getLastName());
        existingPerson.setBirthYear(person.getBirthYear());
        personRepository.save(existingPerson);
        return findAll(page, 5, model);
    }

    @DeleteMapping("/htmx/{id}")
    public String deletePerson(@PathVariable("id") Long id, @RequestParam(name = "page", defaultValue = "0") int page, Model model) {
        personRepository.deleteById(id);
        return findAll(page, 5, model);
    }
}

{% endhighlight %}

Explanation of the Code
* **@Controller**: Indicates that this class is a Spring MVC controller.
* **@RequestMapping("/person-crud-htmx")**: Maps requests to /person-crud-htmx to this controller.
* **private final PersonRepository personRepository**: Injects the repository for CRUD operations.
* **@GetMapping**: Handles GET requests.
* **@PostMapping**: Handles POST requests.
* **@DeleteMapping**: Handles DELETE requests.
* **PageRequest.of(page, size)**: Creates a pageable object for pagination.
* **model.addAttribute("key", value)**: Adds attributes to the model to be used in the view.

## Designing the HTML Template
The Thymeleaf template, `person-crud-htmx.html`, will render the person list and handle dynamic updates using **HTMX**. Here’s the complete template:

{% highlight html %}
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Person CRUD Application</title>
    <link rel="stylesheet" href="https://cdn.simplecss.org/simple.min.css">
    <script src="https://unpkg.com/htmx.org@1.9.12/dist/htmx.js" integrity="sha384-qbtR4rS9RrUMECUWDWM2+YGgN3U4V4ZncZ0BvUcg9FGct0jqXz3PUdVpU1p0yrXS" crossorigin="anonymous"></script>
    <style>
        button + button {
            margin-left: 10px;
        }
    </style>
</head>
<body>
    <div id="app">
        <header>
            <h1>Person CRUD Application</h1>
        </header>
        <main>
            <button th:attr="hx-get=@{/person-crud-htmx/htmx/form(page=${currentPage})}" hx-target="#person-dialog" hx-trigger="click">Add Person</button>
            <table>
                <thead>
                    <tr>
                        <th>First Name</th>
                        <th>Last Name</th>
                        <th>Year of Birth</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody id="persons-list" th:fragment="personRows(persons, currentPage)">
                    <tr th:each="person : ${persons}">
                        <td th:text="${person.firstName}">First Name</td>
                        <td th:text="${person.lastName}">Last Name</td>
                        <td th:text="${person.birthYear}">Year of Birth</td>
                        <td>
                            <button th:attr="hx-get=@{/person-crud-htmx/htmx/form(id=${person.id},page=${currentPage})}" hx-target="#person-dialog" hx-trigger="click">Edit</button>
                            <button th:attr="hx-delete=@{/person-crud-htmx/htmx/{id}(id=${person.id})}" hx-swap="none" hx-trigger="click">Delete</button>
                        </td>
                    </tr>
                </tbody>
            </table>
            <nav id="pagination-nav" style="display: flex; justify-content: center; margin-top: 20px;" th:fragment="pagination(totalPages, currentPage, size)">
                <ul style="display: flex; list-style: none; padding: 0;">
                    <li th:each="pageNum : ${#numbers.sequence(0, totalPages - 1)}">
                        <a href="#" th:text="${pageNum + 1}"
                           th:attr="hx-get=@{/person-crud-htmx/htmx/list(page=${pageNum}, size=${size})}"
                           hx-target="#persons-list"
                           hx-swap="outerHTML"
                           hx-trigger="click"
                           th:style="${pageNum == currentPage} ? 'font-weight: bold;' : ''"></a>
                    </li>
                </ul>
            </nav>
        </main>
        <footer>
            <p>&copy; 2024 Person CRUD Application. All rights reserved.</p>
        </footer>

        <!-- Dialog -->
        <dialog id="person-dialog">
            <!-- Person Form Fragment -->
            <form th:fragment="personForm(editMode, currentPage, person)"
                  th:action="@{${editMode} ? '/person-crud-htmx/htmx/update' : '/person-crud-htmx/htmx/create'}"
                  th:attr="hx-post=@{${editMode} ? '/person-crud-htmx/htmx/update' : '/person-crud-htmx/htmx/create'}"
                  hx-target="#persons-list"
                  hx-swap="outerHTML"
                  method="post">
                
                <input type="hidden" th:if="${editMode}" th:value="${person?.id}" name="id"/>
                <input type="hidden" th:if="${editMode}" name="_method" value="put"/>
                <input type="hidden" name="page" th:value="${currentPage}"/>
                
                <h2 th:text="${editMode} ? 'Edit' : 'Add' + ' Person'"></h2>
                
                <fieldset>
                    <label for="firstName">First Name</label>
                    <input type="text" id="firstName" name="firstName" th:value="${person?.firstName}" placeholder="First Name" required>
                    
                    <label for="lastName">Last Name</label>
                    <input type="text" id="lastName" name="lastName" th:value="${person?.lastName}" placeholder="Last Name" required>
                    
                    <label for="birthYear">Year of Birth</label>
                    <input type="number" id="birthYear" name="birthYear" th:value="${person?.birthYear}" placeholder="Year of birth" required>
                </fieldset>
                
                <menu>
                    <button type="submit" th:text="${editMode} ? 'Update' : 'Add'"></button>
                    <button type="button" onclick="document.getElementById('person-dialog').close()">Cancel</button>
                </menu>
            </form>
        </dialog>
    </div>

    <script>
        document.body.addEventListener('htmx:afterSwap', (event) => {
            if (event.detail.target.id === "person-dialog") {
                document.getElementById('person-dialog').showModal();
            }
        });

        document.body.addEventListener('htmx:beforeRequest', (event) => {
            if (event.detail.elt.closest('#person-dialog')) {
                document.getElementById('person-dialog').close();
            }
        });
    </script>
</body>
</html>

{% endhighlight %}

Explanation of the HTML
* **th:attr="hx-get=@{/person-crud-htmx/htmx/form(page=${currentPage})}" hx-target="#person-dialog" hx-trigger="click"**: When the "Add Person" button is clicked, it sends an HTMX request to get the form fragment and displays it in the dialog.
* **th:each="person : ${persons}"**: Iterates over the list of persons.
* **th:text="${person.firstName}"**: Sets the text content to the person's first name.
* **hx-get, hx-delete**: HTMX attributes for sending GET and DELETE requests.
* **th:fragment="personRows(persons, currentPage)"**: Defines a Thymeleaf fragment for the table rows.
* **th:fragment="pagination(totalPages, currentPage, size)"**: Defines a Thymeleaf fragment for pagination.
* **htmx:afterSwap**: Shows the dialog after the form is loaded.
* **htmx:beforeRequest**: Closes the dialog before making a new request.

## Conclusion
By following this guide, you have created a dynamic web application using **Spring Boot**, **Thymeleaf**, and **HTMX**. This application allows you to perform CRUD operations with a modern and responsive user interface. Explore further by adding more features and enhancing the UI to suit your needs.  

Happy coding!
