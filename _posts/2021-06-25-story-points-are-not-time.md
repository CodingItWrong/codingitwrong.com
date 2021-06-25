---
title: "\"Story Points Are Not Time.\""
---

"Story points are not time."

I'm not resistant to this idea, I just seem to have trouble understanding it. I can understand it when someone explains it to me, but afterward I've had trouble recalling the reason, or how exactly I'm supposed to think about it instead.

This blog post is my attempt to capture an explanation of why story points are not time, so I can get back to it in the future again once I forget.

> Incidentally, this isn't to discount other ways to think about estimation or story points. One of the agile developers I respect most [may have invented story points, and if he did, he's sorry now.](https://ronjeffries.com/articles/019-01ff/story-points/Index.html) He says that points originated from time, and another says that he prefers to estimate with time now. This article focuses only on: within the paradigm "story points are not time," how does that paradigm work?

Say you're using story points on your project right now, and you say that points correspond to time. What does 1 point represent? Some projects might say that it represents about a day of work.

What, a day, like 24 hours? No, a work day.

So 8 hours? You mean no bathroom breaks, no meal breaks, no meetings? Well no, like the time available for development work on a normal work day.

Well, which work day? A day with three quarterly company-wide meetings? Or a day between Christmas and New Years when there are no meetings scheduled? Well, this is getting trickier, but say you average out the number of non-project meetings per day. So maybe with that and breaks it's five hours of heads down development time?

Okay, five hours of development time. Next, imagine the most experienced developer you've ever met. Next to them, imagine someone straight out of code school. No judgment on the latter's capabilities, but you can't deny that they will move slower than the virtuoso. So you have a story that's one point. Which developer is going to finish it in five hours: the code school grad, or the virtuoso?

Maybe you say one-point stories are pretty small so it really is about the same amount of time for both of them. I'm not sure I believe you, but let's grant it. Now, think about the most complex story that you won't split up into smaller stories. Maybe it's five points, but the exact value doesn't matter. That most complex story: are you saying it will take both the code school graduate and the virtuoso the same amount of hours to complete it?

It's not possible. If you have any experience in software development you can't say "every developer works at exactly the same speed." It's absurd.

So why do I keep effectively saying that on projects? We do estimations and we say "one point is one day," we look at a story, and we agree as a team that that story is three points, so three days. We just said it's absurd to say that the same work is three days for *any* developer. So do we really mean that?

Maybe we mean that some assumed mid-level developer would take three days to complete it. But there are a few problems with saying this:

- Developers who could complete that story quickly need to remember that they are not estimating for *themselves*, but for that assumed mid-level developer.
- Developers who would need more time to complete that story need to remember that they are not estimating for themselves either. What’s more, on every story estimate, they need to remember that they are agreeing to “3 days” knowing that it will take them longer than that. This can be discouraging over time, making them feel like they aren’t measuring up.
- If management starts holding developers to the time estimates in terms of numbers of days, then developers will be criticized for many things they can’t control: if they are slowed down by having less experience, technical challenges that weren’t known during estimation, meetings scheduled that they can’t control, or other life circumstances.

So how do you decouple point estimates from days entirely? You think purely in terms of relative complexity. You pick a story, maybe one that’s nontrivial but small, and you say that’s 1 point. Then you compare other stories to it to see if they are about the same complexity, bigger, or smaller. This removes all variables of developer skill level and availability. You will complete a one point story much faster than me if you’re experienced and have lots of heads down time, and I’m just out of code school and have lots of meeting scheduled. But we can think about the work that story entails, and compare it to the work other stories entail.

The reason the “days” estimation hasn’t caused major problems on projects I’ve been on is that (1) management wasn’t holding developers to day estimates, and (2) although we referred to units of time, we actually *were* thinking about complexity without realizing it. What we meant was “imagine the kind of work that takes the average team member about a day of heads-down time. That’s one point.” That's all well and good. But what is problematic is what was unstated: “when you hear more senior team members say ‘that’s one point’ and you think ‘it would take me a lot longer than a day,’ don’t speak up—just adjust your unspoken understanding of what a point is.” So that's not a good team dynamic.

So this helps me understand why it's illogical to say that points are time, how it can become toxic, why it hadn't worked out so badly on my projects, and why it's more correct, safe, and equitable to design your estimation process around the fact that story points are not time.
