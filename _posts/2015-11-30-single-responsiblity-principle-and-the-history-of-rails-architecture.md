---
title: Single Responsibility Principle and the History of Rails Architecture
---

{% include posts/toc_rails_architecture.md %}

Last week I wrote about [the controversy over Rails architecture]({% post_url 2015-11-22-the-rails-architecture-controversy %}). To start out our analysis of it, we need to look at where architectural patterns come from. The idea to add to an application's architecture doesn't just come out of thin air; it should be in response to problems caused by the app's current architecture.

One common problem is that classes tend to grow larger and larger, which makes them harder and harder to change without causing unforeseen bugs. In response to this problem, the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) (SRP) states that a class should only have one thing it's responsible for. If more than one thing needs to be accomplished, additional classes can be created, and the classes can call each other.

Despite how much is said today about how the Model-View-Controller pattern violates SRP, it was actually brought to the web as *an application of* SRP. At the inception of the web, web applications were implemented either as a single script for the whole app or a single script per page. These scripts handled everything from persistence to business logic to display. Splitting out models, views, and controllers was a first pass at applying SRP.

The next significant architectural change was from fat controllers to [fat models](http://weblog.jamisbuck.org/2006/10/18/skinny-controller-fat-model). Models tended to be simple subclasses of an ORM class, sometimes entirely empty. Controllers handled all logic, and tended to get very long. Ultimately, the observation was made that there were really two types of logic in the controllers: domain logic, for things like sending an email when a user registers, and application logic, for things like redirecting to a confirmation page. By moving the domain logic into the model, that leaves only the application logic in the controller, giving each class a single responsibility.

But what a "single responsibility" is changes over time. As web applications, and Rails applications in particular, got larger and larger, "domain logic" went from one responsibility to many. Large model classes became the bottleneck that slowed down development for the same reasons that led to SRP in the first place: too much code in one place making understanding difficult and changes risky.

In response, developers started finding various other responsibilities to separate out, mostly from the models. Unlike with MVC or Fat-Model, there wasn't a consensus on what the new pattern should be. Instead, the different needs of applications led to different patterns.

In a classic [post](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/) and [conference talk](https://youtu.be/5yX6ADjyqyE), Bryan Helmcamp summarized a number of these ways to refactor fat models. These included:

- **Value objects**: extract logic around a single field
- **Service objects**: extract complex business logic
- **Form objects**: extract operations on multiple models
- **Query objects**: extract complex custom SQL queries
- **View objects**: extract logic to display model data
- **Policy objects**: extract permissions checks
- **General Decorators**: extract callback logic

To dig more into Rails architecture options, we'll look into some of these in more detail, as well as a few others:

- **Draper-Style Decorators**: extract model-specific display logic from model
- **Events**: further decouple classes by making the "caller" not need to know everything that's called as a result of its operation

Once we have a sense of what these patterns are and how they work together, we'll be able to handle big-picture questions about how far down this path to go. If there are other patterns that play a significant role in your web application architecture, [let me know](https://twitter.com/CodingItWrong)!