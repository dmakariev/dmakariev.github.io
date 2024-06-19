---
title: Creating a Web Application with Spring Boot and Vue.js
tags: [spring boot, java, jbang, vuejs]
thumbnail-img: "/assets/img/blog/docker-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

Building modern web applications often involves creating a robust backend to handle data operations and a dynamic frontend to provide a smooth user experience. In this tutorial, we'll build a simple CRUD (Create, Read, Update, Delete) application using Spring Boot for the backend and Vue.js for the frontend. Our application will manage a list of persons, allowing you to add, edit, view, and delete person records.

## Related posts 
The series includes the following posts:

1. [Creating a Web Application with Spring Boot and Vue.js](https://makariev.com)
2. [Building a Web Application with Spring Boot and Jakarta Server Faces](https://makariev.com)
3. [Creating a Web Application with Thymeleaf and HTMX](https://makariev.com)
4. [Comparing Web Application Development with Spring Boot Using Vue.js, Jakarta Server Faces, and Thymeleaf/HTMX](https://makariev.com)

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

## Setting Up Spring Boot
We'll start by creating the backend using Spring Boot. Our backend will expose a REST API for managing person data. Let's break down the `VuePersonController.java` file to understand each part.

{% highlight java %}
package com.makariev.examples.jbang;

import org.springframework.web.bind.annotation.*;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import java.util.Optional;
import lombok.RequiredArgsConstructor;

@RestController
@RequestMapping("/api/persons")
@RequiredArgsConstructor
public class VuePersonController {

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

{% endhighlight %}

Explanation
* Annotations:

    * @RestController: Marks this class as a REST controller.
    * @RequestMapping("/api/persons"): Maps all requests to /api/persons to this controller.
    * @RequiredArgsConstructor: Generates a constructor with required arguments (i.e., final fields).

* Dependencies:

    * PersonRepository: An interface for CRUD operations on Person entities, typically extending JpaRepository.

* Methods:

    * findAll(Pageable pageable): Returns a paginated list of persons.
    * findById(Long id): Returns a specific person by ID.
    * create(@RequestBody Person person): Creates a new person.
    * updateById(Long id, @RequestBody Person person): Updates an existing person.
    * deleteById(Long id): Deletes a person by ID.

## #nobuild Vue.js
Next, we'll create the frontend using Vue.js. The person-crud-vue.html file contains the complete HTML and JavaScript needed for our Vue.js application.

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
    <title>Person CRUD Application</title>
    <link rel="stylesheet" href="https://cdn.simplecss.org/simple.min.css">
    <script src="https://cdn.jsdelivr.net/npm/vue@3.4.27/dist/vue.global.prod.js"></script>
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
            <button @click="showPersonModal(null)">Add Person</button>
            <table>
                <thead>
                    <tr>
                        <th>First Name</th>
                        <th>Last Name</th>
                        <th>Year of Birth</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="person in persons" :key="person.id">
                        <td>{{ person.firstName }}</td>
                        <td>{{ person.lastName }}</td>
                        <td>{{ person.birthYear }}</td>
                        <td>
                            <button @click="showPersonModal(person)">Edit</button>
                            <button @click="deletePerson(person.id)">Delete</button>
                        </td>
                    </tr>
                </tbody>
            </table>
            <nav style="display: flex; justify-content: center; margin-top: 20px;">
                <ul style="display: flex; list-style: none; padding: 0;">
                    <li v-for="page in totalPages" :key="page" @click="changePage(page)" style="margin: 0 5px;">
                        <a href="#" :style="{ fontWeight: currentPage === page ? 'bold' : 'normal', cursor: 'pointer' }">{ { page } }</a>
                    </li>
                </ul>
            </nav>
        </main>
        <footer>
            <p>&copy; 2024 Person CRUD Application. All rights reserved.</p>
        </footer>

        <!-- Dialog -->
        <dialog id="person-dialog">
            <h2>{{ editMode ? 'Edit' : 'Add' }} Person</h2>
            <form @submit.prevent="savePerson">
                <fieldset>
                    <label for="firstName">First Name</label>
                    <input type="text" id="firstName" v-model="formData.firstName" placeholder="First Name" required>
                    <label for="lastName">Last Name</label>
                    <input type="text" id="lastName" v-model="formData.lastName" placeholder="Last Name" required>
                    <label for="birthYear">Year of Birth</label>
                    <input type="number" id="birthYear" v-model="formData.birthYear" placeholder="Year of birth" required>
                </fieldset>
                <menu>
                    <button type="submit">{{ editMode ? 'Update' : 'Add' }}</button>
                    <button type="button" @click="closeModal">Cancel</button>
                </menu>
            </form>
        </dialog>
    </div>        

    <script>
        const { createApp, ref, computed } = Vue;

        createApp({
            data() {
                return {
                    persons: [],
                    modalVisible: false,
                    editMode: false,
                    formData: {
                        firstName: '',
                        lastName: '',
                        birthYear: ''
                    },
                    editedPersonId: null,
                    pageSize: 5,
                    currentPage: 1,
                    totalPages: 1
                };
            },
            methods: {
                getAllPersons(page) {
                    fetch('/api/persons?page=${page - 1}&size=${this.pageSize}')
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
                        this.formData = { ...person };
                    } else {
                        this.resetForm();
                    }
                    document.getElementById('person-dialog').showModal();
                },
                savePerson() {
                    if (this.editMode) {
                        fetch('/api/persons/${this.editedPersonId}', {
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
                    fetch('/api/persons/${personId}', {
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
                    document.getElementById('person-dialog').close();
                },
                resetForm() {
                    this.formData = {
                        firstName: '',
                        lastName: '',
                        birthYear: ''
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

{% endhighlight %}

Explanation
* HTML Structure:

    * Basic HTML setup with a link to SimpleCSS for minimal styling and Vue.js.
    * The main container with id="app" is the root element for our Vue.js application.
    * Header, main content, and footer sections provide structure and navigation.

* Vue.js App:

    * Data:
        * persons: Array to store person data.
        * modalVisible, editMode, formData, etc.: Variables to manage the state of the modal form and pagination.
    * Methods:
        * getAllPersons(page): Fetches persons data from the backend and updates the persons array and totalPages.
        * showPersonModal(person): Displays the modal for adding or editing a person.
        * savePerson(): Sends a POST or PUT request to save or update person data.
        * deletePerson(personId): Sends a DELETE request to remove a person.
        * closeModal(), resetForm(), changePage(page): Utility methods to handle form and pagination actions.
    * Lifecycle Hooks:
        * mounted(): Fetches initial data when the component is mounted.

## Bringing It All Together
With this setup, your Spring Boot application will serve the Vue.js frontend, and you can interact with your REST API to perform CRUD operations on person data.

## Related Posts
The series includes the following posts:
1. [Creating a Web Application with Spring Boot and Vue.js](https://makariev.com)
2. [Building a Web Application with Spring Boot and Jakarta Server Faces](https://makariev.com)
3. [Creating a Web Application with Thymeleaf and HTMX](https://makariev.com)
4. [Comparing Web Application Development with Spring Boot Using Vue.js, Jakarta Server Faces, and Thymeleaf/HTMX](https://makariev.com)

## Conclusion
This tutorial demonstrated how to create a simple CRUD application using Spring Boot for the backend and Vue.js for the frontend. By understanding the structure and functionality of each part, you can expand and customize the application to fit your needs. 

Happy coding!
