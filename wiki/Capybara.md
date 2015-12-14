---
title: Capybara
---

[Docs](https://github.com/jnicklas/capybara)

* [Capybara-RSpec](Capybara-RSpec)

Capybara provides a common language to perform web application acceptance testing using both different [Ruby testing frameworks](Ruby-Testing-Frameworks) and different test drivers.

For example, whether you are using [Cucumber](Cucumber) or [RSpec](RSpec), in either case you would call:

- `visit` to load a page
- `fill_in` to fill in a form field
- `click_button` to click a button

(In [Cucumber](Cucumber), these functions would be called within step definitions, not within features.)

You would also use the same language regardless of what test driver you're using:

- **RackTest** to hit Rack interfaces directly without running a web server.
- **Selenium Webdriver** to run your tests in GUI Firefox.
- **Capybara-Webkit** to run your tests in a headless WebKit browser.
- **Poltergeist** to run your tests in the [PhantomJS](http://phantomjs.org/) headless WebKit browser.