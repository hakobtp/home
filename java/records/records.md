# Java Record

```info
Author :        Ter-Petrosyan Hakob
Last Updated:   2022-04-10
````

- [Immutable class](#immutable-class)
- [Record example](#record-example)

---

## Immutable class

Let's write an immutable class, for that we should  maintain the next rules

- private, final field for each piece of data
- getter for each field
- public constructor with a corresponding argument for each field
- equals method that returns true for objects of the same class when all fields match
- hashCode method that returns the same value when all fields match
- toString method that includes the name of the class and the name of each field and its corresponding value

```java
package com.htp.blog;

import java.util.Objects;

public class Employee {

    private final String firstName;
    private final String lastName;
    private final String email;
    private final double salary;

    public Employee(String firstName, String lastName, String email, double salary) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.salary = salary;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public String getEmail() {
        return email;
    }

    public double getSalary() {
        return salary;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employe = (Employee) o;
        return Double.compare(employe.salary, salary) == 0
                && Objects.equals(firstName, employe.firstName)
                && Objects.equals(lastName, employe.lastName)
                && Objects.equals(email, employe.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(firstName, lastName, email, salary);
    }

    @Override
    public String toString() {
        return "Employee{" +
                "firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                ", salary=" + salary +
                '}';
    }
}
```

Although modern IDEA helps us generate boilerplate code our class doesn't look pretty. Starting with Java 17 we can use 
Record.
## Record example

Java records were introduced with the intention to be used as a fast way to create data carrier classes, i.e. the 
classes whose objective is to simply contain data and carry it between modules, also known as POJOs 
(Plain Old Java Objects) and DTOs (Data Transfer Objects)

Let's rewrite our class via Record

```java
package com.htp.blog;

public record Employee(String firstName, String lastName, String email, double salary) {
}
```

Record provides
 
- private, final field for each piece of data
- access method  for each field
- public constructor with a corresponding argument for each field
- equals method that returns true for objects of the same record when all fields match
- hashCode method that returns the same value when all fields match
- toString method that includes the name of the record and the name of each field and its corresponding value

```java
package com.htp.blog;

public class App {
    public static void main(String[] args) {

        var e1 = new Employee("Gamer", "Simson", "gamer@mail.com", 1400);
        var e2 = new Employee("Gamer", "Simson", "gamer@mail.com", 1400);
        var e3 = new Employee("Bart", "Simson", "bart@mail.com", 0);


        System.out.println(e1);
        System.out.println(e2);
        System.out.println(e3);

        System.out.println("e1 equals e2 = " + e1.equals(e2));
        System.out.println("e1 equals e3 = " + e1.equals(e3));

        System.out.println("e1 hash code = " + e1.hashCode());
        System.out.println("e2 hash code = " + e2.hashCode());
        System.out.println("e3 hash code = " + e3.hashCode());

    }
}
```

```log
Employee[firstName=Gamer, lastName=Simson, email=gamer@mail.com, salary=1400.0]
Employee[firstName=Gamer, lastName=Simson, email=gamer@mail.com, salary=1400.0]
Employee[firstName=Bart, lastName=Simson, email=bart@mail.com, salary=0.0]
e1 equals e2 = true
e1 equals e3 = false
e1 hash code = -1706248431
e2 hash code = -1706248431
e3 hash code = 685814257
```

> The Record provides access method without 'get' prefix  ```e1.firstName()```

[Home](./../../README.md) 
| [<< Java Tutorials](./../tutorials.md)




