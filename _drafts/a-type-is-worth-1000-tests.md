A type is worth 1000 tests talk ***

Ensure program correctness with fewer tests
This assumes not doing TDD, because if you were doing TDD you'd still write the same tests to drive out your behavior
Listing all possible program states and avoiding some that are invalid is not practically useful
Types are a design tool. They help the compiler confirming you implemented the design you wanted to implement. But if they hinder evolutionary design, then they require a big upfront design.

Make undesirable states unrepresentable
What if the states later becomes desirable? Then you have to change your types. But if your whole system is strongly coupled to types then that's shotgun surgery. For example, adding an enumeration case.

***

You cannot make all undesired states unrepresentable
Even if you could, the rigidity would cost you

Specifically, a type system reduces the number of undesirable states that are representable.
The argument is that as a result you have to write fewer tests. Just for the undesirable states that are representable.
But many undesirable states aren't something you'd reasonably run across, so you don't test for it. So it's not decreasing the number of tests you write.
The question is, what percentage of the undesirable states that a type system prevents are things you would test for? What percentage of tests does it save you?

Discomfort with the idea that you're making a judgment call about which invalid states are not worth testing for.
But if you still need some tests, you'll still be making some judgment calls.

Argument that whether you would write the tests or not, the type system provides extra guarantees.
Why wouldn't you want those guarantees? Could save you some time debugging sometimes.
Because everything is a tradeoff. The better question is, what are you trading to get those guarantees?
Time. A type system could save you some time debugging, but you're trading time to make changes to your system.

Idea that certainty is worth trading time.
Important to realize that it's a relative certainty: you're gaining certainty about some rare states, but you still have uncertainty about more common states. You still need to test.

Some domains are more mathematically precise than others. Some are more fuzzy than others. This affects how valuable the type systems protections are.
Problem comes with your worldview doesn't match with the domain your working in.

Architecture can be defined as the things that are hard to change. Your type system is usually hard to change. (Although typescript is interested in this regard)

***

Most of the invalid states that a type system protect you against don't matter.
Some of the invalid states that do matter the type system doesn't protect you against, so you still need to test.
Type systems make it harder to change your system.
If you're going to TDD for all the other benefits, you're going to write tests for the significant valid states anyway.

Static typers are concerned about the uncertainty of example-based testing.
I think that static typing bring certainty, it's not a certainty that makes your program behave correctly.