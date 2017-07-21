---
title: Giving Up on Certainty
---

It seems to me that the root of agile development practices is an acknowledgement of human limitations. One such limitation is limited knowledge: we can't know things with certainty. Sandi Metz writes in [_Practical Object-Oriented Design in Ruby_][poodr] that "at [an] early stage of your project you cannot possibly get it right" because “The future is uncertain and you will never know less than you know right now."

Uncertainty explains a lot of the challenges I've experienced and continue to experience in development. The problem comes when the process you follow assumes that you *can* have a high degree of certainty:

- You assume you can be **certain of the requirements** at the beginning of a project. This means not only that you understand the requirements correctly, but also that the business is certain about what they want. If you're comfortable going that far, this also assumes that *customers* know what they want and won't change their minds, and that the industry you're in won't change.
- You assume you can be **certain of the design** of the code at the beginning, that it will give you the flexibility you need and will allow you to extend it in the way it needs to be extended.
- You assume you can be **certain of the implementation** of that design at the beginning. I don't usually distinguish design and implementation (the code *is* the design), but in this context "implementation" emphasizes whether the code is doing what you expect it to do. You assume that you can fully understand the code you've written, including all its interactions with inputs, state, and other code, and that you know for sure it's correct.

In my thirteen years as a full-time developer, these assumptions have been proven false [in my context][context]—not occasionally, but consistently. There's no longer any question in my mind: the project *will* change dramatically, and practices that account for uncertainty and change will be more effective than those that deny it.

That's why I'm drawn to agile development practices:

- I want to **build the minimum possible** for the most important requirement I have right now, then get feedback to see if it works out the way I expected, and then see what's the next most important thing at that time.
- I want to design my code to be **simple to understand and easy to change**, so that I can adapt it to handle future requirements. I don't want it to be highly-coupled so that it's difficult to change. And I don't want to design in "flexibility" from the beginning, because I don't yet know what dimension I'll need flexibility in.
- I want to write **tests that confirm my code behaves the way I expect**: that when I give it certain inputs, I get the outputs and side effects I want. I want to be able to refactor my code to handle new requirements, knowing that its externally-visible behavior is the same.

Do you believe you can have certainty—about requirements, design, and implementation? If not, do you believe that if you just try harder then next time you can have *more* certainty? Or do you believe, like Sandi Metz, that "at [an] early stage of your project you cannot possibly get it right"? Does your approach to development reflect this?

This is not to say that there are *no* systems you can have a high degree of certainty about—they just aren't the kinds of systems I've usually worked on. As David West argues in [_Object Thinking_][thinking], if you're working directly with mathematical formulas or close to the level of the machine, you can be fairly certain of what's needed, and so it may be worth it to trade changeability for higher performance and mathematical proof of correctness. But if you're working in an application where the requirements are determined by user desires or business rules, it's likely you will need to change it over time. In that case, changeability is likely the feature you need to prioritize the most.

When you hear someone introducing a new programming language, paradigm, or principle, ask yourself if they're acknowledging uncertainty and the need for change. If not, look into how that new approach affects changeability before you embrace it.

If you'd like a high-level overview of a kind of development process that takes uncertainty into account, check out Ron Jeffries' [_The Nature of Software Development_][nature].

[context]: /2017/07/16/how-to-be-context-sensitive
[nature]: https://pragprog.com/book/rjnsd/the-nature-of-software-development
[poodr]: http://www.informit.com/store/practical-object-oriented-design-in-ruby-an-agile-primer-9780321721334
[thinking]: http://www.informit.com/store/object-thinking-9780735619654
