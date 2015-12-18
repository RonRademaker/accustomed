# accustomed
Bringing CQRS and Event Sourcing to traditional database design

# Idea
CQRS and Event Sourcing offer many advantages, but they are tied closely to Domain Driven Design. As a result it's very hard to implement this in existing projects. Another disadvantages is that most developers have no knownledge of DDD. This library aims to bring advantages of CQRS and Event Sourcing to applications using traditional database design. 

# Design

## Schema
![Design schema](https://github.com/RonRademaker/accustomed/blob/master/images/design.png)

## Components

### ORM

The ORM ties accustomed into any project that uses an ORM. This library intends to provide support for Propel and Doctrine. Developers should be able to continue using the ORM as they are used to, except for dealing with changes in database design.

### Command Handler

The command handler provides an alternative way of updating the database without using the ORM. This can greatly simplify implementations because write actions are not tied to a database design (allowing you to update the database design later).

### Eventbus

All events, regular events and event related to changes to the database, are passed to an event bus that uses a middleware approach to get them to their correct handler(s). This eventbus is intented to be compatible with existing event dispatcher implementations.

### Event types

#### Atomic events

Any handled command that results in some kind of write (which is usually any command) will result in atomic events. These may later be translated into one or more database queries that should be executed in a single transaction. 

Example event:

``` php
$this->eventbus->dispatch(new ProductBoughtEvent($product, $amount, $customer, $price));
```

Example database write in semi SQL:

``` sql
START TRANSACTION;
UPDATE Product SET instock=instock - $amount WHERE id = $product->getId();
INSERT INTO Orders (customer_id, description, price) VALUES ($customer->getId(), CONCAT($amount, ' x ', $product->getDescription()), $price);
COMMIT;
```

Easy to change to a new design as your application evolves:

``` sql
START TRANSACTION;
INSERT INTO Orders (customer, state) values ($customer->getId(), 'unpaid');
INSERT INTO Orderlines (order, product, amount, price) VALUES ($order_id, $product->getId(), $amount, $price);
COMMIT;
```

#### Related events

The event collector will relate any events that are wrapped in a single transaction in the ORM. These events should always all succeed, or all fail when applying them to the database.

#### SQL events

Low level events that should be avoided. They exist to make transition to accustomed possible without complications. These events are a special kind of related events triggered by interceptions on PDO. This makes sure all database writes are intercepted and the event store will be complete.

### Event Store

A store where all events, related and atomic, are stored.

### Handler 

Handler translate events into views, the translation into the tradtional database will be required to succeed for any write action to succeed.

### Event mapper 

Maps events into database queries, allowing you to completely change any database design without losing information. You'll only need to update the mapping of events.

### Tradtional database

The relational database as we're all so accustomed to.

### Alternate views

Alternative views, like elasticsearch or mongo database, containing segments of the data and allowing really fast reads from your application. 
