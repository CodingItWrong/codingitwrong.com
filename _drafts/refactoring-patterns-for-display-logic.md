In the default Rails stack, four types of component work together to generate HTML: the view templates themselves, partials, models, and helpers. Of course, the main responsibility of the view is to output the markup of the page, based on the data in the model (or multiple models). As display logic gets more complex in the template, there are three places you can abstract that logic to.

- If you just have a portion of markup that needs to be reused, or is complex enough that it makes it hard to understand the overall flow of the page's markup, you can place it in a partial.
- Helpers are more useful for if you need to generate markup but it's more convenient to build up the HTML string from within plain Ruby code.
- If you don't need to generate markup at all, but rather do a data operation like formatting a date or dollar amount, or concatenating fields in a standard way, you can add methods to the model itself.

This separation is a good start, but there are a few additional extractions we can make to make our code simpler and more object-oriented.

## Draper-Style Decorators

Draper-style Decorators provide a place to put model-specific display logic all in one place. This is a way to separate out the core domain responsibilities of the model from display responsibilities. Such logic is also sometimes placed in helpers, but Draper-style Decorators are objects, making them easier to work with and test in some ways.

This is an application of the general Decorator pattern, which involved wrapping an object with another object to add additional behavior. In this case, the model object is wrapped in another object that adds formatting. Usually, all the model's methods are directly accessible, as well as additional formatting methods.

I use the term "Draper-style" because Draper is a common Ruby gem for applying this pattern. The Decorator pattern can also be used for applying secondary effects to model save operations, which I'll discuss in a future post.

## View Objects

The problem that View Objects address is that controllers can get cluttered up with code to load and arrange data in the way the view needs it. This isn't the main responsibility of a controller: what it *needs* to do is accept input, route it to the appropriate models, then return the correct response to the browser: either a view or a redirect. By separating out data loading, we can simplify our controllers and make them more readable. This means that only a single object is loaded in the controller for the view: the View Object. It handles the loading of all other necessary objects.

View Objects can also simplify our view templates. Views should only contain display logic: things like loops and conditionals on the data passed in to it. But even display logic can get pretty cluttered up with things like choosing which CSS class to add to an element, drilling down multiple levels into an object, etc. Logic like this can be moved to the View Object. This results in views that are much more readable, because they're almost entirely markup and simple output ERB tags.

View objects have gone by a few different names, like View Model. Also, Presenter is commonly used in the Rails community, but there's one complexity: the person who coined the term Presenter used it to describe a different pattern, related to form submission.

## Value Objects

Sometimes a single field on a model has logic around it more than what's built into the Ruby data type object. Bryan Helmcamp gives the example of a rating field of A, B, C, D, or F. Some logic that might be needed related to that field includes sorting and grouping.

Initially, you might implement logic like this as methods on the model, like `sort_by_rating`. But once there are three or four 

## Bringing it All Together

When all these patterns are brought together, we have a display stack that splits up responsibilities nicely:

- Model: just the core domain logic
- Value Object: logic around individual fields that have some complexity
- Draper Decorator: adds model-related view logic onto the model
- View Object: handles assembling the data needed for the template
- View Template: renders the markup
- Partials: makes bits of markup reusable

Notice that helpers aren't included in the list. When these other patterns are used, there isn't a lot of need for custom helpers in your application. Of course, this doesn't include built-in helpers like form helpers: those still have a very valuable role.
