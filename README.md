# DDD and Design Patterns

## IMPORTANT NOTE

- don't only focus on strategies (e.g., design patterns, detailed implementation). the most imporant part of DDD is the design phase (e.g., Bounded Context, Aggregate, Entity, and so on). if you wrongly design your domain, your system messes up and even if you use the strategies, you are not following the DDD properly.

## DDD overview

  - why use this?
    - to organize concerns (app, business, ui logic) into its corresponding layer
    - reduce an impact on our code when something change (like requirement)
      - if domain logic changes, then we just need to change the class of the domain layer (ideally)

      - if a logic inside a single aggregate change, we want to reduce the change as less as possible. so dcoupling or setting boundary between BCs or Aggregates helps us to update less code when the change happens.

  - whether you use DDD or not depends on the complexity of the business. 
    - when you try to apply DDD with less complicated business logic, there is no benefit of using DDD. in this case, better to use simple CRUD operation. 


## Terms

  - invariants: happens before the stat has even been reached.
      - business rules that must be enforced by requirement.
      ex)
        - a user can't put items no more than 5 => this is invariants and must follow when programming

      - each Aggregate has serveral numbers of invariants and these must be true always.

  - business logic: 

      - 2 types:
        - application logic
          - to deal with application specific problem, such as how email should be sent, db transaction
        - domain logic
          - deal with pure domain problem, such as a user should not have duplicated account or something.

  - amenic domain model:
    - an entity without any behavior.
    - a procedural style design and not object-oriented design.
    - ends up with spaghetti code or transaction scripts.
    - it is anti-pattern in DDD, but it is not in CRUD.

## Application Service

  def) services for application logic/decision
  resp)  
    - main responsibility is to coordinate tasks on the domain model, so keep this thin.
    - includes db transaction and security  

  
  - not same as domain service.
  - is a direct client of domain model
  - can query for a specific Aggregate instance.
  - one service method per end point (use case)
  - can be implemented by service or command

  - don't mix multiple use case together. duplication is ok for the sake of simplicity and avoid complex code.

  - be sure to focus on coordinating the domains.
    - don't implement domain logic inside application service. (e.g., Order.addItem(...) is OK, but don't do the following:

      ```
        // WRONG 
        OrderItem item = new OrderItem(...);
        Order.getItems().add(item);

        // this should be encapsulated inside Order domain.
      ```

  issue)
    - can I return domain object to ui layer?
      - sould use Command pattern and return DTO instead to avoid domain model froma leaking into user interface.
      - tags: Data Transformer, 
  

## Domain Layer

  - most imortant part of your app since it consists of business logic.

  - frequently change, so classes of this layer should not have any external dependencies (Clean Architecture)

  - Domain concerns such as Entities, Value Objects, and so on are nothing related with Persistence concern (like how it is stored in database)

  - seprate persistent knowledge from domain knowledge, which means that never put persistent knowledge inside domain layer.

  - should create custom exception if you need to throw.

    - don't bring other layer's conern into domain layer such as ResponseStatusException <- this is not something domain should concern.
  
## Domain Service

  formal def)
    1. The operation relates to a domain concept that is not a natural part of an Entity or Value Object.
    2. The interface is defined in terms of other elements of the domain model.
    3. The operation is stateless.

  def)
     - services for domain logic/decision
     - an operation to the domain (must be stateless)
     - any domain logic which does not belong to a entities.

  - no duplication allowed.
    - e.g., don't create multiple same logic. reuse the same implementation for application layer.

  - the method name should be ubiquitous language.
  - the services should be STATELESS: 

      stateless: any client can use any instance of a particular SERVICE without regard to the instance's individual history. The execution of a service will use information that is accessible globally, and may even change global information (that is, it may have side effects).

        - it ***should not have any fields/properties*** to track any information every time it is called.

  - deal with multiple domain objects in a single, atomic operation.

  - Generally, domain services are useful when you have ***business rules/logic that require coordinating or working with more than one aggregate***. If the logic is only involving one aggregate, it should be in a method on that aggregate's entities.

  - you can use repository. but it is for only read. you should delegate its transaction (e.g., save, persist) to application service.

  - should return domain model rather than DTO
  
  - direct client of applicaiton service (e.g., don't call this from controller)

  - should be located domain layer but not inside the Entities nor Value Object.
  - it has dedicated class for this service.

  - if a logic should be belonged to any domain object, the logic should goes to Application services.

  ex)
    - peform a significant business process.
    - transform a domain object from one composition to another.
    - calculate a value requiring input from more than one domain object.

  impl)

    - put the interface and implementation at domain layer.
    - alos you need to put irepository in domain layer to follow the dependency rules.

  - diff domain behaviors and domain service.

      - still don't know. 

      - but, if the service involved more than one aggregate, it should go to the domain service.
      - if the busincess logic can be enclosed inside a single aggregate, it should go to teh domain behaviors 
  
## Entities 

  - identifier euqality: the reference of a object must match when comparing
    - must have id to identify the object (like person, car, and so on)
  - can live by thier own.
    - entities use value objects
  - mutable (not immutable)
  
  - can have a reference of another Aggregate, but this entity must not be accessed from outside this Aggregate (local)
  - can communicate with outside (always through its aggregate)

  - never put persistent conerns/logics in domain layer.

  ### Validations

    - field validation: use self-encapsulation, put the validation logic inside the entity

    - whole entity  validation: need to validate all of fields. => use Specification/Strategy pattern or create ValidationClass for the entity (don't put the validation logic into the entity class, too much responsibility)

    - a set of Aggregates or an Aggregate validation: use domain service. Repository reads the Aggregate instance to validate and use domain event for teh timing when validating

  ### When to create ID
    - prefered: early when constructing this entity

  ### How to create ID
    - prefered: application itself (like uuid, guid)

  ### ID
    - if an entity is local, which means that the entity is not shared outside its Aggregate or itself, don't need to use UUID, you can just put int/varchar
    - if an entity is global, which mean that the entity is shared outside its Aggregate or itself, you need to use UUID.

## Value Objects

  - structural equality: each field must match when comparing
    - can be reused (like money)
  - can't live by their own.
    - value objects are used for entites.
  - immutable
    - this is because a value object can be represented with its fields, so if one of the fields changes, it no longer point to the same value object.
    - if you can safely replace an instance of a class with another one which has the same set of attributes, that’s a good sign this concept is a value object.
  - you can't chage any field after instantiation of a value object.
    -> if you need to modify that, it is entity.

  - how to store value object in database.
    - persistence concern does not relate to DDD. 
      - it does not matter if you create its own table or embed to entity table. 

  - whether it is value object or not depends on the business logic.
    - For example, an address in an e-commerce application might not have an identity at all, since it might only represent a group of attributes of the customer's profile for a person or company. In this case, the address should be classified as a value object. However, in an application for an electric power utility company, the customer address could be important for the business domain. Therefore, the address must have an identity so the billing system can be directly linked to the address. In that case, an address should be classified as a domain entity.

## Aggregates

  def) a cluster of domain objects that can be treated as a single unit.
     - is designed based on its invariants.
     - a set of Entities
     - a transactional boundary, so entities that need to be transactionally consistent are what forms an aggregate. Thinking about transaction operations is probably the best way to identify aggregates. 
     - represents a logical group of entities/value objects to fulfill responsibilities of an area of functionality (ordering, cataloging, inventory)
  ex)
    an order and its line-items.
      - there are two separate objects but it could be useful if treat it as a single unit.

  - Aggregates can't use Repositories. instead, Domain Services can use it.

  - must have global ID
  
  - can communicate with outside (BCs, Aggregates)

  - a repository per aggregate.

  - ideally, it consists of one entity and multiple value objects

  - **only a single aggregate should be modified in a single transaction**. 
    
    - this is because an Aggregate is synonymous for transactional consistency boundary, which means that it represents a boundary of a transaction consistency. esp, an Aggregate is designed based on its invariants (e.g., business rule that must be always consistent). so in order to keep the invariants, we use a transaction on the aggregate. this implys that you don't need to update multiple aggregates in a single transaction. 

    - if you need to modify multiple aggregate at a single request. use event system and handle another transaction for the other aggregate.

  - communication btw Aggregates

    - ***avoid tight coupling btw Aggregates.***

      - e.g., ***calling an aggregate function inside another aggregate.***

      - e.g., NEVER EVER do update an aggreagate from another aggregate.

    - 

## Boundary Context (BC)

  def)
    a single functnality of your system, such as ordering, inventory, catelog)

  issue)
    - should i use a transaction per BC??
  ans)
   - No. it should be per Aggregate. this is because an aggregate is a synonym of transcational consistency boundary, which means that a aggregate represents a doundary of transaction consistency. why? an Aggregate is designed based on its invariants (e.g., business rule that must be always consistent). so, in order to keep the invariants, we need to use a transaction to avoid the invariants from breaking. see [Aggregates](#aggregates) for more detail or textbook (Implemneting Domain Driven Design) page 354.

## Transactional consistency

  - use a single transaction to keep consistency of persistent storage by a request

## Eventual consistency: 

  - use a series of transactions to span multiple database. 
  - often used in microservice architecture.
  - Martin Fowler recommends to use this eventual consistency when communicating different aggregate or BC. but I think it is bit much since it make our project complicated.
  - need to use techinques like Saga or Event Sourcing (but 2PC is not option because of bad performance).
  - in real-life project, you might wonder if you should use thsi eventual consistency if you use a single database and have mutliple aggregates/BCs since you can enclose the communication with different aggregates/BCs in a single transaction. in this case, it might avoid using this eventual consitency in favor of simplicity and easiness. this implies that you need to break the rule of "one aggregate per transaction". you can modify multiple aggregate in a single transaction. it is optional if you use domain events or not. 

## Repositories

  purpose) decouple application and persistence so hide all implementation detail of persistence layer like Data Entities

- encapsulate teh logic required to access data sources. decouple the infrastructure to access databases from the domain model layer.
- only channel for update the database.
      - a repository per aggregate. this is because an aggregate controls teh transactional consistency. 
      - in the case of query (not modification of database), you can create another channel such as CQRS.  

    - repositories allow you to populate data in memory that comes from the database in the form of the domain entities. Once the entities are in memory, they can be changed and then persisted back to the database through transactions.

    -  IMPORTANT: It's important to emphasize again that you should only define one repository for each aggregate root. To achieve the goal of the aggregate root to maintain transactional consistency between all the objects within the aggregate, you should never create a repository for each table in the database. 

    - repositories shouldn't be mandatory.
      - use repositories if you use aggregates/domain models (e.g., to keep the invarints, transactional consistency, you use this repository to make sure there is no inconsistency). otherwise, you can use DAO instead.

    - if you don't use ORM, you might need to use repository with Data Mapper design pattern

## Unit of Work (UoW)

  - run a transaction based on a logical group of more than one Aggregates 

  - used when you need eventual consistency

  - merge multipel opeartions of database into a single batch to improve teh performance.

  - EF (Entity Framework in ASP.NET) provide DbContext which handle a transaction across different Aggregates at once.

  - atomicity
  
## Events

  ### types

    1. Global Event (Integration Event): an event which is involved with different BCs (or different applications)

      - tech:

         - messaging-style events with queuing, brokers, or a service bus using AMQP.

      - note: 

        - always sent asynchronously since it needs to communicate across processes and machines.

    2. Local Event (Domain Event): communicate different aggregates within the same BC

      - tech:

        - within the same application.

      - purpose: decouple concerens btw different aggregates. 

        - putting everything inside service class causes tight coupling and make us hard to maintain, but if you use domain event approach, you can encapsulate each logic into an event handler and make a connection with a domain event in the decoupled way. 

        - only one aggregate should be updated in a single transaction. why? (see [Aggregates Section](#aggregates) for the answer)

      - event handlers:

        - handle update for another aggregate rather than main aggregate for consistency

        - raise integration events for different BCs. 

      - note:

        - atomicity of a domain event (e.g., success or nothing happen)

        - synchronous/asynchronous but the side effect should happen immediately.

        - domain event handling is application concerns. therefore, event handlers reside in application layer.

        - raising domain event is domain concerns.

      - implementation:

        - event: 

          - a data holding structure or class like DTO with all information necessary for the event.

            e.g., orderStarted Event might need to hold orderId, orderNumber, orderitems, or something.

          - use past tense (e.g., OrderShippedDomainEvent)

          - must be immutable (since it happened in the past so shouldn't be changed)

            => should every property is 'read-only'

        - async or sync:
        
          - async: the most standard way to implement eventual consistency.
          
            - scalable.
            - hard & complicated
            
          - sync:
          
            - immediate effect.
            - not scalable since you need to wait for all of event handlers complete its task (e.g., might be a long request handling).


  ### how to raise an event 

    - static class: 
      - create a static class and shared all the place.

    - deferred approach: -> (use MediatR if C#)
      - store the domain events to a collection in Aggregate) and then to dispatch those domain events right before or right after committing the transaction.
      - like js event and its handler.
        - register an event and 

      - transaction management: right before commit (RBC) or right after commit (RAC)?

        - RBC:

          - you must use relational database.
          - side effects (any execution caused by the event) is included in the same transaction 

        - RAC:

          - side effects are not included in the same transaction. -> you need eventual consistency.

        - both approaches are right and it depends on your requirement and business logic.

 ## Commands

    - different from Events.

      1. a command should be handled only once. (e.g., events are handled multiple times by multiole receivers)

    - application concerns

 ## Persistent (PM) Model (or Data Model) vs Domain Model (DM)

  - ref: https://www.mehdi-khalili.com/orm-anti-patterns-part-4-persistence-domain-model

  - why we need to separate two different model?

    - assumption:

      - this is not absolute, which means that it depends on the size of your project. in enterprise size applications, combining Persistent Model and Domain Model causes problems. 

      - if you work with small size applications, it might not be necessary to separate them.

    - reason:

      - object-relational impedance mismatch; a set of conceptual and technical difficulties taht are often encountered when working relational database management system with object-oriented programming language. esp, objects or class definitions must be mapped to database tables defined by a relational schema.

    - definitions:

      PM: a model to map our tables and columns into something we can use in the code (application)

      DM: a model to encapsulate a domain logic

    - justification components:

      1. PM is a property bag and DM is about business logic and behavior

        - if your ORM allows you to set behavior in PM, this is not gonna be a component which justify the use of DM.

      2. Single Responsibility (one of SOLID principle)

        - separating PM and DM allows us to do testing more easily.

        - PM is responsible database logic and DM is responsible for business logic

      3. Dependencies

        - PM has a lot of dependencies and DM is POCO

      4. PM can enter invalid state while DM should not allow invalid state even temporarily.

        - DM allows you to define your custom validation in setters

    - cost of DM
  
      - mapping issue: when/where/how to map PM to DM vice versa.

      - increase codebase since you have to create DM for each PM

  - implementation with Hibernate & JPA  

      - in a nutshell, it does not go well because of following reasons:

        1. Hibernate itself think domain model is the same as persistent model (Entity)

          - hibernate entity is a POJO.

          - it does not mean to separate persistent model (Entity) and domain model since domain model is mapped entity class (e.g., persistent model)

          - ref: https://docs.jboss.org/hibernate/orm/5.0/mappingGuide/en-US/html_single/#entity

      - *** better to combine domain model with persistent model (Entity) together. 

        - if you separate these two, make your application complicated and increase a lot of works (e.g., mapping DM to PM, or how to persist DM)

  - implementation with EntityFramework C#

      - you can define POCO as domain model and you don't need to even create persistent model

        - see: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-implementation-entity-framework-core
        - also: https://github.com/dotnet-architecture/eShopOnContainers/blob/d0cd2830a864a8b975341b02b93a0887a3908c04/src/Services/Ordering/Ordering.Infrastructure/OrderingContext.cs#L17

      - **better to stick this way if you are using C# & EF**.
  

## Deisgn Patterns

### ActiveRecord

add database logic and domain logic in the active record class. don't use it. 
