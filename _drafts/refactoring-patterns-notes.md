More from Help Me talk.

- Query Objects
- Value Objects: no one talks about this much, probably because ActiveRecord makes it hard.
- Policy Objects: for reads, especially permissions checks
- Draper-style Decorator: additional display logic on a model, instead of in helpers
- View Objects: Multiple models out. Many in the Rails community call this a Presenter. Multiple models

- Validators: you can write really complex ones. But just use if you're really submitting one thing.
- Form Objects: Multiple models in. Laravel calls these FormRequests. Jay Fields calls this a Presenter. Can also use for form-specific conditional validation; model validation is just the core. Example is password/conf. Good alternative to accepts-nested-attributes.
- Callbacks: BNR says use for data integrity only.
- General Decorator: wrap a model with secondary effects, instead of callbacks.
- Service Objects: BNR says use for more complex effects instead of callbacks. Especially if coordinating external services like payment.

