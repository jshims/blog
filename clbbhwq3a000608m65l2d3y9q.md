---
title: "What is Dependency Inversion Principle?"
seoTitle: "DIP"
seoDescription: "dependency inversion principle"
datePublished: Tue Dec 06 2022 00:40:59 GMT+0000 (Coordinated Universal Time)
cuid: clbbhwq3a000608m65l2d3y9q
slug: what-is-dependency-inversion-principle
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/WE_Kv_ZB1l0/upload/d072865c7fccdfb55c735fff2d4bc51d.jpeg
tags: dependency-inversion

---

Dependency Inversion Principle (DIP) means that the most flexible systems are those in which dependencies refer only to abstractions, not to concretions. That is because an abstract interface is less volatile than its implementations meaning that it is less likely to change and thus less likely to cause problems to those that refer to it.

![Clean Architecture](https://cdn.hashnode.com/res/hashnode/image/upload/v1670286798273/rxA5OJXrW.png align="center")

*image from Clean Architecture*

### Sample code

Clean architecture is a popular architecture used to implement modern software; [app architecture for Android](https://developer.android.com/topic/architecture). In Clean Architecture the flow of dependencies should be :

*   Presentation → Data → Domain
    

### Presentation

```kotlin
class Presentation(
   private val repository : Repository
){
   val name = repository.getUserName()
   /* show name to UI */
}
```

### Data

```kotlin
class RepositoryImpl : Repository(
   val dataSource : DataSource
){
  override fun getUserName(){
    /* concrete implementation */
  }
}

interface DataSource(
){
   /* remote or cache data */
}
```

### Domain

```kotlin
interface Repository(){
   fun getUserName()
}
```