---
title: Cypress Code Smells
---

User automation tests are intended to closely replicate a real user interacting with your app, and Cypress purports to be even more realistic than past testing tools. But when using Cypress with a modern frontend framework like React or Vue, you will probably run into cases where the app works fine in manual testing but fails in a Cypress test. The test may fail consistently, or intermittently, or may succeed locally but fail on CI (sometimes or always). But regardless, the app works reliably in manual testing but sometimes fails in Cypress tests.

In these cases, the test failure is often due to timing and asynchrony: Cypress moves faster than a real user, so it can get the app into a state that a human tester usually cannot. As a result, it's tempting to reach for `cy.wait()` to slow down the test to human speed. This can get the test passing, but there are several reasons to avoid this approach:

1. `cy.wait()` won't scale beyond a few tests. If you want to thoroughly test your app with Cypress, you'll have a lot of tests, and if they all have `wait` commands your test suite will be extremely slow.
2. Relying on `cy.wait()` makes your tests flaky. Exactly how long do you need to wait? What is fast enough now may not be fast enough in the future, or on CI.
3. `cy.wait()` masks real application bugs.

Let's talk about the last point. How is there a bug if the app works in manual testing? Isn't this expending effort only for the sake of the test? No. Even if a user hasn’t triggered a bug yet, in many cases they *could* trigger it if they were fast enough and the app was running slowly enough. Fixing these issues will lead to an application that is more stable and reliable, so it's often effort well-spent.

So if we shouldn’t use `cy.wait()`, how can we understand what is causing the test failures to know how to fix them? An important starting point is recognizing that you as a developer and tester have an intuitive sense of when operations are done, but Cypress thinks about doneness differently. Cypress will automatically wait:

- For an absent element to appear
- For the completion of a specific network request that you explicitly ask to wait on

But Cypress *won’t* automatically wait:

* For *all* in-flight network requests to complete
* For all scheduled `setTimeout`s to run

What can we do when we *need* the test to wait on something Cypress doesn’t automatically wait for? There are two general approaches. We can either:

* Give Cypress something it *can* wait on, or
* Change the app's behavior so that Cypress *can’t* proceed until the app is ready

These general points are starting to move toward solutions, but they certainly aren’t actionable concrete steps. These async issues aren’t easy to handle. They involve thinking through aspects of Cypress and frontend frameworks that are usually beneath the surface. When these issues don’t occur consistently or locally, it can be difficult to figure out exactly what is going wrong. Even when you’ve fixed an issue, it can be difficult to explain to others on your team how you landed on this fix and why you’re confident it is reliable, and to equip your team members to troubleshoot test failures themselves.

To help, we should talk about code smells. The concept of “code smells” was coined by Kent Beck and introduced in Martin Fowler’s book *Refactoring*. There, it referred to “structures in the code that suggest—sometimes, scream for—the possibility of refactoring.” By naming code smells, this book codified patterns to look for to identify needs for refactoring. (Note that the term “code smell” can come across as judgmental if used inappropriately, but that isn’t the original intent, nor is it the intent of how I use it here. It's only intended to indicate something in the code that is causing a problem, for the purpose of fixing the problem.)

In the same way, we can name code smells to identify common patterns in Cypress errors. This will help make them easier to spot and allow providing more concrete direction around how to address them.

Let’s take a look at the three Cypress code smells I’ve been able to identify so far. For each of these code smells we'll see:

- A definition
* A trivial React example
* A description of how the problem is not limited to tests, but is legitimately a bug that could impact real users
* Possible solutions, and factors to decide which solution to use
* A more substantial example using React, Redux, and Redux-Thunk to show how the problem is more likely to arise in a production application

If you’d like to play with the code samples yourself you can download [the repo](https://github.com/CodingItWrong/cypress-code-smells). It includes Vue/Vuex versions of the examples as well.

Note that I’m writing this code with Cypress 5.3.0. Future versions of Cypress may very well make changes that prevent some of these failures.

The code smells are:

1. [Unprepared Element](#1-unprepared-element)
2. [Flickering Element](#2-flickering-element)
3. [Impatient Test](#3-impatient-test)

## 1. Unprepared Element
An element is interacted with but the data the application expects to be in place is not yet in place, resulting in undesired behavior.

The results of an Unprepared Element can manifest in a number of ways: the app could do nothing in response to that interaction, or log an error to the browser console, or even behavior that is incorrect. The common factor is that the element is present, but the behavior is not the same as when interacting with it manually.

For example, consider this React component:

```jsx
function Example1() {
  const [count, setCount] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    setTimeout(() => {
      setCount(27);
    }, 100);
  }, []);

  const increment = () => {
    if (count === null) {
      setError('Tried to increment count before it was set!');
    } else {
      setCount(count + 1);
    }
  };

  return (
    <>
      {error && <p>{error}</p>}
      <p>
        Count: <span data-cy="count">{count}</span>
      </p>
      <button onClick={increment} data-cy="increment">
        Increment
      </button>
    </>
  );
}
```

Say we have the following Cypress test:

```js
cy.visit('/unprepared/example1');
cy.get('[data-cy=increment]').click();
cy.get('[data-cy=count]').contains(28);
```

We visit the page, and the component mounts. The effect runs, scheduling a timeout to set the `count` after 100 milliseconds. In the meantime, Cypress clicks the `increment` button right away. Because the count is not yet loaded, we get an error that we tried to increment the count before it was set, and the Cypress assertion checking that the count is 28 fails.

The reason the app's state is not ready for the element is because it is being asynchronously loaded. During use by a human, the data is loaded by the time the human can interact with the element, but Cypress can interact with the element faster than the data can load. The Unprepared Element is displayed and active before the data for its operation is ready.

Even if this timing issue never occurs during the developer’s manual testing, it might still show up for an end user. If a user is fast enough and their device or connection is slow enough, they could interact with the element before the backing data is ready. Fixing the application code so that the Cypress test passes consistently will provide real safety to end users.

The solution to an Unprepared Element is to not render an interactable element until the data is ready. Instead, before the data is ready, you avoid rendering the element at all, set a form element’s `disabled` attribute, or cover the element with an overlay that prevents interaction.

To fix our React component example, we could add the following check to disable the `increment` button while the `count` is still `null`:

```jsx
<button
  onClick={increment}
  data-cy="increment"
  disabled={count === null}
>
  Increment
</button>
```

After this change, when Cypress gets to the command `cy.get('[data-cy=increment]').click()`, it sees that the `increment` button is disabled and cannot be clicked yet. So Cypress keeps retrying until it is enabled. The timeout finishes, setting the initial count value. This causes the button to be enabled, then Cypress clicks the button, and the test succeeds.

Instead of changing the Unprepared Element itself, you might be tempted to change the *event handler*, checking for the necessary data and stopping execution of the handler if the data is not ready. But to Cypress, if the element is interactable, the interaction succeeded, so Cypress will not retry the interaction. But if you remove, cover, or disable the element, Cypress knows the interaction can't occur, so it will wait for the element to be interactable.

Let's look at a second example, one that moves the data and async operation out of the component and into Redux. This is closer to how a large production application might be set up, and is a less contrived example of how an Unprepared Element could come about.

This time, we store the count in Redux and load the count from a server:

```jsx
function Example3() {
  const [error, setError] = useState(null);

  const dispatch = useDispatch();
  const count = useSelector(state => state.example3);

  useEffect(() => {
    dispatch(loadCountFromServer);
  }, [dispatch]);

  const handleIncrement = () => {
    if (count === null) {
      setError('Tried to increment count before it was set!');
    } else {
      dispatch(increment());
    }
  };
```

`loadCountFromServer` is a Redux-Thunk asynchronous action that uses `axios` to make a web service request:

```js
const loadCountFromServer = dispatch => {
  axios.get('/api/count.json').then(({data}) => {
    dispatch(setCount(data.count));
  });
};
```

This network request is what introduces the asynchronous delay. On my machine, the network request is really fast because it's local. This means the Cypress tests *sometimes* succeeds when the network request comes back quicker than Cypress clicks. But other times it fails. Even using Cypress network request stubbing, which should makes the response even faster, only causes the test to succeed some of the time.

The more reliable fix for this example’s Unprepared Element is the same as before: disabling the `increment` button until the count is present.

## 2. Flickering Element
An element appears but is then quickly removed from the DOM before Cypress can complete its interaction.

A Flickering Element is accompanied by this error:

```
Timed out retrying: cy.click() failed because this element is detached
from the DOM.

...

Cypress requires elements be attached in the DOM to interact with them.

The previous command that ran was:

  > cy.get()

This DOM element likely became detached somewhere between the previous
and current command.

Common situations why this happens:
  - Your JS framework re-rendered asynchronously
  - Your app code reacted to an event firing and removed the element

You typically need to re-query for the element or add 'guards' which
delay Cypress from running new commands.
```

This error is mysterious because when it occurs you can often *see* the element in the test browser window. Why is Cypress saying it's not in the DOM when it is?

There are actually *two* DOM elements: one that Cypress got but was then removed, and a *second* element that was added and that you see in the test browser window. Because of how fast Cypress is, it caught the first DOM element before it was removed. But because of Cypress' safety checks, it refuses to send a click event to an element that has already been removed from the DOM, instead erroring out.

We can reproduce this problem with the following example:

```jsx
function Example5() {
  const [loading, setLoading] = useState(false);
  const [count, setCount] = useState(null);

  useEffect(() => {
    setLoading(true);
    setTimeout(() => {
      setLoading(false);
      setCount(27);
    }, 1000);
  }, []);

  const increment = () => {
    setCount(count + 1);
  };

  if (loading) {
    return <p>Loading…</p>;
  }

  return (
    <>
      <p>
        Count: <span data-cy="count">{count}</span>
      </p>
      <button onClick={increment} data-cy="increment">
        Increment
      </button>
    </>
  );
}
```

The test is simple:

```js
cy.get('[data-cy=increment]').click();
cy.get('[data-cy=count]').contains(28);
```

This component has a loading state that hides the content while it’s being loaded. When the component mounts, we set the loading state to `true`, send the request, allow it to complete, then set it back to `false`. When we run the test, we get the “element is detached” error.

In this example, you can see a hint of what is going on from the fact that the Cypress `get` command in the sidebar succeeds immediately, before the 1 second loading delay:

![Cypress window showing get command succeeded before loading delay completes](/img/posts/cypress-code-smells/get-already-succeeded.png)

Cypress got the `increment` button before the loading message was even shown. This happens in our example because the initial `loading` state is `false`, so upon the first render the `increment` button is shown. It is set to `true` in the effect, which triggers a rerender, but this doesn’t prevent the first render from completing.

(Note that getting a Flickering Element in this specific case relies on a detail of the behavior of React's `useEffect` hook: React renders the initial state and flushes it to the DOM, then rerenders with the state updated by `useEffect`. The same didn't happen in my testing in Vue using the `mounted()` lifecycle method. But a flickering element can happen in any framework in other circumstances, any time the timing works out where Cypress grabs an element right before it's removed from the DOM.)

A fix to this that sometimes works is to send `{force: true}` as the argument of the `cy.click()` method. This bypasses Cypress' check that the element is in the DOM. One downside is that it bypasses other checks as well, such as that the element is visible and uncovered. Bypassing these checks can lead to false positives or timing issues: for example, Cypress could click on an element covered by an overlay, whereas a real user could not.

The other downside of `force: true` is that it allows you to keep this flickering element in your app. I would argue that it's an application bug that an element appears and then disappears. If the user's device is slow enough they might be able to see that flicker, which is not an ideal user experience. Fixing this flicker instead of changing the test will be a better solution for our users.

To fix the flicker in our example’s application code, we can simply set the initial value of `loading` to `true`, so the `increment` button isn’t rendered before the loading:

```jsx
function Example6() {
  const [loading, setLoading] = useState(true);
  const [count, setCount] = useState(null);

  useEffect(() => {
    setTimeout(() => {
      setLoading(false);
      setCount(27);
    }, 1000);
  }, []);
```

In modern frontend frameworks, flicker like this tends to occur as a side effect of a very *positive* aspect of those frameworks: declarative rendering. We define components that specify how they should appear based on the data available to them, and then every time that data changes, they rerender. This leads to reliable UIs and a productive development flow. But a downside is that it's easy to end up with a data flow where that rerendering causes an element to appear and then disappear quickly.

To figure out how to fix a flickering element, identify the data flow that is causing it to flicker. If the problem can be reproduced in your local development machine, add logging that outputs the component’s props and state/data every time they change. This should help give you visibility into what sequence of events is allowing it to flicker.

Let’s again look at a more substantial example involving Redux:

```jsx
function Example7() {
  const dispatch = useDispatch();
  const count = useSelector(state => state.example7.count);
  const loading = useSelector(state => state.example7.loading);

  useEffect(() => {
    dispatch(loadCountFromServer);
  }, [dispatch]);

  const handleIncrement = () => {
    dispatch(increment());
  };

  if (loading) {
    return <p>Loading…</p>;
  }
```

Here’s `loadCountFromServer`:

```js
const loadCountFromServer = dispatch => {
  dispatch(request());
  axios.get('/api/count.json').then(({data}) => {
    dispatch(setCount(data.count));
  });
};
```

And here are the important parts of the reducer:

```js
const initialState = {
  count: null,
  loading: false,
};

function example1(state = initialState, action) {
  switch (action.type) {
    case EXAMPLE7_REQUEST:
      return {
        ...state,
        loading: true,
      };
    case EXAMPLE7_SET_COUNT:
      return {
        ...state,
        count: action.count,
        loading: false,
      };
//...
}
```

Now that the `loading` state is stored in Redux, it’s not as obvious looking at the component that it will start with a `loading` state of `false` and render the button, leading to a Flickering Element. Real applications can be even more complex, with multiple web service requests in parallel that are sent conditional upon the application state. It can be difficult to trace exactly what sequence of renders will happen with what data in place.

We can fix our Redux example with the same solution: setting the initial `loading` state to `true`. This might not be a good enough solution for a production application, though: if you can navigate to this component multiple times then the `loading` state would start `false` the second time anyway. A more robust solution beyond the scope of this article might be needed to ensure no rendering happens before loading has begun.

## 3. Impatient Test
A test kicks off an asynchronous operation, but then, instead of waiting for it to complete, the test moves on in a way that will cause a problem.

An Impatient Test can happen when the next element to interact with is in the DOM both before and after the asynchronous operation completes, so the test can interact with it immediately.

Here’s an example:

```jsx
function Example9() {
  const [isSignedIn, setIsSignedIn] = useState(true);
  const [guestCount, setGuestCount] = useState(27);
  const [memberCount, setMemberCount] = useState(42);

  const signOut = () => {
    axios.delete('http://localhost:3001/sessions/1').then(() => {
      setIsSignedIn(false);
    });
  };

  const increment = () => {
    if (isSignedIn) {
      setMemberCount(memberCount + 1);
    } else {
      setGuestCount(guestCount + 1);
    }
  };

  return (
    <>
      <p>
        {isSignedIn ? (
          <>
            Signed In
            <button onClick={signOut} data-cy="sign-out">
              Sign Out
            </button>
          </>
        ) : (
          <span data-cy="signed-out">Signed Out</span>
        )}
      </p>
      <button onClick={increment} data-cy="increment">
        Increment
      </button>
      <p>
        Guest Count: <span data-cy="guest-count">{guestCount}</span>
      </p>
      <p>
        Member Count: <span data-cy="member-count">{memberCount}</span>
      </p>
    </>
  );
}
```

This time we have two counters: one for signed-in members, and one for signed-out guests. We have a single `increment` button that increments one or the other of the counters, depending on if you’re signed in or not. The user starts out signed in.

Now, our test:

```js
cy.get('[data-cy=sign-out]').click();

cy.get('[data-cy=increment]').click();
cy.get('[data-cy=guest-count]').contains(28);
cy.get('[data-cy=member-count]').contains(42);
```

We sign out, then click the increment button. We check that the guest count increased by one and that the member count did not increase. But when we run the test, surprisingly, it’s the *member* count that increased instead of the guest count. Why is that?

Cypress’s automatic waiting usually does just what you want, but as a result it can be tempting to think of your Cypress test as synchronous, where each operation runs to completion before the next starts. But that’s not the way Cypress works. It moves forward with each step as soon as it can see the corresponding user interface element.

In the case of our example, the `increment` button is present from the start, and it's not an Unprepared Element: it has the data it needs to work. But its behavior is different before and after the sign out request completes. Our test initiates the sign out request but then impatiently clicks `increment` before the sign out completes, so it's the member count instead of the guest count that's incremented. A real user could run across this case if the sign out network request was slow enough, and would probably be surprised at the behavior.

There are a few different approaches you can use to fix an Impatient Test.

First, you may be able to change the application logic so that the test *can* proceed immediately. For example, in the case of our Sign Out test, we need to send the request to the server, but the frontend doesn’t need to wait for that response to record the fact that it should now consider the user signed out. We can set `signedIn` to false immediately, so that when Cypress clicks the Increment button immediately, it does the right thing:

```jsx
const signOut = () => {
  setIsSignedIn(false);
  axios.delete('http://localhost:3001/sessions/1').then(() => {
    console.log('request complete');
  });
};
```

If you can’t change the application logic so the test can proceed immediately, a second option is to treat the element like you would an Unprepared Element and prevent interaction with it while the async operation is running. It may be too visually disruptive to add an overlay or unrender significant parts of the page, so you might prefer to disable the element. In our example, we can accomplish this by adding a “loading” state of `isSigningOut`:

```jsx
function Example11() {
  const [isSignedIn, setIsSignedIn] = useState(true);
  const [isSigningOut, setIsSigningOut] = useState(false);
  //...

  const signOut = () => {
    setIsSigningOut(true);
    axios.delete('http://localhost:3001/sessions/1').then(() => {
      setIsSignedIn(false);
      setIsSigningOut(false);
    });
  };

  //...
      <button
        onClick={increment}
        disabled={isSigningOut}
        data-cy="increment"
      >
        Increment
      </button>
```

Let’s say you decide you don’t want to disable the element. Maybe you decide that it *does* make sense for the user to interact with the element at any time; just for the sake of *this* test, you don’t want the test to interact until the async operation completes. What can you do?

To make Cypress wait, you can ask it to look for a “green light”: an indication of when it is safe for the test to proceed. A green light can be a UI element or it can be the network request itself. Let's look at each of these two approaches for our example.

If we want to use a UI element as a green light, we can look for the “Signed Out” element to appear on the screen. Recall that it is shown when the user is signed out:

```jsx
{isSignedIn ? (
  <>
    Signed In
    <button onClick={signOut} data-cy="sign-out">
      Sign Out
    </button>
  </>
) : (
  <span data-cy="signed-out">Signed Out</span>
)}
```

We can ask Cypress to get the `signed-out` element:

```js
cy.get('[data-cy=sign-out]').click();
cy.get('[data-cy=signed-out]');

cy.get('[data-cy=increment]').click();
cy.get('[data-cy=guest-count]').contains(28);
cy.get('[data-cy=member-count]').contains(42);
```

Note that we don't need to interact with the `signed-out` element; Cypress will just wait until it appears. When it does, we know that interacting with the *other* element will do what we intend, so it's safe for the test to proceed.

If, instead, we want to use a network request as our green light, we can give the sign out request an alias and wait for it to complete:

```js
cy.server();

cy.route({
  method: 'DELETE',
  url: 'http://localhost:3001/sessions/1',
}).as('signOut');

cy.visit('/impatient/example12');

cy.get('[data-cy=sign-out]').click();
cy.wait('@signOut');

cy.get('[data-cy=increment]').click();
cy.get('[data-cy=guest-count]').contains(28);
cy.get('[data-cy=member-count]').contains(42);
```

We click the `sign-out` button, then wait for the `signOut` network request to complete before moving on to click `increment`. Note that we aren’t stubbing out the response of the network request, only giving the alias `signOut` so we can wait for it later.

Which green-light approach should you take: waiting on the element or on the network request? The difference between these two approaches is what they couple to. The network request approach couples your test to a specific request that is sent, so if that request changes to a different URL, the test would need to be updated. The element-based approach is resilient against changes to network requests, but it might not be an option if there is no visible element that indicates when the operation is complete.

Green lights are a common pattern in Cypress tests, but there is a significant downside to them. Future tests that interact with this element will need to look for the green light as well, and if the developer writing them doesn’t know or forgets, they’ll experience the same Impatient Test problem again. But if instead of updating the test with a green light you update the *application code* with one of the other two solutions, this will prevent the Impatient Test problem for any future test of this element that is written.

Let’s look at a more substantial example of an Impatient Test:

```jsx
function Example13() {
  const dispatch = useDispatch();
  const isSignedIn = useSelector(state => state.example13.isSignedIn);
  const guestCount = useSelector(state => state.example13.guestCount);
  const memberCount = useSelector(state => state.example13.memberCount);

  const handleSignOut = () => {
    dispatch(signOut);
  };

  const handleIncrement = () => {
    dispatch(increment());
  };

  return (
    <>
      <p>
        {isSignedIn ? (
          <>
            Signed In
            <button onClick={handleSignOut} data-cy="sign-out">
              Sign Out
            </button>
          </>
        ) : (
          <span data-cy="signed-out">Signed Out</span>
        )}
      </p>
      <button onClick={handleIncrement} data-cy="increment">
        Increment
      </button>
      <p>
        Guest Count: <span data-cy="guest-count">{guestCount}</span>
      </p>
      <p>
        Member Count: <span data-cy="member-count">{memberCount}</span>
      </p>
    </>
  );
}
```

And the async action:

```js
const signOut = dispatch => {
  axios.delete('http://localhost:3001/sessions/1').then(() => {
    dispatch(setSignedOut());
  });
};
```

Once again, due to the move of the logic and state out of the component into Redux, it’s harder to see the timing issue that will arise.

Of the three possible solutions, the one involving disabling the `increment` button is the most instructive to see in this Redux example, so let’s take a look:

```jsx
function Example14() {
  const isSigningOut = useSelector(state => state.example14.isSigningOut);
  //...
      <button
        onClick={handleIncrement}
        disabled={isSigningOut}
        data-cy="increment"
      >
        Increment
      </button>
```

```js
const signOut = dispatch => {
  dispatch(signOutRequest());
  axios.delete('http://localhost:3001/sessions/1').then(() => {
    dispatch(signOutSuccess());
  });
};
```

```js
const initialState = {
  isSignedIn: true,
  isSigningOut: false,
  guestCount: 27,
  memberCount: 42,
};

function example1(state = initialState, action) {
  switch (action.type) {
    case EXAMPLE14_SIGN_OUT_REQUEST:
      return {
        ...state,
        isSigningOut: true,
      };
    case EXAMPLE14_SIGN_OUT_SUCCESS:
      return {
        ...state,
        isSigningOut: false,
        isSignedIn: false,
      };
//...
```

Now we have an `isSigningOut` state value, and two actions that are dispatched: `EXAMPLE14_SIGN_OUT_REQUEST` when the sign out starts, and `EXAMPLE14_SIGN_OUT_SUCCESS` when it completes. We set the `isSigningOut` state value to `true` upon `REQUEST` and `false` upon `SUCCESS`. This is now a value we can use to disable the `increment` button.

## Conclusion
When it comes to these surprising Cypress test failures, there are several things going on that make the code hard to think about:

* Asynchronous behavior is often hard to wrap our heads around.
* Frontend frameworks' automatic rerendering, which is a great thing, nevertheless also makes this situation hard to trace.
* When the code works in manual testing but fails in the test, potentially only sometimes or only on CI, it's very confusing and often difficult to reproduce.

One resource to help overcome these challenges is the Cypress code smells we've discussed in this post. These code smells serve as patterns to look for in your failing tests and provide suggestions for how to fix them. They can also be helpful for communication in code review when your whole team is familiar with them ("the test is passing now but I think you might have an Impatient Test; it might be safest to check a Green Light. What do you think?”)

Beyond the specific code smells, the general thought patterns above can help you in your own troubleshooting. Occasionally the "it just works" approach to frontend frameworks and Cypress breaks down and you need to understand what is happening in the app and test in-depth. Improving your mental model of these tools will help you identify other asynchronous problems and come up with code smells of your own.

While it makes sense to prioritize bugs found in manual use over failures that only occur in automated tests, the latter are still bugs. It's worthwhile to fix them to make your app's behavior more deterministic. Not everybody’s CPU and network connection are as fast as yours, so things *will* slow down for someone. Making sure your app works on any timing helps ensure your tests won't fail and your users won't run into errors.
