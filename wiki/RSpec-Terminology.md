---
title: RSpec Terminology
---

Because [RSpec] uses a DSL to write tests, there's a layer of separation between the keywords you use and the internal terms for what's instantiated. Here's the mapping:

* `describe` - an ExampleGroup
* `it` - an Example
* `expect()` - an ExpectationTarget
* `to()` - a Matcher
* `let()` - creates a memoized helper method

Sources

* [RSpec: It's Not Actually Magic](https://www.youtube.com/watch?v=Libc0-0TRg4)
* [RSpec Core 3.4 Docs: Let and let!](https://relishapp.com/rspec/rspec-core/v/3-4/docs/helper-methods/let-and-let)
