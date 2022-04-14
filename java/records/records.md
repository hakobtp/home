# Java Record

```info
Author :        Ter-Petrosyan Hakob
Last Updated:   2022-04-14
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

Records automatically generate

- Immutable fields
- A canonical constructor
- An accessor method for each element
- The equals() method
- The hashCode() method
- The toString() method

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
> **NOTE**
> The Record provides access method without 'get' prefix  `e1.firstName()`

You cannot add fields to a record except by defining them in the header. However, static methods, fields, and initializers are allowed.

A record can define methods, but the methods can only read fields, which are automatically final:

```java
package com.htp.blog;

public record Employee(String firstName, String lastName, String email, double salary) {

    public void setFirstName(String firstName){
        // error: cannot assign a value to final variable firstName this.firstName = firstName;
        this.firstName = firstName; 
    }

    public void tryToChangeFirstName() {
        //error: cannot assign a value to final variable firstName this.firstName = firstName.toLowerCase();
        this.firstName = firstName.toLowerCase(); 
    }
}
```

You cannot inherit from a record because it is implicitly final (and cannot be abstract). In addition, a record cannot 
be inherited from another class. However, a record can implement an interface.

```java
package com.htp.blog;

interface IEmployee {

    String firstName();

    String fullName();
}

public record Employee(String firstName, String lastName, String email, double salary) implements IEmployee{

    @Override
    public String fullName() {
        return firstName + " " + lastName;
    }
}
```

The compiler forces you to provide a definition for fullName(), but it doesn’t complain about firstName(). 
That’s because the record automatically generates an accessor for its brightness argument, and that accessor fulfills the contract 
for firstName() in interface IEmployee.

A record can be nested within a class or defined locally within a method. Both nested and local uses of record are implicitly static.

You can add constructor behavior using a compact constructor, which looks like a constructor but has no parameter list.
The compact constructor is typically used to validate the arguments.

```java
package com.htp.blog;

import java.math.BigDecimal;
import java.util.Objects;

public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

    public Employee {
        Objects.requireNonNull(firstName);
        Objects.requireNonNull(lastName);
        Objects.requireNonNull(email);
        Objects.requireNonNull(salary);
    }
}
```

Record provide only a shallow immutability - just like the final fields they're using. If the parameter's type is not immutable itself 
(like List or Date), the record won't prevent modifying its value.

A compact constructor is the right place to ensure the proper immutability, by creating an immutable copy of such object:

```java
package com.htp.blog;

import java.util.List;
import java.util.UUID;

public record User(UUID uuid, List<String> roles) {

    public User {
        roles = List.copyOf(roles);
    }
}
```



The compact constructor's body is being invoked before the actual field values assignment

```java
public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

    public Employee {
        System.out.println("firstName : " + firstName()
                + "lastName : " + lastName()
                + "email : " + email()
                + "salary : " + salary());
    }
}
```

```log
firstName : null lastName : null email : null salary : null
```



> **NOTE**
> Although this seems as if final values are being modified, they are not. Behind the scenes, the compiler is creating an intermediate 
> placeholder for x and then performing a single assignment of the result to this.x at the end of the constructor.

It’s also possible to use the compact constructor to modify the initialization values for the fields.

```java
public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

    public Employee {
        //unlike the standard constructors, we can't use this to reference instance fields
        email = email.toLowerCase();
    }
}
```

You can replace the canonical constructor using normal constructor

```java
public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

    public Employee(String firstName, String lastName, String email, BigDecimal salary) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.salary = salary;
    }
}
```

Of course, the "old" syntax is still supported - records are classes too. They may also have multiple constructors, 
but they can't avoid calling full canonical constructor (initializing all the fields). That's why the code below causes a compilation failure:

```java
public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

    public Employee(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

```log
error: constructor is not canonical, so its first statement must invoke another constructor of class Employee
    public Employee(String firstName, String lastName) {
```


Additionally, an explicit canonical constructor is not allowed to have a more restrictive access level than the record itself.

```java

// package level access
record Employee(String firstName, String lastName, String email, BigDecimal salary) {

    private Employee(String firstName, String lastName, String email, BigDecimal salary) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.salary = salary;
    }
}
```

```log
error: invalid canonical constructor in record Employee
    private Employee(String firstName, String lastName, String email, BigDecimal salary) {
            ^
  (attempting to assign stronger access privileges; was package)
```

To copy a record, you must explicitly pass all fields to the constructor.

```java
public class App {

    public static void main(String[] args) {
        var gamer = new Employee("Gamer", "Simson", "gamer@mail.com", BigDecimal.valueOf(23));
        var gamerCopy = new Employee(gamer.firstName(), gamer.lastName(), gamer.email(), gamer.salary());
    }
}
```



[Home](./../../README.md) 
| [<< Java Tutorials](./../tutorials.md)




