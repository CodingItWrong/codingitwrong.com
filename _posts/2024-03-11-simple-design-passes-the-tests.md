---
title: "Simple Design: Passes the Tests"
---

_This is the fourth post in a [series about the Four Rules of Simple Design](/2024/03/06/simple-design-reveals-intention.html)._

The Rule of Simple Design I'm discussing last is actually the first one listed in order: "passes the tests."

I don't save it for last because it's the lowest priority for me. On the contrary, having good tests is such a high priority that I would be more likely to bring it up with a team than the rest of Simple Design. And there's so much that could be said about the importance of testing that one post only scratches the surface.

At the same time, I'm glad Kent Beck included it in the list of Simple Design rules instead of assuming it. This way, you can't talk about the Rules without mentioning testing as part of it.

I could talk at length about the general benefits of testing for catching bugs, positively influencing design, communicating intent, fostering conversations with the business, and letting you know when you're done a unit of work. But instead, in this post I want to talk about one specific benefit: **comprehensive tests are necessary to enable Simple Design.** In this series I've presented the benefits of the other three Rules of Simple Design. But unless you have a suite of comprehensive tests, you'll be severely limited in your ability to get the benefit of those three Rules.

And in all three cases, the reason the tests are needed is to support change. Let's talk about why.

## How tests enable Simple Design

When you first write code, even without tests you can focus on making it Reveal Intention as much as possible. But once you receive feedback that someone who *didn't* write the code is having trouble understanding your intention, you'll be hesitant to make changes if they risk breaking the program. Revealing Intention doesn’t matter if the program doesn’t work, so how important is it really if the names aren’t quite clear as possible, that there are a few nonessential differences, that there are some large pieces in the code, that the public interface is broader than needed? And as the code changes, a previously-small function will get large, a previously-accurate name no longer covers the scope of what the function is used for, or something in the public interface no longer needs to be there. Is it worth breaking the program just to clean those up? Static typing and IDE support might provide safety for some of those changes, but not all of them.

Also, when you first write code, even without tests you can try to avoid duplicating business rules. You don’t put the same boolean logic in the same place; you put it somewhere central. But how about when you discover that the same business rule was mistakenly implemented in two places? Or that something you thought of as just an incidental bit of logic turns out to be a broadly-shared business rule? Or that two things that seemed to be separate business rules are actually related? Minimizing duplication doesn’t matter if the program is broken, so how big of a problem is it really if there is a little bit of duplication?

Finally, when you first write code, even without tests you can focus on writing the fewest elements. You remove things that aren’t needed and you write things in the simplest possible way. But how about as the code changes, when code that used to be used is no longer used, when a necessary return value is no longer necessary, when something that previously required a low-level algorithm turns into something that a simple call to `map` will handle? Using the fewest elements doesn’t matter if the program doesn’t work, so how important is it really if there are some unnecessary elements in the program?

In all these cases, we can see that when there is a significant risk of changes to the code breaking the functionality of the code, this pits the Rules of Simple Design against program functionality. You have to choose: you can change the code to follow the Rules of Simple Design and get all its benefits, but you risk breaking functionality for your users; or, you can protect the functionality for your users, but miss out on the benefits of the Rules of Simple Design.

## Cutting the Gordian knot

Now, it may seem like the Rules of Simple Design are optional, that your program's functionality is the thing that matters, and that in the long run it's no big deal if you're not following the rules of Simple Design. But that hasn’t been my experience. Over time, if your code does not reveal intention, if there is significant duplication, and if there are unnecessary elements, the code gets harder and harder to understand. It takes longer to make changes, the changes take more effort to make, and they are more likely to cause unexpected breakage in the software. So refusing to make changes to your code can prevent bugs in the short run, but in the long run it leads to more bugs, higher cost, and the inability to adjust your software to users’ changing needs.

But with a suite of comprehensive tests, you don’t have to choose between the Rules of Simple Design and program functionality: you can have both. You can keep your program working now, and continue to keep the code Simple so it works and is adjustable in the long term. If you make a change that breaks something in your program, you can trust the tests to tell you. And if the tests *don’t* tell you, your conclusion is not “we need to stop improving the code;” instead, your conclusion is “ah, there was a hole in our test coverage, let’s fix it and learn from it so that similar bugs won’t get through in the future.”

This is not to say that testing is easy; far from it. It takes a lot of work to learn not only the mechanics of using testing tools, but also the principles of how to structure your code and tests to get a high level of safety for a low maintenance cost. Testing has been a primary professional focus of mine for ten years, and I still run into challenging testing situations all the time. Testing is hard work.

What I'm arguing is that testing is *worth* that hard work. In addition to all the other benefits that comprehensive tests provide, they also transform the tradeoffs between short-term quality and Simple Design. You no longer have to make the impossible choice between whether to keep your code working now or keep it sustainable for the future. You can have both, fully.

If you've been applying the other three Rules of Simple Design but you wouldn’t say your tests are comprehensive, I’m not saying that you haven’t gotten any benefits of Simple Design. You almost certainly have! You can do your best to apply the rules when you first write the code, and when you need to change the code you can apply the Rules when the upside of doing so clearly offsets the risk of making changes. The question I would ask is: does it feel like you’re keeping up? Or does it feel like the code is deteriorating over time? That despite your best efforts the intention is getting more obscured, duplication is proliferating, and there are more and more unnecessary elements in the code?

That deterioration of the code is not inevitable. If you focus on writing comprehensive tests, you’re freed up to take your Simple Design as far as possible, and get all the benefits it provides.
