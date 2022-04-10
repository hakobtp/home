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

## Record example



[Home](./../../README.md) 
| [<< Java Tutorials](./../tutorials.md)




