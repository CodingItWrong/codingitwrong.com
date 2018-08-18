---
title: "There's a New RSpec Book In Town"
tags: [ruby, testing]
---

Working on Big Nerd Ranch's web team is where I first really got a feel for testing, and as I looked for a book that summarized their approach, _The RSpec Book_ was the best I found. It described a behavior-driven, outside-in, mock-based approach to testing. It's the book I recommended to anyone learning testing, regardless of their language.

By the time I found the book, though, a few of its elements were a bit dated: it used the old `should` method that has been superseded by `expect(…).to`, and it recommended using Cucumber for acceptance tests, which seems less common now than just using RSpec itself. This is why I'm excited about the spiritual successor to _The RSpec Book_: Myron Marston and Ian Dees' [_Effective Testing with RSpec 3_][etr3]. It's a comprehensive introduction to modern RSpec and outside-in testing in general.

If you've never used RSpec before, the book walks you through installing and getting started with its API. It introduces testing concepts like examples and expectations. For those who aren't convinced of the value provided by RSpec's `before` hooks, `let` construct, and `context`s, the book walks you through the cases where they're helpful and how they're an improvement over "plain Ruby" alternatives.

In addition to matters of RSpec functionality, the book also gets into testing theory. Rather than assuming you're motivated to write tests, it describes benefits these tests provide. It introduces the TDD cycle ("red-green-refactor") right from the beginning, and the example app that forms the bulk of the book follows the outside-in nested TDD cycle.

Achieving the benefits that testing promises can be challenging, but the book walks you through how to do so. Just about every question I've asked about testing approach is raised and answered, including references to most of the other key testing books and blog posts that have been recommended to me. It's a tour of the most helpful points in a modern view of testing. These points include:

- Whether or not to remove duplication, depending on what is most readable in the situation.
- Where to handle testing edge cases.
- When to use real collaborators and when to use test doubles -- and which kind.
- How to test external integrations.
- When to use features like partial doubles, `any_instance`, and stubbed constants, and when to avoid them.
- How to do large-scale refactoring.

_The RSpec Book_'s example app focused on a command-line app for simplicity. This aids in learning, but let's be honest: at least in the US, most Ruby development is web development. By contrast, _Effective Testing_ doesn't shy away from the complexities of the web. The main example app is a Sinatra app that stores data in SQLite, and later in the book rspec-rails is described.

In Ruby, testing is a well-trodden path, but in many other languages testing is a jungle. As you're blazing a trail, it can be helpful to see what the well-trodden path can look like as a way to help you navigate. That's why I recommend _Effective Testing_ for developers in any language. And you're a Ruby developer, _Effective Testing_ is the new standard reference for RSpec and outside-in testing. It's both an approachable introduction for beginners and an effective summary for experienced testers. It's on my top shelf of books, and I heartily recommend it.

[etr3]: https://pragprog.com/book/rspec3/effective-testing-with-rspec-3
