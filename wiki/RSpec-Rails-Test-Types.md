---
title: RSpec-Rails Test Types
---

The types of test that [RSpec-Rails](RSpec-Rails) makes available.

* *Feature*: An acceptance test, simulating a browser session. Uses [Capybara](https://github.com/jnicklas/capybara). Generated with `rails generate integration_test`. In Rails, integration tests can test for page elements, but can't interact with links or forms.
* *Request*: Simulates one or more HTTP requests. Equivalent to a Rails integration test. Andy Lindeman [recommends](http://www.andylindeman.com/rspec-rails-and-capybara-2.0-what-you-need-to-know/) this for web service testing.
* *Controller*: Calls a controller method directly. Doesn't render a view, but allows you to test what view would be rendered and what data is passed to it. Equivalent to a Rails functional/controller test. In Rails, controller tests do render views.
* *Routing*: Tests mapping of requested paths to controllers.
* *Model*: Tests a model, including hitting the database.
* *View*: Tests a view template in isolation.
* *Helper*: Tests helper methods.

[Docs](https://relishapp.com/rspec/rspec-rails/docs)
