---
layout: post
title: Message Driven Architecture Explained - Basics
category: Architecture
---

You might know about Message Driven Architecture (MDA) if you've heard about the **Domain Events Pattern** or **Event Sourcing.** As its core, MDA means that an application is composed from autonomous components which communicate with each other via messages. MDA is very common in a distributed application, because each component sits on a different server but they still need to work together.

 But the amazing thing about MDA is that it can be applied for local (non distributed) applications as well. This means that a local MDA application can easily become a distributed app, only some configuration is required, but the app code should remained untouched. The main benefit though is that it makes it **very easy to write high quality, maintainable code** i.e lowly coupled, highly cohesive and highly testable code.

 The downside is that it requires a bit of experience to become comfortable with it. It's not straightforward and at first, it seems like over engineering, but once you experiment with it, it will feel like the proper way to implement a non trivial application. But for that, you need to understand MDA properly.

 
## How It Works

 Let's say you have 3 modules: Sales, Accounting, Inventory. These 3 obviously need to work together but we want these modules not to be coupled one to another. Each module knows that the other exist, but they don't know where and how to talk with them. They are pretty decoupled.

 In order to work together, these modules will pass messages to each other via a 3rd party, a 'transporter' which knows where everyone is. That transporter is called a **ServiceBus** and that's its sole responsibility: to deliver messages to a destination.

 In a nuthsell things goes like this:

  
  * Module sends a message to the ServiceBus. 
  * ServicesBus delivers the message to one or many configured destinations. 
  * Destination receives and handle the message.  Note that the sender and the receiver modules don't know about each other, they know only about the ServiceBus. This means that modules can be added or removed without other modules being affected.

 
## Messages

 A message is a simple data transfer object (DTO) which contains message name and details. A message should be as simple as possible and serialization friendly.

 A message can be a:

  
  * **Command** - the message tells that its handler must do something. The message name is the Command while its content are the command arguments. Good names are imperative ones: CreateOrder, RegisterPayment etc. A command is **sent** via the ServiceBus and can have _only one_ handler. 
  * **Event** - the message tells that something have happened. A good event name represents an action in the past: OrderCreated, PaymentSubmitted etc. An event is **published** via the ServiceBus and can have _0 or more_ **subscribers**.  A message handler is the object which processes the message. There are** Command Handlers** (or Executors) and **Event Subscribers**. A message handler can send messages as well.

 
## Idempotency

 One important detail about MDA is that a message is guaranteed to be sent **at least once**. This means that a handler can receive the same message twice. The reasons why that happen are mainly technical in nature and a handler must be able to cope with this behavior. That is no matter how many times a message is processed, the outcome must be the same. This is called **idempotency**.

 Some actions are naturally idempotent, for example a=3. Invoking a handler that assigns 3 to a variable will have the same result no matter how many times is done. But if, let's say, the variable is incremented then each invoke will get a different result. For these cases, the message handler must take additional steps to ensure its idempotency.

 Ensuring idempotency is an app specific action, but usually is done by having in the database a unique key for the processed message. For example, inserting a new Product will also insert the message id. If the operation is a duplicate the db will refuse it. In this case the datastore is the actual enforcer of idempotency. This example also shows that idempotency is a concept that goes beyond the message handler.

 These are the basics of message driven architecture. In the next post I'll talk about the cool stuff.


