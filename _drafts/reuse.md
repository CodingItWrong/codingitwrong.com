A blog post called ["Choose Boring Technology"](http://mcfunley.com/choose-boring-technology) argues that as a company you have about three innovation tokens you can spend on new or unique technologies before the cost of using them overtakes the benefits they provide. I find myself taking this a bit further personally: I would prefer not to use *any* innovation tokens! I would love to use totally unexciting technology and just focus on building apps.

In the last few years I've felt a bit insecure about this preference. Am I being lazy? Is it a factor of me being new to OO design, and once the shine wears off I'll want to move onto something more challenging? Am I indulging in OO overdesign instead of focusing on technology problems that provide real business value?

The book _Domain-Driven Design_ addresses this question—in fact, it seems like it's one of the main reasons it was written: *define domain model here*.

> To make the domain model an asset, the model’s critical core has to be sleek and fully leveraged to create application functionality. But *scarce, highly skilled developers tend to gravitate to technical infrastructure* or neatly definable domain problems that can be understood without specialized domain knowledge.
> Such parts of the system seem interesting to computer scientists, and are perceived to build transferable professional skills and provide better resume material. *The specialized core, that part of the model that really differentiates the application and makes it a business asset, typically ends up being put together by less skilled developers*…

So not only are there business reasons to choose boring technologies, there are *professional* reasons as well: it frees me to focus on domain modeling, which will really stretch my skills.

How far can we take this focus? We can even avoid writing parts of our technical infrastructure by using third-party library code:

> The greatest value of custom software comes from the total control of the CORE DOMAIN. A well-designed framework may be able to provide high-level abstractions that you can specialize for your use. *It may save you from developing the more generic parts and leave you free to concentrate on the CORE*

This isn't news to us in 2017; our apps have dependency config files with dozens of packages, most or all related to technical infrastructure rather than domain-specific concepts. But have we gotten to the point where we are "free to concentrate on the CORE"? No. West describes his view on the limitations of reuse:

> Reuse efforts (as opposed to composability efforts, which have been almost nonexistent) have been characterized by a focus on the solution space—computer code, algorithms, and data structures. True composability will require an understanding of the problem space—of the natural world as advocated by object thinking.”

West's line of thinking seems to be:

- Reusable code related to the "solution space" (the technical infrastructure, in _DDD_'s terminology) is inherently low-level, so it is not very composable
- To gain composability, we need reusable code focused on the problem space (the core domain model), so it will be higher-level

Unfortunately, we haven't been able to achieve very much reusable problem-space code since 2004 when this quote was written. However, our technical code is much higher-level than "algorithms be data structures," but it generally hasn't brought about composability either. I would argue that "composability" suggests being able to plug things together with little effort. But for most of our components, there's still a significant amount of "glue code" that needs to be written to get them to function together.

If neither reusable domain components nor higher-level technical components provide a path to composability, is there such a path? I would argue that this is one of the benefits a batteries-included framework like Ruby on Rails offers. The framework includes much of the required functionality for web applications out of the box. It's surrounded by a rich ecosystem of libraries that are tailored to work with it. And Rails' conventions ensure that these libraries can provide a maximum amount of functionality with a minimum amount of setup.

For example, I just started a hobby Rails app and used the popular Devise gem for authentication. Because of the conventions of how application view layouts and Active Record models work, I was able to get all of the login functionality working without any custom code. Authentication isn't hard, but it's tedious, and it's not what I want to focus on in my limited hobby development time.

Rails isn't by any means the only framework built with this approach. JavaScript's Ember, Elixir's Phoenix, PHP's Laravel, and Swift's Vapor all follow a similar approach to conventions. They're all affected to greater or lesser degrees by the features of their language and the maturity of the surrounding ecosystem. But they all offer at least some degree of the freedom to focus on the business domain.

DHH, the creator of Ruby on Rails, loves to say that we all like to believe our apps are unique, beautiful snowflakes, but they're a lot more the same than different. This has always struck me as liberating, but for many people it's discouraging. I think the above can give us a new perspective on this idea. Your application's *technical infrastructure* is not a unique, beautiful snowflake, and if you try to build it as if it is, all your effort will go into reinventing wheels. When you realize your technical infrastructure isn't unique and give up control over it, you're freed up to focus on your domain model, which *is* unique, and which needs hard work to build well.

My conclusion from all this is that I believe one of the most effective ways to create applications is to focus my talents on business logic, and to try to outsource all of the technical infrastructure I can to existing solutions. And a convention-based framework is one of the most cost-effective ways to do so.

That's what's encouraging to me that I'm not wasting my time using boring technology. I'm focusing on domain modeling, taking advantage of incredibly powerful platforms for code 