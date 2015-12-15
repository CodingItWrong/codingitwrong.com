---
title: The Rails Architecture Controversy
---

{% include posts/toc_rails_architecture.md %}

As someone who's followed Rails for years but never worked in it full-time until now, it had seemed to me that the community had just one view of application architecture: The Rails Way. If you didn't like that Way, you didn't work in Rails. But I've been a software developer long enough that I should have known that every development community has major disagreements! Sure enough, now that I'm looking around for information about architecture in Rails applications, there are a whole spectrum of views.

One view is summarized well by the title of a 2014 blog post, ["Rails Does Not Define Your Application Architecture"](http://naildrivin5.com/blog/2014/05/27/rails-does-not-define-your-application-architecture.html). In a similar vein, [Uncle Bob argues](http://t.co/4XJ8xGcNz4) that the web and the database are I/O devices, and your actual application is everything in between that isn't bound to either. And that's basically the point of [hexagonal architecture](http://fideloper.com/hexagonal-architecture) as well.

The other view seems to be held by DHH and Katz, among others. In their view, Rails *is* attempting to define an application architecture so you don't have to. [Your app isn't a unique, beautiful snowflake](https://youtu.be/9LfmrkyP81M). When experimentation happens, it's ultimately rolled back into [the shared solution](https://youtu.be/9naDS3r4MbY).

Which view of architecture is right? The answer is easy to see in the edge cases. Rails out-of-the-box should be enough at *least* for prototype apps, and something of the scale of Facebook most likely requires some custom-tailored engineering.

But what about the 99% of systems in between? There's some kind of spectrum, but how much of it is on Uncle Bob's side and how much is on DHH's? Spoilers! I won't be answering that. What I will be doing is:

- First, looking at the Single Responsibility Principle, the main OO design principle that drives the discussion.
- Then, reviewing at some of the common patterns people add on top of Rails to accomplish single responsibility.
- Finally, zooming back up to a big-picture level to talk about the two camps' view on the costs and benefits of splitting up these responsibilities vs. sticking with the standard.

In the meantime, feel free to check out my [Rails architecture](http://www.youtube.com/playlist?list=PLHhDPKFbKsTLAVB4dhDCpT7k1-gdMZ3fR) YouTube playlist and start forming your own opinions. And [let me know](https://twitter.com/CodingItWrong) what other talks and blog posts I should be looking at.
