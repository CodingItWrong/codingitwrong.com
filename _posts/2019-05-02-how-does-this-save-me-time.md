---
title: How Does This Save Me Time?
---

The recent [episode of the Copy, Paste, Repeat podcast with Yehuda Katz](https://copypasterepeat.simplecast.fm/4460c8fc) helped me see some big-picture patterns of what people are looking to get out of programming, why frameworks differ, and how we can make strategic choices for our current job and our career. In that episode, Yehuda made an observation about paradigms and how they impact the work that we do as developers:

> Fred Brooks said this a long time ago: there are paradigms, and paradigms early in software helped us a lot. It was like, `if` statements: that is much better than `goto`, that is great, that actually saved us a lot of time.
>
> Eventually, most of the work that everyone does all day has nothing to do with programming paradigms, period. Almost all the work I do is like, “let me talk to a designer, let me think about that. I need to make a few more screens. Okay, now that I have all the screens, how does accessibility work? What happens if I hit tab a bunch of times? I put in a credit card; it seems like it should have the credit card font, not the regular font—that seems like a nice idea. Oh, but now it’s too wide.” None of this has anything to do with functional programming and it would not actually be solved if I removed state.

Beneath Yehuda's direct point is a foundational assumption that I think is extremely valuable to focus in on:  **improvements in software should save us time.** New approaches aren't better because they're novel, more aesthetically appealing, or match some platonic ideal (as the podcast episode also gets into). Instead, they should noticeably reduce the amount of time we have to spend developing.

What should that time savings look like? Yehuda argues that improvements should **save us time building features.** And in his experience stateless functional programming doesn't help with that. What *does* he think helps? We can answer that by looking at what Ember.js does: including features like a build system and routing out of the box, reacting to data mutations directly, and having a recommended data layer with a conventions-based approach that allows you to skip the configuration and boilerplate.

When Yehuda refers to a focus on "functional programming" and "removing state," there's little secret that he's primarily referring to the React ecosystem. Does *that* community think "functional programming" and "removing state" save them time? We can see the answer by going back to [an early conference talk explaining React and Flux](https://youtu.be/nYkdrAPrdcw), the latter being the architecture ultimately popularized by Redux. Facebook Chat's notification count kept getting out of sync with the message lists, and when that happened it was difficult to troubleshoot what was causing the bug. Introducing Flux's one-way data flow allowed fixing this bug for good. So React and the Flux architecture are intended to **save us time troubleshooting difficult bugs.**

From the podcast, Yehuda doesn't seem convinced that this approach saves much time, and I haven't seen that savings in my own context either. So I would rather reach for Ember and the productivity savings it provides.

Yehuda also brought up a third approach to saving time when he described a subtle difference between his own view and and DHH's. He relayed that DHH had said that he isn't interested in building beginner-friendly software—but, by contrast, Yehuda *would* like to do so. So this is a third possible way to save time: **saving newcomers time learning programming.**

When the topic of saving time comes up, I can't help but think about agile development practices, and specifically about change in software. One of the four values of the [Agile Manifesto](https://agilemanifesto.org/) is "responding to change over following a plan." In Kent Beck's [_Extreme Programming Explained_](http://www.informit.com/store/extreme-programming-explained-embrace-change-9780321278654) (subtitled "Embrace Change") he states that "Many XP practices are intended to lower the cost of ongoing design." I think it's a reasonable interpretation to conclude that extreme programming in particular, and agile development in general, attempts to lower the cost of *change*, to **save you time changing software.**

So that's at least four different ways software approaches try to save me time. And I'm finding the question "how does this save me time?" to be quite a helpful lens for comparing software approaches. For example, Ember and React try to save you time in different ways, by addressing very different problems. It's not that React is *slightly worse* than Ember at providing you all the tools you need to build business features; React is *entirely uninterested* in doing so. It's not that Ember is lagging slightly behind React in locking down state mutations; Ember *joyfully embraces* a mutation-based API. This is why when you're immersed in one framework, others can seem absurd: they're so much worse than your framework at the problems *your* framework is trying to solve (but that the other framework is not).

Maybe this all seems obvious to you: of course different frameworks think about things differently. I often hear the phrase "solve different problems"—in the Flux talk linked above, for example. But the problems are usually not clearly defined; the phrase is usually a handwave meaning "do different things somehow." When you actually define *how* different software approaches propose to save you time, the fundamental differences become much more clear.

This perspective suggests a question for assessing your choice of framework: which problems are currently slowing you down the most? Is it:

- Onboarding junior developers?
* Overhead building features?
* Slowdown as the app grows?
* Debugging?

I most often experience slowdown from an overhead of building features and maintaining the app, so this is why I tend to reach for frameworks like Ember and Rails (for the former need), and to apply agile development practices (for the latter). If your slowdown comes from different sources, it will make sense for you to reach for different solutions.

At a higher level, if you have the privilege of *choosing* which companies to work for, which frameworks to learn, which research to do, or which open-source projects to invest in, your focus isn't dictated by where you are currently. Instead, it comes down to which problems you're most interested in addressing. Do you want to help make it quicker:

* For new people to get into programming?
* For experienced developers to build features?
* For experienced developers to change software?
* To troubleshoot subtle bugs?

All of these are areas where we can advance the state of our industry, so each is valuable. And I'm sure there are more problems than these, and more approaches that address them in different ways. Your choice comes down to which kinds of problems you're most passionate about working on.
