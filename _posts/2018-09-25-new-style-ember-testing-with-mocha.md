---
title: New-Style Ember Testing with Mocha
tags: [ember, testing]
---

Here's a walkthrough of setting up my preferred Ember testing stack, getting all the following tools to play nicely together:

- [New-style Ember tests][new-ember-tests] with async/await and no jQuery
- [Mocha][mocha] and [Chai][chai], with [chai-dom][chai-dom] for nicer assertions
- [Sinon][sinon] for test doubles, with [sinon-chai] for nicer assertions
- [Mirage][mirage] for mocking the backend

To start out, since Mocha will take the place of QUnit, we want to remove QUnit-related packages first:

```sh
$ yarn remove ember-cli-qunit qunit-dom
```

We also want to remove some QUnit-specific code from our `tests/test-helper.js` file:

```diff
 import Application from '../app';
 import config from '../config/environment';
 import { setApplication } from '@ember/test-helpers';
-import { start } from 'ember-qunit';

 setApplication(Application.create(config.APP));
-
-start();
```

Next, we install the `ember-cli-mocha` addon. When prompted about overwriting `tests/test-helper.js`, say no. (If you overwrite it, old-style tests will still work, but new-style tests will give you the error "owner.visit is not a function".)

```sh
$ ember install ember-cli-mocha
Yarn: Installed ember-cli-mocha
installing ember-cli-mocha
? Overwrite tests/test-helper.js? No, skip
  create tests/helpers/destroy-app.js
  create tests/helpers/resolver.js
  create tests/helpers/start-app.js
  overwrite tests/test-helper.js
  install package ember-cli-chai
Yarn: Installed ember-cli-chai@^0.4.0
  remove Skipping uninstall because no matching package is installed.
Installed addon package.
```

Then, we want to update to a newer version of `ember-mocha` that what was installed, to enable the new testing API:

```sh
$ yarn add --dev ember-mocha@^0.14.0
```

There's also a newer version of `ember-cli-chai`, and it'll be necessary for us to add `sinon-chai` later. Even if you're not using `sinon-chai`, it's always a good idea to be on the latest version.

```sh
$ ember install ember-cli-chai
```

Now Mocha and Chai should be ready. To see them in action, let's generate a component test:

```sh
$ ember generate component my-component
```

You'll see the following test generated:

```js
{% raw %}import { expect } from 'chai';
import { describe, it } from 'mocha';
import { setupComponentTest } from 'ember-mocha';
import hbs from 'htmlbars-inline-precompile';

describe('Integration | Component | my-component', function() {
  setupComponentTest('my-component', {
    integration: true
  });

  it('renders', function() {
    // Set any properties with this.set('myProperty', 'value');
    // Handle any actions with this.on('myAction', function(val) { ... });
    // Template block usage:
    // this.render(hbs`
    //   {{#my-component}}
    //     template content
    //   {{/my-component}}
    // `);

    this.render(hbs`{{my-component}}`);
    expect(this.$()).to.have.length(1);
  });
});{% endraw %}
```

This is using Mocha and Chai, and it passes. But it's using an older style of test, where `render()` is called on the `this` context, and elements are retrieved using jQuery.

Let's customize our component and then update to the new-style of rendering test. First, let's just drop a "Hello, world!" message into the component's template:

```hbs
<div class="welcome">Hello, world!</div>
```

Now, make the following changes to the component test:

```diff
{% raw %} import { expect } from 'chai';
 import { describe, it } from 'mocha';
-import { setupComponentTest } from 'ember-mocha';
+import { setupRenderingTest } from 'ember-mocha';
+import { render, find } from '@ember/test-helpers';
 import hbs from 'htmlbars-inline-precompile';

 describe('Integration | Component | my-component', function() {
-  setupComponentTest('my-component', {
-    integration: true
-  });
+  setupRenderingTest();

-  it('renders', function() {
+  it('renders', async function() {
...
-    this.render(hbs`{{my-component}}`);
-    expect(this.$()).to.have.length(1);
+    await render(hbs`{{my-component}}`);
+    expect(find('.welcome').textContent).to.include('Hello, world!');
   });
 });{% endraw %}
```

Notice the following differences:

- We call `setupRenderingTest()` instead of `setupComponentTest()`.
- The test is an `async` function, and we use an `await` keyword with the `render()` function. This means that the `render()` function returns a promise, and we pause execution of the test until the promise settles.
- `render()` is a function imported from `@ember/test-helpers` instead of a method on the `this` context.
- Instead of using jQuery via `this.$` to access DOM elements, we call the imported `find()` function to do so.

Run the tests with `ember test` and they should pass.

When using Chai with rendering tests, there's an even more readable way to make assertions: `chai-dom`. Let's install it:

```sh
$ yarn add --dev chai-dom
```

Now we can make the following change to our assertion:

```diff
{% raw %}await render(hbs`{{my-component}}`);
-    expect(find('.welcome').textContent).to.include('Hello, world!');
+    expect(find('.welcome')).to.contain.text('Hello, world!');
});{% endraw %}
```

To me, "expect to contain text" seems more readable than "expect its text content to include".

Next let's add Sinon, the popular test double library. It allows you to create functions that stand in for real functions in your tests, for purposes like asserting that a function was called. This is great for testing the "data down, actions up" approach in your components.

Add the Sinon addon:

```sh
$ ember install ember-sinon
```

Now let's add an action to our component:

```diff
 import Component from '@ember/component';

 export default Component.extend({
+  actions: {
+    handleClick() {
+      this.onClick();
+    }
+  }
});
```

Add a button to the component's template to trigger the action:

```hbs
{% raw %}<button onclick={{action "handleClick"}}>Do Action</button>{% endraw %}
```

When the button is clicked, we call the component's `handleClick` action, which calls a passed-in `onClick` function. How can we test this? Add the following to the test file:

```diff
{% raw %} import { setupRenderingTest } from 'ember-mocha';
-import { render, find } from '@ember/test-helpers';
+import { render, find, click } from '@ember/test-helpers';
 import hbs from 'htmlbars-inline-precompile';
+import sinon from 'sinon';
...
     await render(hbs`{{my-component}}`);
     expect(find('.welcome')).to.contain.text('Hello, world!');
   });
+
+  it('calls a passed-in action', async function() {
+    const handleClick = sinon.spy();
+    this.set('handleClick', handleClick);
+
+    await render(hbs`{{my-component onClick=handleClick}}`);
+    await click('button');
+
+    expect(handleClick.called).to.be.true;
+  });
 });{% endraw %}
```

We call `sinon.spy()` to create a spy function, then we pass it in to the component as an argument. We click the button, then we assert that `handleClick`'s `called` property was set to true. As you might expect, Sinon sets it to true when it's called.

We can do even better on the assertions for Sinon, too, though. Add `sinon-chai`:

```sh
$ yarn add sinon-chai
```

Now change the assertion like so:

```diff
     await click('button');

-    expect(handleClick.called).to.be.true;
+    expect(handleClick).to.have.been.called;
   });
```

To me, "expect handleClick to have been called" is more readable than "expect handleClick called to be true".

There's one last addon that no Ember testing setup can do without: Mirage. It provides a fake web service layer allowing you to set up backend requests and responses in tests, or even in development mode if your backend is not yet ready. Let's install it:

```sh
$ ember install ember-cli-mirage
```

Let's set up a simple model so we can test it with Mirage:

```sh
$ ember generate model widget
```

Add a `name` attribute to the widget:

```diff
 import DS from 'ember-data';

 export default DS.Model.extend({
+  name: DS.attr(),
 });
```

Generate an index route:

```sh
$ ember generate route index
```

In `app/routes/index.js`, load the widgets in the model hook:

```diff
 import Route from '@ember/routing/route';

 export default Route.extend({
+  model() {
+    return this.store.findAll('widget');
+  },
 });
```

And replace the route's template `app/templates/index.hbs` with this:

```hbs
{% raw %}<ul class="widgets">
  {{#each model as |widget|}}
    <li class="widget">{{widget.name}}</li>
  {{/each}}
</ul>{% endraw %}
```

For some reason, with this testing setup generating an acceptance test doesn't work for me--no file is created. But this is okay, because the setup of the test needs to be pretty different than the generated file would be anyway. Let's create an acceptance test by hand. Create `tests/acceptance/mirage-test.js` and add the following contents:

```js
import { describe, it } from 'mocha';
import { expect } from 'chai';
import { visit, find } from '@ember/test-helpers';
import { setupApplicationTest } from 'ember-mocha';
import setupMirage from 'ember-cli-mirage/test-support/setup-mirage';

describe('mirage', function() {
  let hooks = setupApplicationTest();
  setupMirage(hooks);

  it('allows creating models', async function() {
    let widget = server.create('widget', {
      name: 'Awesome Widget',
    });

    await visit('/');

    let widgets = find('.widget');
    expect(widgets).to.contain.text(widget.name);
  });
});
```

Notice how `setupMirage()` is called with `setupApplicationTest()`: we save the return value of `setupApplicationTest()` to a variable, then pass it as an argument to `setupMirage()`.

With that, all our testing tools are set up and working together: Mocha, Chai, Sinon, Mirage, and Ember's new testing APIs. Give them a try and see what you think!

[chai]: https://www.chaijs.com
[chai-dom]: https://github.com/nathanboktae/chai-dom
[mirage]: https://www.ember-cli-mirage.com
[mocha]: https://mochajs.org
[new-ember-tests]: https://youtu.be/8D-O4cSteRk
[sinon]: https://sinonjs.org
[sinon-chai]: https://github.com/domenic/sinon-chai
