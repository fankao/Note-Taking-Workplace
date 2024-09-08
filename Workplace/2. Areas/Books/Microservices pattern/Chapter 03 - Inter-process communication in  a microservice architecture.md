## 3.1 Overview of inter-process communication in a microservice architecture
----

> [!info]
> There are lots of different IPC technologies to choose from:==
> 1. synchronous request/response-based communication mechanisms, such as HTTP based REST or gRPC
> 2. they can use asynchronous, message-based communication mechanisms such as AMQP or STOMP
>    
> There are also a variety of different messages formats:
> 1. Services can use human-readable, text-based formats such as JSON or XML
> 2.  Avro or Protocol Buffers

### 3.1.1 Interaction styles
---
> [!info]
> The first dimension is whether the interaction is one-to-one or one-to-many
> 1. One-to-one—Each client request is processed by exactly one service
> 2. One-to-many—Each request is processed by multiple services.
>    
> The second dimension is whether the interaction is synchronous or asynchronous:
> 1.  Synchronous—The client expects a timely response from the service and might even block while it waits.
> 2.  Asynchronous—The client doesn’t block, and the response, if any, isn’t necessarily sent immediately

![[Pasted image 20240907101038.png]]
### 3.1.2 Defining APIs in a microservice architecture
---
> [!note]
> Regardless of which IPC mechanism you choose, it’s important to precisely define a service’s API using some kind of interface definition language (IDL)

The nature of the API definition depends on which IPC mechanism you’re using:
- If you’re using messaging, the API consists of the message channels, the message types, and the message formats
- If you’re using HTTP, the API consists of the URLs, the HTTP verbs, and the request and response formats

### 3.1.3 Evolving APIs
---
#### Use Semantic Versioning
---
> [!tip]
> The Semantic Versioning specification (http://semver.org) is a useful guide to versioning APIs

> [!note] MAJOR.MINOR.PATCH
> - MAJOR—When you make an incompatible change to the API
> - MINOR—When you make backward-compatible enhancements to the API\
> - PATCH—When you make a backward-compatible bug fix
> 

#### Making Minor, Backward-compatible Changes
---
Backward-compatible changes are additive changes to an API:
- Adding optional attributes to request
- Adding attributes to a response
- Adding new operations

>[!quote] Robustness principle
>	Be conservative in what you do, be liberal in what you accept from others.
> Services should provide default values for missing request attributes. Similarly, clients should ignore any extra response attributes. In order for this to be painless, clients and services must use a request and response format

#### Making MAJOR, Breaking Changes
---
> [!note]
> A service must simultaneously support old and new versions of an API for some period of time

>[!example] REST
> Path Version
> 	/v2/
> MIME type
>		GET /orders/xyz HTTP/1.1 
>		Accept: application/vnd.example.resource+json; version=1

### 3.1.4 Message formats
---
#### Text-based Message Formats

- JSON
- XML

> [!success]
> - human readable
> - Self describing
> - backward-compatible

> [!fail]
> - messages tend to be verbose
> - Every message has the overhead of containing the names of the attributes in addition to their values
> - the overhead of parsing text, especially when messages are large
> 


#### Binary Message Format
---
- Protocol Buffers (https://developers.google.com/protocol-buffers/docs/overview)
- Avro (https://avro.apache.org)

## 3.2 Communication using synchronous Remote procedure invocation pattern
---
> [!note]
> The business logic in the client invokes a proxy interface , implemented by an RPI proxy adapter class. The RPI proxy makes a request to the service. The request is handled by an RPI server adapter class, which invokes the service’s business logic via an interface. It then sends back a reply to the RPI proxy, which returns the result to the client’s business logic
> 

![[Pasted image 20240907111253.png]]

### 3.2.1 Using REST

>[!quote] HTTP. Roy Fielding, the creator of REST, defines REST as follows:
> 	REST provides a set of architectural constraints that, when applied as a whole, emphasizes scalability of component interactions, generality of interfaces, independent deployment of components, and intermediary components to reduce interaction latency, enforce security, and encapsulate legacy systems.

#### The REST Maturity Model
____

![[Pasted image 20240907131755.png]]

### Specifying REST APIs
---
> [!tip]
> The most popular REST IDL is the Open API Specification (www.openapis.org), which evolved from the Swagger open source project. The Swagger project is a set of tools for developing and documenting REST APIs. It includes tools that generate client stubs and server skeletons from an interface definition

#### The Challenge Of Fetching Multiple Resources In A Single Request

#### The Challenge Of Mapping Operations To Http Verbs

#### Benefits and Drawbacks Of REST
---
> [!success] Benefits
> - It’s simple and familiar
> - You can test an HTTP API from within a browser using, for example, the Postman plugin, or from the command line using curl (assuming JSON or some other text format is used).
> - It directly supports request/response style communication.
> - HTTP is, of course, firewall friendly.
> - It doesn’t require an intermediate broker, which simplifies the system’s architecture.

> [!fail] Drawbacks
> - It only supports the request/response style of communication.
> - Reduced availability. Because the client and service communicate directly without an intermediary to buffer messages, they must both be running for the duration of the exchange.
> - Clients must know the locations (URLs) of the service instances(s). Clients must use what is known as a service discovery mechanism to locate service instances.
> - Fetching multiple resources in a single request is challenging.
> - It’s sometimes difficult to map multiple update operations to HTTP verbs.

### 3.2.2 Using gRPC

> [!info] gRPC
> - A binary message-based protocol, and this means—as mentioned earlier in the discussion of binary message formats—you’re forced to take an API-first approach to service design
> - gRPC APIs using a Protocol Buffers-based IDL, which is Google’s language-neutral mechanism for serializing structured data
> - Use the Protocol Buffer compiler to generate client-side stubs and server-side skeletons

> [!info] Protocol Buffers
> - Protocol Buffers is, as mentioned earlier, an efficient, compact, binary format
> - It’s a tagged format. Each field of a Protocol Buffers message is numbered and has a type code
> - A message recipient can extract the fields that it needs and skip over the fields that it doesn’t recognize
> 	gRPC enables APIs to evolve while remaining backward-compatible

> [!example] OrderService
```
	syntax = "proto3";
	
	option java_multiple_files = true;
	option java_package = "net.chrisrichardson.ftgo.orderservice.grpc";
	option java_outer_classname = "OrderServiceProto";
	option objc_class_prefix = "OS";
	
	package orderservice;
	
	service OrderService {
	  rpc createOrder(CreateOrderRequest) returns (CreateOrderReply) {}
	  rpc cancelOrder(CancelOrderRequest) returns (CancelOrderReply) {}
	  rpc reviseOrder(ReviseOrderRequest) returns (ReviseOrderReply) {}
	}
	
	message CreateOrderRequest {
	  int64 restaurantId = 1;
	  int64 consumerId = 2;
	  repeated LineItem lineItems = 3;
	  Address deliveryAddress = 4;
	  string deliveryTime = 5;
	}
	
	message Address {
	    string street1 = 1;
	    string street2 = 2;
	    string city = 3;
	    string state = 4;
	    string zip = 5;
	}
	
	message LineItem {
	    string menuItemId = 1;
	    int32 quantity = 2;
	}
	
	
	message CreateOrderReply {
	  int64 orderId = 1;
	}
	
	message CancelOrderRequest {
	  string name = 1;
	}
	
	message CancelOrderReply {
	  string message = 1;
	}
	
	message ReviseOrderRequest {
	  string name = 1;
	}
	
	message ReviseOrderReply {
	  string message = 1;
	}

```

> [!success] Benefits
> - It’s straightforward to design an API that has a rich set of update operations
> - It has an efficient, compact IPC mechanism, especially when exchanging large messages
> - Bidirectional streaming enables both RPI and messaging styles of communication.
> - It enables interoperability between clients and services written in a wide range of languages.

> [!fail] Drawbacks
> - It takes more work for JavaScript clients to consume gRPC-based API than REST/JSON-based APIs
> - Older firewalls might not support HTTP/2.

### 3.2.3 Handling partial failure using the Circuit breaker pattern
---
> [!info] Pattern: Circuit breaker
> An RPI proxy that immediately rejects invocations for a timeout period after the number of consecutive failures exceeds a specified threshold. See http://microservices.io/patterns/reliability/circuit-breaker.html.

![[Pasted image 20240907133957.png]]

#### Developing Robust RPI Proxies
----
> [!tip]
> Whenever one service synchronously invokes another service, it should protect itself using the approach described by Netflix (http://techblog.netflix.com/2012/02/faulttolerance-in-high-volume.html)

> [!tip] Fault Tolerance in a High Volume, Distributed System
> - ***Network timeouts***—**Never block indefinitely** and always use **timeouts** when waiting for a response. Using **timeouts** ensures that resources are never tied up indefinitely.
> - ***Limiting the number of outstanding requests from a client to a service*** -
> 	- Impose an upper bound on the number of outstanding requests that a client can make to a particular service. 
> 	- If the limit has been reached, it’s probably pointless to make additional requests, and those attempts should fail immediately.
> - ***Circuit breaker pattern:***
> 	- Track the number of successful and failed requests, and if the error rate exceeds some threshold, trip the circuit breaker so that further attempts fail immediately
> 	- A large number of requests failing suggests that the service is unavailable and that sending more requests is pointless
> 	- After a timeout period, the client should try again, and, if successful, close the circuit breaker.

#### Recovering From An Unavailable Service
----
> [!tip] Options to recover
> - the API gateway to return an error to the mobile client.
> - returning a fallback value or default value
> - return either a cached version of its data or omit it from the response.

![[Pasted image 20240907135318.png]]
### 3.2.4 Using service discovery
---
> [!note]
> Service instances have dynamically assigned network locations. Moreover, the set of service instances changes dynamically because of autoscaling, failures, and upgrades. Consequently, your client code must use a service discovery.

![[Pasted image 20240907135605.png]]

#### Overview Of Service Discovery
----
> [!info] Service discovery
> Its key component is a service registry, which is a database of the network locations of an application’s service instances.

> [!info] Two main ways to implement service discovery:
> - The services and their clients interact directly with the service registry.
> - The deployment infrastructure handles service discovery

#### Applying The Application - Level Service Discovery Pattern
----
This approach to service discovery is a combination of two patterns:
- Self registration pattern - A service instance invokes the service registry’s registration API to register its network location. It may also supply a health check URL

> [!tip] Pattern: Self registration
> A service instance registers itself with the service registry. See http://microservices.io/patterns/self-registration.html.

- Client-side discovery pattern 
	- When a service client wants to invoke a service, it queries the service registry to obtain a list of the service’s instances
	- To improve performance, a client might cache the service instances
	- The service client then uses a load-balancing algorithm, such as a round-robin or random, to select a service instance. It then makes a request to a select service instance.
	
>[!tip] Pattern: Client-side discovery
>  A service client retrieves the list of available service instances from the service registry and load balances across them. See http://microservices.io/patterns/clientside-discovery.html.

![[Pasted image 20240907140040.png]]

>[!success] Benefits
> Handles the scenario when services are deployed on multiple deployment platforms

> [!example]
> you’ve deployed only some of services on Kubernetes, discussed in chapter 12, and the rest is running in a legacy environment. Application-level service discovery using Eureka, for example, works across both environments, whereas Kubernetes-based service discovery only works within Kubernetes

> [!fail] Drawback
> - Need a service discovery library for every language—and possibly framework—that you use
> - Are responsible for setting up and managing the service registry, which is a distraction

#### Applying The Platform-provided Service Discovery Patterns
----
> [!info] The deployment platform
> - A service registry that tracks the IP addresses of the deployed services

![[Pasted image 20240907141412.png]]

> [!info] Pattern to implement
> - ***3rd party registration pattern***—Instead of a service registering itself with the service registry, a third party called the registrar, which is typically part of the deployment platform, handles the registration. (See http://microservices.io/patterns/3rd-party-registration.html.)
> - ***Server-side discovery pattern***—Instead of a client querying the service registry, it makes a request to a DNS name, which resolves to a request router that queries the service registry and load balances requests. (See http://microservices.io/patterns/server-side-discovery.html.)

>[!success] Benefits
>- All aspects of service discovery are entirely handled by the deployment platform - Neither the services nor the clients contain any service discovery code
>- The service discovery mechanism is readily available to all services and clients regardless of which language or framework they’re written in

>[!fail] Drawback
>- Only supports the discovery of services that have been deployed using the platform

## 3.3 Communicating using the Asynchronous messaging pattern
---
> [!info] Messaging-based
> A messaging-based application typically uses ***a message broker***, which acts as an intermediary between the services, although another option is to use a brokerless architecture, where the services communicate directly with each other 
> See http://microservices.io/patterns/communication-style/messaging.html.)

### 3.3.1 Overview of messaging
----
 > [!info] Message
 > - The header is a collection of name-value pairs, metadata that describes the data being sent
 > 	- The message header contains name-value pairs, such as a unique **message id** generated by either the sender or the messaging infrastructure, and an optional **return address**, which specifies the message channel that a reply should be written to.
> - The message body is the data being sent, in either text or binary format
> 	- ***Document***—A generic message that contains only data. The receiver decides how to interpret it. The reply to a command is an example of a document message.
> 	- ***Command***—A message that’s the equivalent of an RPC request. It specifies the operation to invoke and its parameters
> 	- ***Event***—A message indicating that something notable has occurred in the sender. An event is often a domain event, which represents a state change of a domain object such as an Order, or a Customer.

#### About Message Channels
---
![[Pasted image 20240907143703.png]]

>[!info] There are two kinds of channels
>- ***A point-to-point*** channel delivers a message to exactly one of the consumers that is reading from the channel. Services use point-to-point channels for the one-to-one interaction styles described earlier.
>	For example, a command message is often sent over a point-to-point channel
>- ***A publish-subscribe*** channel delivers each message to all of the attached consumers. Services use publish-subscribe channels for the one-to-many interaction styles described earlier.
>	For example, an event message is usually sent over a publish-subscribe channel.

### 3.3.2 Implementing the interaction styles using messaging
******
#### Implementing Request/response And Asynchronous Request/response
---
![[Pasted image 20240907144549.png]]

#### Implementing ONE-WAY Notifications
----
The client sends a message, typically a command message
	to a point-to-point channel owned by the service
The service subscribes to the channel and processes the message. It doesn’t send back a reply

#### Implementing Publish/Subscribe
---
#### Implementing Publish/async Responses
----
The publish/async responses interaction style is a higher-level style of interaction that’s implemented by combining elements of publish/subscribe and request/response. A client publishes a message that specifies a reply channel header to a publish-subscribe channel. A consumer writes a reply message containing a correlation id to the reply channel. The client gathers the responses by using the correlation id to match the reply messages with the request

### 3.3.3 Creating an API specification for a messaging-based service API
----
> [!info] A Service
> A service’s asynchronous API consists of operations, invoked by clients, and events, published by the services

![[Pasted image 20240907145851.png]]

#### Documenting Asynchronous Operations
---
- ***Request/async response-style API***—This consists of the service’s command message channel, the types and formats of the command message types that the service accepts, and the types and formats of the reply messages sent by the service
- ***One-way notification-style API***—This consists of the service’s command message channel and the types and format of the command message types that the service accepts
#### Documenting Published Events
---
- A service can also publish events using a publish/subscribe interaction style. The specification of this style of API consists of the event channel and the types and formats of the event messages that are published by the service to the channel

### 3.3.4 Using a message broker
----
![[Pasted image 20240907150524.png]]

#### Overview Of Broker-based Messaging
---
> [!tip] Selecting a message broker, you have various factors to consider, including the following:
> - ***Supported programming languages***—You probably should pick one that supports a variety of programming languages.
> - ***Supported messaging standards***—Does the message broker support any standards, such as AMQP and STOMP, or is it proprietary?
> - ***Messaging ordering***—Does the message broker preserve ordering of messages?
> - ***Delivery guarantees***—What kind of delivery guarantees does the broker make?
> - ***Persistence***—Are messages persisted to disk and able to survive broker crashes?
> - ***Durability***—If a consumer reconnects to the message broker, will it receive the messages that were sent while it was disconnected?
> - ***Scalability***—How scalable is the message broker?
> - **Latency**—What is the end-to-end latency?
> - ***Competing consumers***—Does the message broker support competing consumers?

#### Implementing Message Channels Using A Message Broker
----
![[Pasted image 20240907190516.png]]

> [!success] Benefits
> - Loose coupling—A client makes a request by simply sending a message to the appropriate channel. The client is completely unaware of the service instances. It doesn’t need to use a discovery mechanism to determine the location of a service instance.
> - Message buffering—The message broker buffers messages until they can be processed. With a synchronous request/response protocol such as HTTP, both the client and service must be available for the duration of the exchange. With messaging, though, messages will queue up until they can be processed by the consumer
> - Flexible communication—Messaging supports all the interaction styles described earlier.\
> - Explicit inter-process communication—RPC-based mechanism attempts to make invoking a remote service look the same as calling a local service. But due to the laws of physics and the possibility of partial failure, they’re in fact quite different.

>[!fail] Drawbacks
>- Potential performance bottleneck—There is a risk that the message broker could be a performance bottleneck. Fortunately, many modern message brokers are designed to be highly scalable.
>- Potential single point of failure—It’s essential that the message broker is highly available—otherwise, system reliability will be impacted. Fortunately, most modern brokers have been designed to be highly available
>- Additional operational complexity—The messaging system is yet another system component that must be installed, configured, and operated

### 3.3.5 Competing receivers and message ordering
---
> [!tip] sharded (partitioned) channels
> 1. A sharded channel consists of two or more shards, each of which behaves like a channel.
> 2. The sender specifies a shard key in the message’s header, which is typically an arbitrary string or sequence of bytes. The message broker uses a shard key to assign the message to a particular shard/partition
> 3. The messaging broker groups together multiple instances of a receiver and treats them as the same logical receiver. Apache Kafka, for example, uses the term consumer group. The message broker assigns each shard to a single receiver. It reassigns shards when receivers start up and shut down.

![[Pasted image 20240907191623.png]]

### 3.3.6 Handling duplicate messages
----
> [!tip] There are a couple of different ways to handle duplicate messages:
> -  Write idempotent message handlers.
> - Track messages and discard duplicates.

#### Writing Idempotent Message Handler
---
> [!note] Idempotent
> If the application logic that processes messages is idempotent, then duplicate messages are harmless. Application logic is idempotent if calling it multiple times with the same input values has no additional effect

> [!example]
> For instance, cancelling an already-cancelled order is an idempotent operation. So is creating an order with a client-supplied ID.

> [!note]
> application logic is often not idempotent. Or you may be using a message broker that doesn’t preserve ordering when redelivering messages. Duplicate or out-of-order messages can cause bugs


#### Tracking Message and Discarding Duplicates
---

![[Pasted image 20240907193051.png]]

### 3.3.7 Transactional messaging
---
#### Using A Database Table As A Message Queue
---
![[Pasted image 20240908092547.png]]

> [!quote] Pattern: Transactional outbox
> Publish an event or message as part of a database transaction by saving it in an OUTBOX in the database. See http://microservices.io/patterns/data/transactional-outbox.html.

#### Publishing Events By Using The Polling Publisher Pattern
---
> [!quote] Pattern: Polling publisher
> Publish messages by polling the outbox in the database. See http://microservices.io/patterns/data/polling-publisher.html.

#### Publish Events By Applying The Transaction Log Tailing Pattern
----
![[Pasted image 20240908190947.png]]

> [!quote] Pattern: Transaction log tailing
> Publish changes made to the database by tailing the transaction log. See http://microservices.io/patterns/data/transaction-log-tailing.html.

> [!tip] There are a few examples of this approach in use:
> - Debezium (http://debezium.io)—An open source project that publishes database changes to the Apache Kafka message broker
> - LinkedIn Databus (https://github.com/linkedin/databus)—An open source project that mines the Oracle transaction log and publishes the changes as events. LinkedIn uses Databus to synchronize various derived data stores with the system of record
> - DynamoDB streams (http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)—DynamoDB streams contain the time-ordered sequence of changes (creates, updates, and deletes) made to the items in a DynamoDB table in the last 24 hours. An application can read those changes from the stream and, for example, publish them as events.
> - Eventuate Tram (https://github.com/eventuate-tram/eventuate-tram-core)—Your author’s very own open source transaction messaging library that uses MySQL binlog protocol, Postgres WAL, or polling to read changes made to an OUTBOX table and publish them to Apache Kafka.