Types of test pros and cons:

- Unit: fast and low repetition, test isolation; mocks can couple you to the implementation, and can end up only testing spelling
- Integration: closer to actual usage, but slower and can break
- Acceptance: testing the whole system, what user sees; lot of repetition and slow, can be more brittle b/c UI dependent, not test isolation

You need a strategy to decide how much of each type of test to do, when.
Your strategy depends on bigger questions: what do you think good code is, and therefore what is the purpose of each type of test to that end?

You can't tell someone's strategy by looking at the test.

Why this discussion

- I want correction!
- I want to hear your view, even nuances of it--maybe we all agree, maybe not
- I want resources!

People use different terms in different ways. Ask questions to understand their view, not just the term.
I apologize if categories aren't helpful to you; they are to me

Antipattern: mock purgatory

- Isolation tests with 3+ levels of embedded mocks
- Entirely coupled to implementation
- Very hard to read to understand intent or get design feedback
- Nobody thinks this is good; what do you do?

DHH: acceptance test

- Writing Software talk, TDD is Dead post
- Good code is simple and easy to read
- I can get good code myself, no tests drive me there
- Integration and acceptance tests are valuable to catch regressions; cover what's valuable
- Unit tests are worthless
- TDD causes design damage: code is more complex, harder to understand
- "Is it easier to test?" isn't a good measure of success
- If you're mocking everything, you aren't testing anything. "You can make anything fast if it doesn't have to work."
- You can have unit tests that all work but the system doesn't work - https://twitter.com/SwiftOnSecurity/status/685898921063874560
- Cites James Coplien, ["Why Most Unit Testing is Waste"](http://www.rbcs-us.com/documents/Why-Most-Unit-Testing-is-Waste.pdf)
- Move up to integration tests: model tests
- Regression tests, don't drive your design through test-first
- Kent Beck: "Test as little as possible to reach a given level of confidence"
- DHH: the code should be the documentation, not the tests
- Refactoring can get skipped
- Is there room for safe refactoring in this view?
- TDD gave benefit for regression tests, not design pushback
- Major warning flag: I hardly ever hear DHH use the word "refactoring". And he explicitly argues against code reuse, I.e. For APIs.

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
- Coverage: avoid double-testing—BDD would disagree
- Validations: assume the library works
- TDD discipline: decide when to be in it and when not to
- People do more refactoring as they grow in skill

[BDD:](http://dannorth.net/introducing-bdd/) - RSpec Book - acceptance test everything, then unit test beneath

- Good code is code that tells, not asks
- Acceptance tests and unit tests are both valuable to drive design and catch regressions
- Acceptance and unit tests both cover all cases
- Use mocks to test collaborations
- Designed to answer "where to start, what to test and what not to test" (North)
- Focus on analysis/design, so outside in (Fowler)
- Two concentric red/green/refactor cycles
- Need-driven development, outside-in testing (Fowler)
- System design driven by tests (Fowler)
- Write outside tests, mock first level of collaborators; then build those (Fowler)
- Emphasizing what an object does (interface), not what it is (implementation) (RSpec)
- Focus on building good APIs

Discovery Testing

- GOOS, Jordan and Jeremy posts
- Good code is code that either coordinates collaboration or performs logic, not both
- Unit tests are to drive your design, exposing coupling
- Use mocks to show coupling
- Acceptance tests for safety: "Test the system by using it just as a real user would so you can change implementation details with confidence you won't see false negatives." Minimal coupling so can replace *anything*, including the framework. Don't influence design. May not be able to refactor, but can at least rewrite safely.
- Don't write integration tests: "The'll be both coupled to your implementation and divorced from reality. Don't write or use testing tools that rely on your application's coupling to Rails, Ember, Angular, or any other application library or framework."
- TDD's main benefit is to improve design, regression is secondary to illusory
- Traditional TDD leads you into big objects you leave, or later have to extract, which is painful
- Discourage characterization tests (test-after)
- Separation between classes that facilitate collaboration and tests that implement logic--gets appropriate separation on first shot
- Jordan: acceptance tests don't need to specify all edge/error cases, that can be at unit level
- This results in highly-coupled unit tests, which is fine because that's how you test them. Delete and recreate when requirements change, don't refactor.
- Test the implementation by mocks giving you feedback
- Generally no need to refactor

Bonus: bug reported, replicate with a test first (Bell)

[Adventures in Angular, Ward Bell](https://overcast.fm/+DcFRMRvTE) - 37ish, core moneymaking functions, Lucas, acceptance test critical path, rigorous unit tests; Ward Bell regression main paths

To review:

- [Is TDD Dead?](http://martinfowler.com/articles/is-tdd-dead/)
- [Everzet: Classicist vs. Mockist](https://overcast.fm/+DtC47yT1g)
- Other Full Stack testing episodes
- *TDD* book
- *GOOS* book

Done:

- [BDD](http://dannorth.net/introducing-bdd/)
- [Mocks Aren't Stubs](http://www.martinfowler.com/articles/mocksArentStubs.html)
- [Adventures in Angular, Ward Bell](https://overcast.fm/+DcFRMRvTE)
- [DHH: Writing Software](https://youtu.be/9LfmrkyP81M?t=23m58s) - to 45:00
- [Testing Pyramid](https://github.com/testdouble/contributing-tests/wiki/Testing-Pyramid) - just shows different kinds of tests, but also discourages the middle
- [The Failures of Intro to TDD](http://blog.testdouble.com/posts/2014-01-25-the-failures-of-intro-to-tdd.html) - turned Jordan on to the view of low-level unit for design
- [Tests' Influence on Design](https://github.com/testdouble/contributing-tests/wiki/Tests'-Influence-on-Design) - more info on the approach
- [Test Isolation is About Avoiding Mocks](https://www.destroyallsoftware.com/blog/2014/test-isolation-is-about-avoiding-mocks)
- [A New Look at Test-Driven Development](http://blog.daveastels.com.s3-website-us-west-2.amazonaws.com/2014/09/29/a-new-look-at-test-driven-development.html)
- *The RSpec Book*
