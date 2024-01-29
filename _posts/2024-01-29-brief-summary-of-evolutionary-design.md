---
title: "A Brief Summary of Evolutionary Design"
---

*For years I've wanted to write something to advocate for evolutionary design. There is so much that can be said, and I hope to write more in the future. But I wanted to begin with this short definition of what I mean by "evolutionary design" and explanation of why it's helpful.*

Most developers have experienced a codebase that is hard to work with. What contributes to making code hard to work with? One major factor is over-design or under-design. Under-designed software results when people hack in changes never intended originally in the code, making it harder and harder to work with. Over-designed software is when the developer tries to predict all the ways it will change and plan for it in advance, making a codebase that has a lot of unnecessary indirection, most of which never gets used.

A way to find the middle path between over-design and under-design is evolutionary design. It involves two steps:

1. Building the simplest possible implementation of the requirements of the system today, and
2. When a new requirement comes in, rearranging the code so it's the simplest possible implementation of the new requirement.

The reason for evolutionary design is that we can’t actually know what will change in the codebase in the future. But by setting up the code so it's as easy to understand and change as possible, you can change it in any way it needs to change in the future.

Some key principles for ensuring the code is as simple as possible were [described by Kent Beck](https://www.martinfowler.com/bliki/BeckDesignRules.html):

* Reveals intention: that is, it is set up to help the reader of the code understand what the author of the code was trying to do.
* No duplication: that is, that every bit of business logic in the code is only in one place, so that if the business rule changes you don't risk two different spots in the code getting out of sync.
* Fewest elements: that is, you avoid statements, functions, and types that are unnecessary. This can be because they aren't executed, don't accomplish anything, or because there is a more straightforward way to accomplish the same thing.

The process of changing the code to best fit the new requirements involves refactoring. It’s important to note that this “refactoring” doesn’t involve pre-planned rewrites of large portions of the codebase. Instead, it’s the kind of refactoring described in Martin Fowler’s [*Refactoring* book](https://martinfowler.com/books/refactoring.html): small changes as you go to support the need you have in the moment. You rearrange a bit of code to be a better fit for the new feature you need to implement.

In order to refactor in this way, you need a suite of comprehensive tests: tests that if they pass you can be confident that the code is still working. The reason is because if refactoring is going to be a regular occurrence, you want to automate checking that you haven't broken anything, instead of having to check manually. If you have to check manually, the effort it takes to check and the risk of breaking something means that you will tend to defer refactoring until absolutely necessary, which means the code is not being regularly updated to best fit its new requirements. You don't necessarily need to use test-driven development to write this suite, but test-driven development is one tool you can choose to use to help you ensure your test suite is comprehensive enough to support refactoring.

To learn more about evolutionary design and the kind of simple code, refactoring, and comprehensive tests that support it, I recommend reading the introductory chapters of Fowler’s *Refactoring*.
