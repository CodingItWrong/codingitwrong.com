---
title: Frameworks and Evolutionary Design
---

[JavaScript Jabber episode 360 with James Shore](https://devchat.tv/js-jabber/jsj-360-evolutionary-design-with-james-shore/) contains an extremely helpful description of the concept of evolutionary design; it'll probably be one of the resources I reach for first in the future to introduce the idea of evolutionary design to others. James' high-level outline of it is:

> 1. Build the simplest thing that can possibly work for the problems we're solving today.
> 2. Add new requirements come in, we modify our design to be perfectly designed for what the requirements are today.
> 3. We are designing all the time. Not once at the beginning of the sprint; every day, every hour, we're thinking about design.

As part of this discussion, he shared some ideas on the relationship between evolutionary design and third-party code:

> Whenever you're dealing with third party code, you have to assume it's going to change out from under you at some point. And this is part of what simple design is about. You want to create your code such that, assume that anything you depend on is going to change, and when that happens, how hard is it going to be to change your own code?

He finds arranging his code to support this change much easier with individual libraries than with frameworks:

> Given the choice, I will take a collection of libraries and glue them together myself rather than using a framework, because then I can change the glue rather than being stuck in the world the framework provides.

Interestingly, I learned about evolutionary design in the context of a framework: building Ruby on Rails apps at Big Nerd Ranch. We didn't see a conflict between using the framework and the agile principle of evolutionary design. In fact, "agile" is right in the title (and focus) of the book a lot of people first learned Rails from: [_Agile Web Development with Rails_](https://pragprog.com/book/rails6/agile-web-development-with-rails-6). Since learning evolutionary design I've picked up Ember.js as my frontend framework of choice: in addition to getting the same kind of productivity Rails provides, I find I can apply the same evolutionary design principles there as well.

How do James and I reach different conclusions about the compatibility of frameworks and evolutionary design? I think the difference is that my need to evolve my design is limited to changes in business requirements, rather than extending to changes in libraries.

James, referring to individual libraries, said that he thinks about "*when* this changes, not if." But Rails' architecture is predicated on the idea that most apps *don't* need to change most parts of the framework most of the time—as captured by the oft-repeated line ["you're not a beautiful and unique snowflake."](https://rubyonrails.org/doctrine/#convention-over-configuration) And that's been true in my experience. If you don't need to change your libraries, the time spent writing custom code to glue them together is wasted—time that could have instead been spent implementing business features. So if you can have a high degree of confidence you won't need to change those libraries, you can skip that cost altogether.

What about the cost of handling changes that occur *within* the libraries themselves? In 2019 Rails and Ember are stable enough now that such changes are minimal. Ember in particular has been able to accomplish significant improvements to its internals without any change to the user-facing interface: for example, its rendering engine has been rewritten twice. And even when there are outward-facing changes, Ember provides codemods to make necessary changes to your application code as low-effort as possible. And there are still some ways to wrap libraries within the framework. For example, rather than constructing complex Active Record queries directly in your Rails controller, you can define a custom query method on the model that wraps that query. This simplifies testing and ensures that if anything about Active Record changes, your code that uses it is in one place.

So I don't find changes to my frameworks or libraries a major cost I need to minimize. Instead, most of the changes I need to respond to are changes in business requirements. And that's when I apply the evolutionary design principles just as James described them: I build the minimal functionality the feature requires, then as new features come in, I adjust the code to best fit them. And as I do, using a full-featured framework means that I don't need to slow down to add in new technical capabilities.

It could be argued that my hesitance to wrap my libraries is because I'm not fully embracing an agile approach to handling change. That's a reasonable concern, but in my case I'm generally pretty enthusiastic about taking agile development to extremes, like test-driving everything or building the absolute minimal functionality needed for the current story. I see benefits of going all-in on these practices, but when it comes to isolating libraries, the costs are more apparent: I'm taking away time from implementing business requirements for what is in my case a decoupling that hasn't paid off.

I'm not arguing that *no one* should wrap their libraries; I'm sure it's valuable in many different contexts including James'. But there are contexts like mine where it doesn't provide value, and in which it can therefore be safely omitted. And in those cases evolutionary design is still just as useful to handle the changes my app *does* undergo: changes to business requirements.
