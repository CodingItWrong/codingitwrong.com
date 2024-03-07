---
title: "Simple Design: No* Duplication"
---

_This is the second post in a [series about the Four Rules of Simple Design](/2024/03/06/simple-design-reveals-intention.html)._

“No duplication” means that when a domain concept is represented in two or more places in the code, you adjust the code so that that domain concept is represented in just one place. This usually involves putting shared data or logic in a single shared place so they can be accessed from wherever they’re needed.

## Why no duplication?
Consider a user account record. Say there are multiple conditions that determine whether the account is considered “active:” the user needs to have accepted the latest terms and conditions, and their subscription payment needs to be up-to-date—unless they created their account during the beta period, in which case their account is free. If a user’s account is not active, they aren’t locked out of the application entirely, but there are a number of features throughout the app that they can’t see or are read-only instead of interactive. An example of duplication would be if at each of these places there was a separate calculation that check if the account is active. It would be better to have a single function or method that can tell us if a given user is active, and that function or method is called everywhere that information is needed.

Why does removing the duplication of multiple active checks matter? There are a number of benefits. It makes the code easier to understand because it operates at a single level of abstraction: instead of reading the details of that calculation at each place it’s needed, we just see that we’re asking if the user is active. It prevents errors where a developer might not implement the logic the same way in each place. It reduces testing burden because you can test the function against all the factors that affect being active in one place. Then, when testing each place that the function is called, you don’t have to test all the factors, you can just check one case where the user is active and one where they are inactive.

Those are all important reasons, but there’s one I think is the most important of all: supporting change. If your code is never going to change, having duplicate logic to check for active isn’t too big a deal. But consider if a new condition is added someday—I dunno, say instead of paying cash users can donate NFTs they don’t want anymore. If the logic to check active users is duplicated, you need to update everywhere that logic is run add that condition in. If you miss a case, the application won’t work consistently. That’s a big impediment to changing the app, and can result in you either avoiding change or the app becoming unstable as you make changes. That, I think, is the ultimate reason to avoid duplication of domain concepts: to enable change.

## An invalid counterexample
The absolute of “no” duplication is quite extreme, and objections may immediately rise in your mind. It’s been asked before whether multiple `if` statements constitute duplication, and whether this rule means they should be replaced by a function call.

I have a few responses to that. First, this is why I used the term “domain concept” above. `if` statements aren’t a domain concept, they’re a programming construct, and of course it’s fine to use the same programming construct in multiple places. A domain concept is something that belongs to the problem domain of the application, something that might change in the future based on business rules or laws or user preferences. Another way I've heard this said is "no duplication of *ideas*."

Second, imagine you actually did write a function that took a condition and a function and only executed the function if the condition were true. That doesn’t gain us anything over the `if` statement. It’s no shorter to write. It won’t save us any work when something changes, because whereas business rules change all the time, I can’t imagine programming `if` constructs changing. And it will be less familiar to readers of the code than `if` statements.

## A valid counterexample
I do know of at least one important qualification to “no duplication,” however. Sandi Metz, a prominent software design teacher, has said:

> Duplication is far cheaper than the wrong abstraction… Prefer duplication over the wrong abstraction.

What does "the wrong abstraction" mean? Her [blog post about the wrong abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction) explains in detail. But in short the point is that if you remove duplication, it can turn out that those two things were *not* actually the same, and the way you find that it is if they later have to change differently.

There aren't any foolproof answers for finding out what the right or wrong abstractions are. One heuristic is to think in terms of the business domain: is this duplication based on a common business rule? Another, less abstract, is the "rule of three:" don't remove duplication when there are two instances but wait until there are three, to give you greater confidence that it really is duplication.

Regardless of what heuristics you apply, if you ever prefer duplication over the wrong abstraction, clearly you aren’t following the “no duplication” rule to the letter. So this rule has qualifications. Whereas I wouldn’t say “sometimes you should make sure to have code that doesn’t communicate intent,” sometimes you *should* have duplication in the code.
