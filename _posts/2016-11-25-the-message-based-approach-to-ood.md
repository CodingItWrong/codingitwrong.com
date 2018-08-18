---
title: The Message-Based Approach to OOD
tags: [oo]
---

In software development I usually hear the phrase "calling a method", but have you ever heard the phrase "sending a message" instead? I first ran across it in Objective-C: you *send a message* to an object, and if it has a corresponding method, that method is executed. For a long time I couldn't tell the difference between thinking in terms of method calls vs. thinking in terms of messages, but as I've gotten into Extreme Programming circles, the difference has started to click. The message-based approach is about a particular view of object design.

One of the main goals of OO design is managing dependencies between objects. Few dependencies mean it's less likely that a change in one object forces changes in lots of other objects. When your dependencies are under control, you're more likely to be able to reuse objects in new contexts. Objects with few dependencies are said to be "loosely coupled."

Loose coupling can be accomplished by focusing on encapsulation: hiding the implementation details of an object and exposing only a narrow public interface. (This is also referred to as a small "API surface area.") This can help decrease coupling by exposing fewer methods for other objects to use. The public interface should also be designed such that it's unlikely to change, to minimize changes needed in other objects. By contrast, the private implementation of the object can be changed without risk of affecting other objects, as long as the external behavior remains the same.

Ideally, an object knows what it needs another object to do and simply tells it to do that. It shouldn't need to know *how* the other object does its job and ask it to do each step. The latter approach is effectively the same as procedural code, even though it's implemented within objects. It involves a lot of unnecessary dependencies. If other objects need to perform the same operations, the series of method calls will need to be repeated there. And if changes happen in the receiver, many callers will need to change. Telling an object what you want it to do instead of asking it to perform each step is referred to as the "Tell, Don't Ask" principle.

When you need to send multiple messages to a receiver, this is a sign that there is a concept there that is lacking a name. What's the intent of this series of interactions? You can give that concept a name, send a message with that name, and make a method on the receiver that in turn sends the original *series* of messages to itself. At this point, the smaller methods may no longer need to be public, and you can narrow the public interface. Each caller just needs to send one message to the receiver telling it what it needs.

Here are a few signs that you may be thinking procedurally instead of thinking in terms of sending messages:

1. Sending a sequence of messages to the same collaborator. This may be evidence of knowledge of *how* to accomplish a task that belongs in the receiver, not the sender.
2. Sending a message to an object, receiving a result, and then sending a message to the result. The second object is part of the internals of the first, and shouldn't be exposed: this violates the Law of Demeter. You should be able to send a message to the first object with your overall intent.
3. Conditionals or switch statements that check a condition on an object or its type, and supply behavior based on the response. This behavior should be in the object itself, and polymorphism should be used to handle the differences in behavior. If the check is being performed on a primitive like an integer, then that primitive might need to be replaced with a smarter object that has the desired behavior.

Unit tests are an essential tool for understanding and improving the message interface of your objects. In my experience, it's particularly helpful to use isolation tests (where only the type under test is instantiated) and mocks (test doubles that allow you to check the messages they receive). Both of these tools give visibility into the messages your object sends, what it knows about other objects, and what it expects from the context. In particular, if you have to set up a lot of mocks for a test, especially mocks returning other mocks, that's an indication that the messages you send could be simplified so that you're telling, not asking.

## References

- [*99 Bottles of OOP*](http://www.sandimetz.com/99bottles/), chapter 5
- [*Clean Code*](http://www.informit.com/store/clean-code-a-handbook-of-agile-software-craftsmanship-9780132350884), chapter 6
- [*Growing Object-Oriented Software, Guided by Tests*](http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626), chapter 2
- [*Practical Object-Oriented Design in Ruby*](http://www.informit.com/store/practical-object-oriented-design-in-ruby-an-agile-primer-9780321721334), chapter 4
