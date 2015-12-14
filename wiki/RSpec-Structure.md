---
title: RSpec Structure
---

The following is a suggestion for how to structure RSpec tests:

* Have only one expectation per example. An example is intended to specify *one thing* about the code being tested.
* Use `describe` when everything within the block relates to something concrete, such as a method
* Use `context` when what's in the block doesn't relate to a single concrete thing. For example, one `context` might test multiple methods related to login, each method in a `describe` block within the `context`.