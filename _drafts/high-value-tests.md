—-
title: High-value tests
—-

Recently on the Ember #-testing Slack channel a question was asked about best practices for testing. I’d like to share what I’ve learned, with principles that apply to any frontend or backend framework.

- Acceptance test main user flows for safety
- Use test doubles as little as possible for parts of your own system. But fake out interfaces to external systems.
- Don’t test all edge cases at the acceptance level
- For acceptance tests, make many assertions in one test. For lower level tests, assert one thing per test. Use before blocks to share setup (but not too shared)
- Use factories instead of fixtures. Include only relevant data (but all relevant data)
- Test the contract, the public interface
- Test in isolation. Use test doubles for nontrivial collaborators.
- Instead of testing state changes elsewhere in the app, test outgoing commands with mocks
- Listen to the tests. If the test is complex, adjust the design of your code. This can include wrapping third-party dependencies to simplify the interface
