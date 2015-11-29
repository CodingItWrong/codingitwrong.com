In the default Rails stack, four types of component work together to generate HTML: the view templates themselves, partials, models, and helpers. Of course, the main responsibility of the view is to output the markup of the page, based on the data in the model (or multiple models). As display logic gets more complex in the template, there are three places you can abstract that logic to.

- If you just have a portion of markup that needs to be reused, or is complex enough that it makes it hard to understand the overall flow of the page's markup, you can place it in a partial.
- Helpers are more useful for if you need to generate markup but it's more convenient to build up the HTML string from within plain Ruby code.

Generally, if the logic generates HTML, it's placed into the helper, since it's considered part of the view/controller layer. This leaves the model clean from any markup, but it may still have some display logic, such as a `display_name` field that combines `first_name` and `last_name`.

This separation is a good start, but there are a few additional extractions we can make to make our code simpler and more object-oriented

- View: just markup output. For real.
- View object, view model, presenter: model-agnostic display logic. Multiple models out. Many in the Rails community call this a Presenter. Multiple models. View display logic not directly related to one model.
- Draper decorator: model-specific display logic
- Value objects: field-specific display logic. no one talks about this much, probably because ActiveRecord makes it hard.
- Policy Objects: for reads, especially permissions checks
- Query Objects: for complex queries
- Model: just the core concerns of the business object