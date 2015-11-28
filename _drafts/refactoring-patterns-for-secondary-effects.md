So far in the Rails architecture series, we've talked about the controversy over different approaches, as well as talking about the why and how of creating new types of class. I'd like to look in more detail about the most controversial of those classes: the ones related to secondary effects.

When I say "secondary effects," here's what I mean. The primary effect of a write operation on the web operation is almost always a create, update, or delete on a database records. But there are often many secondary effects as well: creating or updating additional records, writing to logs, sending notifications via email, text, or mobile apps. I would even consider taking payment a secondary effect, for this reason: no system ever *just* takes payment: they always save a database record for an order or a donation. So placing the order or donation is the primary effect, and charging the card is the secondary effect.



## Callbacks

This is the mechanism built in to Active Record for firing off secondary effects when models are created, updated, or deleted. There are a few downsides to this mechanism, though. The model is responsible for having full knowledge of all secondary effects. Also, the callbacks are always fired, so if they're only needed some of the time then the model is *also* responsible for knowing the conditions when they should be fired. Because of this, Big Nerd Ranch's "Ruby on the Server" course materials recommend only using callbacks for updating fields on the model for data consistency, such as setting a GUID or permalink field. Other needs for secondary effects should be handled by another pattern.

## Form Objects

Form objects represent the specific form being submitted (or the specific web service request being made) as its own object, independent of the model itself. There are at least two motivations for this. First, when a form corresponds to multiple model objects, the single form object can take the input, validate it, and save each separate model as needed. Second, when certain validations are only needed on one form and not on the model in general, the form object can implement that validation, simplifying the model's validations to only include rules that always apply. In particular, Bryan from Code Climate recommends this approach to using `accepts_nested_attributes_for`.

This pattern goes by a few other names. Jay Fields introduced the pattern and referred to it as the Presenter pattern, but that name is now also used for the View Object pattern. The Laravel framework has this pattern built-in, calling them FormRequests.

## General Decorators

Decorator is a classic OO pattern where an object is wrapped by another object with the same or a similar interface, to add behavior. This can be used for models to add secondary effects upon saving, such as sending email. The decorator's `save` method would call the model's `save` method, then, upon success, the decorator would send an email.

With general decorators, callbacks aren't always fired. The caller is responsible for wrapping the model in whichever decorators it needs. This is a very flexible approach. It also means that knowledge of secondary effects is the responsibility of the caller. If a new secondary effect needs to be added, all callers that need that effect must be updated. The implications of this will be explored below.

I'm calling this pattern "General Decorator" to contrast it with "Draper-style Decorators," which are for display logic instead.

## Service Object

Service Objects are the poster child for not doing things "The Rails Way," but they do offer unique benefits compared to other patterns. The service object corresponds a single business use case, and handles calling the model for persistence as well as initiating any necessary secondary effects. Because of this, neither the model nor the caller need to know about secondary effects, simplifying them. However, the caller *does* need to know which Service Object to call, but this should be clear if the Service Object really is named after a use case.

Someone observed that Service Objects are conceptually very similar to Active Jobs, and that this can offer a "Rails Way" to do Service Objects. This is in fact exactly what the Laravel framework does: jobs can be synchronous or asynchronous.

## Events and Event Handlers

Events aren't discussed much in the Rails ecosystem, but they offer a unique way to reduce coupling. In an Event/Handler approach, when a model is saved, it can tell the framework that a certain event has happened, such as "UserRegistered". Then, the framework checks to see what handlers are registered to handle that event: for example, a RegistrationMailer. Each of these handlers is called in turn.

Events are similar to Service Objects in that the knowledge of secondary effects lives in an independent location, not in the model or the caller. The difference is that the caller uses the model directly, and the secondary effects are always called.

Support for Events is built into Rails as `ActiveSupport::Notification`. It's not very widely used, perhaps because it's described in terms of specifically monitoring instrumentation; but it can be used for any application-specific event/handler needs.

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

Decision recommendation
- If you just need to update fields in the same model, use callbacks. This can be used with other patterns.
- If you have a form with multiple models, or form-specific validation, but no secondary effects, use a form object
- If you have a few secondary effects used rarely, use model decorators
- If you have secondary effects that are always used, use events
- If you have many secondary effects called from multiple places in different combinations, use service objects
- If your service objects are getting complex or repetitive, you can have them use general decorators or broadcast events instead of issuing effects directly.

- General decorator: secondary effects wrapped around model. Knowledge of effects in caller, not always fired. Best when there aren't many, and when conceptually you are mainly interacting with the model. Good for critical-path effects.
- Service object: secondary effects in independent class; an active job is a category of this. Knowledge of effects in independent class, but caller needs to know about it in the abstract, so not always fired. Could coordinate general decorators. Good for critical-path effects, like payment. If you aren't conceptually interacting with just one model, this is best. BNR says use for more complex effects instead of callbacks. Especially if coordinating external services like payment. Laravel calls these jobs. Also called command handlers.
- Events: secondary effects originating from model, bound outside of model. Always fired, but knowledge of effects not coupled to model OR caller. Good for non-critical-path effects, like logging or emailing.
- Callbacks: secondary effects IN model. BNR says just for internal stuff, like data consistency. Always fired, knowledge of effects in model.

- Concerns: separate module, included in same model; for related domains; could include callbacks, events, validators, value objects. Just a way for organizing built-in or external patterns within model itself.
- Validators
- Model

DIFFERENT APPLICATION LAYERS

- Rails web UI
- API (might be separate controllers)
- Rake tasks
- Queued jobs

What needs to be shared across all of those? And what is separate for each?
