---
title: A Rubyist's Perspective on Testing in Swift
---

I'd messed around with [some basic TDD in Swift](http://learntdd.in/cocoa-touch) before, but this week I took a hackcation to start a new iOS app project and I wanted to really try out TDD in more depth. Here are the things I learned:

## General

I test-drove all the functionality of the app, but when I switched to user interface tweaks, I didn't write any tests for those: I just reran the test suite after changes to make sure I didn't break anything. This _did_ mean that I sometimes got out of the habit of writing tests first, which bit me a few times when I started writing testable functionality before the tests for it.

Adjusting to explicit protocols was interesting. In Ruby there's no such thing as interfaces or protocols: if you want a test double to substitute in for a real object, just create a different "duck type" object that responds to the same methods. But in Swift (like in many languages), protocols have to be made explicit. So if you're planning on doing any unit testing, go ahead and put protocols around service classes, since you'll probably end up creating test doubles of them eventually.

## Acceptance Tests

Xcode's built-in XCUI testing had significant reliability issues when I tested it: a "hello world" test I recorded would immediately fail upon replaying it. Manually editing it didn't help. The [KIF framework](https://github.com/kif-framework/KIF) has been a lot more reliable for me. It also provides access to the real UIView instances, so I can do things like check the number of rows in a table.

KIF's main examples are in Objective-C, but it works well with Swift too. Just follow [the instructions in the readme](https://github.com/kif-framework/KIF#use-with-swift) and you shouldn't have any trouble.

KIF doesn't reset the app after each test, so you'll want to go back to the main screen of your app after each test, so that tests can be run independently without breaking each other. If there's always a consistent button you can tap to go back to the main screen, you can put that tap in the `setUp()` method of a base test class. That usually worked for me, although I ended one test on a modal so I needed to close it as part of its own test.

I tried writing my KIF UI tests with the Quick framework, but it uses a lot of closures, and Swift 3 requires an explicit `self` to call instance methods from within closures. This meant I had to write `self.tester()` instead of `tester()`. This may seem minor, but in my mind it was a big hindrance to readability, especially because the tester is referenced so many times. I tried working around this by saving the return value of `self.tester()` to a local variable--but this ended up breaking line number reporting for test failures, so this wasn't an option.

Even though I didn't use Quick, [Nimble expectations](https://github.com/Quick/Nimble) worked great with KIF tests, even using normal Xcode test classes instead of Quick specs. This allows me to replace `XCTAssert(…)` with `expect(value).to(…)`, which I find a lot more readable. Assertion-style isn't very appealing to me even when it's as readable as `assert_equal …`, and the fact that Xcode assertions have an `XCT` prefix makes the readability even worse.

Direct model access (e.g. instantiating models in tests to set up test data) works well with KIF, with a few caveats. First, you will want to reset the model store after each test to make sure your tests don't pollute each other's data. And if you're updating data for a view that's on-screen (like a table view on the main screen of your app), you may need to programmatically tell it to reload its data.

## Unit Tests

I did end up using [Quick](https://github.com/Quick/Quick) for unit tests, along with Nimble expectations. One difficulty I had with Quick is in getting it to compile with method calls that can throw exceptions that I don't _want_ to catch. In XCTest I just declare test methods with `throws` when necessary. But the closures passed to `it` can't be declared to throw. And putting a do/catch construct around each hinders readability. Instead, I prefix the calls to the throwable methods with `try!` instead of `try`. This means that no do/catch construct is required; you'll just get a runtime error if one throws an exception you aren't testing for.

Another downside with Quick is that Xcode doesn't show a preview of its tests in the testing sidebar as reliably, because Quick's testing methods are dynamically created. But the sidebar preview isn't 100% reliable with XCTest either. And I didn't find myself missing the sidebar preview much. I thought I would need a reliable testing sidebar so I could click individual tests to run them. But, as it turns out, Xcode isn't that reliable at running individual tests either: it would often run _no_ test then say it succeeded. To work around this, when I wanted to run individual tests I right-clicked on test classes and suites to disable them, only leaving enabled the ones I wanted to run at the time. I didn't find myself needing to disable individual test methods/examples: acceptance tests mostly had one test case per class, and unit tests ran so quickly I usually just left all of them enabled. This worked okay, but it was definitely cumbersome. If I wasn't already sold-out on TDD, it likely would have presented a barrier to me getting excited about it.

[Automatically creating test doubles is hard in Swift](http://blog.pragmaticengineer.com/swift-the-only-modern-language-with-no-mocking-framework/) because of the lack of reflection capabilities. There are a few libraries out there to help, but I decided to start out hand-rolling my own test doubles, and that's worked fine so far. I added `Double` to the class names instead of `Stub` or `Mock` because I'm not rigidly following [the most commonly-accepted definitions of the terms](http://martinfowler.com/bliki/TestDouble.html). Instead, I'm adding just enough features to each to satisfy the tests I need to write.

## Core Data Models

Usually, I like naming my protocols the simplest name for the concept (i.e. `WidgetStore`) and the adding a qualifier to the concrete implementation (i.e. `CoreDataWidgetStore`). However, this wasn't possible when it came to creating a protocol for my Core Data models. (I usually don't create test doubles for models in other languages, but Core Data models were too hard to use because I couldn't even find a way to instantiate them without an `NSManagedObjectContext`.) But for Core Data models I couldn't name the protocol the simplest possible name (`Widget`) because that was the name of the concrete Core Data model that's generated by the "Create NSManagedObject Subclass…" command. Instead, I appended `-Model` to the protocol name (i.e. `WidgetModel`), and that seemed clear enough.

The advice I got from my [Big Nerd Ranch](https://bignerdranch.com) backend team for Rails Active Record models worked great for Core Data models as well: don't feel the need to test-drive simple stored attributes, but test-drive calculated properties and custom logic.

Without a factory framework like [`factory_girl`](https://github.com/thoughtbot/factory_girl), it was cumbersome to instantiate models. Because of this, I ended up falling into the trap of using a repository class (i.e. `WidgetStore`) in my model tests (i.e. `Widget`), just because it saved me some typing! But this store depended on _another_ store, and all of a sudden my model test was instantiating too many classes. When I realized this, I changed my model test to instantiate the models using Core Data directly, wrapped in test helper methods. I made as many helper methods as I needed to make each line as clear as possible. For example, I had a `createWidget(for: owner, completedAt: date)` method, then I added a `creatingCompletedWidget(for: owner)` for when the exact date didn't matter.

## Was it worth it?

Was all this testing worth it? Well, it's already supported a number of refactorings I've done to keep my code clean, including changing out views for different types of views, changing storyboard flows between view controllers, improving smoothness of UI animations, abstracting out duplicate code, extracting code from view controllers into new view subclasses, and renaming classes to better reflect what they're used for. That's a lot of improvements I wouldn't have been able to make in such a short time without tests as a safety net.

So, overall, starting out a Swift project with TDD was a great experience. I highly recommend trying out KIF, Nimble, and Quick to figure out what workflow works best for you!
