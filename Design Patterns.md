---
author:
  - Dainish Jabeen
bibliography:
  - refs.bib
date: 2023-10-01
title: Design Patterns
---

# Introduction

Patterns describe evolved methods through trail and error, of commonly
occurring problems.

# SOLID

### Single Responsibility Principle

A class should have just one reason to change.
### Open/Close Principle

Classes should be open for extension but closed for modification.
### Liskov Substitution Principle

Extending a class, should be able to use objects of the subclass instead of the parent class.
### Interface Segregation Principle

Clients shouldn't be forced to depend on methods they dont use.
### Dependancy Inversion Principle

High level classes should not depend on low level classes. Details should depend on abstractions.
# Composition

Preferred to inheritance, the concept is to use interfaces to expand behaviours easily (in a plugin method).

# Types of patterns

**Creational Patterns:** Increase flexibility and reusability.
**Structural Patterns:** Assemble classes to large structures while keeping them flexible and efficient.
**Behavioural Patterns:** Effective communication and responsibility between modules.

# Creational Patterns

## Factory

Create objects in superclass, alter in subclass.The creation of objects is done using the factory method, that changes the object returned based on context (normally conditional statements) which is the subclass of a 'Creator' class . The return type of the function is an abstract so the client receiving does not know the details of the object. This abstract type is an interface, which is detailed in concrete implementations that specify which object is created. Which are directly called via a subclass of the creator.

This eases the creation of new objects to be used, and is useful if it is context dependant which object should be created and used. Can also create a pool of objects, which the factory method checks before creating a new object.

![[Factory.png]]

## Abstract Factory

Create families of objects without specifying its implementation. If there matrix of implementations ie feature 1 x feature 2 (furniture x style). An interface is created for each characteristic on one dimension (interface for each furniture type) and an implementation of each the second dimensions characteristic on each interface (implementation for each style type).

The abstract factory is the creator interface with an implementation of each of the second dimensions characteristics (style types) that create each of first dimensions object with that chosen second dim. The return type is an interface of the first dimension allowing to use any first dim methods without the client knowing the second dim. Allowing easy additions of either dims. Extension method for first dimension and plugin method for second dim.

![[Abstract Factory.jpeg]]

## Builder

Using the same construction code to build complex objects. Solving the issues of classes with many parameters, create a builder modules there implementations of an interface for the variations in the object. In each builder the parameters will be set, you dont need to use all parameters.

For easier construction reuse a director module with coded methods of using the builders. The client code will use the director via a concrete builder to get the object. This gives you an easy way to extend variations of an object.

![[Builder.png]]
## Prototype

Copy existing objects without dependancy. Create a cloning interface common to objects that are to be duplicated. The implementation of the interface will have a method to clone all
fields of that object.

Good for when many objects are storing state, easy way to get an object with a basic configuration. May not have access to all fields of an object to clone.

![[Prototype.png]]
## Singleton

Uses only one instance that is globally available. Help protect from threading issues a shared resource. Make constructor of object private, used via method in module that creates ans stores object. When called will return created object.

![[Singleton.png]]
# Structural Patterns

## Adapter

Communication between incompatible interfaces. Using an interface when implemented will convert the type of one object to another by wrapping it (referencing it). Access to the methods of the object are then via the interface. Allowing a plugin model for adapters.

![[Adapters.png]]
## Bridge

Separate modules that are the implementations and abstract. Modules write high level methods that uses an interface for details to create a plugin nature. The abstract methods use the implementations via the interface.

![[Bridge.png]]
## Composite

Objects in tree structure. Deal with all bottom leaves and higher leaves with children in the same, allowing recursion to navigate through the tree and provide a result. A common interface is used to access each leaf, allowing an implementation be for a bottom leaf or an adult leaf, for the client its does not matter.

![[Composite.png]]
## Decorator

Add new behaviours to objects via wrappers. With a common interface between current object and the wrapper/decorator, a base decorator is created which is extended and has access to the base object. Allowing use of either base object or decorator without the client being aware.

![[Decorator.png]]
## Facade

Simple interface to complex objects. Use an intermediary module handling all calls to the subsystem, so the client doesn't call it directly.

![[Facade.png]]
## Flyweight

Save RAM by sharing objects. Create objects \"flyweights\" that hold all intrinsic (not changing/identical across objects) data of objects, which is referenced to by objects that hold the extrinsic data. The flyweights are immutable, another module can be used to manage pools of flyweights.

![[Flyweights.png]]
## Proxy

Substitute or placeholder for object, controls access to objects so can perform methods before/after. Using an interface for both the base object and proxy, with proxy linking to the service.

![[Proxy.png]]
# Behavioural

## Chain of responsibility

Pass request along chain of handlers, each handler deciding if to use it then passes it on. A handler is a single module with a single method (execute) and a ref to the next handler in chain . All handler implement the same interface, this allows the chain to be dynamically created at run time. 

The handler will make a decision on if the data should be passed on or if the chain can stop, therefore only executing the number of steps required. For a cleaner setup can have a base handler which is extended by all handlers.

![[Chain.png]]
## Command

Turn a request into an object for easier handling; can parametrise, delay or queue requests. Using a 'command' module implementing an interface between procedure and low level details with a single execute method. Allowing the command of a interface to become a plugin. The interface will simply alert the command, the command will then gather the data needed on its own to
further decouple them. If an invoker module is used to setup the required command modules and execute them when receiving a signal from the interface can stack commands and reuse the objects.

![[Command.png]]
## Iterator

Traverse elements without exposing its details. Storage patterns may require different traversal methods, this can be separated into modules that implement an interface. To give the traversal a plugin nature. Can use a interable collection interface so grouping of storage method and iterators is easy.

![[Iterator.png]]
## Mediator

Reduce dependencies between objects and by controlling communication. Components that are independent should not directly communicate, better to go via a mediator that redirects calls based on needs. So the components depend on the mediator and not on each other. Can use an interface for using multiple mediators, so that each component use the interface further decoupling further the interfaces. Should be careful of the size of the mediator.

![[Mediator.png]]
## Memento

Save and restore previous state of an object. Best method is to allow the object to create a copy of its own state to not void the encapsulation. The copy of the state is a special object called memento. A separate Caretaker can then store this object. The memento can be created by the original object via an interface, allowing multiple memento to be used polymorphically.

![[Memento.png]]
## Observer

Notify events occurring of an observed object. Via an interface a publisher module will communicate to subscribers. The publisher will hold a list of all subscribers, with methods to add or remove a sub. Can also have a interface for the publisher.

![[Publisher.png]]
## State

Change behaviour based on state. Finite state machine are normally based on conditionals, but this lacks evolvability. Each state has its own module, with the original object holding a ref to its current state. And a method to change states with the state implementations holding the state specific behaviour which is accessed via the interface.

The states are implemented via an interface, easy adding of states.

![[State.png]]

## Strategy

Define a family of algorithms with each object interchangeable. Split modules an implement via an interface an algorithm that does something specific in many ways. Use a context module to keep track of the strategies and allow use via interface. The client will then decide which strategy to use. Context module used for cleanliness.


Can use a set of anonymous functions instead of this interface pattern; functions without an identifier. This allows you to set the different functions in the context module at compile time to be used and easily add.

![[Strategy.png]]
## Template Method

Define structure of procedure and then adjust via subclasses. The high level module will have abstract methods which have to be implemented by extensions and optional methods.

![[Template.png]]
## Visitor

Separate algorithms from objects on which they operate. Create a separate module for the algorithms via an interface and choose which method is require for the current object via double dispatch. Double dispatch is when the object has a method that will use the correct method in the algorithm module accessed via the interface.

![[Visitor.png]]