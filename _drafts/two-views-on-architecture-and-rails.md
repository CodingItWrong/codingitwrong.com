---
title: Two Views on Architecture and Rails
---

As someone who's followed Rails for years but never worked in it full-time until now, it seemed to me that there was just one view of application architecture: The Rails Way. If you didn't like that Way, you didn't work in Rails. But I've been a software developer long enough that I should have known that every development community has major disagreements!

Sure enough, now that I'm looking around for information about architecture in Rails applications, there are a whole spectrum of views.

The one view is summarized well by the title of a 2014 blog post, ["Rails Does Not Define Your Application Architecture"](http://naildrivin5.com/blog/2014/05/27/rails-does-not-define-your-application-architecture.html). In a similar vein, [Uncle Bob argues](http://t.co/4XJ8xGcNz4) that the web and the database are I/O devices, and your application is everything in between that isn't bound to either. And that's basically the point of [hexagonal architecture](http://fideloper.com/hexagonal-architecture) as well.
