---
title: Isolated and Integrated Tests in Redux and Vuex
---

How do you test your frontend data store? Frontend data stores commonly have at least two different aspects: asynchronous logic to make web service requests, and synchronous logic to persist data in the store. Vuex, the first-party data store for Vue.js, supports both of these out of the box in the form of actions and mutations, respectively. Redux, one of the most popular data store options for React, includes only synchronous operations by default; `redux-thunk` is a common add-on library to add asynchronous logic at the data store level.

In both libraries, the synchronous and asynchronous logic are implemented as separate functions. How do we test them? We could test functions in isolation to see what they return or mutate. Alternatively, since these functions usually work closely together, we could test them in integration, seeing if an asynchronous function ultimately persists the right data to the store.

Let's look at examples of how to write both isolated and integrated tests in Redux and Vuex, and see the tradeoffs of the two approaches.

## The Store Code

Here's the Redux and `redux-thunk` code to implement loading data from a web service. First, the `loadRestaurants()` thunk:

```js
export const loadRestaurants = () => async (dispatch, getState, api) => {
  const restaurants = await api.loadRestaurants();
  dispatch(storeRestaurants(restaurants));
};

export const storeRestaurants = restaurants => ({
  type: STORE_RESTAURANTS,
  restaurants,
});
```

You may not have seen the third argument to a thunk before. Where does it come from? We use `redux-thunk`'s `[.withExtraArgument()](https://github.com/reduxjs/redux-thunk#injecting-a-custom-argument)` method to pass our API in as a dependency, so that it's passed as the third argument to the asynchronous function. We make a call to load restaurants from the API. We don't need to look at the implementation here, it just kicks off an HTTP request. When we get the restaurants back, we dispatch `storeRestaurants()`, which is an action creator do dispatch the action that will persist the restaurants in our store.

When we set up our store, we do so in a function that allows the `api` argument to be passed-in, so that it's not even hard-coded at that location. We'll see the value of this for testing later:

```js
const myCreateStore = api =>
  createStore(
    rootReducer,
    compose(applyMiddleware(thunk.withExtraArgument(api)), devToolsEnhancer()),
  );
```

Here's the `restaurants` reducer that responds to the `STORE_RESTAURANTS` action:

```js
export function restaurants(state = [], action) {
  switch (action.type) {
    case STORE_RESTAURANTS:
      return action.restaurants;
    default:
      return state;
  }
}
```

Pretty trivial.

Let's see the equivalent Vuex module. Vuex's modules combine the asynchronous logic and data storage in one place:

```js
const restaurants = api => ({
  namespaced: true,
  state: {
    restaurants: []
  },
  mutations: {
    storeRestaurants(state, restaurants) {
      state.restaurants = restaurants;
    }
  },
  actions: {
    load({ commit }) {
      return api.loadRestaurants().then(restaurants => {
        commit("storeRestaurants", restaurants);
      });
    }
  }
});

export default restaurants;
```

## Testing Functions in Isolation

The most isolated way to test these stores would be to test the functions in an isolated way. This is the most common approach in the Redux world. Let's see what those tests look like.

We'll start with the test of the Redux reducer function. It's as trivial as you might expect:

```js
describe('restaurant reducers', () => {
  it('starts with an empty array', () => {
    const initialState = restaurants(undefined, {type: 'INITIAL'});
    expect(initialState).toEqual([]);
  });

  describe('storeRestaurants', () => {
    it('overwrites the restaurant list', () => {
      const initialState = [
        { id: 1, name: 'Sushi Place' },
      ];
      const newRestaurants = [
        { id: 2, name: 'Pizza Place' },
      ];

      const resultState = restaurants(
        initialState,
        storeRestaurants(newRestaurants),
      );

      expect(resultState).toEqual(newRestaurants);
    });
  });
});
```

The reducer test has no knowledge of Redux, it's just testing a simple function.

Now, the test for the thunk:

```js
describe('loadRestaurants', () => {
  it('queries the API and stores the results', async () => {
    const stubbedRestaurants = [
      { id: 1, name: 'Sushi Place' },
      { id: 2, name: 'Pizza Place' },
    ];

    const dispatch = jest.fn();
    const api = {
      loadRestaurants: () => Promise.resolve(stubbedRestaurants),
    };

    await loadRestaurants()(dispatch, null, api);

    expect(dispatch).toHaveBeenCalledWith(
      storeRestaurants(stubbedRestaurants),
    );
  });
});
```

The only slight bit of complexity here is the fact that the thunk is a thunk (a function that returns a function) so we need to call it and then call the resulting function. Other than that, this test is just a simple function invocation, where we stub one passed-in function (`api.loadRestaurants`) to return a result and we set up another function (`dispatch`) as a mock so we can ensure it was called with the right arguments. Again here, `redux-thunk`'s interface means our test has no dependencies on any libraries; we are just testing a function.

Let's compare these isolation tests to the ones for Vuex. First, the action test:

```js
describe("load action", () => {
  it("commits the restaurants returned from the server", async () => {
    const api = {
      loadRestaurants: () => Promise.resolve(stubbedRestaurants)
    };
    const commit = jest.fn();

    const restaurantsModule = restaurants(api);
    await restaurantsModule.actions.load({ commit });

    expect(commit).toHaveBeenCalledWith(
      "storeRestaurants",
      stubbedRestaurants
    );
  });
});
```

Similarly to the `redux-thunk` test, we stub the API call, and mock a passed-in function to check. The only difference is the function signatures.

Now, the test for the mutation:

```js
describe("storeRestaurants mutation", () => {
  it("overwrites the restaurants in the state", () => {
    const restaurantsModule = restaurants();

    const state = {
      restaurants: [{ id: 3, name: "Old Restaurant" }]
    };

    restaurantsModule.mutations.storeRestaurants(state, stubbedRestaurants);

    expect(state.restaurants).toEqual(stubbedRestaurants);
  });
});
```

Because Vuex mutates the state object, we test the same state object afterward to see its new contents.

One consequence of this approach to testing functions in isolation, both for Redux and Vuex, is that these test don't ensure the functions work together. Ideally our thunk/action test would confirm that the appropriate dispatch/commit happen, and then that the reducer/mutation
test covers the identical situation, ensuring the resulting state is correct. But in production applications I've seen a mismatch between the two types of test. If they are only tested together in end-to-end tests, then that puts high pressure on those tests to be reliable, fast, and run frequently, to ensure our stores work as we expect.

## Integration Testing

Let's try to get earlier and quicker feedback about how our asynchronous and synchronous store functions work together. We'll do this by testing our stores in integration to ensure they work as a whole. This time we'll start with Vuex, as I've seen the integrated approach more frequently in the Vuex world.

Let's see the tests. First, the store module:

```js
it("allows loading restaurants from the server", async () => {
  const stubbedRestaurants = [
    { id: 1, name: "Sushi Place" },
    { id: 2, name: "Pizza Place" }
  ];

  const api = {
    loadRestaurants: jest.fn()
  };

  const store = new Vuex.Store({
    modules: {
      restaurants: restaurants(api)
    }
  });

  api.loadRestaurants.mockResolvedValue(stubbedRestaurants);

  await store.dispatch("restaurants/load");

  expect(store.state.restaurants.restaurants).toEqual(stubbedRestaurants);
});
```

Note that we aren't testing the action, mutation, and state separately: we're testing them in integration with one another. This test treats the store module as a unit: when we dispatch a certain action, we expect the relevant state to be exposed. Fittingly, this relationship between the action and the state is the same information the component assumes and uses in its own design.

To test the Vuex module in integration, we need to create a real Vuex instance. This is what passes the `commit` function to the action, and what passes the state to the mutation function when `commit` is called. So, unlike testing our module functions in isolation, this couples the test to Vuex. The benefit we get is that we ensure that Vuex connects our action and mutation together to give us our expected result.

How can we test in this way with Redux? Well, Redux isn't only used in the context of React: Redux creates a plain store object we can interact with directly.

```js
describe('store', () => {
  it('allows loading restaurants from the server', async () => {
    const stubbedRestaurants = [
      {
        id: 1,
        name: 'Sushi Place',
      },
      {
        id: 2,
        name: 'Pizza Place',
      },
    ];

    const api = {
      loadRestaurants: () => Promise.resolve(stubbedRestaurants),
    };

    const store = createStore(api);
    await store.dispatch(loadRestaurants());

    expect(store.getState().restaurants.restaurants).toEqual(
      stubbedRestaurants,
    );
  });
});
```

Notice why we defined our store in a function to pass the `api` in: it allows us to easily pass in a mocked version. Jest's module mocking could help as well, but this makes the interface and dependencies more clear and straightforward. Again, we just dispatch an action, wait for the returned promise to resolve, then check the state in the store.

Although our test doesn't reference Redux, we're importing our store, which is set up using Redux. So our test does depend on the Redux library.

## Isolation or Integration?

Even the name of a Vuex "[module](https://en.wikipedia.org/wiki/Modular_programming)" suggests that it is a self-contained unit and that we should interact with it through its public interface.
