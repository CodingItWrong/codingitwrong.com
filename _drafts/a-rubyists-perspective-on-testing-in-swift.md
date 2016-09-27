---
title: A Rubyist's Perspective on Testing in Swift
---

* Xcode's built-in UI testing had significant reliability issues when I tested it: a "hello world" test I recorded would immediately fail upon replaying it. Manually editing it didn't help. The [KIF framework](https://github.com/kif-framework/KIF) has been a lot more reliable for me. It also provides access to the real view instances, so I can do things like check the number of rows in a table.
* Direct model access (e.g. instantiating models in tests to set up test data) works well with KIF, but you may need to manually kick off a view to redload/redraw since you're changing its data in an atypical way.
* KIF's main examples are in Objective-C, but it works well with Swift too. Just follow the instructions in the readme and you shouldn't have any trouble.
* I tried writing my KIF UI tests with Quick, but the explicit `self` required in its closures was a big detriment to readability: I had to write `self.tester()` instead of 'tester()`, which was a big drag when the tester is referenced so many times. Saving the return value of `self.tester()` to a local variable wasn't a good workaround, because this broke line number reporting.
* But Nimble expectations worked great with KIF tests, even using normal Xcode test classes instead of Quick specs. I already find assertion-style less readable than expectation-style, and the fact that Xcode assertions have an `XCT` prefix makes the readability even worse.
* Use Quick
* Running individual tests unreliable; disable/enable instead
* throws with XCTest, try! with Quick
* Go ahead and put protocols around services, since you'll want mock ones for unit testing
* deleteAll()
* Hand mocks