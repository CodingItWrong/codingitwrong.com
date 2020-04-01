---
title: Unit Tests and Framework Coupling
---

I've been evaluating React+Redux and Vue+Vuex to use as a basis for some testing content I want to write. To try to pick between them, I've written some unit tests for the components and the data layer. And it's given me helpful insights into the differences between React and Vue.

One of the benefits of unit tests is that they help you see the interface of the units you're testing. When you're testing a piece of code where you define the interface yourself, it can give you the insight you need to adjust the interface to be as simple and straightforward as possible. Due to the nature of these two frontend frameworks, we have a bit less control over the interface: React and Vue define how components work, and Redux and Vuex define how the data layer is connected to components. So instead, our unit tests give us some insight into how the _framework_ chose to set up the interfaces, and into the tradeoffs between them.

## React and Redux

With Redux, I've gone with `redux-thunk` as well: it strikes the right balance between separating out my async data layer logic from my components, while avoiding the complexity of `redux-saga` or `redux-observable`. For this example, we're just loading some data from a web service.

Here's our component:

```jsx
import React, {useEffect} from 'react';
import {connect} from 'react-redux';
import {loadRestaurants} from './store/restaurants/actions';

export const RestaurantList = ({restaurants, loadRestaurants}) => {
  useEffect(() => {
    loadRestaurants();
  }, [loadRestaurants]);

  return (
    <div>
      <h2>Restaurants</h2>
      <ul>
        {restaurants.map(restaurant => (
          <li key={restaurant.id}>{restaurant.name}</li>
        ))}
      </ul>
    </div>
  );
};

const mapStateToProps = state => ({
  restaurants: state.restaurants.restaurants,
});

const mapDispatchToProps = {loadRestaurants};

export default connect(mapStateToProps, mapDispatchToProps)(RestaurantList);
```

The component simply kicks off a `loadRestaurants` action, and renders the `restaurants` given to it. The implementation of these was given in the previous post.

Let's see what the tests of this component show us about `react-redux`'s interface. Our test uses React Testing Library.

```jsx
import React from 'react';
import {render} from '@testing-library/react';
import {RestaurantList} from '../RestaurantList';

describe('RestaurantList', () => {
  const stubbedRestaurants = [
    { id: 1, name: 'Sushi Place' },
    { id: 2, name: 'Pizza Place' },
  ];

  let loadRestaurants;
  let context;

  beforeEach(() => {
    loadRestaurants = jest.fn();
    context = render(
      <RestaurantList
        restaurants={stubbedRestaurants}
        loadRestaurants={loadRestaurants}
      />,
    );
  });

  it('displays the passed-in restaurants', async () => {
    const {findByText} = context;
    expect(await findByText('Sushi Place')).not.toBeNull();
    expect(await findByText('Pizza Place')).not.toBeNull();
  });

  it('loads restaurants', async () => {
    expect(loadRestaurants).toHaveBeenCalled();
  });
});
```

This test helps us see that `RestaurantList` has two responsibilities: render the restaurants it's given, and kick off a request to load restaurants from the server. The component itself doesn't actually see the relationship between those two things; it just receives a list of restaurants and a function to call. Note that the test itself has no knowledge of Redux; we test the unconnected version of the component. This is a very simple aspect of `react-redux`'s interface: the state and dispatched actions are passed to the component as simple props.

## Vue and Vuex

Here's our Vue component implementing the same functionality:

```html
<template>
  <div>
    <h2>Restaurants</h2>
    <ul>
      <li
        data-test="restaurant"
        v-for="restaurant in restaurants"
        :key="restaurant.id"
      >
        {{ restaurant.name }}
      </li>
    </ul>
  </div>
</template>

<script>
import { mapState, mapActions } from "vuex";

export default {
  name: "RestaurantList",
  mounted() {
    this.loadRestaurants();
  },
  methods: mapActions({
    loadRestaurants: "restaurants/load"
  }),
  computed: mapState({
    restaurants: state => state.restaurants.restaurants
  })
};
</script>
```

The implementation of our Vuex module, again, was defined in the previous post.

Let's see the test. We're using the first-party Vue Test Utils to help us.

```js
import Vuex from "vuex";
import { mount, createLocalVue } from "@vue/test-utils";
import RestaurantList from "@/components/RestaurantList";

const localVue = createLocalVue();
localVue.use(Vuex);

describe("RestaurantList", () => {
  const stubbedRestaurants = [
    { id: 1, name: "Sushi Place" },
    { id: 2, name: "Pizza Place" }
  ];

  let restaurantsModule;
  let wrapper;

  beforeEach(() => {
    restaurantsModule = {
      namespaced: true,
      state: {
        restaurants: stubbedRestaurants
      },
      actions: {
        load: jest.fn()
      }
    };
    const store = new Vuex.Store({
      modules: {
        restaurants: restaurantsModule
      }
    });

    wrapper = mount(RestaurantList, { localVue, store });
  });

  it("loads restaurants on mount", () => {
    expect(restaurantsModule.actions.load).toHaveBeenCalled();
  });

  it("displays the restaurants", () => {
    expect(
      wrapper
        .findAll("[data-test='restaurant']")
        .at(0)
        .text()
    ).toBe("Sushi Place");
    expect(
      wrapper
        .findAll("[data-test='restaurant']")
        .at(1)
        .text()
    ).toBe("Pizza Place");
  });
});
```

The first difference between this and the React test is how the component is mounted. In React Testing Library, the component is set up using the same JSX you use in production code:

```jsx
render(
  <RestaurantList
    restaurants={stubbedRestaurants}
    loadRestaurants={loadRestaurants}
  />,
);
```

Whereas with Vue Test Utils, we use a different API:

```js
mount(RestaurantList, { localVue, store });
```

This is because Vue by default doesn't define component trees in JavaScript using JSX (although the option is available); by default, you use separate templates. So in our test, which is JavaScript, you need to use a different mechanism to define the component to test.

A lot has been written about the tradeoffs between JSX and templates, and both the upcoming Vue 3.0 and Ember.js get significant performance benefits from the extra constraints that templates provide. But one of the benefits of JSX is that, as a syntax that operates inside JavaScript files, it can be used in a wider variety of contexts, such as test files. So with JSX the interface you use to configure a component in a test is the same one you use in production code. This is a nice benefit to test simplicity and readability.

The next difference to look at is how the connection of the component to the store is tested:

```js
restaurantsModule = {
  namespaced: true,
  state: {
    restaurants: stubbedRestaurants
  },
  actions: {
    load: jest.fn()
  }
};
const store = new Vuex.Store({
  modules: {
    restaurants: restaurantsModule
  }
});

wrapper = mount(RestaurantList, { localVue, store });
```

We want our test in isolation from the real Vuex store module, so instead we define a "fake" module inline. We only include the bits of the module that the component uses: the `restaurants` state item and the `load` action. We don't need to include the mutation in the test because our component isn't aware of it; it functions as an implementation detail of the store module.

But note that we _do_ create a real Vuex store, just configured with our minimal store module. Then we pass that `store` as an option when we `mount()` the component. This sets it up so that when we map the state and actions in the component, they are ready. Here's that component code again as a reminder:

```js
import { mapState, mapActions } from "vuex";

export default {
  name: "RestaurantList",
  mounted() {
    this.loadRestaurants();
  },
  methods: mapActions({
    loadRestaurants: "restaurants/load"
  }),
  computed: mapState({
    restaurants: state => state.restaurants.restaurants
  })
};
```

So note that both the component and its test are aware of Vuex: they reach out to the Vuex library to ask it to set things up. This is different from `react-redux`, where the library passes simple data and functions to the components, so that testing them means just passing data and functions ourselves.

There is an alternative to creating a real Vuex store in our tests, as described in the book [*Testing Vue.js Applications*](https://www.manning.com/books/testing-vue-js-applications): creating a mock store. Here's the example the book gives:

```js
const $store = {
  actions: {
    fetchListData: jest.fn()
  }
}
shallowMount(TestComponent, {
  mocks: {
    $store
  }
})
```

I tried this approach in my application, and it didn't work. I think the problem is related to the fact that I'm using a store module, and Vuex remaps things a bit. I don't provide a `state.restaurants.restaurants` item or a `restaurants/load` action; I provide a `restaurants` module that has a `restaurants` state item and a `load` action. So there is some logic Vuex runs to get from the interface of the store code provided to the interface presented to the component. Even if I could get it working, this means that the Vuex module config in the test would be set up differently than the production one, reducing my confidence that the components would work in isolation. Because of that, I feel more confident in my tests if I'm _really_ passing a store module to Vuex and _really_ hooking Vuex up to the component.

Our tests show that there is extra coupling involved on the Vue side: either the tests need to actively use Vuex, or they need to be aware of some of the mapping logic Vuex uses with modules. This makes the tests harder to write and harder to see what is happening at a glance.
