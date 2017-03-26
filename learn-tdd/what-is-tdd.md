---
title: What is Test-Driven Development?
---

Test-Driven Development (TDD) is an approach to automated software testing that involves writing a failing test before writing the production code to make it pass.

The order of steps is usually described is:

* **Red**: write just enough test to make it fail.
* **Green**: write just enough production code to make it pass.
* **Refactor**: look over your code to see if there's anything you can do to improve it while keeping the tests green.

## Why Use TDD?

Code naturally gets messy over time. As fixes and enhancements accumulate, the code gets harder and slower to understand and change. Attempting to do a "Big Design Up Front" doesn't help, because you'll inevitably guess wrong and over-design some parts of the system, slowing development down even further.

As an alternative to this, TDD enables an evolutionary design process where you can:

- **Start small** with only the features necessary to meet today's requirements. This keeps you from speculating and building more features that aren't used and only serve to make the code harder to change.
- **Keep it simple** with objects that are easy to understand and test. Isolation tests help you create classes with a clear purpose (high cohesion) and well-defined dependencies (low coupling). This makes them easier to change when new requirements come along.
- **Make quick, confident changes** with a regression test suite that covers all of your functionality. Your well-factored classes allow for easy changes, and any time a change isn't easy you can refactor with the tests proving that you aren't changing existing behavior.

## This Site's Approach

There are a few different schools of TDD, and this site follows the London school. There are a few terms closely related to the London school:

- **Behavior Specification**: referring to the mindset being less about testing for correctness and more about defining the behavior of the system.
- **Outside-In Testing**: referring to the fact that you first test the outside of your system the way a user interacts with it, then you let that test lead you to the individual classes inside your system you need to specify and build.
- **Isolation Testing**: referring to the fact that it tests each class in complete isolation from other classes.
- **Mockist TDD**: referring to mocks, a type of test double used to isolate units and provide visibility into the messages your application sends between its objects.

For more information on the different schools of TDD, see [Approaches to Testing: A Survey](http://codingitwrong.com/2016/02/08/approaches-to-testing-a-survey.html).

## Resources

* [The Wikipedia article](https://en.wikipedia.org/wiki/Test-driven_development) on TDD
* [*Test-Driven Development by Example*](http://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530), the original work on TDD, which corresponds to another school of TDD: the classicist school
* [*Growing Object-Oriented Software, Guided by Tests*](http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626), the authoritative work on the London school of TDD
