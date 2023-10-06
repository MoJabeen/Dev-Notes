---
author:
  - Dainish Jabeen
bibliography:
  - refs.bib
date: 2023-10-01
title: Clean Architecture
---

Notes based on [1]  [2].

# Introduction

The goal of software architecture is to minimize the human resources required to build and maintain the system.

There are two values stakeholders care about: the behaviour and structure of a system.

**Behaviour:** Make the machine fulfill the users requirements.

**Architecture:** Soft ware should be easy to change, the difficulty of the change should be proportional to the scope of change and not the "shape". Shape being the type of change requested.

**Eisenhower matrix:** A digram showing the combination of important and urgent. The conclusion is that generally things that are urgent should not be important and things that are important should not be urgent. Three big areas of architecture: function, separation of components and data management.

# Object orientated

First area thought to be introduced is Encapsulation of data and functions. Data should be kept within concepts and only accessed through this concept.

Good encapsulation is when the users of a program have or need no knowledge of the implementation of the data structure or function.

-   OO does not require perfect encapsulation, modern languages have declaration and definition of the classes tied together needing knowledge of one and other to use them.

**Inheritance:** Able to reuse classes and modules in different scopes. Encapsulation can exist without OO so this is not its feature.

## What is polymorphism ?

At its core polymorphism is pointers to functions. Allowing the pointer to dictate based on the given parameter the functions behaviour. This can be done without OO, however it makes it much safer and more convenient.
## How did OO effect the flow of dependency (Dependency inversion)

Prior to OO (polymorphism) flow of control had to match the direction of dependency.

-   All higher level functions are dependant on the implementation and
    the flow of control stems down to them from higher level function.

The dependency can be inverted using OO and interfaces; giving the architect control over the dependency tree. **Control order does not have to match dependency**

*Green arrow = Control , Red arrow = dependency*

![[Poly.png]]

-   The UI can depend on the business rules and the control flow stem
    from the rules.

![[Flow.png]]

The main thing OO introduces is a safe effective method of controlling source code dependencies !!

Vital to allow modularity and plugin nature of development.

### Functional Programming

Functional programs variable are not mutable, they do not vary.

**All concurrent update problems derive from mutable variables**

## How do you utilize immutable variables

If there was infinite resources (memory and processing) using immutable variables would be an option. Instead need to split the program into mutable and immutable components. The immutable components will feed the mutable components which then use transactional memory.

![[Components.png]]

Transactional memory acts as disk database with safety measures against race conditions; locking mechanisms on reading and writing.

## What is event sourcing ?

Store transactions instead of states to avoid mutable variables. The state is then calculated by summing transactions.

# Components

Well designed components are independently deployable and therefore
independently developable. Module managment tools: Maven, Leiningen and RVM.

Three principles associated with component design :
## Reuse/Release equivalence principle

Should be an overarching theme of the modules inside a component. To allow effective developing and reuse of a component there should be scheduled releases of new versions. And therefore the classes and components should be releasable together as a unit.

## Common closure principle

Gather classes/modules into a component that change for the same reasons at the same time. Much easier to change related classes in a single component than across many components.

## Common reuse principle

Dont force users of a component to depend on code they dont need. Place reused classes and modules together into a component. There should be a high dependency in a component's classes and modules.

If dependant on a component better to be completely dependant on the entire component.

## How do you balance the three principles

It is dependant on the needs of the software system and there will be tension between all three. i.e Early on development CCP is more important than REP ! As development is more important than reuse.

![[space.png]]
# Component Cohesion

## Acyclic dependencies principle

No cycles in the component dependency graph. Released components allow devs to work in isolated teams and decide which version to integrate with their dependant component.

If there is a dependency cycle between components multiple components will be forced to use the same version of a dependant components disrupting the isolated teams. To break a cycle create an interface component which will inverse the dependency. Or create another component between.

## Stable dependency principle

Do not make hard to change modules depend on easy to change modules. A method to make software stable and therefore hard to change is to have a lot of software dependant on it.

### How do you measure stability ?

By the number of dependencies:

Fan in: Incoming dependencies (Increasing this number is good for stability) 
Fan out: Outgoing dependencies

Instability : I = Fan out/(Fan in + Fan out), I=0 is maximum stability and I=1 is max unstable.

## Stable abstraction principle

A component should be as abstract as it is stable.

A stable component should be abstracted so it can be **extended**, it should be hard to change however being extendable does not effect this.

### How can you measure abstractness

Abstractness = Number of abstract classes and interfaces / number of classes in component

### Metric to determine good design

Distance from main sequence :

$$ D = A + I -1 $$

# Architecture

Architecture is about deployment, maintenance and ongoing development. Not directly linked with behaviour or operation however it can aid in this.

The focus is on reliability and development speed not performance. Good architecture is about making a system easy to understand, develop, deploy and maintain.

As many options as possible should be kept open for as long as possible by good design. Until you are in the most optimum position to make a judgment on an option (database, language, hardware etc).

The concept is to ensure policy is completely decoupled from details. Good architecture is about balancing when boundaries are needed and not needed and being flexible about this as it will be revealed as time progresses. Trade offs are always present.

![[main_seq.png]]
## Technical Breadth

All architecture will be iterative due to unknown unknown, the response is to reduce this by increasing your known unknowns - technical breadth. Careful to avoid the frozen caveman problem, constantly revisiting the same risk due to PTSD.

## Areas

Software architecture can be thought of as involving work in four paradigms.

-   Structure: Deployment method (Monolithic,Distributed)
-   Characteristics: Properties to focus on based on domain problem
    (Performance,Security,Scalability)
-   Decisions: Hard rules for constructing the system (only procedures
    can access the database)
-   Principles: Soft rules/guidance (Only use asynchronous comms)

## Characteristics

The list of characteristics supported should be as small as possible for simplicity.

### Operational Characteristics 

| Metric              | Definition                                                                                                                                                                  |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Availability**    | How long the system will need to be available (if 24/7, steps need to be in place to allow the system to be up and running quickly in case of any failure).                |
| **Performance**     | Includes stress testing, peak analysis, analysis of the frequency of functions used, capacity required, and response times. Performance acceptance sometimes requires an exercise of its own, taking months to complete. |
| **Reliability/safety** | Assess if the system needs to be fail-safe, or if it is mission critical in a way that affects lives. If it fails, will it cost the company large sums of money?             |
| **Scalability**     | Ability for the system to perform and operate as the number of users or requests increases.                                                                                  |

### Structural Characteristics

| Metric                  | Definition                                                                                                                                   |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **Configurability**     | Ability for the end users to easily change aspects of the software’s configuration (through usable interfaces).                               |
| **Extensibility**       | How important it is to plug new pieces of functionality in.                                                                                   |
| **Installability**      | Ease of system installation on all necessary platforms.                                                                                        |
| **Localization**        | Support for multiple languages on entry/query screens in data fields; on reports, multibyte character requirements and units of measure or currencies. |
| **Portability**         | Does the system need to run on more than one platform? (For example, does the frontend need to run against Oracle as well as SAP DB?)           |
| **Upgradeability**      | Ability to easily/quickly upgrade from a previous version of this application/solution to a newer version on servers and clients.                |
| **Leverageability/reuse** | Ability to leverage common components across multiple products.                                                                              |
| **Maintainability**     | How easy it is to apply changes and enhance the system?                                                                                        |
| **Supportability**      | What level of technical support is needed by the application? What level of logging and other facilities are required to debug errors in the system?|

### Cross cutting characteristics

| Metric               | Definition                                                                                                                                                                                                                               |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Accessibility**    | Access to all your users, including those with disabilities like colorblindness or hearing loss.                                                                                                                                         |
| **Archivability**    | Will the data need to be archived or deleted after a period of time? (For example, customer accounts are to be deleted after three months or marked as obsolete and archived to a secondary database for future access.)              |
| **Authentication**   | Security requirements to ensure users are who they say they are.                                                                                                                                                                          |
| **Legal**            | What legislative constraints is the system operating in (data protection, Sarbanes Oxley, GDPR, etc.)? What reservation rights does the company require? Any regulations regarding the way the application is to be built or deployed? |
| **Security**         | Does the data need to be encrypted in the database? Encrypted for network communication between internal systems? What type of authentication needs to be in place for remote user access?                                                    |
| **Usability/achievability** | Level of training required for users to achieve their goals with the application/solution. Usability requirements need to be treated as seriously as any other architectural issue.                                                |
| **Authorization**    | Security requirements to ensure users can access only certain functions within the application (by use case, subsystem, webpage, business rule, field level, etc.).                                                                         |
| **Privacy**          | Ability to hide transactions from internal company employees (encrypted transactions so even DBAs and network architects cannot see them).                                                                                                  |
| **Supportability**   | What level of technical support is needed by the application? What level of logging and other facilities are required to debug errors in the system?                                                                                        |

### Domain

| Domain concern             | Architecture characteristics               | Example                                                                                                                         |
|----------------------------|--------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| **Mergers and acquisitions** | Interoperability, scalability, adaptability, extensibility | The system needs to be able to integrate with the systems of the acquired company and scale to meet the needs of the larger organization. |
| **User satisfaction**      | Performance, availability, fault tolerance, testability, deployability, agility, security | The system needs to be responsive, always available, and secure in order to meet the needs of users. |
| **Time and budget**        | Simplicity, feasibility                | The system needs to be easy to develop and maintain within the constraints of time and budget. |
| **Time to market**         | Agility, testability, deployability     | The system needs to be developed and deployed quickly in order to meet market demands. |
| **Competitive advantage**  | Agility, testability, deployability, scalability, availability, fault tolerance | The system needs to be able to adapt to changing market conditions and provide a competitive advantage for the organization. |

## Size

The needs for different sized groups will be different. Small groups may find many interfaces and complete modularization a hindrance. A component per team may not be the optimal method of balancing deployment, operation and maintenance even if it makes development easier.

I.e Following strict micro service method will add heavier interconnections burden and bottleneck deployment, even if it helps development via strong boundaries. Immediate deployment should always be the goal.

## Maintenance

The primary maintenance cost is splunking; finding the best place in software to add a feature or repair. With the risk of changes causing unwanted bugs. Clean architecture should via screaming its use cases and strong boundaries limit this.

## Use Cases

The use case should be made clear by the architecture, behavior/features are easy to find via prominent modules. Many human resources have been wasted by decisions made on the details not relevant to the use case. These should be deffered as long as possible.
## Diagrams

Good to simplfy diagrams by only showing interface points and using splits to show different data flow streams.
# Dependency

Dependency can be thought of as import mechanisms, if a module or files requires an import or using statement then it is **dependant on the thing its is importing!**

**Connascense:** Change in one module requires change in another to keep the system working.

An interface allows for dependency inversion without changing the flow of control. This is vital to a modularized methodology. The key thing to check in dependency is if changes or breakages in a module effect other modules, if they do they are dependant.

Generally an interface is called via a data object allowing for polymorphic methods and independence.

![[Interface.jpeg]]

The lower level component should depend on the higher level ones. I.e DB access method should depend on the procedure that uses that data and not the other way. This allows for plugin design where details are dependant on procedures so they can be easily swapped.

**Vital to remember to create reusable interfaces/frameworks, must
create them concurrently with several plugins/reusing apps.**

Error handling becomes clear when there is clear dependency as it can happen post interface.

# Component Cohesion

A module should be determined by the axis of changes, is it expected for all elements in this module to change for the same reason and rate! A module hard to change should be easy to extend.
# Decoupling

Separate things that change for different reasons collect things that change for the same reason. (UI and procedures change for different reasons). All implementation details should change for different reasons than core procedures and therefore should be independent. Technical elements are the horizontal layers, and use cases/actors are the vertical layers. A use case may need many technical elements across the layers (UI,procedure and DB).

This may cause duplication, however if this will eventually diverge (accidental duplication) it is acceptable.The data passing between boundaries should not have structure that causes dependencies, as this will cause dependency inversion to be broken. It should be kept as simple as possible and then formatted after the boundary.

To determine coupling determine if any new features require coordination between components.

## Methods

-   Source code level: Monolithic structure, comms via functions calls
-   Deployment level: Individually deployed units, comms via sockets or
    shared mem.
-   Service level: Independent at source level, comms via network

Tip: Best to keep modules ready for separation as services, however not to separate unless good reason.

# Policies

Policies lead to procedures and should be separated and grouped together based on axis of change. The grouped policies are components on acyclic graph.

```        
func encrypt() {
	while(true)
    writeChar(translate(readChar()))
```

This is bad design as the procedure encrypt is dependant on the details.

A better design would be #Improved
## Critical

The highest level procedures should be those that could be done manually, at the core of product. These can be considered critical procedures.

# Partial Boundaries

If a full boundary might be needed in the future.

Facade classes are stand in place methods that don't have dependency inversion to access services.
# Testing

**Humble object pattern:** Separate easy to test presenter and hard to test view.


![[Improved.jpeg]] #Improved 

The view is the detail hardware and the presenter is the methods used to get the data to it. The view module should be kept very simple to allow the rest of the code to be easily tested. Interfaces allow views/controllers to be replaced with stubs or test doubles easily.

**Fragile tests:** Strong coupling between tests and code mean changes in the code cause all tests to break.

**Create a test API to access the procedures, decoupling the structure of the tests and procedures.**

# Main component

This should oversee and coordinate everything, it is the lowest level policy. Nothing should be dependant on it, other than the operating system.

Main should setup initial conditions and configurations and hand this over to higher level policies. If designed as a plugin allows many different configurations.

# Services

Services that don't follow the dependency inversion rule (using interfaces) are not architecturally significant, despite being computationally separate they maybe coupled via the data shared. If a change to the shared data causes many services to need change.

Each service should have its own boundaries dividing it into components, the real boundaries are not between the services they are interfaces communicating with each other. Unless the service is a single component surrounded by boundaries.

The benefit of service design is outside the goal of good architecture.

# Structures

-   Big ball of mud: None

-   Monolithic
	-   Layered
	-   Pipeline
	-   Microkernal

-   Distributed
	
	-   Service-based
	-   Event driven
	-   Space based
	-   Service-orientated
	-   Microservices

Distributed gives the benefit of performance,scalability and availability with a bunch of networking headaches.

## Layered

Separated components based on technical differences.

![[Layered.png]]
## Pipeline

Filter and pipe method, where data is passed through many stateless filters via pipes.

![[Pipeline.png]]
## Microkernal

Division of components by technical and domain. Gives plugin structure.

![[Microkernal.png]]
## Service-based

The basic topology of service-based architecture follows a distributed macro layered structure consisting of a separately deployed user interface, separately deployed remote coarse- grained services, and a monolithic database. The services split by domain.

![[Service-based.png]]
## Event-driven

Event-driven architecture is made up of decoupled event processing components that asynchronously receive and process events. It can be used as a standalone architecture style or embedded within other architecture styles (such as an event-driven microservices architecture).

Requests made to the system to perform some sort of action are send to a request orchestrator. The request orchestrator is to deterministically and synchronously direct the request to various request processors. The request processors handle the request, either retrieving or updating information in a database.

![[Event_Driven.png]]
## Space based

Space-based architecture gets its name from the concept of tuple space, the technique of using multiple parallel processors communicating through shared memory. High scalability, high elasticity, and high performance are achieved by removing the central database as a synchronous constraint in the system and instead leveraging replicated in-memory data grids. 

Application data is kept in- memory and replicated among all the active processing units. When a processing unit updates data, it asynchronously sends that data to the database, usually via messaging with persistent queues. Processing units start up and shut down dynamically as user load increases and decreases, thereby addressing variable scalability. Because there is no central database involved in the standard transactional processing of the application, the database bottleneck is removed, thus providing near- infinite scalability within the application.

![[Space_based.png]]
## Microservices

Microservices contain an API layer which guide request to each service which is a standalone program and database. each service runs in its own process, which originally implied a physical computer but quickly evolved to virtual machines and containers.

![[Microservices.png]]
# Embedded software

Issue of deep mingling between hardware elements and procedures.

## Hardware abstraction layers (HAL)

Line between firmware and software is the HAL, the HAL handles the softwares need and access to the the hardware. The software should not know anything about the hardware instead use concepts. Ie BatteryLow NOT blinkLED.

The HAL also provides substitution points for the hardware to be used for testing to avoid the target hardware bottleneck.

**Target Hardware Bottleneck:** Code can be only tested on the hardware.

## Processor abstraction layer (PAL)

Separates the firmware into separate pieces to allow off hardware testing. Making the processor a plugin element. Can do a similar thing with an OS.

# Example

1.  Determine actors and use cases
2.  Create component architecture
3.  Ensure dependency is appropriate for policy levels
4.  Determine deployment method of components

## Use Cases

Actors should be connected to use cases, abstract use cases can be used that connect other use cases which add details to the abstract.

## Components

Example pattern: View, presenter, interactor and controller are separated

-   Controller: Handle input
-   Interactor: Process input into usable data
-   Presenter: Format results
-   Viewer: Display result

The vertical splits will be different actors/use cases.
## Types

### Layered Architecture

Boundaries drawn by technical difference, ie web, procedure and storage.

Doesn't have any link to the business or use case.

### Feature Architecture

The boundaries are around features, but doesn't effectively separate technologies.

### Component Architecture

Use boundaries and deployment method to separate technologies and use cases. A hybrid of the previous.

## Encapsulation

If components have free access to each other, the architecture can be easily ignored. Utilizing encapsulation allows the complier to ensure the architecture.

# Risk

Use risk rating system which scores the impact of risk (1-3) and then the likelihood (1-3) and multiples for the final risk score. This score should be marked against the supported characteristics, showing direction from previous scores.

# Bibliography

[1 ] R. C. Martin, Clean Architecture: A Craftsman’s Guide to Software Structure and Design (Robert C. Martin Series). Boston, MA: Prentice Hall, 2017, ISBN: 978-0-13-449416-6. [On- line]. Available: https://www.safaribooksonline.com/library/view/clean- architecture-a/9780134494272/.  
[2] M. Richards and N. Ford, Fundamentals of Software Architecture: An Engineering Approach. O’Reilly Media, Incorporated, 2020, ISBN: 9781492043454. [Online]. Available: https:// books.google.co.uk/books?id=%5C_pNdwgEACAAJ.