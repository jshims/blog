---
title: "Understanding Java Annotation And Reflection"
datePublished: Fri Apr 12 2024 00:24:36 GMT+0000 (Coordinated Universal Time)
cuid: cluvxelyl001b08l92d09ef1k
slug: understanding-java-annotation-and-reflection
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/WE_Kv_ZB1l0/upload/624a772c513aa604d163659e4c0d9ffc.jpeg
tags: spring, java, kotlin, springboot, annotations, reflection

---

As a Spring developer, I found myself needing to brush up on Java annotations and Reflection. While working on tasks like data validation and JPA, I realized I was using these features without a solid grasp of the underlying concepts. To clear up my confusion, I delved into documentation and blog posts by fellow programmers. Here, I'll share what I've learned to help anyone else navigating these concepts.

Let's dive in!

---

# Java Annotation

## What is it?

In Java, an annotation serves as metadata—a way to provide additional information about code elements. For example, in the snippet below, `@field:NotBlank()` describes the `name` variable as one that should not be blank.

```kotlin
data class UserInfo(
  @field:NotBlank(message = "user name cannot be blank")
  val name: String
)
```

As a Java or Kotlin developer, you've likely encountered various annotations like `@Override`, `@Autowired`, `@Size`, and more. Each serves to describe aspects of classes, methods, fields, and so on.

## How are they used?

Java annotations are used at either compilation or at runtime.

### Compile Time

During compilation, annotations can instruct the compiler to perform specific actions, such as checking code correctness or generating code. For instance, `@Override` prompts the compiler to ensure a class properly overrides its superclass method.

```java
class Animal {
  public void eat() {}
}

class Dog extends Animal {
  @Override
  public void eat() {}
}
```

Similarly, annotations like `@Getter` and `@Setter` from Project Lombok automatically generate getter and setter methods.

```java
public class Person {
  @Getter @Setter private String name;
}
```

Compile-time annotation processing involves the compiler checking annotations in the source code before producing bytecode. During this process the compiler checks if there are annotation processors inside the source code and runs the logic within it. I haven’t created a custom processor before so I have to look up the specifics but if you are interested check out [Abstract Processer documentation](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/AbstractProcessor.html).

### Runtime

Annotations can also be used at runtime. At runtime, annotations are processed using a feature called Reflection. We will look into reflection at the second section of this post. For now, think of Reflection as a feature allowing the runtime environment to check annotations in a class and modify execution if needed.

For instance, the `@Size` annotation validates payload data in a Spring application. At runtime, the environment checks for the annotation, validates the data, and proceeds accordingly.

```kotlin
data class UserInfo(
  @field:Size(min=0, max=10, message = "name length must be 0~10")
  val name: String
)
```

Similarly, the `@Autowired` annotation facilitates dependency injection in Spring applications. On startup, the application scans for annotated fields, checks for corresponding beans in the application context, and injects dependencies as needed.

```java
@Service
public class UserService{
  @Autowired private UserRepository repository;
}
```

For custom annotations you must declare a method that knows how or what to do for its corresponding annotation. I haven’t created one before so the specifications should be checked if you are interested.

## Why does it exist?

Annotations serve several purposes, including providing additional compiler instructions, enhancing code readability by providing metadata directly in source code (rather than XML), and reducing boilerplate code. For other reasons please view the [JSR 308 Explained: Java Type Annotations](https://www.oracle.com/technical-resources/articles/java/ma14-architect-annotations.html) post from Oracle.

---

# Java Reflections

## What is it?

Java Reflection is a powerful feature in Java programming that allows a program to examine and modify its own structure and behavior at runtime. This capability is used in but not limited to:

* **Introspection**: Examining classes to discover fields, methods, annotations, and more.
    
* **Dynamic Instance Creation**: Instantiating classes dynamically during runtime.
    
* **Dynamic Invocation**: Invoking methods during runtime.
    

## How is it used?

There are numerous ways Reflection is used. In this post I will share a few scenarios on how it is used in Spring.

### Introspection → Dependency Injection

Java Reflection’s ability to introspect a class is used in Dependency Injection (`@Autowired`). During the startup process, Reflection is used to inspect classes for fields or constructors annotated with `@Autowired` and injects the appropriate bean at runtime.

```java
@Service
public class UserService{
  @Autowired private UserRepository repository;
}
```

### Dynamic Instance Creation → Component Scanning

Java Reflection’s ability to dynamically create instances at runtime is used in component scanning. During the startup process, Spring scans classes annotated with `@Component`, `@Service`, `@Repository`, and other similar annotations. When a class is identified as a component, Spring dynamically creates an instance of the class and registers it as a bean in the application context.

```java
@ComponentScan(basePackages = "com.example.app")
public class AppApplication {
  public static void main(String[] args) {
    SpringApplication.run(AppApplication.class, args);
  }
}
```

### Dynamic Invocation → AOP

Java Reflection’s ability to dynamically invoke methods is used in Aspect Oriented Programming (AOP). For instance, let’s look at `@Transactional`. During the startup process, Spring looks for methods (pointcuts) and creates proxies for them. These proxies are responsible for intercepting method calls to an object and running certain logic (aspect) based on the advice. In the case of `@Transactional`, dynamic invocation will work like:

1. A `@Transactional` method is invoked.
    
2. Proxy intercepts the request and starts a transaction.
    
3. Proxy dynamically invokes the target method.
    
4. Proxy commits or rollbacks the transaction.
    

## Why does it exist?

Java Reflection exists to bring flexible and dynamic development. It supports advanced capabilities that enable frameworks and libraries to implement features like dependency injection, component scanning, and AOP features seamlessly. However, it does have its drawbacks. Reflection should be used with an understanding of its potential performance and security drawbacks.

This post was intended to review what and how annotations and reflections are used in the background of Spring development. So I won’t go into the drawbacks but, for those considering using Reflection directly, I would recommend diving deeper into its implications and best practices.

---

Thank you for reading! I hope this post has made it clearer how annotations and reflections operate in the background when you're working with Spring.