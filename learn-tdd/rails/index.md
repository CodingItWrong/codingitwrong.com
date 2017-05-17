---
title: Learn TDD in Rails
---

{% include tutorial-intro.md %}

To see how TDD works in Rails, let's walk through a simple real-world example of building a feature. We'll be using Rails 5.1 and RSpec, one of the most popular test frameworks for Ruby. Each section of the article is linked to a corresponding commit in the [Git repo](https://github.com/learn-tdd-in/rails) that shows the process step-by-step. This tutorial assumes you have some [familiarity with Rails](http://guides.rubyonrails.org/) and with [automated testing concepts](/learn-tdd/concepts).

You can also watch a [meetup presentation video](https://youtu.be/fXlLbhuIc34) of an earlier version of this tutorial.

The feature we'll build is the age-old tutorial feature: creating a blog post.

{% include_relative _commits.md %}

## More Resources

To learn more about TDD, I recommend:

* [_The RSpec Book_](https://www.amazon.com/RSpec-Book-Behaviour-Development-Cucumber/dp/1934356379) - uses older versions of Rails and RSpec, but it's a great simple walkthrough of outside-in testing.
{% include tutorial-goos.md %}
* [_Rails 4 Test Prescriptions_](https://pragprog.com/book/nrtest2/rails-4-test-prescriptions) by Noel Rappin

{% include tutorial-contact.md %}
