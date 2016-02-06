*These are my presentation notes from a Web Jibber Jabber presentation. This will probably end up on another blog soon--my own, if nothing else. But for now, here it is!*

As I've been getting into testing, I've been asking myself how much testing to do--specifically, how much unit, integration, and acceptance testing. Turns out Folks Got Opinions™ on this! As I looked, I found at least four different approaches to testing, that answered a number of questions I was asking:

1. What are the different test types for?
2. How much do I test?
3. What process do I go through to write tests?
4. How do I use test doubles?
5. What do I check for in my tests?

But it turns out these different approaches to testing *also* answered some bigger-picture questions I wasn't expecting:

6. How do I deal with change in my application?
7. What even **is** good code?

I wanted to share what I found. And as a bonus, I have a **crazy theory** to share: each approach to testing is tailor-made to guide you towards a certain kind of architecture. So, if you try to use a testing approach for an architecture it's not a good fit for, you're gonna have a bad time.

One quick note: there aren't hard boundaries between these types of testing. Don't think of them as things you have to pick exactly one of. Think of them as tools you can apply in different situations as they fit best.

First, a clarification of how I use testing terms:

- **Acceptance test**: tests the whole system from the outside (i.e. web user interface).
- **Integration test**: tests an object connected to other objects and external systems (i.e. the database).
- **Unit test**: tests an object in isolation from any other objects.
- **“Unit” test**: when I am being sarcastic about Kent Beck saying “unit test” to refer to what the industry considers an integration test. More on this in a minute!
- **Stub**: a test double that provides hard-coded responses, and may record state changes.
- **Mock**: a test double that allows you to verify that certain methods were called

- - -

# Test Approach #1: Whatever it is DHH does.
(Maybe "Regression Testing"?)

![Regression Testing](assets/images/2016-02-04-regression-testing.png)

- Related terms?
    - Test-After Development
    - ⌘-R-Driven Development
- Major representatives? DHH
- Resources?
    - "[Writing Software][]" RailsConf 2014 keynote
    - "[TDD is Dead][]" blog post
    - "[Is TDD Dead?][]" panels
- **1. What are the test types for?** Acceptance and integration tests are for regression. Unit tests are **useless**.
- **2. How much do I test?** Test only what's valuable. Maybe the most common use cases of your app, or what makes you money.
- **3. What process do I go through to achieve this?** Write the code, ⌘-R, then write the regression test. Test in the browser.
- **4. How do I use test doubles?** Don't. Always use your real, connected production code.
- **5. What do I check for in my tests?** Check page content or database entries.
- **6. How do I deal with change?** Just do it! Refactoring is de-emphasized; code reuse is discouraged. [[Is TDD Dead?][]]
- **7. What even is good code?** Simple, readable code.
- Criticisms
    - There are a lot of dependencies between classes.
    - Difficult to follow this approach as apps grow in size.
    - It's concerning that there's no mention of refactoring.

# Test Approach #2: Classical TDD

- Related terms?
    - Detroit School of TDD
    - Middle-Out Testing
- Major representatives? Kent Beck, Martin Fowler; there is a lot of leeway here so specifically representing Ian Cooper's view
- Resources
    - "[Classicist and Mockist TDD][CAM]" podcast episode
    - "[Mocks Aren't Stubs][MAS]" blog post
    - "[Is TDD Dead?][]" panels
    - "[TDD: Where Did It All Go Wrong][AGR]" conference talk
    - *[Test-Driven Development: By Example][TDD By Example]* book
- **1. What are the different test types for?** "Unit" tests are for driving design...for sufficiently large definitions of "unit." It's more similar to what others mean when they say "integration testing." [[AGR][]]
- **2. How much do I test?** Test everything once and only once. [[AGR][]]
- **3. What process do I go through to write tests?** Red, green, refactor. [[MAS][]]
- **4. How do I use test doubles?** Use stubs *only* when the real collaborators are difficult to work with.
- **5. What do I check for in my tests?** Check state, internal and external. For example, you might send a message to one object then check a value three objects away. You might also have to expose some internal properties of an object just for testing.
- **6. How do I deal with change??** Refactor the class's implementation, which includes its dependencies that aren't directly accessed from the outside.
- **7. What even is good code?** A black box with well-defined external functionality.
- Criticisms
  - Exposing state for the sake of testing--but practicioners say this doesn't cause a big problem in practice. [[CAM][]]
  - Doesn't exert pressure on you to refactor--but practicioners say as your skills grow you do this more. [[FITDD][]]
  - Extracting objects is painful. [[FITDD][]]

# Test Approahc #3: Isolation Testing

- Related terms?
    - Mockist TDD--this is the most common term for it, but you'll see below why I think the term "Isolation Testing" better captures the essence for the sake of this comparison
    - London School of TDD
    - Outside-In Testing
    - Need-Driven Development
    - Behavior-Driven Development (BDD; but that has a larger scope)
- Major representatives? Dan North, David Chelimsky, Dave Astels, Steve Freeman, Nat Pryce
- Resources?
  - "[Classicist and Mockist TDD][CAM]" podcast episode
  - *[The RSpec Book][RB]*
  - "[Test Isolation is About Avoiding Mocks][AM]" blog post
  - *[Growing Object-Oriented Software, Guided by Tests][GOOS]* book
  - "[Outside-in TDD and Dependency Injection in Rails][OITDD]" podcast episode
- **1. What are the different test types for?** Acceptance and unit tests are for driving your design. Acceptance tests are also for catching regressions. [[GOOS][]]
- **2. How much do you test?** Test everything at both the acceptance and unit level, since acceptance tests drive you to unit level. [[RB][]]
- **3. What process do I go through to write tests?** Do two red-green-refactor circles, acceptance-level and unit-level. [[RB][]]
- **4. How do I use test doubles?** Stub or mock all collaborators.
- **5. What do I check for in my tests?** Check behavior, not state--i.e. check messages sent. (But when there are return values you also check those.)
- **6. How do I deal with change?** Reconfigure your component, by which I mean...
- **7. What even is good code?** Small, reusable components. [[GOOS][]]
- Criticisms
  - Mocks not actually testing that the system works--but that's what acceptance tests are for.
  - Too many levels of mocks--but this is exactly the design feedback you're supposed to respond to, simplifying your code by applying the Law of Demeter and the Tell-Don't-Ask principle. [[AM][]]
  - Mocks couple you to the implementation--but if you want reusable components, outgoing messages *are* the interface. The implementation is what's in the method and any private methods, not what's an outgoing message.
  - Extracting a class then adding tests is test-after development--but you can TDD the abstraction as you pull it out one bit at a time [[OITDD][]]

# Test Approach #4: Discovery Testing

- Major Representatives? Justin Searls, [Test Double](http://testdouble.com/)
- Resources?
    - "[The Failures of Intro to TDD][FITDD]" blog post
    - "[Tests' Influence on Design][]" page
    - "[The Testing Pyramid][TP]" diagram
- **1. What are the different test types for?** Unit tests are for driving your design, acceptance tests are for regression. [[TP][]]
- **2. How much do I test?** Test-drive everything at the unit level, then test at the acceptance level to catch regressions.
- **3. What process do I go through to write tests?** Red, green, then implement the collaborators you mocked...not quite as catchy! Decompose the problem into smaller parts from the start, instead of refactoring after the fact. [[FITDD][]]
- **4. How do I use test doubles?** Stub or mock all collaborators.
- **5. What do I check for in my tests?** Check behavior, just like in Isolation Testing.
- **6. How do I deal with change?** Throw out and rewrite a subtree of your classes.
- **7. What even is good code?** A tree of tiny, disposable classes. [[FITDD][]]
- I haven't found any responses to Discovery Testing online, but here are the questions I have:
    - Does disposable code work for complex features? Even if each class is tiny, one feature might be implemented with dozens of classes that you'd have to throw out if the feature changed at a high level.
    - Is code reuse *never* worth the cost?
    - Do you always know the right way to decompose your problem at first?

- - -

So how do you decide what testing approach to take? I would suggest that the better question is, what kind of system do you want to create?

- Refactorable black boxes?
- Pluggable components?
- Disposable subtrees?
- ...Literature? I guess??

The answer is It Depends™. On a number of things:

- On your personal wiring.
- On your experience level with each testing style.
- On your experience level in software design.
- On your experience level in the problem domain.
- On team and organization structure.
- On the needs and constraints of the system.
- On your project working style: waterfall vs. iterative.

But you want to make sure you choose the right testing tool for the kind of system you want to create; otherwise you'll be frustrated at the direction the tests are steering you. I think this is the cause of a lot of people's frustration with certain testing styles.

- If you want to do a lot of black-box refactoring but you use Isolation Testing, you'll get frustrated that you have to update mocks every time you refactor.
- If you want "simple, readable code" the way DHH defines it but you use *any* TDD approach, you'll resent the small objects the tests lead you to.
- If you want to do a lot of component reuse but you use Classical TDD, it won't give you the design pressure to help you with that.

What's your personal testing apporach? Does it match up with the kind of system you're trying to create? Did you get any ideas for new techniques you might want to try?

[AGR]: https://vimeo.com/68375232
[AM]: https://www.destroyallsoftware.com/blog/2014/test-isolation-is-about-avoiding-mocks
[CAM]: https://overcast.fm/+DtC47yT1g
[FITDD]: http://blog.testdouble.com/posts/2014-01-25-the-failures-of-intro-to-tdd.html
[GOOS]: http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626
[Is TDD Dead?]: http://martinfowler.com/articles/is-tdd-dead/
[MAS]: http://www.martinfowler.com/articles/mocksArentStubs.html
[RB]: https://pragprog.com/book/achbd/the-rspec-book
[OITDD]: https://overcast.fm/+DtC4BiOdM/7:54
[TDD By Example]: http://www.amazon.com/Test-Driven-Development-By-Example/dp/0321146530
[TDD is Dead]: http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html
[TDD: Where Did It All Go Wrong]: https://vimeo.com/68375232
[The Testing Pyramid]: https://github.com/testdouble/contributing-tests/wiki/Testing-Pyramid
[Tests' Influence on Design]: https://github.com/testdouble/contributing-tests/wiki/Tests'-Influence-on-Design
[TP]: https://github.com/testdouble/contributing-tests/wiki/Testing-Pyramid
[Writing Software]: https://youtu.be/9LfmrkyP81M?t=23m58s
