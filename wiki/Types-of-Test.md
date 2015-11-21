---
title: Types of Test
---

* **Unit test**: tests a single class. Does not instantiate any other classes except for test doubles, and does not require a framework or any external services, including a database.
* **Integration test**: any test that operates on multiple parts of your application. Includes tests of more than one class at once, tests that require a framework, and tests that hit external services including a database.
* **Functional test**: tests a controller in an MVC application.
* **Acceptance test**: tests an entire application from a user's perspective. For web applications, this includes interacting with links and forms on pages.

Most of the above comes from [*Laravel Testing Decoded*](https://leanpub.com/laravel-testing-decoded).