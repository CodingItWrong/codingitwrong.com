So far in the Rails architecture series, we've talked about the controversy over different approaches, as well as talking about the why and how of creating new types of class. I'd like to look in more detail about the most controversial of those classes: the ones related to secondary effects.

When I say "secondary effects," here's what I mean. The primary effect of a write operation on the web operation is almost always a create, update, or delete on a database records. But there are often many secondary effects as well: creating or updating additional records, writing to logs, sending notifications via email, text, or mobile apps. I would even consider taking payment a secondary effect, for this reason: no system ever *just* takes payment: they always save a database record for an order or a donation. So placing the order or donation is the primary effect, and charging the card is the secondary effect.

There are several different patterns about how to implement secondary effects, with vocal adherents of each. What makes secondary effects more controversial than other application needs? I think the reason is because one of the biggest differences between hobby apps and enterprise apps is the quantity and complexity of callbacks. A hobby app might not have any secondary effects for most of its operations, whereas an enterprise app might do extensive logging, profiling, and auditing for even the simplest operation. With needs that different, it's no wonder very different patterns might be needed for the two cases.

With that in mind, let's take a look at the different patterns for secondary effects and when they're useful.

## Callbacks

[Callbacks](http://guides.rubyonrails.org/active_record_callbacks.html) are the mechanism built in to Active Record for firing off secondary effects when models are created, updated, or deleted. There are a few downsides to this mechanism, though. The model is responsible for having full knowledge of all secondary effects, which contributes to large, complex models. Also, the callbacks are always fired, so if they're only needed some of the time then the model is *also* responsible for conditional logic around the secondary effects.

Because of this, Big Nerd Ranch's "Ruby on the Server" course materials recommend only using callbacks for the simplest cases: specifically, updating fields on the model for data consistency, such as setting a GUID or permalink field. Other needs for secondary effects are often best handled by another pattern.

## Form Objects

[Form objects](http://culttt.com/2015/11/04/using-form-objects-in-ruby-on-rails) represent the specific form being submitted (or the specific web service request being made) as its own object, independent of the model itself. There are at least two motivations for this.

First, when a form corresponds to multiple model objects, the single form object can take the input, validate it, and save each separate model as needed. In particular, Bryan Helpcamp from Code Climate recommends this approach to using `accepts_nested_attributes_for`.

Second, when certain validations are only needed on one form and not on the model in general, the form object can implement that validation, simplifying the model's validations to only include rules that always apply. The canonical example of conditional validations is user signup. On the signup form, a cleartext password and confirmation are required, but they are not required on other forms, such as an email update form. If all validation is on the model, this conditional validation needs to be handled in the model. But with form objects, the model itself will just not require a cleartext password, while the signup form *will* require it.

The Form Object pattern goes by a few other names. Jay Fields introduced the pattern and referred to it as [the Presenter pattern](http://blog.jayfields.com/2007/03/rails-presenter-pattern.html), but that name is now also used for the View Object pattern. The Laravel framework has this pattern built-in, calling them [Form Requests](http://laravel.com/docs/5.1/validation#form-request-validation).

## General Decorators

Decorator is a classic OO pattern where an object is wrapped by another object with the same or a similar interface, to add behavior. (I'm calling this pattern "General Decorator" to contrast it with "Draper-style Decorators," which are for display logic instead.) General decorators can be used for models to [add secondary effects upon saving](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/), such as sending email. The decorator's `save` method would call the model's `save` method. Then, upon success, the decorator would send an email.

With general decorators, callbacks aren't always fired. The caller is responsible for wrapping the model in whichever decorators it needs. This is a very flexible approach, but it means that the caller is responsible to know which secondary effects it needs. If a new secondary effect needs to be added, all callers that need that effect must be updated. The implications of this will be explored below.

## Service Object

Service Objects are the poster child for not doing things "The Rails Way," but they do offer unique benefits compared to other patterns. The service object corresponds a single business use case, and handles calling the model for persistence as well as executing any necessary secondary effects. Because of this, neither the model nor the caller need to know about secondary effects, simplifying both of them. The caller *does* need to know which Service Object to call, but this should be fairly obvious if the Service Object really is named after a use case.

Andy Croll observed that [Rails Active Jobs are conceptually very similar to Service Objects](http://youtu.be/60LH3em78V8), and that they can offer a "Rails Way" to do Service Objects. This is in fact exactly the way the Laravel framework implements service objects out of the box: it has a concept of [jobs that can be synchronous or asynchronous](http://laravel.com/docs/5.1/queues) (although the documentation currently emphasizes their asynchronous aspect).

In a sample project of design patterns](http://youtu.be/bHpVdOzrvkE) code reviewed by DHH, Service Object was the pattern he most strenuously objected to. But in this case it seems like the Service Object was less about wrapping secondary effects and more about application logic that really belonged in the controller. So although DHH says he has never seen a good scenario for a Service Object, I haven't yet run across an example of what specifically he would say about a good one.

## Events and Event Handlers

Events aren't discussed much in the Rails ecosystem, but they offer a unique way to reduce coupling. In an Event/Handler approach, when a model is saved, it can tell the framework that a certain event has happened, such as "UserRegistered". Then, the framework checks to see what handlers are registered to handle that event: for example, a RegistrationMailer. Each of these handlers is called in turn.

Events are similar to Service Objects in that the knowledge of secondary effects lives in an independent location, not in the model or the caller. The difference is that the caller uses the model directly, and the secondary effects are always called.

Support for Events is built into Rails in the [`ActiveSupport::Notifications`](http://api.rubyonrails.org/classes/ActiveSupport/Notifications.html) module. It's not very widely used, perhaps because it's described specifically in terms of instrumentation; but [it can be used](http://youtu.be/dgUhP606F9w) for any application-specific event/handler needs.

## How to Decide

I hinted at some of the differences between these patterns above, but how do you decide which, if any, are right for your application? Here are my suggestions:

- If you just need to update fields in the same model, use callbacks. You can still do this for field updates even if you also need other patterns for more complex needs.
- If you have a form with multiple models, or form-specific validation, use a form object. But I don't recommend using form objects if you have other secondary effects: a service object is probably better in that case.
- If you have a few secondary effects that you use rarely, or that aren't on the critical path, use model decorators
- If you have secondary effects that aren't on the critical path, and they're always used, use events
- If you have many secondary effects called from multiple places in different combinations, or if they're on the critical path, use service objects
- If your service objects are getting complex or repetitive, you can have them use general decorators or broadcast events instead of issuing effects directly.

Callbacks
- Maps to model
- Always called
- Knowledge in model

Form Requests
- Does not map to model
- Always called
- Knowledge in form request

General Decorators
- Maps to model
- Not always called
- Knowledge in caller

Service Objects
- Does not map to model
- Not always called
- Knowledge in 3rd party object

Events and Handlers
- Maps to model
- Always called
- Knowledge in 3rd party object
