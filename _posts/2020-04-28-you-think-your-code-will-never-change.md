---
title: "You Think Your Code Will Never Change"
---

You think your code will never change. You may not realize you think that, but you do. The reason I know this is because you make a variety of decisions that will save you money if and only if your code never changes.

Decisions like:

* **Little emphasis on code quality, including naming and large functions and classes.** If the code works, you won't need to touch it again, so you don't need to make it easy to understand.
* **Minimal code review.** As long as there aren't obvious bugs or security issues, the code is fine as-is. You don't need to transfer knowledge of it to anybody.
* **Hard-coded dependencies.** If the dependency doesn't need to change now, it won't need to change later.
* **Thinking of refactoring only in terms of large sweeping changes.** The system doesn't change much, so when it does change, it's because something new significant is needed.
* **Focus on "confidence" as the only goal of testing.** Unit tests give you insight into the internal quality of your code, but the internal quality doesn't matter since that code isn't going to change. Internal quality issues will make unit tests hard to write, pressuring you to improve the internal quality: but you don't need to care about improving the internal quality.
* **Stopping at partial test coverage.** You aren't going to be making lots of small refactorings. So if something is working and isn't likely to get broken, you don't need to test it.
* **More higher-level tests, fewer lower-level tests.** Higher- level tests give you more confidence and support large sweeping changes. They make it harder to get full test coverage, but that isn't important.

I wonder if you might say "I don't think all of that. I know my code will change! I just didn't know that things like code quality, small refactorings, and unit tests support change." No, that couldn't be the case. There is an abundance of information out there about code quality, small refactorings, and unit tests. But whenever that information is presented, it is almost always either silently ignored or vocally opposed as not worth the effort.

No, you might *consciously* think your code will change, but your decisions reveal the fact that, deep down, you don't believe investment in changeable code will pay off. You adopt practices that are cost effective if, and only if, your code never changes.

So the question is, are you right? Has it turned out that your code has never changed?

* Do you never run into code you need to understand to troubleshoot it, but you can't understand it?
* Do you never have to hack in a change that the original code wasn't meant to support and can't be easily changed to support?
* Do you never introduce a bug by making a change to seemingly-unrelated code?

I don't know you; for all I know, you may very well be right that your code never needs to change. If that's the case then you made the right decision: you chose the practices that are the most cost-effective when your code doesn't need to change. You avoided costly overhead to support changes that never came.

But I just wonder sometimes. I wonder if there is anyone out there who has code that sometimes needs to change. Do you think it's possible?
