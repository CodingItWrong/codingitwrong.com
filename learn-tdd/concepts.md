---
title: Testing Concepts
---

* [Object-Oriented Concepts](#oo-concepts)
* [Test Terminology](#test-terminology)
* [Testing Approaches](#approaches)
* [Types of Test](#types-of-test)
* [Types of Test Double](#types-of-test-double)

## <a name="oo-concepts"></a>Object-Oriented Concepts

* **<a name="behavior"></a>Behavior**: the messages sent by an object-oriented program. Behavior verification involves checking what messages were sent by a program, not the results of those messages in terms of data stored.
* **<a name="collaborator"></a>Collaborator**: an object that works with another object to accomplish its goals.
* **<a name="dependency"></a>Dependency**: knowledge that an object has about another object. An object must have some dependencies on its [collaborators](#collaborator), but it's possible for an object to have more dependencies on other objects that necessary.
* **<a name="refactor"></a>Refactor**: improving the design of a piece of code through small transformations without changing its behavior.
* **<a name="state"></a>State**: the data constructs in a program. State verification involves checking the results of operations in terms of data, not checking that the operations themselves happened.

## <a name="test-terminology"></a>Test Terminology

* **<a name="assertion"></a>Assertion**: a check within a test that ensures a given condition is true, and report an error message if not.
* **<a name="example"></a>Example**: the "spec"-style equivalent of a [test case](#test-case). An example writes out something a piece of code should do, and can be run to determine whether the system actually does it.
* **<a name="expectation"></a>Expectation**: the "spec"-style testing equivalent of an [assertion](#assertion). Indicates that a certain condition is expected to be true. When the spec is run, if the condition is not true, an error will be shown.
* **<a name="red-green-refactor"></a>Red-Green-Refactor**: a cycle of steps followed in TDD. First, a failing test is written and run to confirm that it fails ("red"). Then, production code is written to make the test pass ("green"). Finally, as necessary, the test and production code are [refactored](#refactor) to improve their design, while the test is repeatedly run to make sure it is still green.
* **<a name="regression"></a>Regression**: the reintroduction of an error in code that was previously working correctly. One of the main goals of testing is to catch regressions.
* **<a name="spec"></a>Specification (Spec)**: a test, considered primarily as a way to indicate the desired [behavior](#behavior) of a system, rather than to confirm the behavior of an existing system. Used in [Mockist TDD](#mockist-tdd).
* **<a name=""></a>Subject**: the object being tested in a given context.
* **<a name="test-case"></a>Test Case**: the smallest unit of a test suite that can be run on its own.
* **<a name="test-double"></a>Test Double**: an object that stands in for a production object during testing. Includes [dummies](#dummy), [fakes](#fake), [mocks](#mock), [spies](#spy), and [stubs](#stub).
* **<a name="test-suite"></a>Test Suite**: a collection of test cases.

## <a name="approaches"></a>Testing Approaches

* **<a name="bdd"></a>Behavior-Driven Development (BDD)**: an approach to software development that involves working with the user to specify the behavior of a system and build it in terms of those specifications. BDD is closely related to [Mockist TDD](#mockist-tdd), but whereas Mockist TDD usually begins with features already defined, BDD includes the process of coming up with the features list.
* **<a name="classical-tdd"></a>Classical TDD**: the original form of TDD proposed by Kent Beck. It focused on [middle-out testing](#middle-out-testing), [unit tests](#unit-test) against real [collaborators](#collaborator), [stubbing](#stub), verifying [state](#state), and "[test case](#test-case)" and "[assertion](#assertion)" terminology.
* **<a name="middle-out-testing"></a>Middle-Out Testing**: an approach to testing that begins with domain objects and works from there out to user-facing code.
* **<a name="mockist-tdd"></a>Mockist TDD**: a refinement of [classical TDD](#classical-tdd) proposed by the London TDD community. Mockist TDD focuses on [mock objects](#mock), [behavior](#behavior) verification, [outside-in testing](#outside-in-testing), and [isolation testing](#isolation-testing).
* **<a name="outside-in-testing"></a>Outside-in Testing**: an approach to testing that begins with the outside of the system, i.e. with [acceptance tests](#acceptance-test), and then writes [isolation tests](#isolation-test) as needed to specify classes needed to satisfy the acceptance test.

## <a name="types-of-test"></a>Types of Test

* **<a name="acceptance-test"></a>Acceptance Test**: a test that confirms that the system correctly implements a user-visible feature. Often accomplished with [end-to-end testing](#end-to-end-test).
* **<a name="characterization-test"></a>Characterization Test**: tests written against a pre-existing system to document its current behavior, bugs and all. Includes both first-party and third-party code. Similar to [exploratory tests](#exploratory-test), but tends to be more comprehensive and permanent.
* **<a name="end-to-end-test"></a>End-to-End Test**: a test that accesses the entire system from the outside, e.g. through the user interface or HTTP requests. Often used to accomplish [acceptance testing](#acceptance-test).
* **<a name="exploratory-test"></a>Exploratory Test**: tests written against a pre-existing system to understand its current behavior. Includes both first-party and third-party code. Similar to [characterization tests](#characterization-test), but tends to be less comprehensive and more disposable.
* **<a name="functional-test"></a>Functional Test**: a test of a controller in an MVC application.
* **<a name="integration-test"></a>Integration Test**: multiple usages:
	* Aby test that exercises more than one production class; the opposite of an [isolation test](#isolation-test).
	* A test that checks that the application's code works correctly with third-party code.
* **<a name="isolation-test"></a>Isolation Test**: a test that exercises only one production class. Any [dependencies](#dependency) of that class are replaced by [test doubles](#test-double).
* **<a name="request-test"></a>Request Test**: a test of a request sent into a system, such as an HTTP request to a server-rendered web application or a web service.
* **<a name="unit-test"></a>Unit Test**: multiple usages:
	* A test that exercises only one production class; equivalent to "[isolation test](#isolation-test)." This is the definition used in [Mockist TDD](#mockist-tdd).
	* A test of a class and all its real [collaborators](#collaborator). Called a "unit" test because it can be run in isolation without affecting or being affected by any other tests. This is the definition used in [Classical TDD](#classical-tdd).

## <a name="types-of-test-double"></a>Types of Test Double

* **<a name="dummy"></a>Dummy**: a [test double](#test-double) that is not actually used for anything other than to fill in an argument list.
* **<a name="fake"></a>Fake**: a [test double](#test-double) that has real behavior implemented in a simple way. For example, a fake equivalent of a database connection might store data in-memory.
* **<a name="mock"></a>Mock**: a [test double](#test-double) that allows specifying in advance the messages it must receive, which are then verified at the end of a test case. Mocks and [spies](#spy) are the primary methods of [behavior](#behavior) verification.
* **<a name="spy"></a>Spy**: a [test double](#test-double) that records messages it receives, which can then be tested against at the end of a test case. Spies and [mocks](#mock) are the primary methods of [behavior verification](#behavior).
* **<a name="stub"></a>Stub**: a [test double](#test-double) with hard-coded method responses.

---

##  References

* [*Growing Object-Oriented Software, Guided by Tests*](http://www.informit.com/store/growing-object-oriented-software-guided-by-tests-9780321503626)
* ["Introducing BDD"](https://dannorth.net/introducing-bdd/), DanNorth.net
* ["Mocks Aren't Stubs"](http://martinfowler.com/articles/mocksArentStubs.html), MartinFowler.com
* [*Practical Object-Oriented Design in Ruby*](http://www.poodr.com/)
* ["Test Doubles"](http://www.martinfowler.com/bliki/TestDouble.html), MartinFowler.com
* [*Test-Driven Development by Example*](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)
