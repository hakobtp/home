# Spring Data JPA projections

```info
Author :        Ter-Petrosyan Hakob
Last Updated:   2022-04-29
```

- [Overview](#overview)
- [Project setup](#project-setup)
- [Create entities](#create-entities)

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

![student-course_relationship](./assets/spring-jpa-projection/spring-%20jpa-projections.png)

<!-- gradle -->


<!-- To show how it works let's create two entities. -->

## A entity

## B entity

[Home](./../../README.md) 
| [<< Spring Data JPA](./tutorials.md)