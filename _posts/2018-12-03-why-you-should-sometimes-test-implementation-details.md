---
title: Why You Should Sometimes Test "Implementation Details"
---

*Update: removed a reference to Kent C. Dodds not addressing Redux connected components.*

"Don't test implementation details" is a phrase that seemed straightforward to me at first, but is seeming less and less clear to me over time.

It seems to me that some kinds of implementation details should not be tested. But other things that seem like implementation details, many of us agree you should test.

I want to add a little nuance to the phrase to help prevent misunderstanding of that phrase that could lead to testing frustration. When _should_ you test “implementation details”, and when shouldn’t you? Let’s look at five different situations.

## 1. Don’t test implementation details of individual components

The blog post that got me thinking about this topic is Kent C. Dodds’ [“Testing Implementation Details”][kent-implementation] from a few weeks ago. He shows how checking the state values within a component can lead to false negatives, and how testing component methods directly can lead to false positives. This is because neither of these are visible to the user of your component. Instead, Kent recommends testing only in terms of inputs like props and user interactions, and outputs like the render tree. I totally agree with this: you shouldn’t test implementation details of individual components. I can’t think of any significant upsides to doing so.

Edd Yerburgh provides a helpful term for this approach in his book [_Testing Vue.js Applications_][edd-book]—he refers to it as “testing the contract”. In this approach, when you look at any piece of code, you think about the expectations that the rest of the application has on that piece of code. What outputs does the application expect when you give it certain inputs? For a single component, these would be just what Kent refers to: props and user interactions for the input, the render tree as the output.

## 2. Maybe test implementation details of component trees

How does this principle apply when you have *a tree* of components to test, instead of just one? In other words, a component that renders other components instead of just markup. Earlier last year, Kent wrote a [blog post arguing for preferring integration tests][kent-integration] over unit tests because of the increased confidence you get from testing the pieces together. This involves testing against real dependencies instead of mocking, as well as avoiding shallow rendering.

This isn’t the only view in the React community, though. Jack Franklin tends to advocate shallow rendering, and he explains why in video 5 of his [Testing JavaScript with React and Enzyme course][jack-course]. As an example, he wrote a test that asserted how many of a certain child component class were rendered out:

```jsx
it('can add more counters when we click the button', () => {
  const wrapper = shallow(<CounterList />)

  const btn = wrapper.find('button')
  btn.simulate('click')
  const counters = wrapper.find('Counter')
  expect(counters.length).toEqual(2)
})
```

It’s true that this test is coupled to an implementation detail of `CounterList` — the fact that it delegates to `Counter` children. An alternative would be to use `mount` and check for the rendered content of the `Counter` component.

```jsx
it('can add more counters when we click the button', () => {
  const wrapper = mount(<CounterList />)

  const btn = wrapper.find('button')
  btn.simulate('click')
  const counters = wrapper.find('[data-testid=counter]')
  expect(counters.length).toEqual(2)
})
```

But such a test would still be coupled to an implementation detail—specifically, an implementation detail of *another* component! If you change the content of the `Counter` component, this could result in a number of different parent components’ tests breaking.

Recently I goaded Kent and Jack into having a discussion comparing and contrasting their views on testing, and their main point of difference was the above point about how to test component trees. You can [watch the video here](https://youtu.be/z4DNlVlOfjU?t=1533).

Which view should you take? I'd recommend watching the video of their discussion and decide which pros and cons apply most to your application. But in this case the phrase “don’t test implementation details” doesn’t even lead us to a clear answer—we would need to decide *which* implementation details we don’t want to test.

## 3. Don’t test only end-to-end

Next let’s consider whether “don’t test implementation details” applies to our end-to-end-tests. A few days ago, Dan Abramov tweeted this:

<div class="card mb-3">
  <div class="card-body">
    <blockquote class="blockquote mb-0">
      <p class="mb-0">
        How to know if a test sucks? You changed something that wouldn’t be observable to the user of your app/library, but the test broke and you had to fix the test.
      </p>
      <footer class="blockquote-footer">
        <a href="https://twitter.com/dan_abramov/status/1065663012541992963">@dan_abramov, Nov 30</a>
      </footer>
    </blockquote>
  </div>
</div>

Dan doesn't draw a distinction between “app” and “library” here, but the testing for these two would likely look significantly different. If you're building a frontend component library, the "outside edge" of your system is the exported components. In this case we would write tests for component trees, and so Dan’s advice aligns with Kent’s: render component trees fully to avoid breakage that wouldn’t be observable to the user of our app. But we’ve seen that there's also an argument to be made for shallow rendering, in which case an internal change *could* break the tests. So there isn’t universal agreement with the sentiment that such tests suck.

How about the case where we have an application rather than a component library? The outside edge of the application is its user interface, so it seems to me that following Dan’s advice and only testing externally-visible behavior would require only using end-to-end tests. Component integration tests wouldn't fit the spirit of Dan’s tweet, because the user of your app doesn’t care if there is a `TodoListContainer` that renders properly given certain props; they just care if they can see todos. If you’re writing component tests at all, you're already testing an implementation detail of your application.

I’m not sure if Dan would agree with this conclusion that you should test exclusively with end-to-end tests, but someone would: this is what DHH, the creator of Rails, argued for in a [blog post][dhh-post] and [conference talk][dhh-talk] in 2014. So it's worth considering: can we actually test exclusively with end-to-end tests, avoiding the implementation details of our app?

While it's important to have *some* end-to-end tests, there are a number of problems with relying exclusively on them:

* **End-to-end tests provide less precise test failure messages.** If you are testing whether an element is rendered by the application, and it *isn't* rendered, you don't get a lot of feedback as to what is going wrong. This may be fine if there isn't much logic involved. But if there are significant data transformations that are applied before data is shown, you could be better served by also testing some of those operations directly to pinpoint where the failure occurs.
* **End-to-end tests run slower.** Tests that need to run through your entire application stack, including HTTP requests and animations, are slower than tests that just execute a function. 10 seconds for an end-to-end test may not feel too bad, but slowness can have a number of different negative effects. Running the tests during development can be slow, which encourages you to wait longer before you run them, which means you will have more code changes, and so a harder time determining which change caused a failure. Also, if your application continues to grow in size over time, the full test suite will get longer and longer—I’ve heard of production applications whose full test suites take hours. The creators of [the Cypress end-to-end testing library](https://cypress.io) rightly point out that improvements in tooling can change this tradeoff over time. But since there are limits to the speed a real web application can run, there will also be limits to the speed that real end-to-end tests can run.
* **End-to-end tests can’t realistically cover all possible code paths.** J.B. Rainsberger has a classic blog post and talk [“Integrated Tests are a Scam”][jbrains-post] (where he uses the term "integrated test" to refer to any test that covers multiple pieces of code, including what we're referring to as integration and end-to-end tests). He points to the fact that, in complex systems, fully covering all possible code paths with integrated tests requires a combinatorial explosion of tests that would take an exorbitant amount of time to write. So although an integrated test gives you more confidence than the corresponding unit test, you can’t get that level of confidence for every code path in your app. You need a different approach to get that confidence.

These aren’t arguments to not end-to-end test *at all*: you still want to have some such tests to give you confidence that your entire application works together. But they *are* arguments for having fewer end-to-end tests and more integration or unit tests, in which case, as I argued above, you are in one sense testing implementation details of your application.

## 4. Do test implementation details of component/state-store integration

We’ve assessed whether or not to test implementation details at different levels of UI test. How does this question apply to testing the integration between components and state stores?

Jack Franklin and Edd Yerburgh have recommendations in the React and Vue ecosystems, respectively. Both recommend avoiding testing against your real state store, but they have two different recommendations for how to test, based on differences in how Redux and Vuex connect to components.

Redux connects to components by wrapping your component in a higher-order “connected component” that passes down props for state and actions. Because of this, Jack recommends avoiding testing your connected component (since all it does is wire things up to Redux) and just testing your non-connected component directly. In the test you can pass it the data it needs via React's usual props mechanism, and you can pass it mock functions in place of actions. Of course, you still need to ensure that the component is correctly connected to Redux; your end-to-end tests can cover this need.

In Vuex, Vue.js’s state store, state isn’t connected to components via props; instead, the Vuex store is injected into your component, and you access data and dispatch actions through it. So, unlike with Redux, there is no “plain” component that has no knowledge of the state store. Because of this, Edd generally recommends creating a real Vuex store instance in tests—but not your application’s fully-configured Vuex store. Instead, it’s a test-specific store with only the hard-codex data and mock actions your test case needs. The strategy is the same as with Redux; the only difference is that the mechanism here requires a real Vuex store instance.

Whether you’re working in Redux or Vuex, both of these these authors’ recommendations involve testing implementation details: we are testing which actions are dispatched, and the end user doesn’t care about that! The alternative, though, would be hooking up each of your component tests to the full global state store of your application, which might not be feasible or performant. At that point you’re edging toward doing a full end-to-end test of your application, and we’ve already seen the downsides of relying entirely on end-to-end tests.

Thinking in terms of Edd’s “contract testing” concept, data provided by a store is one kind of input, and a dispatched action is one kind of output from a component. Testing these shouldn’t be that foreign a concept, because if you were writing a test for a component library and a component could take a passed-in function prop, you would need to test that that function is called at the appropriate time. This approach simply applies the same thinking to dispatched actions.

## 5. Maybe test implementation details of the state store

If we aren’t testing our state store integrated with our components, how *should* we test it?

If you've used Redux you’ve probably tested individual reducer and action creator functions directly, ensuring each one produces the correct output based on the input passed to it. This is one way to test Vuex store functions as well. But this is the extreme of testing implementation details: it confirms the behavior of each function, but it doesn’t ensure that they work together correctly. The rest of your app doesn’t care about how these different functions within your state store interact; all it cares about is that when the right commands are executed, the right data comes out the other end. And tests of individual functions don't ensure this. For example, maybe an asynchronous action creator dispatches actions that don’t make the changes to the state that you expect. Or maybe a Vuex mutation function sets a value in one state field, but then a getter function expects to read it from a _different_ state field.

In this approach of testing individual state store functions, you need to rely on your end-to-end tests to ensure these functions all work together. And with Redux, this may be your only option—depending on the library you’re using for asynchrony, there may not be a good way to test your store all together.

With Vuex, on the other hand, there *is* an alternate way to test your stores that decouples you from its implementation details. As Edd Yerburgh writes in [_Testing Vue.js Applications_][edd-book] (emphasis mine):

<div class="card mb-3">
  <div class="card-body">
    <blockquote class="blockquote mb-0">
      <p class="mb-0">
        There are two approaches to unit testing a store. We can either test each of the parts separately, or we can <em>combine them together and run tests against a running store instance</em>. Both testing approaches have benefits and drawbacks…
      </p>
    </blockquote>
  </div>
</div>

If you want to test your Vuex store module as a whole, you can simply instantiate it in a test. You can dispatch an action which commits a mutation, then check that a getter function returns the value you expect. This approach allows the mutations and state storage to be an implementation detail. As a result, you get confidence that all your store's functions work together correctly, and you can change the state storage without refactoring any tests. If something goes wrong, you can get feedback that is more closely related to the store, which can make it easier to debug.

## So, Sometimes Test Implementation Details

This review of different parts of frontend apps should make it clear that “don’t test implementation details” isn’t as clear-cut as it seems at first. Here are the answers-plural that I would give to whether you should test implementation details:

* Don't test implementation details of individual components (methods and state)
* Maybe test the implementation details of a component tree, maybe not (shallow rendering vs. mounted)
* Do test the implementation details of your app as a whole (component tests instead of just end-to-end tests)
* Do test the implementation details of how components communicate with a state store (instead of connecting to the real store)
* Maybe test the implementation details of a Vuex store/module, maybe not (individual functions vs. the full store)

So it seems like if someone asks you "should you test the implementation?" it's difficult to answer until you ask "which implementation do you mean?"

A root cause of the complexity is that "the implementation" isn't just one thing. Any smaller or larger grouping of code has an interface and an implementation: the application does, components do, state stores do, and even individual functions do. It's a good idea to write different types of tests to gain different benefits, and each type of test will consider different parts of your app implementation details or not. The key is to evaluate each type of test separately. When you can avoid testing *nonessential* implementation detail from a test that's better, but be sure to consider the tradeoffs.

I'm always interested in learning from different approaches to testing, so I'd love to hear from you on Twitter or on the Reactiflux or Vue Land Discords! Reach out and let me know what implementation details you do or don't find it valuable to test.

[dhh-post]: http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html
[dhh-talk]: https://www.youtube.com/watch?v=9LfmrkyP81M&feature=share
[edd-book]: https://www.manning.com/books/testing-vue-js-applications
[jack-course]: https://javascriptplayground.com/testing-react-enzyme-jest/
[jbrains-post]: https://blog.thecodewhisperer.com/permalink/integrated-tests-are-a-scam
[kent-integration]: https://blog.kentcdodds.com/write-tests-not-too-many-mostly-integration-5e8c7fff591c
[kent-implementation]: https://blog.kentcdodds.com/testing-implementation-details-ccb8d269586
