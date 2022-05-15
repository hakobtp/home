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

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

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

Let’s declare a projection interface for the Course and Student entities as shown below.

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
---

`Student View`

```java
public interface StudentView {

    String getEmail();

    String getFirstName();

    StudentAddressView getAddress();

    List<StudentCourseView> getCourses();

    interface StudentCourseView {
        String getTitle();

        Integer getNumber();

    }

    interface StudentAddressView {

        String getZipCode();
    }
}
```

```java

public interface StudentRepository extends JpaRepository<Student, Long> {

    Optional<StudentView> getByEmail(String email);
}
```


Ok let's summarize in the course repository  we have the getByNumber method which should return CourseView  and 
the getByAuthor which should return CurseAuthorStudentView.

In the student repository we have getByEmail mehtod which should return StudentView.

Let's write some tests for our repository

`test application.properties`

```properties
jdbc.driverClassName=org.h2.Driver
jdbc.url=jdbc:h2:mem:myDb;DB_CLOSE_DELAY=-1

hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.show_sql=true
hibernate.hbm2ddl.auto=create

hibernate.cache.use_second_level_cache=false
hibernate.cache.use_query_cache=false
```

`insert_data.sql`
```sql
INSERT INTO student (id, email, first_name, last_name)
VALUES(1, 'gurgen@mail.com', 'Gurgen', 'Aloyan');

INSERT INTO address (id, student_id, state, city, street, zip_code)
VALUES(1, 1, 'AR', 'Armavir', 'Test', '00001');

INSERT INTO course (id, author, description, number, title)
VALUES(1, 'James Gosling', 'course description', 1, 'Java');

INSERT INTO student_course (course_id, student_id)
VALUES(1,1);
```

`clean_up_data.sql`
```sql
DELETE FROM address;
DELETE FROM student_course;
DELETE FROM course;
DELETE FROM student;
```


`unit test for CourseRepository`

```java

@DataJpaTest
@Sql(scripts = "/insert_data.sql")
@Sql(scripts = "/clean_up_data.sql", executionPhase = AFTER_TEST_METHOD)
class CourseRepositoryTest {

    @Autowired
    private CourseRepository courseRepository;

    @Test
    void getByNumberTest() {
        var courseViewOptional = courseRepository.getByNumber(1);

        assertFalse(courseViewOptional.isEmpty());
        var courseView = courseViewOptional.get();

        assertEquals(courseView.getNumber(), 1);
        assertEquals(courseView.getTitle(), "Java");
        assertEquals(courseView.getAuthor(), "James Gosling");
    }

    @Test
    void getByAuthorTest() {
        var list = courseRepository.getByAuthor("James Gosling");

        assertFalse(list.isEmpty());
        assertEquals(list.size(), 1);

        var course = list.get(0);
        assertEquals(course.getNumber(), 1);
        assertEquals(course.getTitle(), "Java");

        var students = course.getStudents();
        assertFalse(students.isEmpty());
        assertEquals(students.size(), 1);

        var student = students.get(0);
        assertEquals(student.getEmail(), "gurgen@mail.com");
        assertEquals(student.getFirstName(), "Gurgen");
    }
}
```



### Open Projection:

## Class Based Projections
## Dynamic Projections

[Home](./../../README.md) 
| [<< Spring Data JPA](./tutorials.md)