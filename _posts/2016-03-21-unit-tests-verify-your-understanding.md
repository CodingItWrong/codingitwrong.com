---
title: Unit Tests Verify Your Understanding
tags: [testing]
---

In [The Deep Synergy Between Testability and Good Design](http://vimeo.com/15007792), Michael Feathers argues that unit tests are a way to understand your code. This got me thinking about something I read in [Growing Object-Oriented Software, Guided By Tests](http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626). When I checked, sure enough, it makes a similar point: while acceptance tests verify the _external_ quality of your system (does it function?), unit tests verify the _internal_ quality (including whether developers can understand the code).

This led me to a new way to think about the difference between acceptance and unit tests: **acceptance tests verify _that_ your system works, and unit tests verify that you understand _how_ it works.**

In other words, if you have an acceptance test for a feature, you can hack away at the code until you get the results you want without having _any_ idea how the code achieves those results. But once you try to write a unit test, you have to _understand_ how the code works: how it reacts to different inputs, what collaborators it needs, and (if you're using test doubles) what messages it sends to those collaborators.

This same effect happens whether your acceptance testing is done manually or automatically. But I'd argue that, surprisingly, it might be _worse_ if you have automated acceptance tests. If it takes you three minutes to click through a web browser to see if your latest change worked, it's in your best interests to understand what your code is doing first. But if it takes five seconds to run an automated acceptance test, it's hard to avoid the temptation to experiment and make code changes you don't understand, just to see if they make the test pass.

But is understanding really important if you have acceptance tests that specify the system's behavior—and will fail if something changes? I'd argue that understanding is still important. Acceptance tests rarely test every possible combination of events and state, so a situation will likely arise in production that you haven't tested for. (If that wasn't the case, the idea of "write a test to reproduce the bug, then fix it" would make no sense.) When that unanticipated situation arises, you don't want your system to be a black box of code that nobody understands; you want it to be a set of components that behave in predictable ways.

Of course, this doesn't mean that if you don't write unit tests you don't understand your code. But writing unit tests is a discipline that will help you make sure you understand it. They also serve as documentation to help "Other Developers Including Your Future Self(tm)" understand the code.
