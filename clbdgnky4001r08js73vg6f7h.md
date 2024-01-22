---
title: "Facade Design Pattern"
seoTitle: "Facade Design Pattern"
seoDescription: "Facade Design Pattern"
datePublished: Wed Dec 07 2022 09:41:26 GMT+0000 (Coordinated Universal Time)
cuid: clbdgnky4001r08js73vg6f7h
slug: facade-design-pattern
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/WE_Kv_ZB1l0/upload/d072865c7fccdfb55c735fff2d4bc51d.jpeg
tags: design-patterns

---

A Facade design pattern is a **structural concept** that helps reduce complexity between clients and subsystems by providing a higher-level interface known as a facade.

![Design Patterns: Elements of Reusable Object-Oriented Software](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fe94be462-62b0-463a-a0a1-f1b3033ae3ea%2FUntitled.png?id=7ac054c2-0dfd-414f-9866-3a6258cc3c09&table=block&spaceId=816299f4-9160-4967-96cb-515deac4ce28&width=2000&userId=c20ad251-590b-4de2-951d-e6be6f44d715&cache=v2 align="center")

*image from Design Patterns: Elements of Reusable Object-Oriented Software*

For instance, consider an e-commerce web application server that has subsystem services such as InventoryService, PaymentService, and ShippingService. If each client had access to each service, the subsystem classes’ will be tightly coupled with the client. To decouple the two and reduce complexity, a facade class or interface can be introduced as a liaison between the two. Clients can be agnostic to the implementation of the subsystem and the subsystem can just concentrate on the implementation of its logic.

### Sample Code

**Domain**

```kotlin
data class Product(
  val productId : Int,
  val name : String,
)
```

**Client/Controller**

```kotlin
class Controller(
  val facade : OrderServiceFacade
){
  fun orderProduct(productId : Int) : Boolean {
    val isOrderComplete = facade.placeOrder(productId)
    return isOrderComplete
  }
}
```

**Facade**

```kotlin
interface OrderServiceFacade {
  fun placeOrder(productId : Int) : Boolean
}

class OrderServiceFacadeImpl(
  val inventoryService : InventoryService,
  val paymentService : PaymentService,
  val shippingService : ShippingService
) : OrderServiceFacade(){
  override fun placeOrder(productId : Int) : Boolean {
    return if(val product = inventoryService.isAvailable(productId)){
      val paymentConfirmed = paymentService.makePayment()
      if(paymentConfirmed){
        shippingService.shipProduct(product)
        true
      } else false
    } else false
  }
}
```

**Service/Subsystem**

```kotlin
class InventoryService {
  fun isAvailable (productId : Int) : Product {
    /* Check persistent data to see if inventory is available */
    return product
  }
}

class PaymentService {
  fun makePayment () : Boolean {
    /* Connect with payment such as payment gateway or in app purchase */
    return true / false
  }
}

class ShippingServicxe{
  fun shipProduct(product : Product) : Unit {
    /* connect with external shipment service to ship product */
  }
}
```