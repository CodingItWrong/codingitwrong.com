---
title: How to Be Context-Sensitive
---

There's a joke that says that a consultant's answer to every question is "it depends" — but there's a good amount of truth to it, and this hit home for me recently.

When I had heard Node and Elixir developers talk about high throughput on their systems, I thought it was premature optimization. Sure, WhatsApp needed to scale to millions of users, but it's kind of arrogant to assume you'll need to. The kind of throughput that Rails offers [ought to be enough for everyone](http://quoteinvestigator.com/2011/09/08/640k-enough/), right? But then I attended a developer conference and met a developer who works for a company processing data from Internet of Things devices used for monitoring industrial equipment. For him, high throughput wasn't about future speculation, it was about his business even being viable.

That experience helped it finally sink in for me that not everybody builds the kinds of apps I build. It's led me to think a lot about the context I work in, how there are others who work in different contexts, and how those contexts could mean that things that work for me won't work for them.

So here's my context as a developer:

- I have a **"traditional" computer science background**: I grew up in the 90s with the privilege of having easy access to computers, programmed from early on, and got a great computer science college education.
- I've mostly done **server-side web development**, with only small amounts of JavaScript on top. I've also done six months of full-time iOS development recently. So my experience in "frontend" development (both native and in-browser) is limited.
- The systems I've worked on have mostly been **small, greenfield systems** that I've either kicked off or joined while they were still under initial development.
- These systems have mostly been what I'll refer to as "**information systems**," by which I mean systems for taking input from humans, performing some simple operations on it, and displaying it back to humans. They aren't systems focused on storing or processing large amounts of non-human-generated data, or on advanced visualizations. Justin Searls recently referred to these as "spreadsheets on the web."
- As is common these days, these systems have often been **integrated** with external first- and third-party systems, rather than being standalone.
- I've mostly worked on **small teams**. For many of my projects, I've been the sole developer as well as performing project management. For others, I've been one of maybe five developers max working on a given system.
- I've worked with developers of all **different experience levels**. This spans from people who were just learning programming to published authors of dev books, and from people who grew up with iPhones to people who used punch cards.
- I've worked in **product shops, consultancies, through placement firms**, but almost never as a freelancer drumming up my own business (thankfully!)
- I've worked in **all kinds of process**: waterfall, agile, and no-process.

Why do we hesitate to talk about the context we work in like this? Maybe we don't think it affects us. Maybe we don't want to admit our limitations because we're afraid it will affect how people see us or our hireability. Maybe it's because we're afraid of others giving an excuse for not following our advice. If anyone can say "that doesn't apply to me because my context is different," we can't make them learn anything.

But that last phrase is telling: no matter _what_ approach we take, we can't _make_ anyone learn anything. The best we can do is try to get to know people and their context, hear what their challenges are, share what we've learned and how it's helped us, and make suggestions for things they could try.

It's overwhelming to think about analyzing all these factors of our context to figure out which principles apply in other contexts and which don't; but thankfully, we don't have to. We can just share what we know, being explicit about our context, and let others make application as they will. We can refuse to make black-and-white statements about contexts we don't have experience in.

When you hear people talk about ideas without sharing their context, try to find out their context to see if that helps you know how to apply it. Try to share your own context when you share your ideas. And if you hear someone making pronouncements about "all developers" or "all software," think about whether they really have the breadth of experience to be able to make such a statement.
