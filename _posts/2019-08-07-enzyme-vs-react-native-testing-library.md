---
title: "Choosing a React Native Component Testing Library: Enzyme vs. React Native Testing Library"
---

In my writing and workshop about React Native testing, I demonstrate component testing using [React Native Testing Library][rntl]. But what about the other big React Native component testing library, [Enzyme][enzyme]? What are the tradeoffs between the two? And how strong is my recommendation?

It’s pretty strong. By contrast, on the web I'm *somewhat* in favor of the analogous React Testing Library over Enzyme. But on React Native I'm *strongly* in favor of React Native Testing Library (RNTL) over Enzyme. Here's why.

## Support
Enzyme is primarily a library for testing React. Although it also supports React Native, the latter has always been a second-class citizen in Enzyme. To get it working you have to follow [complex instructions](https://airbnb.io/enzyme/docs/guides/react-native.html), including either using [a fork of a mocking library](https://github.com/RealOrangeOne/react-native-mock) or a complex jsdom setup. Setting up RNTL, by contrast, is just `yarn add --dev react-native-testing-library` and you're done.

So that’s the state of support as of today. But might it get better in the future? Enzyme is maintained by Airbnb, who has publicly sunsetted their use of React Native. So it seems most likely that Enzyme's React Native support will get *worse* over time, not better. RNTL was created by Callstack, one of the most committed companies to contributing to React Native.

## Full Mounting
Enzyme on React Native only supports shallow rendering, not full mounting. (There is a way to get it working with full mounting using JSDOM, but this makes your environment less realistic for React Native, and can be fragile.) RNTL supports the opposite: only full mounting, not shallow rendering. At a theoretical level there are tradeoffs between the two approaches, and you could go either way depending on your testing philosophy. But although I’m more predisposed towards isolated testing, I’ve found that in React full-mounting components works better.

The tradeoff is between what your test is coupled to. For example, say you have a component that renders a list of buttons:

```jsx
const ButtonList = ({ rows, onPress }) => (
  <View>
    {rows.map((row, index) => (
      <ThirdPartyStylishButton
        key={row.title}
        title={row.title}
        onPress={onPress}
      />
    ))}
  </View>
);
```

Now, say you want to write a test for `ButtonList` that the right number of buttons is rendered. With shallow mounting in Enzyme, the test doesn’t know anything about what’s inside a `ThirdPartyStylishButton`; it just knows how many `ThirdPartyStylishButton`s there are. So you would assert that the right number of `ThirdPartyStylishButton`s are rendered:

```jsx
describe('ButtonList', () => {
  it('renders the correct number of buttons', () => {
    const rows = [{ title: 'Foo' }, { title: 'Bar' }];
    const wrapper = shallow(<ButtonList rows={rows} />);
    expect(wrapper.find(ThirdPartyStylishButton).length).toBe(2);
  });
});
```

So your test is coupled to the fact that `ButtonList` uses `ThirdPartyStylishButton` for the children. If you decide to change later to use your own stylish button implementation, you would need to change your tests.

By contrast, with full mounting in RNTL, the contents of `ThirdPartyStylishButton` are rendered as well. There is _temporarily_ a `getByType` function that lets you check the `ThirdPartyStylishButton`s rendered, but it will be removed in RNTL 2.0 in favor of the following. Instead, you would query by something user-visible, such as the text on the button, or the accessibility ID or label of the button:

```jsx
describe('ButtonList', () => {
  it('renders the correct buttons', () => {
    const rows = [{ title: 'Foo' }, { title: 'Bar' }];
    const { queryByText } = render(<ButtonList rows={rows} />);
    expect(queryByText('Foo')).not.toBeNull();
    expect(queryByText('Bar')).not.toBeNull();
  });
});
```

This means that you could change to using your own stylish button implementation, as long as the same text or accessibility info is used, the test would continue to work. But on the flip side, if the implementation of `ThirdPartyStylishButton` changes, your test might fail at that time. So your test is coupled to the _implementation_ of `ThirdPartyStylishButton`—but if you’re testing against text or accessibility attributes, hopefully those aspects of its implementation will be stable.

## Testing the Contract
Enzyme is more flexible in terms of allowing all kinds of different testing approaches, including what I would consider testing implementation details. RNTL only allows testing a component through its public interface, which I and many others argue is a better approach.

For example, in Enzyme you can directly set the state of your component, call a method on your component directly, then check that the state was changed appropriately:

```jsx
describe('increment()`, () => {
  it('increases the count by 1', () => {
    const counter = shallow(<Counter />);
    counter.setState({ count: 1 });
    counter.instance.increment();
    expect(counter.state('count')).toBe(2);
  });
});
```

Let’s see the simplest component implementation that satisfies this test:

```js
class Counter extends Component {
  state = { count: 0 };

  increment() {
    const { count } = this.state;
    this.setState({ count: count + 1 });
  }

  render() {}
}
```

It doesn’t render anything! Because we’re just testing internal state and methods of the component, we aren’t guaranteeing that it outputs anything at all. Now, you could do a separate test to inspect the rendered output. But if you did so, this wouldn’t guarantee that the rendering and behavior of the component work together; they could be using different names for the state property, for example. With tests like that, you aren’t even verifying that a single component works on its own.

Also, such a low-level test couples you to the current implementation of your component, preventing you from making changes in the future. What if, instead of just storing an integer of the count, you wanted to store a record for each time the counter was incremented, including the time it was done? To do so, you would need to rewrite this test.

With RNTL, instead, you would focus on what the user can see. How does the component end up in the initial state you want? Then, how does the user interact with it to bring about a change? Then, how can the user that it’s correctly ended up in the final state it needs to be in? That’s what you would test.

```jsx
describe('Counter', () => {
  it('allows incrementing the count', () => {
    const { getByText } = mount(<Counter initialCount={1} />);
    fireEvent.press(getByTestId('incrementButton'));
    expect(getByText('Count: 2')).not.toBeNull();
  });
});
```

This test ensures that both the rendering and the behavior of the component work together. It also allows you to refactor the implementation of the component, so that the internal state can be changed for future requirements. As you do so, the test will continue to pass, giving you protection to encourage refactoring.

Dan Abramov made this point in a tweet:

> “We don’t encourage reading implementation details like state variables themselves…Instead, test observable behavior — i.e. what your component renders or does."

-- @dan_abramov, <https://twitter.com/dan_abramov/status/1103439786122141701>

Vue.js core team member Edd Yerburgh refers to this as “testing the contract”:

> A component contract is the agreement between a component and the rest of the application…Other components can assume the component will fulfill its contractual agreement and produce the agreed output if it’s provided the correct input.

-- Edd Yerburgh, _Testing Vue.js Applications_

I’ve found that testing the contract of components is the right level of abstraction for testing components: it allows you to treat what happens inside them as an implementation detail that can be refactored in the future. Because of this, all the extra flexibility that Enzyme provides in terms of testing styles is unnecessary in my mind.  And when the docs and tutorials describe these approaches, it tempts newer developers into using these approaches and getting themselves in trouble.

## What Matters
Between these different pros and cons, the topic of support is the strongest argument: RNTL is easier to set up and more likely to be supported in the long term than Enzyme. This is the best argument for using RNTL, in my mind.

The topics of full mounting and testing the contract are more debatable. Some would argue that it’s better to shallow mount and to test methods directly. There are pretty strong arguments in favor of testing the contract instead (Dan Abramov and Edd Yerburgh, as cited), so I would recommend doing so unless you’re strongly convinced that you need to test at a lower level. When it comes to full mounting vs. shallow mounting, there are different views. I've just seen full mounting work out better in React Native in particular.

[enzyme]: https://airbnb.io/enzyme/
[rntl]: https://callstack.github.io/react-native-testing-library/
