
## 2.2. Defining an application’s microservice architecture
---
### 2.2.0. Overall
----
#### 2.2.0.1 Defining application's architecture
Like much of software development, defining an architecture is more art than science. This section describes a simple, three-step process, shown in figure 2.5, for defining an application’s architecture.

![[Pasted image 20240906200056.png]]
![[Pasted image 20240906200125.png]]
A ***system operation*** is an abstraction of a request that the application must handle - The architectural scenarios that illustrate how the services collaborate.
- command - updates data
- query - retrieves data

**The second step** in the process is to determine the decomposition into services:
 - One strategy, which has its origins in the discipline of business architecture, is to define services corresponding to business capabilities
 - Another strategy is to organize services around domain-driven design subdomains.

> [!NOTE] Noted
> The end result is services that are organized around business concepts rather than technical concepts.

**The third step** in defining the application’s architecture is to determine each service’s API:
- Assign each system operation identified in the first step to a service. A service might implement an operation entirely by itself => A service might implement an operation entirely by itself.
- Collaborate with other services => you determine how the services collaborate (IPC mechanisms)
#### 2.2.0.2 Obstacles to decomposition
 - Network latency
 - Synchronous communication between services reduces availability
 - Requirement to maintain data consistency across services
 - God classes, which are used throughout an application
 
### 2.2.1. Identifying the system operations
-----
![[Pasted image 20240906202324.png]]

1. Creates the high-level domain model consisting of the key classes that provide a **vocabulary** with which to describe the system operations  -  The domain model is derived primarily from the nouns of the user stories (Find objects and relationship)
2. Identifies the system operations and describes each one's behavior in terms of the domain model - The system operations are derived mostly from the verbs (Define actions for each objects)

	You could also define the domain model using a technique called Event Storming

![[Pasted image 20240906203003.png]]


#### Creating A High - Level Domain Model
----

> [!NOTE] Noted
> A high-level domain model is useful at this stage because it defines the **vocabulary** for describing the behavior of the system operations.


```
Given a consumer

	And a restaurant
	
	And a delivery address/time that can be served by that restaurant
	
	And an order total that meets the restaurant's order minimum

When the consumer places an order for the restaurant

Then consumer's credit card is authorized

	And an order is created in the PENDING_ACCEPTANCE state
	
	And the order is associated with the consumer
	
	And the order is associated with the restaurant
```

=> Nouns indicates domain model including Consumer, Order, Restaurant, and CreditCard.

```
Given an order that is in the PENDING_ACCEPTANCE state

	and a courier that is available to deliver the order

When a restaurant accepts an order with a promise to prepare by a particular

	time

Then the state of the order is changed to ACCEPTED

	And the order's promiseByTime is updated to the promised time
	
	And the courier is assigned to deliver the order
```

=> Nouns indicates domain model including Courier, Delivery, MenuItem, Address

![[Pasted image 20240906204408.png]]

The responsibilities of each class are as follows:
- Consumer—A consumer who places orders.
- Order—An order placed by a consumer. It describes the order and tracks its status.
- OrderLineItem—A line item of an Order.
- DeliveryInfo—The time and place to deliver an order.
- Restaurant—A restaurant that prepares orders for delivery to consumers.
- MenuItem—An item on the restaurant’s menu.
- Courier—A courier who deliver orders to consumers. It tracks the availability of the courier and their current location.
- Address—The address of a Consumer or a Restaurant.
- Location—The latitude and longitude of a Courier.

#### Defining System Operations
----
There are two types of system operations:
- Commands - System operations that create, update, and delete
- Queries - System operations that read data

> [!NOTE] Noted
> A good starting point for identifying system commands is to analyze the verbs in the user stories and scenarios

![[Pasted image 20240906205045.png]]
![[Pasted image 20240906205058.png]]


 A command has a specification: 
 - parameters
 - return value
 - behavior in terms of the domain model classes

 The behavior specification
 - Preconditions that must be true when the operation is invoked - *Given*
 - Post-conditions that are true after the operation is invoked - *Then*
 
![[Pasted image 20240906205458.png]]
![[Pasted image 20240906205707.png]]

### 2.2.2. Defining services by applying the Decompose by business capability pattern
---

> [!HIGHLIGHT] Noted
> a *business capability* is something that a business does in order to generate value. The set of capabilities for a given business depends on the kind of business (See in https://microservices.io/patterns/decomposition/decompose-by-business-capability.html)
> 

#### Business Capabilities Define What An Organization Does
---
- what an organization’s business is - stable
- how an organization conducts its business - changes over time, sometimes dramatically


> [!EXAMPLE] Example
> it wasn’t that long ago that you deposited checks at your bank by handing them to a teller. It then became possible to deposit checks using an ATM. Today you can conveniently deposit most checks using your smartphone. As you can see, the Deposit check business capability has remained stable (WHAT), but the manner in which it’s done has drastically changed (HOW).

#### Identifying Business Capabilites
----
An organization’s business capabilities are identified by analyzing the organization’s purpose, structure, and business processes

Each business capability can be thought of as a service, except it’s business-oriented rather than technical

Its specification consists of various components, including inputs, outputs, and service-level agreements


> [!EXAMPLE] Example
> For example, the input to an Insurance underwriting capability is the consumer’s application, and the outputs include approval and price.

A business capability is often focused on a particular business object

> [!EXAMPLE] Example
> For example, the Claim business object is the focus of the Claim management capability

A capability can often be decomposed into sub-capabilities

> [!EXAMPLE] Example
> For example, the Claim management capability has several sub-capabilities, including Claim information management, Claim review, and Claim payment management

It is not difficult to imagine that the business capabilities for FTGO include the following:
- Supplier management:
	- Courier management—Managing courier information
	- Restaurant information management—Managing restaurant menus and other information, including location and open hours
- Consumer management—Managing information about consumers
- Order taking and fulfillment
	- Order management—Enabling consumers to create and manage orders
	- Restaurant order management—Managing the preparation of orders at a restaurant
	- Logistics
		- Courier availability management—Managing the real-time availability of couriers to delivery orders
		- Delivery management—Delivering orders to consumers
- Accounting
	- Consumer accounting—Managing billing of consumers
	- Restaurant accounting—Managing payments to restaurants
	- Courier accounting—Managing payments to couriers
- ...


> [!NOTE] Noted
> On interesting aspect of this capability hierarchy is that there are three restaurant related capabilities: Restaurant information management, Restaurant order management, and Restaurant accounting. That’s because they represent three very different aspects of restaurant operations

#### From Business Capabilities To Services
![[Pasted image 20240906212752.png]]


> [!NOTE] Noted
> A key benefit of organizing services around capabilities is that because they’re stable (WHAT), the resulting architecture will also be relatively stable. The individual components of the architecture may evolve as the HOW aspect of the business changes, but the architecture remains unchanged.

> [!NOTE] Noted
> An important step in the architecture definition process is investigating how the services collaborate in each of the key architectural services


### 2.2.3. Defining services by applying the Decompose by sub-domain pattern
---

> [!NOTE] Definition
> A domain mode captures knowledge about a domain in a form that can be used to solve problems within that domain
> It defines the vocabulary used by the team, what DDD calls the Ubiquitous Language

DDD has two concepts that are incredibly useful when applying the microservice architecture:
- subdomains
- bounded contexts.


> [!LINK] References
> See http://microservices.io/patterns/decomposition/decompose-by-subdomain.html

Problems of traditional approach to enterprise modeling:
- getting different parts of an organization to agree on a single model is a monumental task
- from the perspective of a given part of the organization, the model is overly complex for their needs
- the domain model can be confusing because different parts of the organization might use either the same term for different concepts or different terms for the same concept
And, DDD avoids these problems by defining multiple domain models, each with an explicit scope

DDD defines a separate domain model for each subdomain. A subdomain is a part of the domain, DDD’s term for the application’s problem space

Subdomains are identified using the same approach as identifying business capabilities: analyze the business and identify the different areas of expertise. 
	The end result is very likely to be subdomains that are similar to the business capabilities

DDD calls the scope of a domain model a bounded context - includes the code artifacts that implement the model


> [!NOTE] Noted
> When using the microservice architecture, each bounded context is a service or possibly a set of services.

![[Pasted image 20240907081213.png]]

### 2.2.4 Decomposition guidelines
----
#### Single Responsibility Principle

> [!quote] Robert C. Martin
>  *A class should have only one reason to change*

#### Common Closure Principle

> [!quote] Robert C. Martin
> *The classes in a package should be closed together against the same kinds of changes. A change that affects a package affects all the classes in that package.*

### 2.2.5 Obstacles to decomposing an application into services
---
#### Network Latency

> [!CONCEPT] An ever-present concern in a distributed system

#### Synchronous Inter-process Communication Reduces Availability

> [!CONCEPT] how to implement interservice communication in a way that doesn’t reduce availability

#### Maintaining Data Consistency Across Services

 > [!tip]
 > A saga is a sequence of local transactions that are coordinated using messaging. Sagas are more complex than traditional ACID transactions but they work well in many situations. One limitation of sagas is that they are eventually consistent



####  Obtaining A Consistent View Of The Data

>[!warning]
>In a monolithic application, the properties of ACID transactions guarantee that a query will return a consistent view of the database
>
>In contrast, in a microservice architecture, even though each service’s database is consistent, you can’t obtain a globally consistent view of the data

#### God Classes Prevent Decomposition

> [!info]
> God classes are the bloated classes that are used throughout an application (http://wiki.c2.com/?GodClass)
> A god class typically implements business logic for many different aspects of the application

>[!example]
>The *Order* class is a great example of a god class in the FTGO application. That’s not surprising—after all, the purpose of FTGO is to deliver food orders to customers. Most parts of the system involve orders. If the FTGO application had a single domain model, the *Order* class would be a very large class. It would have state and behavior corresponding to many different parts of the application

![[Pasted image 20240907090905.png]]

> [!tip]
> A much better approach is to apply DDD and treat each service as a separate subdomain with its own domain model
> This means that each of the services in the FTGO application that has anything to do with orders has its  

Take advantages of DDD to split God class - Order into versions corresponding to subdomains

![[Pasted image 20240907091226.png]]

![[Pasted image 20240907091410.png]]

### 2.2.6 Defining service APIs
---
> [!info]
> Service’s API: its operations and events

A service API operation exists for one of two reasons:
- Some operations correspond to system operations. They are invoked by external clients and perhaps by other services
- The other operations exist to support collaboration between services. These operations are only invoked by other services
A service publishes events primarily to enable it to collaborate with other services

> [!note]
> 1. The starting point for defining the service APIs is to map each system operation to a service
> 2. we decide whether a service needs to collaborate with others to implement a system operation
> 	> we then determine what APIs those other services must provide in order to support the collaboration
> 

#### Step 1 - Assigning System Operations To Services
---
![[Pasted image 20240907092844.png]]

#### Step 2 - Determining The APIS Required To Support Collaboration Between Services
----

> [!note]
> In order to fully define the service APIs, you need to analyze each system operation and determine what collaboration is required.

![[Pasted image 20240907093206.png]]

> [!note]
> Select IPC technology (REST, GPC, Message Broker) after sketching Services - Operations - Collaborators
## Summary
---
1. Architecture determines your application’s -ilities, including maintainability, testability, and deployability, which directly impact development velocity.
2. The microservice architecture is an architecture style that gives an application high maintainability, testability, and deployability
3. Services in a microservice architecture are organized around business concerns — business capabilities or subdomains—rather than technical concerns
4. There are two patterns for decomposition:
	- Decompose by business capability, which has its origins in business architecture
	- Decompose by subdomain, based on concepts from domain-driven design
5. You can eliminate god classes, which cause tangled dependencies that prevent decomposition, by applying DDD and defining a separate domain model for each service