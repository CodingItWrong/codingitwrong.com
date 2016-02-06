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
