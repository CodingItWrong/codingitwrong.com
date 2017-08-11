---
title: Can Tests Serve As Documentation?
---

When the benefits of testing are described, one of the points usually made is that the tests serve as documentation of the system. The idea as I understood it was that your tests could be readable enough that a new developer could literally read through them to learn the capabilities of the system. This idea was initially appealing to me, but I never saw any examples of developers doing this, and I heard more and more about making your _production_ code readable enough that it was its _own_ documentation.

Is tests-as-documentation, then, just a pipe dream? I thought it might be, but recent experiences on a project have made me think again.

I had been working on an Apple TV app with a coworker of mine. I usually like all my projects to have end-to-end tests, but there aren't yet any good UI testing tools for tvOS. (The built-in Xcode UI test tools for tvOS are too rudimentary to be useful as-is.)

As a result, we were limited to manually retesting the app after each change. We ran into all the usual difficulties with manual testing: it's hard to reproduce all possible scenarios, and it's easy to get bored and take shortcuts.

But the most significant problem we ran into with manual testing was not knowing what to test. Since my coworker and I are building different features, we don't necessarily know all the details of the features the other person builds. At times our work broke existing functionality that we didn't even know existed.

For some people this is just typical development life. But for me, fresh out of Rails development, it was rough. For a variety of reasons I won't get into in this post, the benefit-cost ratio for end-to-end tests is higher on the web. As a result, the projects I was working on had robust end-to-end tests that covered all their major functionality. When a change I made broke something, a test would fail and tell me. I would go and read the test. And because the Big Nerd Ranch web team puts a lot of effort into the readability of their tests, I was easily able to learn what the expected behavior was. I could roll back my change and see the test pass, then put my test back in and trace what exactly caused the failure.

I think _that_ is what it looks like for tests to serve as documentation. It's not that a new developer is expected to read the test suite from top to bottom. It's that, when something breaks, the tests document what the expected behavior is in a way that makes it easy to fix. That information isn't locked in the brain of another developer who needs to manually test after each change.
