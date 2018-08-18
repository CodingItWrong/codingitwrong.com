---
title: Refactoring Patterns for Secondary Effects
tags: [rails]
---

{% include posts/toc_rails_architecture.md %}

In my last post, we looked at [patterns for display logic]({% post_url 2015-12-07-refactoring-patterns-for-display-logic %}) in Rails architecture. Now I'd like to look at a more controversial category of pattern: the ones related to secondary effects.

When I say "secondary effects," here's what I mean. The primary effect of a write operation on the web is usually a create, update, or delete on a database record. But there are often secondary effects as well: for example, creating or updating secondary records, writing to logs, or sending notifications via email, text, or mobile apps. I would even consider taking payment to be a secondary effect, for this reason: no system ever *just* takes payment; they always save a database record for an order or a donation. So placing the order or donation is the primary effect, and charging the card is the secondary effect.

There are several different patterns about how to implement secondary effects, with vocal adherents of each. What makes secondary effects more controversial than other application needs? I think the reason is that the biggest difference between hobby apps and enterprise apps is the quantity and complexity of these secondary effects. A hobby app might not have any secondary effects for most of its operations, whereas an enterprise app might do extensive logging, profiling, and auditing for even the simplest operation. With such different needs, it's no wonder very different patterns might be preferred for the two cases.

One additional term that will be helpful as we consider secondary effects is "critical path." When I say "critical path," I mean a secondary effect that must be completed before you can save the record. For example, charging a credit card is on the critical path, because you don't want to save an order record until the payment is taken. But sending a confirmation email is not on the critical path, because it can be sent after the order record is saved--even after a delay. ([The *GOOS* book](http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626) calls the critical path "dependencies" and things not on the critical path "notifications".)

With that in mind, let's take a look at some patterns for implementing secondary effects.

## Callbacks

[Callbacks](http://guides.rubyonrails.org/active_record_callbacks.html) are the mechanism built in to Active Record for firing off secondary effects when models are created, updated, or deleted. All you have to do is configure a method on the model to be called before or after one of those operations. There are a few downsides to this mechanism, though. The model is responsible for having full knowledge of all secondary effects, which contributes to large, complex models. Also, the callbacks are always fired, so if they're only needed some of the time then the model is *also* responsible for conditional logic around the secondary effects. Because they're always fired immediately, callbacks are best for critical path effects.

Because of this, Big Nerd Ranch's ["Ruby on the Server"](https://training.bignerdranch.com/classes/ruby-on-the-server) course materials recommend only using callbacks for the simplest cases: specifically, updating fields on the model for data consistency, such as setting a GUID or permalink field. Other needs for secondary effects are often best handled by another pattern.

## Form Objects

[Form objects](http://culttt.com/2015/11/04/using-form-objects-in-ruby-on-rails) represent the form being submitted as its own object that doesn't directly correspond to a database model. Even though it's named after forms, this pattern isn't limited to GUI web applications: it can be applied to web service requests as well. There are at least two reasons you might use this pattern.

First, when a form includes fields corresponding to multiple model objects, a single form object can take the input, validate it all at once, then save each model. Bryan Helmcamp from Code Climate [recommends this approach](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/) over using `accepts_nested_attributes_for`.

Second, when certain validations aren't always needed for a model but only for certain forms, the form object can implement that situational validation. Then the model's validations can be limited to the rules that always apply, keeping them simpler and making the model more reusable. The canonical example of conditional validations is user signup. On the signup form, a cleartext password and confirmation are required, but they are not required on other forms, such as an email update form. If all validation is on the model, then these rules need to be defined as conditionals. But with form objects, the model allows the cleartext password field to be optional and the signup form requires it. This way, neither requires more complex conditional validation rules.

The Form Object pattern goes by a few other names. Jay Fields introduced the pattern and referred to it as [the Presenter pattern](http://blog.jayfields.com/2007/03/rails-presenter-pattern.html), but that name is now commonly used in the Rails community for the View Object pattern. The Laravel framework has the Form Object pattern built-in, referred to as [Form Requests](http://laravel.com/docs/5.1/validation#form-request-validation).

## General Decorators

Decorator is a [classic OO pattern](https://en.wikipedia.org/wiki/Design_Patterns) where an object is wrapped in another object with the same or a similar interface, to add additional behavior. (I'm calling this pattern "General Decorator" to contrast it with ["Draper-style Decorators,"]({% post_url 2015-12-07-refactoring-patterns-for-display-logic %}) which are for display logic instead.) General Decorators can be used for models to [add secondary effects upon saving](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/), such as sending email. The decorator's `save` method would call the model's `save` method, as well as additional secondary effects before or afterward. Different secondary effects can even be separated into their own General Decorators, to provide more flexibility to use them in different combinations,

With General Decorators, the caller is responsible for picking and choosing which General Decorators it needs to use. This approach provides a lot of flexibility because secondary effects don't always need to be called. However, it also means that the caller is responsible to know which secondary effects it needs. If a new General Decorator is added, all callers that need that effect must be updated. Because of this "optional" nature, I don't recommend General Decorators for the critical path: a more deterministic pattern can be a better fit for those.

## Service Object

Service Objects are the poster child for not doing things "The Rails Way," but they do offer unique benefits compared to other patterns. The Service Object corresponds a single business use case, and handles calling the model for persistence as well as executing any necessary secondary effects. Because of this, neither the model nor the caller need to know about the secondary effects, which simplifies both of them. The caller *does* need to know which Service Object to call—but this should be fairly obvious if the Service Object really is named after a use case. Service Objects are great for orchestrating secondary effects both on and off the critical path.

Andy Croll observed that [Rails Active Jobs are conceptually very similar to Service Objects](http://youtu.be/60LH3em78V8), and that they can offer a "Rails Way" to do Service Objects. This is in fact exactly the way the Laravel framework implements service objects out of the box: it has a concept of [jobs that can be synchronous or asynchronous](http://laravel.com/docs/5.1/queues) (although the documentation currently emphasizes their asynchronous aspect).

In a [sample project of design patterns](http://youtu.be/bHpVdOzrvkE) code-reviewed by DHH, Service Object was the pattern he most strenuously objected to. But in that case it seems like the Service Object was less about wrapping secondary effects and more about application and validation logic--which really belong in the controller and model, respectively. So although DHH says he has never seen a good scenario for a Service Object, I feel like secondary effects are just such a scenario, and I haven't seen what DHH would say about those.

## Events and Event Handlers

Events aren't discussed much in the Rails ecosystem, but they offer a unique way to reduce coupling. In an Event/Event Handler approach, when a model is saved, it can inform the framework that an event occurred, using an application-specific name such as "UserRegistered". Then the framework checks to see if any handlers are registered to handle that event. For example, a UserMailer might be registered to send a confirmation email. The framework then calls each of these event handlers in turn.

Events are similar to Service Objects in that neither the model nor the caller has knowledge of the secondary effects—that knowledge lives in an independent location. The difference is that, with Events, the caller uses the model directly and the secondary effects are always called. Also, compared to service objects, events are a looser coupling, because it adds a level of indirection between calling the primary and secondary effects. This is often better for code reuse, but it can also be harder to follow what's going on. For that reason, I recommend using a more explicit pattern for the critical path, so that those essential secondary effects can be seen directly in the code.

Support for Events is built into Rails in the [`ActiveSupport::Notifications`](http://api.rubyonrails.org/classes/ActiveSupport/Notifications.html) module. It's not very widely used for custom application events, perhaps because the documentation sounds like it's only intended to be used for instrumentation; but [it can be used](http://youtu.be/dgUhP606F9w) for any other event/handler needs that your application has as well.

## How to Decide

I described some of the differences between these patterns above, but how do you decide which, if any, are right for your application? I haven't used all of these patterns extensively enough to have a decisive opinion, but here are my initial thoughts:

- If you just need to update fields in the same model, use Callbacks. You can still do this for field updates even if you also need other patterns for more complex needs.
- If you have a form with multiple models or with form-specific validation, use a Form Object. But I don't recommend adding secondary effects to a Form Object.
- If you have only few secondary effects that you use rarely, or that aren't on the critical path, use General Decorators.
- If you have secondary effects that aren't on the critical path, and they're always used, use Events.
- If you have many secondary effects called from multiple places in different combinations, or if they're on the critical path, use Service Objects.
- If your Service Objects are getting complex or repetitive, you can have *them* use General Decorators or broadcast Events instead of calling secondary effects directly.

What do you think? How do you decide what patterns to use for secondary effects? Are there other patterns I missed? [Let me know!](https://twitter.com/CodingItWrong)
