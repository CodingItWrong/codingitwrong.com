---
title: '"I Regret Nothing!" or, How to Evaluate a New Technology'
---

When I was applying for my current job at [Big Nerd Ranch](http://bignerdranch.com), one of the things they asked me to do was to prepare a ten-minute presentation on anything I would like. At my then-current job I had been thinking a lot about how to evaluate new technologies, so I decided to present on that. It was a lot of fun, so I thought I'd share my thoughts here as well.

- - -

I love tooling: setting up a standard technology stack for my team that solves a lot of common problems. So when I hear about a new technology that might fit into my stack, I get really excited. It feels a little like this:

![Mario Galaxy](/img/posts/i-regret-nothing/galaxy.jpg)

It's going to be a whole new universe of possibilities! But once I get past the tutorial and try to use the technology for real work, often times it feels more like this:

![Kaizo Mario](/img/posts/i-regret-nothing/kaizo.gif)

We've all had experience with a technology that didn't end up being all it was cracked up to be. It can be discouraging when it's a technology we got excited about. And when it's a technology that someone *else* picked for us, it can make you downright angry.

![Angry Birds](/img/posts/i-regret-nothing/angry-birds.jpg)

So the question is, s there a way to avoid these ups and downs of new technologies? Is there a way to get off the roller-coaster?

![Mario Kart](/img/posts/i-regret-nothing/rainbow-road.gif)

I think there is. I think the way to get off the roller coaster is to interrogate your technologies.

![Phoenix Wright](/img/posts/i-regret-nothing/phoenix-wright.jpg)

When I say interrogate, I mean to ask good questions about them. I'd like to propose five questions to ask of any new technologies to help you evaluate them. When you're excited about a new tool and its benefits, it's not always easy to see the costs. These questions will help you think about the tools from all sides.

## Question 1: How much does it do?

![Sword-Chucks](/img/posts/i-regret-nothing/sword-chucks.jpg)

Does the tool you're considering do *everything* you need it to do? Or does it leave holes you'll have to do yourself? Modern web applications have to do a lot: validation, sanitation, serialization to a database, connecting to web services, rendering templates, compiling Sass. Could your old tool be doing something you're taking for granted? If so, there will be a high cost to reimplement that feature yourself.

At my last employer, we experienced this cost when we switched our frontend build tools from [Grunt](http://gruntjs.com/) to [Laravel Elixir](http://laravel.com/docs/5.1/elixir). The latter has a much nicer syntax, making it easier to understand and improve our build script. But all it does out of the box is asset compilation, and we had been using Grunt for everything: database backups, setting folder permissions, running commands... Because we didn't consider those needs before we switched, we ended up doing those remaining needs by hand, because we didn't have time to find out how to do them in Elixir.

## Question 2: Does it scale?

![Scaling Mario](/img/posts/i-regret-nothing/scale.gif)

I'm not talking about the age-old question "Does Rails scale?" That's been done to death. I'm asking, does the tool you're considering meet the needs of your smallest systems *and* your largest systems? If not, you'll need multiple tools, which means a higher learning cost for your team members, as well as a higher ongoing support cost for switching contexts.

A success story for this was switching MVC frameworks at my last employer. We went with [Laravel](http://laravel.com). Other frameworks were rated more highly for Rapid Application Development, or for large enterprise systems. But Laravel had the best rating across web applications of all sizes. It lets you start quickly, but it also sets you up for success when you need to make your architecture more robust as you grow. We used it for everything from a single-table no-GUI app to a 10,000-user video streaming platform, and it worked great for all of it.

## Question 3: Can we learn it?

![Eve Online](/img/posts/i-regret-nothing/eve.jpg)

## Question 4: Is there community support?

![Zerg Rush](/img/posts/i-regret-nothing/zerg.jpg)

## Question 5: Is it maintained?

![SimCity](/img/posts/i-regret-nothing/simcity.jpg)

## Conclusion

![A winner is you.](/img/posts/i-regret-nothing/a-winner-is-you.jpg)