---
title: Patterns
---
Software design patterns are reusable solutions to common problems.

The seminal books on design patterns are [*Design Patterns*](https://en.wikipedia.org/wiki/Design_Patterns) by the "Gang of Four" and [*Patterns of Enterprise Application Architecture*](http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420).

Notable patterns related to web development:

* **ActiveRecord Model**: an object corresponding to a record in the database, along with all the operations necessary to make changes to it in the database.
* **Command**: store a request as an object, so that different operations can be done to it, including queueing, logging, etc.
* **Decorator**: wraps an object with another object of the same interface, to add additional functionality.
* **Domain Model**: objects that represent busines entities and operations, but are unrelated to database persistence.
* **Event**: a mechanism for notifying other objects ("listeners") of things that happen in this object.
* **Form Object**: an object that corresponds directly to fields on a form, which may not directly correspond to the fields in the database. For example, a form may have fields from two different database tables.
* **Presenter**: a representation of the data to be displayed in a view. Allows removing almost all logic from the view.
* **Queued Jobs**: records a request to do a task to be done separately from the user's request/response cycle.
* **Serializer**: handles transforming an object into another representation, such as JSON.
* **Service Object**: performs a single business operation, independently of a web user interface or database persistence.
