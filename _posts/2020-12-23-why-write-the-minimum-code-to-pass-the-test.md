---
title: Why Write the Minimum Code to Pass the Test?
---

Probably the least intuitive thing about test-driven development is the fact that you're supposed to write the *minimum* code to pass the current test. Learners struggle with doing that when they *know* what the code ultimately needs to be. Even for experienced TDDers, it can sometimes be *more* work to figure out how to write the minimum code than the final code. So why bother? Why is it valuable to write only the minimum code to pass the test?

To answer this, consider a scenario. Say you’ve written a piece of code, and now you want to write unit tests for it. What should you test?

Maybe you think through success and error scenarios. Maybe there are several significant logic paths that you test. How do you know when you’re done? When code coverage says you’ve covered every line? It can be discouraging to get there. And multiple code paths means there could be many more important combinations.

A better way to get thorough tests is to think in terms of **behavior specification.** *Before* you write the code, you’ll write specifications that describe what you need the code to do—in the form of tests. Then you’ll implement the behavior that you specified. This way, you don’t need to ask what to test—if you need functionality, then you need to specify that functionality.

And you’ll **write only the behavior that you specified:** that is, you’ll write the minimum functionality necessary to get the specs (specifications) to pass. Why? Because that is the only behavior they’ve specified. If you need more behavior than that, you need to specify it.

If you write more behavior than you specified, then **you are writing unspecified behavior.** And unspecified behavior can have all kinds of problems. It isn’t described anywhere so it’s hard to find out about. There’s no guarantee that you’ve executed it so it may not work as intended. If it breaks there is no warning. And if it is removed there is no way to know.

So the answer to the question “why should I only write the minimum functionality to pass the test?” is “because you shouldn’t write unspecified behavior,” or, “because you should specify all behavior.”

This demonstrates one of the benefits of the term “spec” over the term “test”. Linguistically, it’s totally valid to test just one case or one bit of code. But it’s a bit of a contradiction to say “I’ve written a specification for one scenario.” Specification implies thoroughness. And that’s what we want in our tests.
