---
title: Experimentation
---

When you become convinced that [different contexts have different needs]() and that [you can't force people to learn](), something interesting happens. You start to experiment.

When I say "experiment," I don't necessarily mean in the scientific sense. Certainly some agile teams track metrics, and measure whether changes to their methodology helps or hurts those metrics. That can work fine as long as you pay attention to more than just the numbers. But I've never been on a team that has done that kind of measurement.

Kent Beck makes a statement in passing in [_Extreme Programming Explained_][xp-explained] that captures what I mean when I say "experiment" (emphasis mine):

> “Experiment with XP using these practices as your hypotheses. For example, let’s try deploying more frequently *and see if that helps*.”

It's informal, and based on the assumption that the team can tell when something is helping or hurting their development.

I was surprised when I started naturally taking this experimental approach without intending it. Here are a few examples.

I was asked to join a planning meeting for a project that was having trouble breaking stories into small pieces. In the meeting, the story under consideration was reimplementing a modal window with a form. The reimplementation would take several weeks, but iterations were only a week long so we wanted to break it into smaller stories. We didn't want to replace the working form with an only-partially functional form, because the form was currently in use. And we didn't want to gradually refactor the old form into the new one: the new implementation was so different that refactoring would be prohibitively expensive.

Instead, I proposed the idea of having *both* the old and new forms in the app at the same time. This is related to the concept of a [feature toggle]. There were a few different reasons that I proposed this as an experiment rather than an absolute:

- I had never tried this technique, so I wasn't sure how well it would work.
- I had never done it on iOS in particular, so I wasn't sure what the technical overhead would be.
- I hadn't worked with this client much, so I didn't know how they would feel about having two versions of the same form in their testing build, an old complete one and a new incomplete one.

The team ended up having trouble delivering the feature in small stories, but since I wasn't on the project team I didn't get to find out why.

Another example of an experiment was a refactoring I did for a video chat app. We had a `Call` class that handled logic related to connecting calls. It had a lot of conditional logic depending on whether the call was incoming or outgoing. We were considering whether to separate it into an `IncomingCall` and `OutgoingCall` class to remove these conditions and make each easier to understand. Another dev on the project was trying to think through in advance if it would help or not. Instead, I proposed that we just try to gradually refactor it and see if we like it. He was hesitant, but agreed to let me give it a try.

My first experiment was to duplicate the `Call` to a new `IncomingCall` class, get it working, then gradually remove any unnecessary functionality from it. This proved to be difficult, though. This was a Swift codebase, and there were a number of method signatures in the codebase referencing the `Call` type. Attempting to pass an `IncomingCall` instead resulted in a type error. Since I wasn't as familiar with static type systems, I wasn't able to figure out how to do this refactoring gradually. So I backed up and tried another experiment.

This time I started with an _empty_ `IncomingCall` class that subclassed `Call`, then I gradually copied over methods from `Call`. This experiment succeeded--since `IncomingCall` was a kind of `Call`, the type system handled it just fine.

As a long-time Java programmer, my next impulse was to make the parent `Call` class "abstract," since it should never be instantiated. But rather than abstract classes, Swift has a closely-related concept: protocols with extensions allow supplying default behavior and specifying what behavior descendants need to implement. I began this refactoring, but differences between protocol extensions and abstract classes made it difficult. I ended up giving up on this experiment; separating out the `IncomingCall` and `OutgoingCall` was good enough for now.

Sandi Metz took the idea of experimentation even further in some advice she gave when I took a training class from her. She said that when you are having a disagreement with a teammate about how to do something and you're convinced that you're right but you can't convince the other person, try doing it their way together. If it ends up working out, then you've learned something! And if you end up being right and it doesn't work, you've learned together.

If you're spending a lot of time trying to figure out plans that you're sure will work, or trying to convince team members of something, try doing an experiment instead. It can be a much better way to get information than speculating.

[xp-explained]: http://www.informit.com/store/extreme-programming-explained-embrace-change-9780321278654
[feature toggle]: https://martinfowler.com/bliki/FeatureToggle.html
