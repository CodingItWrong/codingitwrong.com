DISPLAY LOGIC

- View: just markup output. For real.
- View object, view model, presenter: model-agnostic display logic. Multiple models out. Many in the Rails community call this a Presenter. Multiple models. View display logic not directly related to one model.
- Draper decorator: model-specific display logic
- Value objects: field-specific display logic. no one talks about this much, probably because ActiveRecord makes it hard.
- Policy Objects: for reads, especially permissions checks
- Model: just the core concerns of the business object

SECONDARY EFFECTS

- Form object: for form-specific validation or coordinating of multiple models. If have secondary effects, probably better to call it a service object. Multiple models in. Laravel calls these FormRequests. Jay Fields calls this a Presenter. Can also use for form-specific conditional validation; model validation is just the core. Example is password/conf. Good alternative to accepts-nested-attributes.
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
