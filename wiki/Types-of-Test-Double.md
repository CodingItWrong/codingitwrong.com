---
title: Types of Test Double
---

Terminology differs, but in general "test double" is an overaching term for a non-production object that is used as part of a test in place of a production object. There are several different types:

- **Dummy**: No behavior and no verification. Often used just to fill in parameter lists.
- **Stub**: Hard-coded behavior. May record state for state verification, i.e. which emails were sent.
- **Mock**: Allows setting expectations on behavior before the test runs.
- **Spy**: Allows verifying behavior after the fact. Martin Fowler associates spies with stubs, but to me they seem more closely associated with mocks because they allow behavior verification.
- **Fake**: A functioning version of the object that has some limitation that prevents it from being used in production. For example, it might use an in-memory database. Fakes allow productive manual testing.

The same test double framework may be able to be used to create different types of test double. For example, in `rspec-mocks`:

- A double with `as_null_object` called on it functions as a dummy
- A double with `allow().to receive()` called on it functions as a stub
- A double with `expect().to receive()` called on it functions as a mock
- An object created with `spy()` is a spy

[More Info](http://martinfowler.com/articles/mocksArentStubs.html)
