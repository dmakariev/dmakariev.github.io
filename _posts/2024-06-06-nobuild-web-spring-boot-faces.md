---
title: Building a Web Application with Spring Boot and Jakarta Server Faces
tags: [spring boot, java, jbang, jsf, faces, jakarta, joinfaces]
thumbnail-img: "/assets/img/blog/docker-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

Creating a web application using Spring Boot and Jakarta Server Faces (JSF) combines the strengths of Spring Boot's rapid development capabilities with JSF's rich component-based UI framework. This tutorial will walk you through creating a simple CRUD (Create, Read, Update, Delete) application using these technologies.

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
Start by creating a Spring Boot project and add the necessary dependencies for Jakarta Server Faces (JSF). You can use the following dependencies in your `pom.xml`:
```xml
<dependencies>
    <!-- JoinFaces library to integrate JSF with Spring Boot -->
    <dependency>
        <groupId>org.joinfaces</groupId>
        <artifactId>faces-spring-boot-starter</artifactId>
        <version>5.3.0</version>
    </dependency>
    <!-- minimal CSS for styling -->
    <dependency>
        <groupId>org.mvnpm</groupId>
        <artifactId>simpledotcss</artifactId>
        <version>2.3.1</version>
    </dependency>    
</dependencies>
```

add to `application.properties`
```
joinfaces.faces-servlet.enabled=true
joinfaces.faces.automatic-extensionless-mapping=true
```

## Creating the Backing Bean, aka. the Controller
First, let's set up our project dependencies and configuration (used by JBang). We'll use the JoinFaces library to integrate JSF with Spring Boot. The [`simpledotcss`](https://simplecss.org/){:target="_blank"} library will provide minimal CSS for styling.

`FacesPersonBean.java`
{% highlight java %}

//DEPS org.joinfaces:faces-spring-boot-starter:5.3.0
//DEPS org.mvnpm:simpledotcss:2.3.1

//JAVA_OPTIONS -Djoinfaces.faces-servlet.enabled=true
//JAVA_OPTIONS -Djoinfaces.faces.automatic-extensionless-mapping=true
//FILES META-INF/resources/person-crud-faces.xhtml=person-crud-faces.xhtml

package com.makariev.examples.jbang;

import org.springframework.web.context.annotation.SessionScope;
import java.util.ArrayList;
import java.util.List;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.domain.PageRequest;
import org.springframework.stereotype.Component;
import jakarta.annotation.PostConstruct;
import jakarta.faces.context.FacesContext;
import jakarta.faces.application.FacesMessage;
import jakarta.faces.component.UIViewRoot;
import jakarta.faces.application.ViewHandler;
import java.io.Serializable;

@Component("personBean")
@SessionScope
@Getter
@Setter
public class FacesPersonBean implements Serializable {
    private final PersonRepository personRepository;
    private List<Person> persons;
    private Person formData;
    private boolean editMode;
    private Long editedPersonId;
    private int pageSize = 5;
    private int currentPage = 1;
    private long totalPages;

    public FacesPersonBean(PersonRepository personRepository) {
        this.personRepository = personRepository;
    }

    @PostConstruct
    public void init() {
        formData = new Person();
        loadPersons();
    }

    public void loadPersons() {
        var page = personRepository.findAll(PageRequest.of(currentPage - 1, pageSize));
        persons = page.getContent();
        totalPages = page.getTotalPages();
    }

    public void create() {
        formData = new Person();
        editMode = false;
        executeScript("showPersonDialog()");
    }

    public void save() {
        FacesContext facesContext = FacesContext.getCurrentInstance();
        List<FacesMessage> messages = new ArrayList<>();

        if (formData.getFirstName() == null || formData.getFirstName().isEmpty()) {
            messages.add(new FacesMessage(FacesMessage.SEVERITY_ERROR, "Validation Error", "First Name is required."));
        }

        if (formData.getLastName() == null || formData.getLastName().isEmpty()) {
            messages.add(new FacesMessage(FacesMessage.SEVERITY_ERROR, "Validation Error", "Last Name is required."));
        }

        if (formData.getBirthYear() == null || formData.getBirthYear() <= 0) {
            messages.add(new FacesMessage(FacesMessage.SEVERITY_ERROR, "Validation Error", "Birth Year must be a positive number."));
        }

        if (!messages.isEmpty()) {
            for (FacesMessage message : messages) {
                facesContext.addMessage(null, message);
            }
            return;
        }

        if (editMode) {
            Person person = personRepository.findById(editedPersonId).orElseThrow();
            person.setFirstName(formData.getFirstName());
            person.setLastName(formData.getLastName());
            person.setBirthYear(formData.getBirthYear());
            personRepository.save(person);
        } else {
            personRepository.save(formData);
        }

        loadPersons();
        formData = new Person();
        executeScript("closeDialog()");
    }

    public void edit(Person person) {
        this.formData = person;
        this.editMode = true;
        this.editedPersonId = person.getId();
        executeScript("showPersonDialog()");
    }

    public void delete(Person person) {
        personRepository.delete(person);
        loadPersons();
    }

    public void changePage(int page) {
        currentPage = page;
        loadPersons();
    }

    public List<Integer> getPageNumbers() {
        List<Integer> pageNumbers = new ArrayList<>();
        for (int i = 1; i <= totalPages; i++) {
            pageNumbers.add(i);
        }
        return pageNumbers;
    }

    private void executeScript(String script) {
        FacesContext facesContext = FacesContext.getCurrentInstance();
        facesContext.getPartialViewContext().getEvalScripts().add(script);
    }
}

{% endhighlight %}

This Java class, `FacesPersonBean`, is a Spring component with session scope. It manages the CRUD operations for `Person` entities. Let's break down its key parts:

* **Dependencies**: We include `JoinFaces` and `simpledotcss` libraries for JSF integration and minimal CSS styling.
* **SessionScope**: `@SessionScope` indicates that the bean's lifecycle is tied to the user's session.
* **PostConstruct** Initialization: The init method is annotated with `@PostConstruct` to initialize the `formData` and load the list of persons when the bean is created.
* **CRUD Methods**: Methods like `loadPersons`, `create`, `save`, `edit`, and `delete` handle the CRUD operations. These methods interact with the `PersonRepository` to perform database operations.
* **Pagination**: The `changePage` method and `getPageNumbers` provide pagination functionality.


`person-crud-faces.xhtml`
{% highlight html %}
<!DOCTYPE html [
    <!ENTITY nbsp "&#160;">  
    <!ENTITY copy "&#169;"> 
    <!ENTITY bull "&#8226;"> ]>
<html xmlns:f="jakarta.faces.core"
      xmlns:h="jakarta.faces.html"
      xmlns:p="jakarta.faces.passthrough"
      xmlns:ui="jakarta.faces.facelets"
      xmlns:jsf="jakarta.faces">
    <h:head>
        <title>Person CRUD Application</title>
        <h:outputStylesheet library="_static" name="simpledotcss/2.3.1/simple.css"/>
    </h:head>
    <h:body>
        <h:form id="mainForm">
            <header>
                <h1>Person CRUD Application</h1>
            </header>
            <main>
                <h:commandButton value="Add Person" action="#{personBean.create}">
                    <f:ajax execute="@this" render=":mainForm:personDialogContent" onevent="handleDialog"/>
                </h:commandButton>

                <table jsf:id="table">
                    <thead>
                        <tr>
                            <th>First Name</th>
                            <th>Last Name</th>
                            <th>Year of Birth</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                    <ui:repeat value="#{personBean.persons}" var="person">
                        <tr>
                            <td>#{person.firstName}</td>
                            <td>#{person.lastName}</td>
                            <td>#{person.birthYear}</td>
                            <td>
                        <h:commandButton value="Edit" action="#{personBean.edit(person)}">
                            <f:ajax execute="@this" render=":mainForm:personDialogContent" onevent="handleDialog"/>
                        </h:commandButton>
                        <h:commandButton value="Delete" action="#{personBean.delete(person)}" style="margin-left:10px">
                            <f:ajax execute="@this" render="@form"/>
                        </h:commandButton>
                        </td>
                        </tr>
                    </ui:repeat>
                    </tbody>
                </table>

                <nav style="display: flex; justify-content: center; margin-top: 20px;">
                    <ul style="display: flex; list-style: none; padding: 0;">
                        <ui:repeat value="#{personBean.pageNumbers}" var="page">
                            <li style="margin: 0 5px;">
                            <h:commandLink action="#{personBean.changePage(page)}" value="#{page}" style="#{personBean.currentPage == page ? 'font-weight:bold' : ''}">
                                <f:ajax execute="@this" render="@form"/>
                            </h:commandLink>
                            </li>
                        </ui:repeat>
                    </ul>
                </nav>
            </main>
            <footer>
                <p>&copy; 2024 Person CRUD Application. All rights reserved.</p>
            </footer>

            <dialog id="personDialog">
                <h:panelGroup id="personDialogContent">
                    <h2><h:outputText value="#{personBean.editMode ? 'Edit' : 'Add'} Person" /></h2>
                    <h:messages globalOnly="true" layout="table" id="personFormMessages" showDetail="true"/>
                    <fieldset>
                        <h:outputLabel for="firstName" value="First Name" />
                        <h:inputText id="firstName" value="#{personBean.formData.firstName}" p:placeholder="First Name"/>
                        <h:outputLabel for="lastName" value="Last Name" />
                        <h:inputText id="lastName" value="#{personBean.formData.lastName}" p:placeholder="Last Name"/>
                        <h:outputLabel for="birthYear" value="Year of Birth" />
                        <h:inputText id="birthYear" value="#{personBean.formData.birthYear}" type="number"  p:placeholder="Year of Birth"/>
                    </fieldset>
                    <menu>
                        <h:commandButton value="#{personBean.editMode ? 'Update' : 'Add'}" action="#{personBean.save}">
                            <f:ajax execute="@form" render="personFormMessages table"/>
                        </h:commandButton>
                        <h:commandButton value="Cancel" type="button" onclick="closeDialog(); return false;" style="margin-left:10px">
                        </h:commandButton>
                    </menu>
                </h:panelGroup>
            </dialog>
        </h:form>

        <script>
            function handleDialog(data) {
                if (data.status === 'success') {
                    showPersonDialog();
                }
            }

            function showPersonDialog() {
                document.getElementById('personDialog').showModal();
            }

            function closeDialog() {
                document.getElementById('personDialog').close();
            }
        </script>
    </h:body>
</html>

{% endhighlight %}

This XHTML file defines the user interface of our CRUD application. Let's break down its key parts:

* **Namespaces**: The xmlns attributes define the namespaces for JSF core, HTML, passthrough attributes, Facelets, and JSF components.
* **Header**: The h:head tag includes the page title and links to the CSS stylesheet.
* **Main Form**: The h:form tag wraps the entire form. Inside it, we have:
  * **Header**: Displays the application title.
  * **Main Section**: Contains the "Add Person" button, the table of persons, and pagination controls.
  * **Footer**: Displays a copyright message.
  * **Dialog**: A modal dialog for adding or editing persons.

## Conclusion
In this tutorial, we created a web application using **Spring Boot** and **Jakarta Server Faces**. We covered setting up the project, defining the backend logic, and creating the frontend UI. This combination provides a powerful framework for building modern web applications with rich user interfaces and robust backend support.

Happy coding!
