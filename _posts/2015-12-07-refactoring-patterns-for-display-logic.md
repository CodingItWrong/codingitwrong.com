---
title:Refactoring Patterns for Display Logic
---

So far in the Rails Architecture series we've talked about [what all the fuss is about]({% post_url 2015-11-22-the-rails-architecture-controversy %}), then about [how the Single Responsibility Principle leads us to seek out new design patterns]({% post_url 2015-11-30-single-responsiblity-principle-and-the-history-of-rails-architecture %}). Now let's dive in to some of these patterns to see how they can work together to simplify your classes. I won't be providing detailed code samples, but I'll link to articles for additional detail on each pattern.

In this post we'll start with patterns related to display logic. In the default Rails stack, four types of component work together to generate HTML: the view templates themselves, partials, models, and helpers. Of course, the main responsibility of the view is to output the markup of the page, based on the data in the model (or multiple models). As display logic gets more complex in the template, there are three places you can abstract that logic to:

- If you just have a portion of markup that needs to be reused, or is complex enough that it makes it hard to understand the overall flow of the page's markup, you can place it in a **view partial**.
- **Helpers** are more useful for if you need to generate markup but it's more convenient to build up the HTML string from within plain Ruby code.
- If you don't need to generate markup at all, but rather do a data operation like formatting a date or dollar amount, or concatenating fields in a standard way, you can add methods to the **model** itself.

This separation is a good start, but there are a few additional extractions we can make to improve the modularity of our code.

## Draper-Style Decorators

[Draper-style Decorators](https://github.com/drapergem/draper) provide a place to put model-specific display logic all in one place. This is a way to separate out the core domain responsibilities of the model from display responsibilities. Such logic is also sometimes placed in helpers, but Draper-style Decorators are objects, making them easier to work with and test in some ways.

This is an application of the general [Decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern), which involves wrapping an object with another object to add additional behavior. In this case, the model object is wrapped in another object that adds formatting. Usually, all the model's methods are directly accessible, as well as additional formatting methods.

I use the term "Draper-style" because Draper is a common Ruby gem for applying this pattern. The Decorator pattern can also be used for applying secondary effects to model save operations, which I'll discuss in a future post. If you don't want to use Draper, you can implement the "Draper-style Decorator" pattern yourself with a little more work.

## View Objects

The problem that [View Objects](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/) address is that controllers can get cluttered up with code to load and arrange data in the way the view needs it. This isn't the main responsibility of a controller: what it *needs* to do is accept input, route it to the appropriate models, then return the correct response to the browser: either a view or a redirect. By separating out data loading, we can simplify our controllers and make them more readable. This means that only a single object is loaded in the controller for the view: the View Object. It handles the loading of all other necessary objects.

View Objects can also simplify our view templates. Views should only contain display logic: things like loops and conditionals on the data passed in to it. But even display logic can get pretty cluttered up with things like choosing which CSS class to add to an element, drilling down multiple levels into an object, etc. Logic like this can be moved to the View Object. This results in views that are much more readable, because they're almost entirely markup and simple output tags.

View Objects have gone by a few different names, like "View Model" and "Presenter." The latter is commonly used in the Rails community, but there's one complexity: Jay Fields [coined the term Presenter](http://blog.jayfields.com/2007/03/rails-presenter-pattern.html) to describe a different pattern, related to form submission.

## Value Objects

Sometimes a single field on a model has more logic around it than what's built into the Ruby data type. Bryan Helmcamp gives the example of a rating field of A, B, C, D, or F. Some logic that might be needed related to that field includes sorting and grouping.

Initially, you might implement logic like this as methods on the model, like `sort_by_rating`. But once there are three or four method all relating to the same field, that starts to feel like a separate responsibility that belongs in its own object. This is called a [Value Object](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/).

I haven't heard a lot of talk about value objects in Rails or other frameworks that use the Active Record pattern. One reason may be that, by default, Active Record objects expose all database columns as basic data types--so you either have to jump through some hoops to prevent accidental access to the non-value-object version of the field, or else be disciplined to never use the raw version. It's not the cleanest.

## Bringing it All Together

When all these patterns are brought together, we have a display stack that splits up responsibilities nicely:

- **Model**: just the core domain logic
- **Value Object**: logic around individual fields that have some complexity
- **Draper Decorator**: adds model-related view logic onto the model
- **View Object**: handles assembling the models needed for the template
- **View Template**: renders the markup
- **Partials**: makes bits of markup reusable

Notice that helpers aren't included in the list. When these other patterns are used, there isn't a lot of need for custom helpers in your application. The other patterns handle all the needs in a more object-oriented way. Of course, this doesn't include built-ins like form helpers: those still have a very valuable role.

This isn't to say that every model and view of every application needs all these patterns. It's best to start simple, and only add additional objects when there really *is* a separate responsibility to pull out. And even then, you may only need Decorators for a few models, and you certainly don't want Value Objects for all of your fields.

How do you use these patterns in your apps? Do you have other patterns for display logic? [Let me know!](https://twitter.com/CodingItWrong)
