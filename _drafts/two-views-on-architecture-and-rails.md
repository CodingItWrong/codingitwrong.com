---
title: Two Views on Architecture and Rails
---

As someone who's followed Rails for years but never worked in it full-time until now, it seemed to me that there was just one view of application architecture: The Rails Way. If you didn't like that Way, you didn't work in Rails. But I've been a software developer long enough that I should have known that every development community has major disagreements!

Sure enough, now that I'm looking around for information about architecture in Rails applications, there are a whole spectrum of views.

One view is summarized well by the title of a 2014 blog post, ["Rails Does Not Define Your Application Architecture"](http://naildrivin5.com/blog/2014/05/27/rails-does-not-define-your-application-architecture.html). In a similar vein, [Uncle Bob argues](http://t.co/4XJ8xGcNz4) that the web and the database are I/O devices, and your application is everything in between that isn't bound to either. And that's basically the point of [hexagonal architecture](http://fideloper.com/hexagonal-architecture) as well.

The other view seems to be held by DHH and Katz, among others. In their view, Rails *is* attempting to define an application architecture so you don't have to. [Your app isn't a unique, beautiful snowflake](https://youtu.be/9LfmrkyP81M). When experimentation happens, it's ultimately rolled back into [the shared solution](https://youtu.be/9naDS3r4MbY).

Which answer is right is easy to see at the extremes. Rails out-of-the-box should be enough at *least* for prototype apps, and something of the scale of Facebook most likely requires some custom-tailored engineering.

But what about the 99% of systems in between? 