---
title: 'The Fourth Developer'
---

In our software development process we want more value: more useful software delivered. The first technique we reach for to try to do this is to add more developers to the team. How well does this work?

To assess this, let's assume the following project setup:

- We’re talking about multiple developers on a single codebase where any developer can work in any area of the code. No siloing by programming language or feature.
- All developers on the team have the same skill level and are fully onboarded onto the project.
- Developers create a feature branch working solo, then open a PR, which is reviewed by one other developer.

In this scenario, then the amount of useful code delivered increases linearly with the number of developers. Two developers deliver two units of code, three deliver three, etc. The amount of time needed for PR review doesn’t increase, because no matter how many developers you have, if each one reviews just one other developer’s code, everyone gets a review. (I don’t include the one-developer case in this list because one developer isn’t doing any code review, so they have more time to write code and their output isn’t directly comparable.)

But the linear increase of output doesn’t increase indefinitely. At some point, the amount of useful code added slows down, even to the point of the team being less productive than before the last developer was added. And I think this point is far earlier than almost any of us realize.

Let's add a few additional assumptions:

- Most of the time developers are modifying or extending existing code, rather than adding totally new code.
- PR reviews are thorough, not just looking for obvious errors or code style, but really understanding the code.
- PR reviews are the main time developers are getting familiar with code. They are not regularly reading through other code to the same level of thoroughness as in PR reviews.

With these assumptions added, we can now pinpoint exactly when adding developers starts to cause a problem. **Specifically, the team is hindered more than they’re helped at the moment when you add a fourth developer.**

## A Fourth Developer Adds Unfamiliar Code
How can I make such a specific claim? It’s based on the logistics of being familiar with the code you’re working on. It’s significantly easier to add functionality when you're familiar with the code you're modifying or extending: you’re more productive and the result is more reliable. And three developers will be significantly more familiar with the codebase than four. Let’s see why.

As our goal, let’s say that **for every change to the code, we want someone who is familiar with the pre-existing code involved:** that is, either writing the code or reviewing it before merge. Having someone familiar involved increases quality, because it minimizes our chance of introducing bugs, duplicating functionality, or just writing code that fits poorly with the existing code. How well can we succeed at this with different team sizes?

When you are a **solo developer** on a project, you're familiar with all of the code. This means a high level of quality for whatever you’re working on, although of course the limitations on output for a solo developer are clear.

Over time, your memory of code fades, so you don't have perfect familiarity of all the code you've ever worked on. For simplicity of analysis, we'll set aside the impact of memory on code familiarity for most of this post, then bring it back in at the end to account for it.

When you have a **team of two developers,** each is still familiar with all of the code, because for each chunk of it, you either wrote it or reviewed it. This means you’ll still have a high level of quality in the code you write.

What about a **team of three developers**? If you think about it, it becomes clear that for every PR of new code, either the author or the reviewer will be familiar with that code. Here's why we can conclude this.

Call the developer who authored the new PR A, and the reviewer B. Who wrote the code that this new PR is modifying? If A or B wrote that code, then that’s someone in the PR process who is familiar. What about if C wrote that code? If so, then the reviewer of that previous code was either A or B, so that’s still someone involved in the new PR who is familiar.

Now, what about **four developers**? Here things get difficult. Again, call the developer who authored the PR A, and the reviewer B. Is one of them familiar with the pre-existng code?

There are 12 possible combinations of who wrote and reviewed this code previously:

![](/img/posts/the-fourth-developer/01-12-cases.png)

In 10 of the 12 cases, either A or B wrote or reviewed the previous code, so someone in this PR is familiar. But in two of the cases, neither is familiar with the code: the case where C wrote it and D reviewed it, and the case where D wrote it and C reviewed it. This is a new situation: now there is a PR that will be merged into the codebase modifying code that neither the author nor the reviewer will be familiar with. This means the quality of that PR will be impacted. Previously, there was no such unfamiliar code; all code was familiar either to the author of the PR or the reviewer. **Now, only in 10 of 12 cases, or 83% of the time, will the code be familiar to those involved in the PR.**

## The Impact of Unfamiliar Code
How big an impact is that loss of familiarity? There are at least three ways to think about it.

The first way to think about the impact is to focus on the 83% of new code that will be familiar. 83% is still a fairly high percentage, and you gained the benefit of an additional fourth developer’s worth of output. But on the other hand, with 1, 2, or 3 developers someone in the pair was familiar with 100% of the code written. A sudden drop from 100% to less than 90% is significant. This is a cost you take on when you add a fourth developer, to get the benefit of that fourth unit of code being written.

The second way to think about the impact is to focus on the new cases for author/reviewer combinations:

![](/img/posts/the-fourth-developer/02-old-and-new-cases.png)

For the six pre-existing cases, there was 100% familiarity. For the six new cases, in four of the six you have familiarity, and in two of the six you don't. That's 67% familiarity. That number doesn't sound quite as good as 83%, but you’re still adding more familiar cases than unfamiliar ones.

But there’s a third way to think about the impact that more directly relates to value delivered, and its picture isn't good. This is to think in terms of the new code added.

Think back to the amount of useful code we described. Ideally, two developers would deliver two units, three developers three, four would deliver four, etc. Combine that total amount of code written with the percentage of code understood. With two developers all the code is familiar, so they deliver two units of familiar code. With three developers all the code is still familiar, so they deliver three units of familiar code. But with four developers, only 83% of the code is familiar. So of the four units of code they deliver, 3.33 units are familiar, and 0.67 units are unfamiliar.

How should we assess the code added by the four-developer case? Three developers deliver three units of familiar code, and four developers deliver 3.3 units, so 0.33 more. Three developers deliver zero units of unfamiliar code, and four developers deliver 0.67 units. By adding a fourth developer, you're adding 0.33 units of familiar code and 0.67 units of unfamiliar code.

**By adding a fourth developer, for every bit of familiar, well-understood, fits-with-the-system code you add, you’re adding *twice as much* unfamiliar, less-understood, clutters-up-the-system code.**

The change from three developers to four is drastic. We went from three developers adding *no* unfamiliar code, to a fourth developer adding *twice as much* unfamiliar code as familiar. In a very real sense it's one step forward, two steps back. This is probably very surprising to hear; it was to me.

But is this difference mitigated by the fact that your familiarity with code fades over time? It's true that that means that in the three-developer case you don't have *perfect* familiarity with all the code. But it still means that your *only* unfamiliarity comes from fading memory, whereas with a fourth developer you're adding twice as much code that you've *never seen* right from the beginning. The difference in familiarity is still drastic.

## How Can This Be?
If you’re not sure how this math can be correct, let's think about the cases that lead to it.

First, let's look at the new code that D will be writing now that they have joined the team:

![](/img/posts/the-fourth-developer/03-new-code.png)

We added a fourth developer D who is now writing code that would not have been written before, that A will write a PR to extend in the future. 2/3 of the time A or B reviewed that code, so it’s familiar. But 1/3 of the time, when C reviewed D’s code, that the code is unfamiliar.

Next, let's look at the old code, that A, B, and C would have been writing anyway:

![](/img/posts/the-fourth-developer/04-old-code.png)

When there were only three developers, all the code that A, B, or C wrote was familiar to A or B. But now, D is in the review rotation for C's code. So the 1/3 of the time when D reviews C's code, it is no longer familiar to A or B. So the presence of D on the team disrupts the review process, so that C's code actually becomes *less* familiar to A and B than it would have been before.

This is why adding D increases the amount of unfamiliar code more than it increases familiar code. The new code written isn't always familiar, and now code that would previously have been familiar is less frequently familiar.

To be clear, the problem isn't the fourth developer added to an existing team of three: we said in the assumptions that all developers have the same skill level and are fully onboarded to the project. D is not "the new developer"; D is just "one of the two developers who isn't writing or reviewing a given PR." The blame isn't on one person; the blame is on the unavoidable dynamics of a four-developer team.

You added a fourth developer to the team because you wanted an increase in the amount of code written, with the implicit assumption that most of that code would be high quality. But your goal hasn’t been achieved. Adding the fourth developer has only increased the amount of *familiar* code from 3 units to 3.33, and it has introduced the costly dynamic of unfamiliar code, 0.67 units of it.

## Digging Deeper
To see how this dynamic continues as a fifth and more developers are added to the project, we can write an equation to describe the different review scenarios and chart it for higher values of team members. As you might expect, things continue to get worse:

![](/img/posts/the-fourth-developer/05-getting-worse.png)

The amount of familiar code increases by less and less each time, approaching a hard limit of 4 units no matter how many developers you add. Meanwhile, the amount of unfamiliar code increases linearly. Adding more developers only serves to cause the amount of unfamiliar code to dwarf the amount of familiar code, destroying the quality of your codebase.

A drop in quality is seen far earlier than this, though, and at a precise moment: when you add the fourth developer to the team. All of a sudden, you go from adding a developer who adds all familiar code, to a developer who results in twice as much unfamiliar code as familiar code. This is not a good tradeoff.

## What Do We Do?
What should you do with this information? One option would be to reject the premise that unfamiliar code is a problem at all. If it’s okay for an unfamiliar developer to change code and an unfamiliar reviewer to approve it, the problem disappears. But I would warn against this view: I would argue that many of the problems that happen in codebases as they grow can be traced to changes that are made without thorough understanding.

If, on the other hand, you accept the premise that unfamiliar code is a problem, then there are a few ways to respond:

1. You could mitigate the problem by taking a development approach where unfamiliar code is not *as* unfamiliar, and therefore not as risky. One way would be by adopting stringent code conventions. If you’re automatically familiar with many aspects of the code because it’s written in a familiar way, that reduces the chance of misunderstandings and quality problems
2. You could arrange your system so it can be built by teams of three developers max. This may seem impossible, but there are a few different ways: by using high-level abstractions to minimize the code that needs to be written, by focusing on building the most important features one at a time, or by splitting up the system into smaller self-contained pieces.

These approaches have all been discussed at length elsewhere better than they could be summarized here. But the surprising thing may be how early you need to reach for those approaches. They aren’t for when your team has reached a dozen developers. Impacts on the quality of your app happen at a precise moment: when you add a fourth developer. To maximize quality, the moment when you’re considering that fourth developer is the moment to apply these techniques.
