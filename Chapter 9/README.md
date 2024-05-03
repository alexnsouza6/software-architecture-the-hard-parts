# Chapter 9 - Data Ownership and Distributed Transactions

Once data is pulled apart, it must be stitched back together to make the system work. This means figuring out which services own what data, how to manage distributed transactions, and how services can access data they need (but no longer own). In this chapter, we explore the ownership and transactional aspects of putting distributed data back together.

## Assigning Data Ownership

The general rule of thumb for data ownership is that the service that performs write operations to a table is the owner of that table. However, joint ownership makes this simple rule complex! 

To illustrate some of the complexities with data ownership, consider the example illustrated in Figure 9-1 showing three services: a Wishlist Service that manages all of the customer wish lists, a Catalog Service that maintains the product catalog, and an Inventory Service that maintains the inventory and restocking functionality for all products in the product catalog.

![](chapter-9-1.png)

In this chapter, we unravel this complexity by discussing the three scenarios encountered when assigning data ownership to services (single ownership, common ownership, and joint ownership), and exploring techniques for resolving these scenarios.


## Single Ownership Scenario

Single table ownership occurs when only one service writes to a table. This is the most straightforward of the data ownership scenarios and is relatively easy to resolve.

![](chapter-9-2.png)


## Common Ownership Scenario

Common table ownership occurs when most (or all) of the services need to write to the same table.

While this simple example includes only three services, imagine a more realistic example where potentially hundreds (or even thousands) of services must write to the same Audit table.

The solution of simply putting the Audit table in a shared database or shared schema that is used by all services unfortunately reintroduces all of the data-sharing issues described at the beginning of Chapter 6, including change control, connection starvation, scalability, and fault tolerance.

A popular technique for addressing common table ownership is to assign a dedicated single service as the primary (and only) owner of that data, meaning only one service is responsible for writing data to the table.

![](chapter-9-3.png)

## Joint Ownership Scenario

Figure 9-4 shows the isolated joint ownership example from Figure 9-1. The Catalog Service inserts new products into the table, removes products no longer offered, and updates static product information as it changes, whereas the Inventory Service is responsible for reading and updating the current inventory for each product as products are queried, sold, or returned.

![](chapter-9-4.png)

Fortunately, several techniques exist to address this type of ownership scenario—the table split technique, the data domain technique, the delegation technique, and the service consolidation technique. Each is discussed in detail in the following sections.


## Table Split Technique

The table split technique breaks a single table into multiple tables so that each service owns a part of the data it’s responsible for. 

![](chapter-9-5.png)

![](chapter-9-6.png)

## Data Domain Technique

This is formed when data ownership is shared between the services, thus creating multiple owners for the table. With this technique, the tables shared by the same services are put into the same schema or database, therefore forming a broader bounded context between the services and the data.

![](chapter-9-7.png)

![](chapter-9-8.png)

## Delegate Technique

With this technique, one service is assigned single ownership of the table and becomes the delegate, and the other service (or services) communicates with the delegate to perform updates on its behalf.

One of the challenges of the delegate technique is knowing which service to assign as the delegate (the sole owner of the table). The first option, called primary domain priority, assigns table ownership to the service that most closely represents the primary domain of the data—in other words, the service that does most of the primary entity CRUD operations for the particular entity within that domain. The second option,
called operational characteristics priority, assigns table ownership to the service needing higher operational architecture characteristics, such as performance, scalability, availability, and throughput.

![](chapter-9-9.png)

![](chapter-9-10.png)

With synchronous communication, the Inventory Service must wait for the inventory to be updated by the Catalog Service, which impacts overall performance but ensures data consistency.


## Service Consolidation Technique

The service consolidation technique resolves service dependency and addresses joint ownership by combining multiple table owners (services) into a single consolidated service, thus moving joint ownership into a single ownership scenario

![](chapter-9-11.png)

![](chapter-9-12.png)

## Data Ownership Summary

![](chapter-9-13.png)


## Distributed Transactions

When architects and developers think about transactions, they usually think about a single atomic unit of work where multiple database updates are either committed together or all rolled back when an error occurs. This type of atomic transaction is commonly referred to as an ACID.transaction.

To understand how distributed transactions work and the trade-offs involved with using a distributed transaction, it’s necessary to fully understand the four properties of an ACID transaction.

Atomicity means a transaction must either commit or roll back all of its updates in a single unit of work, regardless of the number of updates made during that transaction. In other words, all updates are treated as a collective whole, so all changes either get committed or get rolled back as one unit.

Consistency means that during the course of a transaction, the database would never be left in an inconsistent state or violate any of the integrity constraints specified in the database.

Isolation refers to the degree to which individual transactions interact with each other. Isolation protects uncommitted transaction data from being visible to other transactions during the course of the business request.

Durability means that once a successful response from a transaction commit occurs, it is guaranteed that all data updates are permanent, regardless of further system failures.

![](chapter-9-14.png)

![](chapter-9-15.png)


## Eventual Consistency Patterns

Distributed architectures rely heavily on eventual consistency as a trade-off for better operational architecture characteristics such as performance, scalability, elasticity, fault tolerance, and availability.

While there are numerous ways to achieve eventual consistency between data sources and systems, the three main patterns in use today
are the background synchronization pattern, orchestrated request-based pattern, and the event-based pattern.

![](chapter-9-16.png)

Customer 123 decides they are no longer interested in the Sysops Squad support plan, so they unsubscribe from the service.

![](chapter-9-17.png)

## Background Synchronization Pattern

The background synchronization pattern uses a separate external service or process to periodically check data sources and keep them in sync with one another. The length of time for data sources to become eventually consistent using this pattern can vary based on whether the background process is implemented as a batch job running sometime in the middle of the night, or a service that wakes up periodically (say,
every hour) to check the consistency of the data sources.

![](chapter-9-18.png)

![](chapter-9-19.png)

## Orchestrated Request-Based Pattern

A common approach for managing distributed transactions is to make sure all of the data sources are synchronized during the course of the business request (in other words, while the end user is waiting). This approach is implemented through what is known as the orchestrated request-based pattern.

![](chapter-9-20.png)

![](chapter-9-21.png)

![](chapter-9-22.png)

![](chapter-9-23.png)

## Event-Based Pattern

The event-based pattern is one of the most popular and reliable eventual consistency patterns for most modern distributed architectures, including microservices and event-driven architectures. With this pattern, events are used in conjunction with an asynchronous publish-and-subscribe (pub/sub) messaging model to post events (such as customer unsubscribed) or command messages (such as unsubscribe cus
tomer) to a topic or event stream. Services involved in the distributed transaction listen for certain events and respond to those events.

![](chapter-9-24.png)










