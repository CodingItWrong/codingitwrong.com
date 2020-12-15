---
title: Why Chasing Red Doesn't Work
---

You need to make a change to your code. So you think about it and decide what change you believe will work. Then you make the change.

But then something goes wrong: either a test fails, or you run the code and it doesn’t work. You thought you understood the code and the change you needed to make, but you weren’t quite right. You understood something incorrectly.

So you think about it again: what change do you need to make *now* to get the code to work? You decide and make the change. You’re hoping to both (1) get the *first* change working correctly and (2) get the *rest* of the application’s functionality working again.

If the tests now pass and the app works, you can breathe a sigh of relief—you’re done. You got the app back on track.

But what if that still doesn’t happen? What if after two rounds of changes, the app still isn’t working—what do you do?

The most obvious option—maybe the only one you can think of—is to keep going down the path you’re on. When something isn’t working right in your code, you fix it, of course. So you just think about the errors you’re getting and try again to fix them.

There’s just one problem—the last two times you tried to change the code, it didn’t work. That pattern suggests that if you keep trying to make *more* changes, those won’t work either.

There’s a second, deeper reason to think that continuing to make more changes isn’t likely to fix the app. Before your first change, the app was in a known, working state. You thought you understood what the app was doing well enough to be able to make this change, but the fact that that first change didn’t work means that your understanding of the app was incorrect. And, since that time, the app has had two changes made to it that you didn’t understand correctly. So in the code's current state your understanding of it is pretty low. And every time you make an additional change that doesn’t correctly fix the problem, the code drifts increasingly far into the realm of incomprehensibility, further decreasing your chances of ever fixing it.

At this point, it would be easy to feel pretty discouraged. If “make more changes” is in fact the only option, and it doesn’t work, then what does that say about your skills as a developer? And it can feel like I’m being pretty critical to point out in great detail how unlikely you are to succeed.

But the reason I’m pointing this out is because there *is* an alternative. It’s one that *is* likely to succeed. And it shows that the problem wasn’t your skills as a developer; the problem was that you got stuck going down a path that was making it harder for you.

What is the alternative approach to get the code working? You discard all the changes, resetting the code to the last known good state, and try again. You make a *different* initial change to attempt to get it right from the start. Ideally, a smaller change. You take a small step in the direction you need to go—a step that keeps the tests passing and the app working.

[_99 Bottles of OOP_](https://sandimetz.com/99bottles) is an excellent book that walks through a long small-step process just like this. Warning against the trap we discussed above, the authors refer to it as “chasing after red”:

> “If you simultaneously change many things and something breaks, you’re forced to understand everything in order to fix anything. You could end up chasing after red, with increasing desperation, before eventually discarding all of the changes and beginning anew.
>
> Making a slew of simultaneous changes is not refactoring—it’s re*hack*toring. It would be much better to make a series of tiny changes and run the tests after each. If the tests fail, you know the exact change that caused the failure, and can undo back to green and make a better change. If the tests pass, you know that the current code works, even if the refactoring is only partially complete.”

Many developers’ experience backs up the idea that making more and more changes leads to “increasing desperation.” And in this post we’ve seen one reason why that tends to happen. Every time you make a change that does not correctly fix the app, the code moves into more and more of an unknown state, further decreasing the likelihood that you will be able to correctly understand it to fix all of the errors.

The pragmatic thing is to discard your changes and try again. You aren’t throwing out work, you’re throwing out *bugs*. And the time you spent isn’t without benefit—you’ve learned a path that doesn’t work, and that will help you as you assess the code from the known starting state.
