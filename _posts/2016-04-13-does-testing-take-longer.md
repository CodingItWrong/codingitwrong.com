---
title: Does Testing Take Longer?
tags: [testing]
---

It bothers me any time there are two different views on a topic and I don't know where I stand on it. Lately, I've been thinking about the question of whether writing automated tests makes your development time longer.

One view is that writing tests self-evidently takes longer: it's code you're writing that you wouldn't otherwise need to take time to do. Even if you make allowances for the time it takes in the short term to learn how to write tests efficiently, in the long term tests still take more time to write than no tests.

The opposing view is that you always test your code, and that if you don't take time to write automated tests, you'll spend the same time manually testing--more time in the long run.

So which is it? First I'd like to give what I think is the answer when using test-driven development. In my experience, test-driven development *does* in fact take more time than writing code without tests--but not because of writing the tests themselves. Instead, it's the *refactoring* step that takes longer. Looking over your code, finding duplication, deciding whether it's the right time to create an abstraction--all of these take time.

If that's true, then the most helpful question isn't whether tests are worth extra time--it's whether *refactoring* is worth extra time. That's a tradeoff, too: if a system isn't going to be used for very long, or if it's not likely to change very much, refactoring might not be worth it. A lot has been written on the value of refactoring, so there's plenty to learn from to inform your decision.

If it's true that refactoring and not writing tests takes longer, this suggests an interesting possibility. What if you're only writing acceptance tests, and not writing unit tests or doing regular refactoring? It seems like these acceptance tests shouldn't add significantly to development time, once you've learned enough to be efficient at writing them. I'd be interested to hear if anyone has experience with this approach.
