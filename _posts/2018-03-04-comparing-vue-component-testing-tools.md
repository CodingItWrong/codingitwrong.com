---
title: "Comparing Vue Component Testing Tools"
tags: [testing]
---

When you want to test your Vue components, there are a few different tools we can use:

- [Vue’s unit testing guide](https://vuejs.org/v2/guide/unit-testing.html) describes an approach using new Constructor() and $mount()
- [vue-test-utils](https://github.com/vuejs/vue-test-utils) is a first-party testing library that is currently in beta
- The Cypress testing framework has a [Cypress Vue unit test](https://github.com/bahmutov/cypress-vue-unit-test) library

Let’s take a look at each and see how they compare.

## Goals

First we need to talk about what the goals of our testing are. One important principle of testing is to **avoid being coupled to implementation details.** You want to be able to change how your component is implemented. Instead, you want to test the externally-visible behavior.

What’s externally visible in a component? For example, you can provide input to a component by sending or changing props, or by interacting with its DOM elements. The component can provide output by changing its DOM elements, emitting an event, calling a function passed in as a prop, or dispatching a Vuex action.

There are some things I see in Vue testing guides that aren’t externally visible, though: checking data items, calling a component method directly, confirming that a component method is called. Data items and methods are implementation details of components.

With that stated, let’s take a look at our testing options. We’ll look at an example from [my Vue TDD tutorial](http://learntdd.in/vue): a form that allows you to enter a message and send it.

## Built-in Vue Unit Testing

```js
import Vue from 'vue';
import NewMessageForm from '@/components/NewMessageForm';
import { spy } from 'sinon';

describe('NewMessageForm.vue', () => {
  let vm;

  beforeEach(() => {
    const Constructor = Vue.extend(NewMessageForm);
    vm = new Constructor().$mount();
  });

  describe('clicking the save button', (done) => {
    let spy;

    beforeEach(() => {
      spy = sinon.spy();
      vm.$on('save', spy);

      const messageTextField = vm.$el.querySelector("[data-test='messageText']");
      messageTextField.value = 'New message';
      messageTextField.dispatchEvent(new window.Event('input'));

      const saveButton = vm.$el.querySelector("[data-test='saveButton']");
      saveButton.click();
    });

    it('clears the text field', done => {
      Vue.nextTick(() => {
        const messageField = vm.$el.querySelector("[data-test='messageText']");
        expect(messageField.value).to.eq('');
        done();
      });
    });

    it('emits the save event', () => {
      expect(spy).to.have.been.calledWith('New message');
    });
  });
});
```

Interacting with form elements with Vue unit test is a bit cumbersome. Vue instances expose the `$el` property, which points to the root DOM element of the component. This allows us to programmatically interact with form elements just like we can in basic DOM scripting. For clicks we can just call `click()`, but entering text is harder:

```js
let messageField = vm.$el.querySelector("[data-test='messageText']");
messageField.value = 'New message';
messageField.dispatchEvent(new window.Event('input'));

// use Vue.nextTick() as necessary
```

There are a few things that make this complex:

- Know to use an "input" event instead of a "change" event (which is more typical in DOM scripting)
- Set the `value` property of the element separately from issuing the event
- Create a new "input" Event and pass it to `dispatchEvent()`, rather than calling an event-specific method directly

I couldn't find a reference to any of this in the Vue unit test docs. It's likely that it's obvious to someone more experienced in front end development than me, but a big part of Vue's appeal is to newer frontend developers, and this won't help them.

Also, because Vue unit tests don’t handle asynchrony for us, we have to know to manually call `Vue.nextTick()` to wait for the DOM to be updated.

## vue-test-utils

```js
import Vue from 'vue';
import { mount } from '@vue/test-utils';
import NewMessageForm from '@/components/NewMessageForm';

describe('NewMessageForm.vue via vue-test-utils', () => {
  let wrapper;

  beforeEach(() => {
    wrapper = mount(NewMessageForm);
  });

  describe('clicking the save button', (done) => {
    beforeEach(() => {
      const messageField = wrapper.find("[data-test='messageText']");
      messageField.element.value = 'New message';
      messageField.trigger('input');

      wrapper.find("[data-test='saveButton']").trigger('click');
    });

    it('clears the text field', () => {
      const messageField = wrapper.find("[data-test='messageText']");
      expect(messageField.element.value).to.eq('');
    });

    it('emits the save event', () => {
      expect(wrapper.emitted().save.length).to.eq(1);
    });
  });
});
```

Issuing events is a bit simpler in Vue Test Utils: it handles asynchrony for us, so we don’t need to manually call anything. And `trigger()` is a nice consistent interface to trigger any kind of event.

```js
let messageField = wrapper.find("[data-test='messageText']");
messageField.element.value = 'New message';
messageField.trigger('input');
```

But setting the value on the text field is actually a bit *more* indirect. We have to know to retrieve the element from the wrapper.

Another thing that’s nice about Vue Test Utils is the `emitted()` method that exposes events emitted by the component. That makes it clear how you can test events, and encourages that kind of testing:

```
expect(wrapper.emitted().save.length).to.eq(1);
```

## Cypress

```js
import mountVue from 'cypress-vue-unit-test';
import NewMessageForm from '../../src/components/NewMessageForm.vue';

describe('NewMessageForm', () => {
  beforeEach(mountVue(NewMessageForm));

  describe('clicking the save button', () => {
    let spy;

    beforeEach(() => {
      spy = cy.spy();
      Cypress.vue.$on('save', spy);

      cy.get("[data-test='messageText']")
        .type('New message');

      cy.get("[data-test='saveButton']")
        .click();
    });

    it('clears the text field', () => {
      cy.get("[data-test='messageText']")
        .should('have.value', '');
    });

    it('emits the "save" event', () => {
      expect(spy).to.have.been.calledWith('New message');
    });
  });
});
```

### Entering Text

In a sense, Cypress component tests are a higher level of abstraction. You aren’t just calling methods on component objects; you can also interact with the UI in the same way you do in end-to-end tests:

```js
cy.get("[data-test='messageText']")
  .type('New message');
```

This is convenient but it may raise questions in your head: aren’t unit tests supposed to be lower level? This is why I would argue for using the term "component test" rather than "unit test." In unit tests we tend to make function calls. But function calls aren’t how we use components; we interact with them in the UI. Therefore that’s the right level of abstraction for testing components. We can see evidence for this in the fact that we don’t easily know how to call the functions to test components; that’s not how we use them.

## Why does it matter?

All of these approaches will work to test components, and if you're on a project that's already chosen a component test appraoch, you don't need to change right away. But what *is* the value of picking a cleaner approach to component testing?

- **Friction:** we developers are human, and it’s hard to get motivated to do something that’s tedious. The more work it is to test the external interface of our components, the more we’ll be tempted to write easier tests of component internals—or not test at all.
- **Reliability of tests:** when you have several lines of code to write to execute one conceptual step, that’s more places for errors to creep in. Errors in tests can manifest as mysterious failures or (maybe worse) tests that pass even though the functionality is broken.
- **Test-Driven Development:** because of the friction and unreliability of these test approaches, if you do write this style of component test at all you’ll be tempted to do it after the component is already written, so you can focus on getting the tests right. But if you do this you’ll miss out on the benefits of test-driven development.
