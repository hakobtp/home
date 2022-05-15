# Spring Data JPA projections

```info
Author :        Ter-Petrosyan Hakob
Last Updated:   2022-05-15
```

- [Overview](#overview)
- [Project setup](#project-setup)
- [Create entities](#create-entities)
  - [Student entity](#student-entity)
  - [Course entity](#course-entity)
  - [Address entity](#address-entity)
- [Create repository](#create-repositories)
  - [Student repository](#student-repository)
  - [Course repository ](#course-repository)
  - [Address repository ](#address-repository)
- [Interface Based Projections](#interface-based-projections)  
  - [Close Projection](#close-projection)
  - [Open Projection](#open-projection)
- [Class Based Projections](#class-based-projections)
- [Dynamic Projections](#dynamic-projections)

---

<!-- 

write test
interface projection
close
open
class projection
dynamic projection -->


## Overview

Spring JPA gives us the ability via repository easy work with our data but the repository returned all entity representations, 
sometimes we don't need all representations, in this case, we can use projections. 


## Project setup

For demo purposes let's create a simple project:

```groovy
plugins {
  id 'org.springframework.boot' version '2.6.7'
  id 'io.spring.dependency-management' version '1.0.11.RELEASE'
  id 'java'
}

group = 'com.htp'
version = '0.0.1'
sourceCompatibility = '17'

repositories {
  mavenCentral()
}

ext {
  set('testcontainersVersion', "1.16.2")
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  runtimeOnly 'org.postgresql:postgresql'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'org.testcontainers:junit-jupiter'
}

dependencyManagement {
  imports {
    mavenBom "org.testcontainers:testcontainers-bom:${testcontainersVersion}"
  }
}

tasks.named('test') {
  useJUnitPlatform()
}
```

For the integration test, I will use the testcontainer if you want to know much about the testcontainer please visit the 
[testcontainer's official site](https://www.testcontainers.org/){:target="\_blank"} 
or read my article [testcontainer introduction](../../test/testcontainer/testcontainer-introduction.md).

## Create entities

To make things simple let's create three simple entities

![student-course_relationship](./assets/spring-jpa-projection/spring-%20jpa-projections.png)


### Student entity

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    private String firstName;
    private String lastName;
    private String email;

    @ManyToMany(mappedBy = "students", fetch = FetchType.EAGER)
    public List<Course> courses = new ArrayList<>();

    @OneToOne(mappedBy = "student")
    private Address address;

    // getter and setter

}    

```

### Course entity

```java
@Entity
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    private String title;
    private String description;
    private Integer number;
    private String author;

    @ManyToMany(cascade = {
            CascadeType.PERSIST,
            CascadeType.MERGE
    })
    @JoinTable(
            name = "student_course",
            joinColumns = @JoinColumn(name = "course_id", referencedColumnName = "id"),
            inverseJoinColumns = @JoinColumn(name = "student_id", referencedColumnName = "id")
    )
    private List<Student> students = new ArrayList<>();

    // getter and setter

}  
```

### Address entity

```java
@Entity
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    @OneToOne
    private Student student;

    private String state;

    private String city;

    private String street;

    private String zipCode;
}

    // getter and setter
```    

## Create repositories

Let's create repositories for our entities.

### Student repository

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
}

```

### Course repository 

```java
public interface CourseRepository extends JpaRepository<Course, Long> {
}
```

### Address repository

```java
public interface AddressRepository extends JpaRepository<Address, Long> {
}
```

## Interface Based Projections

In this type, we create an interface with only getter methods of properties we want from an entity class. 
This interface will be the return type of query method we write in repository.

### Close Projection

In Close Projection, the getter methods of interface match exactly with the getter methods of Entity’s properties. 

Let’s declare a projection interface for the Course and Student Entits as shown below.

`Course view`

```java

public interface CourseView {

    String getTitle();

    Integer getNumber();

    String getAuthor();
}
```

```java
public interface CurseAuthorStudentView {

    String getTitle();

    Integer getNumber();

    List<CourseStudentView> getStudents();

    interface CourseStudentView {

        String getFirstName();

        String getEmail();
    }
}
```

Let's add two methods to our course repository

```java
public interface CourseRepository extends JpaRepository<Course, Long> {

    Optional<CourseView> getByNumber(Integer number);

    List<CurseAuthorStudentView> getByAuthor(String author);
}

```

Ok let's summarize in the course repository  we have the getByNumber method which should return CourseView  and 
the getByAuthor which should return CurseAuthorStudentView.



### Open Projection:

## Class Based Projections
## Dynamic Projections

[Home](./../../README.md) 
| [<< Spring Data JPA](./tutorials.md)