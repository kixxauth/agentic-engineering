## Coding conventions and guidelines for designing and creating good code

The primary objective of any codebase is to create software which provides some useful function. The secondary objective of any codebase is to remain malleable -- easily updated -- to accommodate changes detected in the requirements or environment in which the software operates.

This document addresses the second objective of remaining malleable; easily updated.

Most future changes to any codebase will be made by extending it to add new capabilities. It is important that you or someone else will be able to add new capabilities without breaking the current functionality. Your most important role is to facilitate those future extensions even when you cannot know what they will be.

Your most significant problem when adopting future changes will be the complexity of the codebase. But, there are two different forms of complexity to consider:

1. **Inherent Complexity** is the complexity inherent in the problem domain the codebase is solving for. Inherent complexity is necessary complexity, because without that complexity the codebase would provide no value. The codebase is valuable because it encapsulates and hides inherent complexity, allowing the system to take on higher order problems. For example; a codebase may encapsulate the complexity of working with a disk drive so the operating system can focus on providing a unified filesystem interface.
2. **Accidental complexity** is introduced into the codebase because non-optimal design decisions were made when writing code. Accidental complexity is bad because it makes the code hard to extend and does not add value. Your focus when designing solutions and writing code is to eliminate accidental complexity, and that is what the remainder of this document is about.

So, do not think of working code as your primary goal. Your primary goal is a good design for the code that hides inherent complexity, eliminates accidental complexity, and which also happens to work.

## Object Oriented Programming
Your primary tool for encapsulating Inherent Complexity and eliminating Accidental Complexity is the Object Oriented Programming paradigm (OOP). In OOP, software is designed as a collection of interacting objects rather than a mere list of instructions or functions. By breaking a program into modular, self-contained objects, it becomes significantly easier to maintain, extend, and debug over time. It shifts the focus from how a task is done to what entity is responsible for doing it.

A good mental model for you to use to use OOP is to think of the codebase as made up of a community of tiny little cooperating machines (objects). And, each machine has more cooperating machines within it. An object is a machine capable of performing specific tasks. You assign tasks for the object to perform, it is responsible for those tasks, and has access to all the knowledge and resources required to perform those tasks.

## Everything is an object
As you begin to think about the problem domain and requirements of the system, it helps if you assume that every aspect of the problem domain can be modeled as an object in the system you are designing.

- Think of data as objects and no object is passive. Whatever manipulations and transformations are required of an object, are realized by that object itself instead of some other kind of thing acting upon that object.
- Think of relationships as objects with their own responsibilities and capabilities.
- Think of procedures as objects. Procedures are the vital force which compose other objects together and allows them to interact. A procedure is nothing more than an organized collection of messages, and both the collection and the messages are objects.

An object can be a component in the system made up of several JavaScript modules. An object can be a single JavaScript module. An object can be a class, or instances of a class. An object can even be a method on a class, or a conditional logic within that method.

## Information Hiding
In an ideal world, each object would be completely independent of the others: you could work in any of the objects without knowing anything about any of the other objects. When an object is independent it encapsulates the good inherent complexity and avoids leaking out the bad accidental complexity. In this world, the complexity of a system would be the complexity of its worst object.

Encapsulating inherent complexity in an object is also known as "information hiding". The information hidden within a module usually consists of details about how to implement some mechanism.

Information hiding reduces accidental complexity in two ways:

1. The interface reflects a simpler, more abstract view of the object's functionality and hides the details.
2. If a piece of information is hidden, there are no dependencies on that information outside the object which makes it easier to extend the codebase.

Your objects should hide their data and information behind abstractions and expose methods which operate on that data.

## Objects Should be General Purpose
General purpose objects are almost always better than special purpose ones. General purpose classes and methods usually have simpler interfaces and hide more complexity than specialized ones.

One way to separate specialized code is to push it downwards. An example of this is device drivers. An operating system typically must support many different device types of devices. Each of these device types has its own specialized command set. In order to prevent specialized device characteristics from leaking into the main operating system code, operating systems define an interface with general-purpose operations that any secondary storage device must implement.

Another way to separate specialized code is to push it upwards. The top-level classes of an application, which provide specific features, will necessarily be specialized for those features. That specialization should be contained in those classes.

## Open Closed Principle
The Open-Closed Principle (OCP) states that classes, modules, and methods should be open for extension, but closed for modification.

This means you should be able to add new functionality to a system without altering the existing, working code. Think about that very carefully: If you could add new features to the codebase without modifying any old code then features would be added solely by writing new code.

Breaking Down the Concept:

- **Open for Extension:** The behavior of an object can be extended. As the requirements of the application change, we can make the module behave in new and different ways to meet those requirements.
- **Closed for Modification:** Extending the behavior of an object does not result in changes to the existing behavior. The original code remains untouched and stable.

This means that extending a class to override methods on it is almost always a bad idea. If a method needs to be overridden, then it probably should not exist on the base class. Instead you should move implementations of that method to specialized sub classes which are designed to encapsulate that specific functionality.

Instead, you should construct interfaces or abstract base classes rather than concrete implementations to create a "plug-and-play" architecture where new logic can be plugged in without rewriting the host.

## Overdoing Decomposition
It might appear that the best way to achieve the objective of good design is to divide the system into a large number of small components: the smaller the components, the simpler each individual component is likely to be.

However, the act of subdividing creates additional complexity:

- Some complexity comes just from the high number of components.
- Subdivision usually results in more interfaces, and every new interface adds complexity.
- Subdivision can result in additional code to manage the components.
- Subdivided components will be farther apart making it harder to map dependencies.
- Separation makes it harder for you to understand the components at the same time which makes it difficult to map the dependencies between them.
- Subdivision can result in duplication: code that was present in a single instance before subdivision may need to be present in each of the subdivided components.

Creating small classes and small methods is not your goal when writing or refactoring code. In fact, there are times when you should bring code together instead. Bringing pieces of code together is most beneficial if they are closely related:

- They share information; for example, both pieces of code might depend on information about the HTTP protocol.
- They are used together: anyone using one of the pieces of code is likely to use the other as well.
- They overlap conceptually, in that there is a simple higher-level category that includes both of the pieces of code.
- It is hard to understand one of the pieces of code without looking at the other.

**Bring code together** if information is shared, so it is all in one place.

**Bring code together** if it will simplify the interface.

But, there are still reasons you should separate code:

**Separate code** when information is not shared.

**Separate code** when it does not address the same concern.

**Separate code** so that specialized code is separate from general purpose code.

## Dependency Injection
Dependency Injection (DI) is a software design pattern where an object’s dependencies—the services or objects it needs to function—are supplied by an external source rather than being created by the object itself.

Essentially, instead of a component saying, "I need to build a database connection," it says, "I need a database connection provided to me."

You should use dependency injection whenever you can because it makes the system simpler and easier to test. Dependency injection helps to decouple an object, make it more flexible for future changes, and makes it more testable.

Dependency Injection is typically achieved through:

- Constructor Injection: Passing dependencies through a class constructor.
- Setter Injection: Providing dependencies via a setter method after the object is created.
- Interface Injection: The dependency provides an interface that the client must implement to receive the dependency.

## Dependency Inversion Principle
The **Dependency Inversion Principle** (DIP) states:

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

Without DIP, code often looks like this:

```javascript
class EmailService {
  send(to, message) {
    // Implementation details
  }
}

class UserRegistration {
  constructor() {
    this.emailService = new EmailService();  // Direct dependency
  }
  
  register(user) {
    // ... registration logic
    this.emailService.send(user.email, 'Welcome!');
  }
}
```

The problem: `UserRegistration` (high-level) directly depends on `EmailService` (low-level). If you want to change email providers or test with a mock service, you have to modify `UserRegistration`.

Instead, both depend on an abstraction (interface/contract):

```javascript
class UserRegistration {
  constructor(notificationService) {
    this.notificationService = notificationService;  // Depends on abstraction
  }
  
  register(user) {
    // ... registration logic
    this.notificationService.notify(user.email, 'Welcome!');
  }
}

// Now you can swap implementations
class EmailService {
  notify(to, message) { /* ... */ }
}

class SlackService {
  notify(userId, message) { /* ... */ }
}

// Use whichever you want
const registration = new UserRegistration(new EmailService());
```

The benefits of dependency inversion for you are:

- **Loose coupling**: High-level code doesn't care about implementation details
- **Testability**: Easy to inject mock services for testing
- **Flexibility**: Swap implementations without changing the core logic
- **Maintainability**: Changes to low-level code don't break high-level code

The dependency inversion principle is about depending on *what something does* (the interface) rather than *how it does it* (the implementation).

## Avoid Throwing Exceptions
An exception is a particularly complex element of an interface. It can propagate up through several stack levels before being caught, so it affects not just the method’s caller, but potentially also higher-level callers and their interfaces. The best way to eliminate exception handling complexity is to define your APIs so that there are no exceptions to handle.

For example: When defining an `unset()` method, it may be tempting to throw an exception if the value to be unset does not exist. Instead, the purpose of `unset()` should be to ensure the pointer no longer exists, so if it does not exist then `unset()` silently returns as a no-op instead of throwing an exception.

The second technique for reducing the number of places where exceptions must be handled is exception masking. With this approach, an exceptional condition is detected and handled at a low level in the system, so that higher levels of software need not be aware of the condition. For instance, in a network transport protocol such as TCP, packets can be dropped for various reasons such as corruption and congestion. TCP masks packet loss by resending lost packets within its implementation, so all data eventually gets through and clients are unaware of the dropped packets.

## Create Good Methods
You should create methods which do one thing and only that thing. Methods should either do something or answer something, but not both. Either your method should change the state of an object, or it should return some information about that object. Doing both often leads to confusion.

In order to make sure our functions are doing one thing, you need to make sure that the statements within your method are all at the same level of abstraction. You want every method to be followed by those at the next level of abstraction so that we can read the program, descending one level of abstraction at a time as we read down the list of methods.

The ideal number of arguments for a method is zero (niladic). Next comes one (monadic), followed closely by two (dyadic). Three arguments (triadic) should be avoided where possible. More than three (polyadic) shouldn’t be used. When a method seems to need more than two or three arguments, it is likely that some of those arguments ought to be wrapped into a class or object of their own.

## Choose Good Names
Names for constants, classes, methods, and variables are important to reducing accidentall complexity. A good name conveys a lot of information about what the underlying entity is, and, just as important, what it is not. When considering a particular name, ask yourself: “When I see this name in the future, how closely will I be able to guess what the name refers to?".

When you look up an implementation of a thing by its name, there should be no surprises in what it does. The name of it should be a dead giveaway.

