In the default Rails stack, four types of component work together to generate HTML: the view templates themselves, partials, models, and helpers. Of course, the main responsibility of the view is to output the markup of the page, based on the data in the model (or multiple models). As display logic gets more complex in the template, there are three places you can abstract that logic to.

- If you just have a portion of markup that needs to be reused, or is complex enough that it makes it hard to understand the overall flow of the page's markup, you can place it in a partial.
- Helpers are more useful for if you need to generate markup but it's more convenient to build up the HTML string from within plain Ruby code.
- If you don't need to generate markup at all, but rather do a data operation like formatting a date or dollar amount, or concatenating fields in a standard way, you can add methods to the model itself.

This separation is a good start, but there are a few additional extractions we can make to make our code simpler and more object-oriented.

## Draper-Style Decorators

## View Objects

The problem that View Objects address is that controllers can get cluttered up with code to load and arrange data in the way the view needs it. This isn't the main responsibility of a controller: what it *needs* to do is accept input, route it to the appropriate models, then return the correct response to the browser: either a view or a redirect. By separating out data loading, we can simplify our controllers and make them more readable. This means that only a single object is loaded in the controller for the view: the View Object. It handles the loading of all other necessary objects.

View Objects can also simplify our view templates. We all know views should only contain display logic: things like loops and conditionals on the data passed in to it. But even display logic can get pretty cluttered up with things like choosing which CSS class to add to an element, drilling down multiple levels into an object, etc. Logic like this can be moved to the View Object. This results in views that are much more readable, because they're almost entirely markup and simple output ERB tags.

## Value Objects



- View: just markup output. For real.
- View object, view model, presenter: model-agnostic display logic. Multiple models out. Many in the Rails community call this a Presenter. Multiple models. View display logic not directly related to one model.
- Draper decorator: model-specific display logic
- Value objects: field-specific display logic. no one talks about this much, probably because ActiveRecord makes it hard.
- Policy Objects: for reads, especially permissions checks
- Query Objects: for complex queries
- Model: just the core concerns of the business object