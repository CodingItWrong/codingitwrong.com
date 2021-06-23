---
title: Taking a Joke About Code Quality Too Seriously
---

I saw a joke on social media about code quality. It wasn't funny enough to repeat here, but the format may be familiar. It highlights the contradiction of a senior developer who would on the one hand require job interviewees to show they can write maintainable code, but on the other hand that senior developer's code isn't very maintainable at all.

This definitely does happen at many companies. So how do you respond to that reality? A few options are:

* You could call that developer a hypocrite and say they should write the code they say they want.
* You could say that code quality doesn't really matter and they should stop saying it does.
* You could say that this is an unfortunate reality of the pressures of real work, but there's nothing to be done about it.

I think there's a better response. I think that although high-quality code is the ideal, you often can't write the code that way right away. You have to get down a first idea and mess with it until it works. That code won't meet code quality standards at first, because the focus is exploration and getting something working. This is where most code has to start. And this is where most code stops: you got it working so you move on.

But it's true that in that form the code isn't maintainable. That's why you need a second step, refactoring: rearranging that small chunk of code so that it behaves the same but is arranged to be more readable, understandable, and modifiable. That's the step that gets skipped, which leads to a low-quality codebase. The developer didn't write the code wrong, per se: they did the first step correctly, but they just didn't do the second step.

Why don't people refactor? Because sometimes it's hard to rearrange the code without breaking it, so it doesn't seem worth the effort. When is it hard to rearrange the code without breaking it? When you don't have thorough tests that you can trust to fail if something changes.

So yes, the conclusion is me killing the joke with something utterly boring: you should write thorough tests.

A lot of folks will say that tests aren't pleasant but are good for you, like eating your vegetables. I don't think it needs to be that way. Test-driven development makes writing thorough tests fun, and to some degree that's biologically demonstrable: you get small shots of endorphins as you have small successes every few minutes getting a test to pass.
