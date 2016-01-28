Types of test pros and cons:

- Unit/Isolated: fast and low repetition, test isolation; mocks can couple you to the implementation, and can end up only testing spelling
- Integration: closer to actual usage, but slower and can break
- Acceptance/system/end-to-end: testing the whole system, what user sees; lot of repetition and slow, can be more brittle b/c UI dependent, not test isolation

Been interested in testing for a long time, but this is the first place I've been able to consistently practice it.
How much of each type of test should I do? Unit, Integration, Acceptance.
There are different views on this!
As I tried to understand the different views, I found their answer to "how much of each type of test should I do?" was inseparable from bigger questions: what process do you go through? What is your end goal as far as good code?

This is a discussion, not a talk:

- I want correction when I'm wrong!
- I want to hear your view, even nuances of it--maybe we all agree, maybe not
- I want more resources to read/watch!

Some qualifications:

- There are more approaches than these
- There are variations within each approach
- Terms change meaning over time and person-to-person
- You don't have to always do only one of these. You can think of them as disciplines that apply for certain systems or parts of your code.
- But categories are still helpful for understanding differences and alternatives

# Regression testing

- Who? DHH
- Resources?
    - [Writing Software][]
    - [TDD is Dead][]
    - [Is TDD Dead?][]
- **What is good code?** Simple, readable code
- **How can I get good code?** I can write good code myself; having tests drive it makes it worse
- **What are the test types for?** acceptance and integration for regression, unit useless
- **How much do you test?** Test only what's valuable
- **What process do I go through to achieve this?** Write the code, then write the regression test
- How much code do you write at first? All of it.
- Dependencies? The real thing
- How do you verify? Check output on screen or in DB
- No refactoring or code reuse. He explicitly advocates against architecting for code reuse, i.e. command-line or APIs

# Classical TDD

- Who? Kent Beck, Martin Fowler; there is a lot of leeway here so specifically representing Ian Cooper's view
- Resources
  - [Classicist and Mockist TDD][]
  - [Mocks Aren't Stubs][]
  - [Is TDD Dead?][]
  - [TDD: Where Did It All Go Wrong][]
  - *[Test-Driven Development: By Example][TDD By Example]*
- Associated terms and concepts: Detroit/Boston school, middle-out
- **What is good code?** Classical OO principles and design patterns
- **How can I get good code?** Let tests provide feedback to consider
- **What are the test types for?** "unit" (not isolated) for driving design, then refactor without test feedback
- **How much do you test?** Test everything once; avoid double-testing, especially in extracted classes
- **What process do I go through to achieve this?** Red, green, refactor. In middle-out, domain classes, then build up
- How much code do you write at first? Absolute bare minimum to get the test to pass.
- Dependencies? The real thing when possible, stubs next, mocks last
- How do you verify? Check state
- High support for refactoring, low for code reuse
- Criticisms
  - Exposing state for the sake of testing, but "not a big deal"
  - Doesn't exert pressure on you to refactor; do more as you get more experienced
  - Extracting objects is painful

# Mockist TDD

- Who? Dan North, Steve Freeman, Nat Pryce, XP Tuesdays
- Resources?
  - [Classicist and Mockist TDD][]
  - *[The RSpec Book][]*
  - [Test Isolation is About Avoiding Mocks][Avoiding Mocks]
  - [Introducing BDD][]
  - *[Growing Object-Oriented Software, Guided by Tests][GOOS]*
- Associated terms and concepts: BDD (but that has a larger scope), isolation testing, London school, outside-in, need-driven development
- **What is good code?** Code that can be reused and recombined in different ways. Code with low coupling. Code that tells, doesn't ask.
- **How can I get good code?** Let tests drive your design at each layer, with mocks exerting pressure to reduce coupling.
- **What are the test types for?** Acceptance and unit for driving your design, regression secondary. Integration good because a mix.
- **How much do you test?** Test many things at both the acceptance and unit level, since acceptance drives you to unit level. But if one test at the unit level fixes multiple at the acceptance level, you're done. This can happen when you see patterns in acceptance tests failing, knowing what you know about the implementation. When you wrote the acceptance test, you didn't know the implementation.
- **What process do I go through to achieve this?** Two RGR circles, for application and for classes. But the outer circle writes a whole test case at once.
- How much code do you write at first? The code you wish you had.
- Dependencies? Mocks
- How do you verify? Check messages sent
- Balanced support for refactoring and code reuse
- Criticisms
  - Mocks not actually testing anything, but that's what acceptance tests are for
  - Too many levels of mocks, but this is exactly the design feedback you're supposed to respond to, and apply Demeter/Tell-Don't-Ask [Test Isolation is About Avoiding Mocks]
  - Mocks couple you to the implementation: no, if you plan to reuse the classes, outgoing messages are the interface. The implementation is what's in the method and any private methods, not what's an outgoing message.
 - Extracting a class then adding tests is test-after development; but you can TDD the abstraction as you pull it out one bit at a tim

# Discovery Testing

- Who? Test Double, Justin Searls
- Resources?
  - [The Failures of Intro to TDD][]
  - [Tests' Influence on Design][]
  - [The Testing Pyramid][]
- **What is good code?** A tree of classes that either coordinate collaboration or perform logic, not both. Disposable classes.
- **How can I get good code?** By driving it with unit tests only, with mocks helping you think through decomposing the problem.
- **What are the test types for?** Unit for driving your design, acceptance for regression. Integration bad because has downsides of both.
- **How much do you test?** Test everything at the unit level to design and the acceptance level to catch regressions?
- **What process do I go through to achieve this?** Red, green, then step down the tree to implement the collaborators you mocked.
- How much code do you write at first? The code you wish you had.
- Dependencies? mocks
- How do you verify? For collaboration nodes, check mocks. For leaf notes, check return values.
- Discourages refactoring or code reuse--rewrite instead. But how does this work for nontrivial needs?

[Avoiding Mocks]: https://www.destroyallsoftware.com/blog/2014/test-isolation-is-about-avoiding-mocks
[Classicist and Mockist TDD]: https://overcast.fm/+DtC47yT1g
[The Failures of Intro to TDD]: http://blog.testdouble.com/posts/2014-01-25-the-failures-of-intro-to-tdd.html
[GOOS]: http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626
[Introducing BDD]: http://dannorth.net/introducing-bdd/
[Is TDD Dead?]: http://martinfowler.com/articles/is-tdd-dead/
[Mocks Aren't Stubs]: http://www.martinfowler.com/articles/mocksArentStubs.html
[The RSpec Book]: https://pragprog.com/book/achbd/the-rspec-book
[TDD By Example]: http://www.amazon.com/Test-Driven-Development-By-Example/dp/0321146530
[TDD is Dead]: http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html
[TDD: Where Did It All Go Wrong]: https://vimeo.com/68375232
[The Testing Pyramid]: https://github.com/testdouble/contributing-tests/wiki/Testing-Pyramid
[Tests' Influence on Design]: https://github.com/testdouble/contributing-tests/wiki/Tests'-Influence-on-Design
[Writing Software]: https://youtu.be/9LfmrkyP81M?t=23m58s

***

More notes (may or may not be incorporated into discussion)

DHH: acceptance test

- The code should be the documentation, not the tests.
- Integration and acceptance tests are valuable to catch regressions; only cover what's valuable
- Unit tests are worthless
- TDD causes design damage: code is more complex, harder to understand
- "Is it easier to test?" isn't a good measure of success
- If you're mocking everything, you aren't testing anything. "You can make anything fast if it doesn't have to work."
- You can have unit tests that all work but the system doesn't work - https://twitter.com/SwiftOnSecurity/status/685898921063874560
- Cites James Coplien, ["Why Most Unit Testing is Waste"](http://www.rbcs-us.com/documents/Why-Most-Unit-Testing-is-Waste.pdf)
- Move up to integration tests: model tests
- Kent Beck: "Test as little as possible to reach a given level of confidence"

Middle-out, domain-model-out: integration test starting from the middle, then acceptance

- TDD book, Kent Beck
- Acceptance and unit tests are valuable to drive your design and catch regressions; cover every case
- Isolation with mocks isn't as valuable
- One option in classical TDD (Fowler)
- Build domain objects for a feature first, then layer UI on. (Fowler)
- Avoids having to fake anything (Fowler)
- Test the interfaces by using real collaborators or stubs
- Refactor step: emergent design (RSpec book)
- But design not determined: you choose designs to solve problems, how much complexity to trade for testability
- Traditional TDD has trade offs
- Example of ActiveRecord validations: assume the framework works
- Reorganizing the classes is changing the implementation; view of the domain as a whole
- Leads you to simple APIs with unguided design behind them

[BDD:](http://dannorth.net/introducing-bdd/) - RSpec Book, GOOS - acceptance test everything, then unit test beneath

- Good code is code that tells, not asks
- Acceptance tests and unit tests are both valuable to drive design and catch regressions
- Acceptance and unit tests both cover all cases
- Use mocks to test collaborations. Seem to have the goal that mocks minimize coupling
- Designed to answer "where to start, what to test and what not to test" (North)
- Focus on analysis/design, so outside in (Fowler)
- Emphasizing what an object does (interface), not what it is (implementation) (RSpec) -- different issue
- Focus on building good APIs
- Mocks couple you to implementation? Nope, they couple you to reusable components. Their outgoing messages aren't implementation, they're their interface.
- Leads you to reusable components.

Discovery Testing

- Acceptance tests for safety: "Test the system by using it just as a real user would so you can change implementation details with confidence you won't see false negatives." Minimal coupling so can replace *anything*, including the framework. Don't influence design. May not be able to refactor, but can at least rewrite safely.
- Don't write integration tests: "The'll be both coupled to your implementation and divorced from reality. Don't write or use testing tools that rely on your application's coupling to Rails, Ember, Angular, or any other application library or framework."
- TDD's main benefit is to improve design, regression is secondary to illusory
- Separation between classes that facilitate collaboration and tests that implement logic--gets appropriate separation on first shot
- Jordan: acceptance tests don't need to specify all edge/error cases, that can be at unit level
- This results in highly-coupled unit tests, which is fine because that's how you test them. Delete and recreate when requirements change, don't refactor.

To review:

- *TDD* book
- *GOOS* book
