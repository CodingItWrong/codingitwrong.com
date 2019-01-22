---
title: Testing with Third Party Form Components
tags: [testing, javascript]
---

I got a great question via e-mail about how to test your app when you're using a third-party form component library. In my tutorials I recommend putting a `data-test` attribute on your `<input>` elements to select them by:

```html
<input data-test="email" />
```

In Cypress, for example, you can select this element like so:

```js
cy.get("[data-test='email']").type("example@example.com");
```

But what do you do when you don't create the `<input>` elements yourself--a third party library does? Depending on the library you're using, you can get an error like the following:

```sh
CypressError: cy.type() failed because it requires a valid typeable element
```

Cypress found the element that was being selected, but apparently it's not an element that can be typed into. What's going on here?

The key to testing these form controls is to dive into the details of the HTML elements the third-party form component is creating. Cypress doesn't know about your components; it only knows about the DOM elements that are created.

Let's look at how a few different Material Design libraries render and how we can test each.

## Vuetify

In Vuetify, for Vue.js, a text field component is written like this:

```html
<v-text-field
  id="myfield"
  class="myfield"
  data-test="myfield"
/>
```

Note that I've added an ID, a class, and a `data-test` attribute. Although I recommend selecting elements via `data-test` attributes, I want to demo IDs and classes as well because your existing tests may already be using IDs or classes, and component libraries can handle them differently.

Here are the DOM elements created by this component:

```html
<div class="v-input myfield v-text-field v-text-field--placeholder theme--light">
  <div class="v-input__control">
    <div class="v-input__slot">
      <div class="v-text-field__slot">
        <input id="myfield" data-test="myfield" type="text">
      </div>
    </div>
  </div>
</div>
```

Notice that the `id` and `data-test` attribute are placed on the `<input>` element itself, so they can be used to get the `<input>` directly:

```js
cy.get("#myfield").type("By ID");
cy.get("[data-test='myfield']").type("By data attribute");
```

By contrast, the `class` is applied to the outermost `<div>`. This makes sense from a styling standpoint, but if we try to select the element with that class:

```js
cy.get(".myfield").type("By class");
```

Cypress gives us the error I mentioned earlier:

```sh
CypressError: cy.type() failed because it requires a valid typeable element
```

This is because `.myfield` is the `div`, not the `input`, and you can't type into a `div`. Instead, we need to ask for the `input` element *inside* the element with class `.myfield`:

```js
cy.get(".myfield input").type("By class and input element");
```

This works. Unfortunately, this means *your test is coupled to implementation details of a third-party component library*. If Vuetify is updated to apply the class to the `input` instead of the containing `div`, your tests will break. Luckily, if you use the `data-test` attribute I recommend it's applied to the `input` directly, so your tests avoid coupling to the third-party library.

## Material-UI

Let's compare this to one of the popular Material Design libraries for React: Material-UI. Here's how you create a text field in Material-UI:

```html
<TextField
  id="myfield"
  class="myfield"
  data-test="myfield"
/>
```

And here's the rendered DOM:

```html
<div class="myfield" data-test="myfield">
  <div class="MuiInputBase-root-18 MuiInput-root-5 MuiInput-underline-9 MuiInputBase-formControl-19 MuiInput-formControl-6">
    <input id="myfield" type="text" value="" aria-invalid="false" class="MuiInputBase-input-28 MuiInput-input-13">
  </div>
</div>
```

As before, the `id` is applied to the `input`, and the `class` is applied to the containing `div`. The `data-test` attribute is different, though: whereas with Vuetify it was applied to the `input`, in Material-UI the `data-test` attribute is applied to the containing `div`.

This means that for Material-UI it's only the ID field that you can select directly:

```js
cy.get("#myfield").type("By ID");
```

For either `data-test` attributes or classes, you need to explicitly select the containing `input`:

```js
cy.get("[data-test='myfield'] input").type("By data attribute");
cy.get(".myfield input").type("By class and input element");
```

## Ember-Paper

To round out our comparison, let's look at Ember-Paper, a Material Design library for Ember.js:

```html
{% raw %}<Form.input
  data-test-myfield
  id="myfield"
  class="myfield"
  @type="text"
  @value={{myfield}}
  @onChange={{action (mut myField)}}
/>{% endraw %}
```

(Note that the form of the `data` attribute is slightly different, because of how the `ember-test-selectors` library sets up `data` attributes.)

Here's the DOM it renders:

```html
<md-input-container id="myfield" data-test-myfield="" class="md-default-theme ember-view myfield">
  <input class="md-input   ng-dirty" id="input-ember206" aria-describedby="ember206-char-count ember206-error-messages" type="text">
</md-input-container>
```

(Yes, that's literally a DOM element called `md-input-container`. Weird, huh? Your guess is as good as mine!)

Anyways, the way attributes are handled is consistent, but not in the direction we'd prefer: the `id`, class, and `data-test` attribute are all applied to the containing element, not to the `input`. So no matter which we use, we need to select the `input` explicitly:

```js
await fillIn('#myfield input', 'By ID');
await fillIn('.myfield input', 'By Class');
await fillIn('[data-test-myfield] input', 'By Data Attribute')
```

## The General Case

There are dozens and dozens of different UI component libraries, so you may need to do your own research to get the tests working in your app. The key is to look into how the third-party components render to the DOM. There's nothing standardized about how attributes are applied: the components have full control over what they render. So you'll need to see where they put `data-test` attributes, or whatever alternate attribute you use for selecting elements.

I think it's ideal for `data-test` attributes to be applied to the `input` themselves, so if your UI library doesn't do it that way, it could be worth you opening an issue requesting a change--or, even better, a pull request to add that functionality!
