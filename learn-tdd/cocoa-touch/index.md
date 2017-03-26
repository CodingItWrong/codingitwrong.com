---
title: Learn TDD in Cocoa for iOS
---

{% include tutorial-intro.md %}

To see how TDD works in Cocoa Touch, let's walk through a simple real-world example of building a feature.
We'll be using Xcode 8 and its built-in XCTest framework.
Even though Cocoa Touch has UI testing built-in, I ran into some reliability issues with it,
so instead we'll be using [the KIF framework](https://github.com/kif-framework/KIF/) for UI testing.
Each section of the article is linked to a corresponding commit in the [Git repo](https://github.com/learn-tdd-in/cocoa-touch) that shows the process step-by-step.
This tutorial assumes you have some [familiarity with iPhone development](https://developer.apple.com/ios/documentation/) and with [automated testing concepts](/concepts).

The feature we'll build is a simple list of messages.

{% include_relative _commits.md %}

## More Resources

To learn more about TDD, I recommend:

* [iPhreaks podcast episode 95: TDD](https://devchat.tv/iphreaks/095-ips-tdd-test-driven-development)
* [_Test-Driven iOS Development_](http://www.informit.com/store/test-driven-ios-development-9780321774187) - it's a bit dated: it's in Objective-C and precedes XCTest, but the principles of how to break up an iOS app for testing are still very useful.
{% include tutorial-goos.md %}

{% include tutorial-contact.md %}
