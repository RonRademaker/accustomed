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

### Event collector

The event collector collects all update and insert instructions from the ORM and transforms these into events that will be stored. 

### Related events

The event collector will relate any events that are wrapped in a single transaction. These events should always all succeed, or all fail when applying them to the database.

### Command Handler

The command handler provides an alternative way of updating the database without using the ORM. This can greatly simplify implementations because write actions are not tied to a database design (allowing you to update the database design later).

### Atomic events

Any handled command that results in some kind of write (which is usually any command) will result in atomic events. These may later be translated into one or more database queries that should be executed in a single transaction.

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
