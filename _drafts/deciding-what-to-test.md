Types of test pros and cons:
- Unit: fast and low repetition, test isolation; mocks can couple you to the implementation, and can end up only testing spelling
- Integration: closer to actual usage, but slower and can break
- Acceptance: testing the whole system, what user sees; lot of repetition and slow, can be more brittle b/c UI dependent, not test isolation

Question: how much do you write acceptance vs integration vs unit tests, and in what order?

[BDD:](http://dannorth.net/introducing-bdd/) acceptance test everything, then unit test beneath
- Designed to answer "where to start, what to test and what not to test" (North)
- Focus on analysis/design, so outside in (Fowler)
- Need-driven development, outside-in testing (Fowler)
- System design driven by tests (Fowler)
- Write outside tests, mock first level of collaborators; then build those (Fowler)
- Emphasizing what an object does (interface), not what it is (implementation) (RSpec)
Middle-out, domain-model-out: integration test starting from the middle, then acceptance
- One option in classical TDD (Fowler)
- Build domain objects for a feature first, then layer UI on. (Fowler)
- Avoids having to fake anything (Fowler)
DHH: acceptance test
- TDD causes design damage: code is more complex, harder to understand
- "Is it easier to test?" isn't a good measure of success
- If you're mocking everything, you aren't testing anything. "You can make anything fast if it doesn't have to work."
- You can have unit tests that all work but the system doesn't work - https://twitter.com/SwiftOnSecurity/status/685898921063874560
- Cites James Coplien, ["Why Most Unit Testing is Waste"](http://www.rbcs-us.com/documents/Why-Most-Unit-Testing-is-Waste.pdf)
- Move up to integration tests: model tests
- Regression tests, don't drive your design through test-first
- Kent Beck: "Test as little as possible to reach a given level of confidence"
Ward Bell: unit test rigorously, acceptance test only the key moneymaking features (due to JS difficulties?)
Testing Pyramid:
- Unit tests for design,
- Acceptance tests for safety: "Test the system by using it just as a real user would so you can change implementation details with confidence you won't see false negatives." Minimal coupling so can replace *anything*, including the framework. Don't influence design. May not be able to refactor, but can at least rewrite safely.
- Don't write integration tests: "The'll be both coupled to your implementation and divorced from reality. Don't write or use testing tools that rely on your application's coupling to Rails, Ember, Angular, or any other application library or framework."
- TDD's main benefit is to improve design, regression is secondary to illusory
- Traditional TDD leads you into big objects you leave, or later have to extract, which is painful
- Discourage characterization tests (test-after)
- Separation between classes that facilitate collaboration and tests that implement logic--gets appropriate separation on first shot
- Jordan: acceptance tests don't need to specify all edge/error cases, that can be at unit level
- This results in highly-coupled unit tests, which is fine because that's how you test them. Delete and recreate when requirements change, don't refactor.

Bonus: bug reported, replicate with a test first (Bell)

[Adventures in Angular, Ward Bell](Adventures in Angular: 026 AiA Testing Tools
https://overcast.fm/+DcFRMRvTE) - 37ish, core moneymaking functions, Lucas, acceptance test critical path, rigorous unit tests; Ward Bell regression main paths

To review:
- [Is TDD Dead?](http://martinfowler.com/articles/is-tdd-dead/)
- [Everzet: Classicist vs. Mockist](https://overcast.fm/+DtC47yT1g)
- Other Full Stack testing episodes
- *The RSpec Book*

Done:
- [BDD](http://dannorth.net/introducing-bdd/)
- [Mocks Aren't Stubs](http://www.martinfowler.com/articles/mocksArentStubs.html)
- [Adventures in Angular, Ward Bell](Adventures in Angular: 026 AiA Testing Tools
https://overcast.fm/+DcFRMRvTE)
- [DHH: Writing Software](https://youtu.be/9LfmrkyP81M?t=23m58s) - to 45:00
- [Testing Pyramid](https://github.com/testdouble/contributing-tests/wiki/Testing-Pyramid) - just shows different kinds of tests, but also discourages the middle
- [The Failures of Intro to TDD](http://blog.testdouble.com/posts/2014-01-25-the-failures-of-intro-to-tdd.html) - turned Jordan on to the view of low-level unit for design
- [Tests' Influence on Design](https://github.com/testdouble/contributing-tests/wiki/Tests'-Influence-on-Design) - more info on the approach
