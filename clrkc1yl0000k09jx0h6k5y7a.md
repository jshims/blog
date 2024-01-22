---
title: "Understanding Spring Boot @Transactional"
seoTitle: "Understanding Spring Boot @Transactional"
seoDescription: "Understanding Spring Boot @Transactional"
datePublished: Fri Jan 19 2024 07:42:19 GMT+0000 (Coordinated Universal Time)
cuid: clrkc1yl0000k09jx0h6k5y7a
slug: understanding-spring-boot-transactional
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/WE_Kv_ZB1l0/upload/96f26929b96f038c9ff4d397b7ab5f35.jpeg
tags: springboot, transactions

---

# What I learned

Today, I had the opportunity to review the concept of transactions and explore how the `@Transactional` annotation operates within a Spring Boot environment.

### What are transactions

A transaction is a unit of work that ensures either all of its operations are successful or none of them are. It is usually used on data sensitive operations in which the integrity of data depends on all operations being executed successfully. For instance, one can think of the transfer of money as a group of operations that need to be arranged as a unit of transaction. The money transfer process usually involves a series of operations, including:

* Withdraw from one account
    
* Transfer of money to another account
    
* Depositing money to the other account
    

In the mentioned scenario, for the transfer of money to take place, all three operations must successfully execute.

In Spring Boot, developers can utilize the `@Transactional` annotation to specify a function or functions to run as a transaction. Spring Boot employs aspect-oriented programming, a programming paradigm, to seamlessly implement transactions without requiring significant intervention from the developer.

### What is Aspect Oriented Programming (AOP)

AOP is a programming approach that helps to modularize a codebase. It is a pattern that groups different functionalities of code into “aspects” and reuses it whenever it is needed. This improves reusability of code and helps maintain a leaner codebase. For example logging is usually a function that a log of methods reuse. By following or using AOP principles, a codebase can define a function to run before and after certain functions are called in order to log the execution time of said functions.

[Spring Start Here](https://www.manning.com/books/spring-start-here?ar=true&lpse=A) succinctly summarized some concepts utilized in AOP:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705645815621/7302c5d1-eaae-4609-8c01-c2724d71ad6c.png align="center")

Terminologies used in AOP (reference: [Spring documentation](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html))

* Aspect: what code is being injected
    
* Advice: when is the code being injected.
    
* Pointcut: which methods are targeted for injection
    
* Join point: the trigger for aspects to run. In Spring it is always method invocation
    
* Target object: the object that contains the join point
    

Having a simple understanding of AOP will help in further understanding how `@Transactional` is used in Spring.

## `@Transactional`

*reference:* [*Spring documentation*](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)

The `@Transactional` annotation utilizes AOP to run a function or functions as a transaction. In the case of transactions:

* Aspect: starting and ending a transaction
    
* Advice: around the pointcut or target method, meaning before and after the pointcut is run
    
* Pointcut: the method that is to be invoked (transaction unit)
    
* Join point: method call
    
* Target object: usually a service component in a Spring Boot app
    

To illustrate a typical flow using the `@Transactional` annotation, let's refer to a diagram from the Spring documentation:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705645867841/2e852600-7efb-495b-915e-ded049e66abe.png align="center")

In code, an aspect using the `@Transactional` annotation will appear as follows:

```kotlin
val manager = TransactionManager.getInstance()

fun aspect(){
  try{
    manager.begin()
    target.invoke()
    manager.commit()
  }catch(e : Exception){
    manager.rollback()
  }
}
```

### Parameter `(readonly = true)`

As a Spring Boot developer, you've likely encountered numerous methods annotated with the `@Transactional(readOnly = true)` annotation. While exploring transaction concepts, I found myself intrigued by the advantages of the `readOnly` parameter and the reasons behind annotating a method specifically for read-only queries.

**Benefits**

* Performance
    
    Applying `@Transactional(readOnly=true)` provides several optimizations:
    
    1. No Flushing in Hibernate: No need to synchronize data between the persistence context and DBMS.
        
    2. No Dirty Checking: No need to detect changes in entity states (data changes).
        
    3. DBMS Optimizations: Potential optimizations, such as avoiding locking or reading from a read replica instead of a write-read instance (varies by DBMS).
        
* Clear Intent
    
    Other developers can easily understand that the operation is a read-only transaction. Additionally, the annotation prevents Create, Update, Delete (CUD) operations and discourages future additions of such operations.
    

**Drawbacks**

* Locking
    
    Transactions occupy a database connection, and in the case of large transactions or operational logic within the annotated method, the connection might hinder other attempts to interact with the database.
    

> <div data-node-type="callout">
> <div data-node-type="callout-emoji">💡</div>
> <div data-node-type="callout-text">Why not omit the <code>readonly</code> parameter for readonly operations?</div>
> </div>
> 
> After reading various resources the aforementioned performance and clear intent benefits were main reasons for using the annotation. Moreover, another performance benefit was caching. When reading from the same entity multiple times, the persistence context acts as a first-level cache, reducing the need for direct reads from the database.