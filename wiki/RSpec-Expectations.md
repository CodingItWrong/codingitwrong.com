---
title: RSpec Expectations
---

[RSpec](RSpec) has at least four ways to define expectations. As of 2012, the recommended syntax is `expect()` and `it { is_expected.to }`. They are more reliable than the older `should` and `it { should }` syntaxes. For more details, see [this post](http://rspec.info/blog/2012/06/rspecs-new-expectation-syntax/).

# Expect

`expect()` syntax lets you define an expectation on an explicit object.

```ruby
expect(actual).to eq(expected)
expect(actual).to be > expected
expect([1, 2, 3]).to_not include(4)
```

[More Info](https://github.com/rspec/rspec-expectations)

# It Is Expected To

`it { is_expected.to }` syntax lets you define an expectation on a previously-defined subject.

```ruby
subject { Person.new(:birthdate => 19.years.ago) }
it { is_expected.to be_eligible_to_vote }
```

[More Info](http://rspec.info/documentation/3.3/rspec-core/RSpec/Core/MemoizedHelpers.html)

# Should (Deprecated)

`should` syntax lets you define an expectation on an explicit object.

```ruby
actual.should eq expected
actual.should be > 3
[1, 2, 3].should_not include 4
```

[More Info](https://github.com/rspec/rspec-expectations/blob/master/Should.md)

# It Should (Deprecated)

`it { should }` syntax lets you define an expectation on a previously-defined subject.

```ruby
subject { Person.new(:birthdate => 19.years.ago) }
it { should be_eligible_to_vote }
```

[More Info](http://rspec.info/documentation/3.3/rspec-core/RSpec/Core/MemoizedHelpers.html)

