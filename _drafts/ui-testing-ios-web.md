---
layout: post
author: Josh Justice
date: 201X-MM-DD 10:00:53+00:00
title: "Giving Up on UI Testing: A Tale of Grief and Hope"
description: Why UI testing on iOS is hard but important, and ways we could make it easier.
excerpt: UI testing on iOS is particularly difficult. The cost of building and maintaining tests tends to outweigh the benefit. With these challenges in place, I'd like to talk about why it's still important to UI test on iOS, what is causing these challenges, and some ideas for what we can do to get past them.
categories:
- iOS
---

Since the [last time][last-time] I wrote about testing on iOS, my optimistic view has been…moderated a bit.
In fact, when it comes to UI testing, I'd say I've gone through the five stages of grief.
See if you can identify with any of these:

1. **Denial:** "There's no problem UI testing on iOS.
    Apple provides their own UI testing framework, and you can even *record* a test!
    And if you don't like that framework, KIF is really popular too.
    This shouldn't be a problem."
2. **Anger:** "Why doesn't anybody write UI tests?
    Do you *like* manually retesting every time you make a change?"
3. **Bargaining:** "Alright, I'll write the UI tests for the project, and if they break, I'll take responsibility for fixing them.
    Anything to get some kind of test coverage!"
4. **Depression:** "Maintaining these tests is way more work than any benefit they're providing.
    What am I gonna do?
    Is iOS the wrong platform for someone who cares about testing?"
5. **Acceptance:** "I think I need to give up on UI testing for now and find another way to get the confidence I want in my codebase."

Okay, maybe I'm letting myself get a bit too emotionally affected by testing, but the fact remains that UI testing on iOS is particularly difficult.
The cost of building and maintaining tests tends to outweigh the benefit.

With these challenges in place, I'd like to talk about why it's still important to UI test on iOS, what is causing these challenges, and some ideas for what we can do to get past them.

## Why UI test?

If you can get good unit test coverage for the rest of your app, does it matter if the UI isn't tested?
Well, the most important part isn't the UI per se, but rather *end-to-end* testing.
Even if you've unit-tested every part of your app, if they don't work *together* properly your users will still be upset.
The best way to get confidence that your entire app works is to have "end-to-end tests" that run through as many layers of your app as possible.
Ideally these tests interact with your app in the same way a user does, and that's where UI testing comes in.
By scripting taps and gestures and text input, you're starting your testing right where the user does.

However, there's an argument to be made that end-to-end testing may not be as necessary in iOS apps as in, for example, web applications.
Many iOS apps function primarily as a user interface on top of a web service.
Much of the nontrivial "business logic" often lives on the server.
If there isn't much business logic in your app itself, then manually tapping through the screens may be all the testing you need.

But sometimes our apps have more logic in them then we realize.
If you're feeling the pain of old code, you may want to rewrite the offending class from scratch.
But if you've ever done a rewrite of a significant chunk of code you know how risky it can be—so we tend to put it off.
A better alternative to rewriting is **refactoring**: making a series of small transformations to the code that preserve its behavior.
But how can you ensure you haven't changed the behavior after each small change?
That's where tests come in.
Unit tests allow you to refactor individual classes, but when you're reorganizing code between multiple classes, you need something to ensure the _overall_ behavior of the app stays the same.
That's where end-to-end tests really shine.

## Why is UI testing hard?

So that's a reason you may want end-to-end tests for your app.
So why is UI testing so much harder on iOS than on the web?
For server-rendered pages, the simplest answer is that there's a lot less going on.
You click a link, this sends an HTTP request, you wait for the HTTP response, then you check for elements on the page.
By comparison, on iOS you might have any number of animations and asynchronous loads going on at once.
It's hard to tell when you should test for something.

I don't think page loads are the core difference between iOS and the web, though: and this is clarified by JavaScript apps.
These also have animations and asynchronous loading, and some JavaScript frameworks have pretty good support for UI testing — [Ember][ember] and [React][enzyme], for example.
How have these open-source projects been able to accomplish what Apple hasn't?

I think the reason is the surprising power of open systems.
Everything in the web stack — [HTTP][http], [HTML][html], the [DOM][dom], and [JavaScript][ecmascript] — is based on a published specification with well-defined behavior.
On top of these stable primitives, an open market of ideas has formed.
Different projects pursue different priorities, resulting in a community that, as a whole, innovates in every direction at once.
In particular, JavaScript rendering engines innovate in bringing simplicity and predictability to the rendering cycle.
And simplicity and predictability make for good testability.

By contrast, Apple's platforms are closed: you may be able to see private APIs in the debugger, but you can't rely on them.
This closed platform allows Apple to offer great features, some of which are far easier to use than the DOM.
But that closed system also prevents the same kind of open market of ideas built on top of it.
There aren't nearly the same number of open-source projects, and their innovation is limited by a lack of access to low-level primitives.
The only organization with that access is Apple, and a single organization can only prioritize so many things that once — and historically UI testing hasn't made the cut.
That isn't a criticism of Apple, it's just the reality of closed systems.

## How Can We End-to-End Test?

If UI testing on iOS is so hard, is there another way to test end-to-end?
I've gotten ideas from a few different sources:

- Fellow nerd John Gallagher pointed out that there's often significant behavior in the UI that can't be verified by automated tests, because it doesn't have clear pass and fail states: things like animation smoothness and frame rate.
- In an influential presentation called "[Boundaries][boundaries]," Gary Bernhardt describes an architecture where the user interface layer has had all "decisions" pulled out of it into a more easily testable layer.
    In this approach, he doesn't feel the need to test the user interface layer at all.
- Martin Fowler describes a pattern called [Presentation Model][presentation-model], where you "represent the state and behavior of the presentation independently of the GUI controls used in the interface".

Putting these ideas together, they suggest separating the concepts of UI testing and end-to-end testing.
What if we isolated the UI in its own untested layer?
What if we moved as much logic as possible into a layer *just beneath* the UI, and started our end-to-end testing there?
In an approach like this, view controllers would handle only things like view setup and transitions.
They would delegate all data requests and user actions to a tested object.
End-to-end tests would still cover an entire sequence of user interaction, they would just exclude the UI portion that requires manual testing to be done well anyway.

The idea of thinning out ViewControllers is nothing new in the iOS community.
We haven't reached a consensus about which approach to take, and I think presentation models are an approach worth investigating.
An approach that makes end-to-end testing easier could have a transformative effect on the way we're able to refactor our old code and keep our development running quickly and smoothly.
I'll be doing some research in this area, so keep an eye out for an upcoming post!

[last-time]: https://www.bignerdranch.com/blog/a-rubyists-perspective-on-testing-in-swift/
[ember]: https://guides.emberjs.com/v2.11.0/testing/acceptance/
[enzyme]: https://github.com/airbnb/enzyme
[http]: https://www.w3.org/Protocols/rfc2616/rfc2616.html
[html]: https://www.w3.org/TR/html5/
[dom]: https://www.w3.org/DOM/DOMTR
[ecmascript]: http://www.ecma-international.org/publications/standards/Ecma-262.htm
[boundaries]: https://www.destroyallsoftware.com/talks/boundaries
[presentation-model]: https://martinfowler.com/eaaDev/PresentationModel.html
