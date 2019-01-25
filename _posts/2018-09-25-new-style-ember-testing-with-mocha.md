---
title: New-Style Ember Testing with Mocha
tags: [ember, testing]
---

*Updated 2019-01-25: updated to use `ember-mocha` directly.*

Here's a walkthrough of setting up my preferred Ember testing stack, getting all the following tools to play nicely together:

- [New-style Ember tests][new-ember-tests] with async/await and no jQuery
- [Mocha][mocha] and [Chai][chai], with [chai-dom][chai-dom] for nicer assertions
- [Sinon][sinon] for test doubles, with [sinon-chai] for nicer assertions
- [Mirage][mirage] for mocking the backend

To start out, since Mocha will take the place of QUnit, we want to remove QUnit-related packages first:

```sh
$ yarn remove ember-cli-qunit ember-qunit qunit-dom
```

(You might only have `ember-cli-qunit` or only `ember-qunit`; the above command should cover either case.)

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

Next, we install the `ember-mocha` addon:

```sh
$ ember install ember-mocha
```

We also want to add `ember-cli-chai` to get access to Chai, an assertion framework:

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
import { setupRenderingTest } from 'ember-mocha';
import { render } from '@ember/test-helpers';
import hbs from 'htmlbars-inline-precompile';

describe('Integration | Component | my-component', function() {
  setupRenderingTest();

  it('renders', async function() {
    // Set any properties with this.set('myProperty', 'value');
    // Handle any actions with this.set('myAction', function(val) { ... });

    await render(hbs`{{my-component}}`);

    expect(this.element.textContent.trim()).to.equal('');

    // Template block usage:
    await render(hbs`
      {{#my-component}}
        template block text
      {{/my-component}}
    `);

    expect(this.element.textContent.trim()).to.equal('template block text');
  });
});{% endraw %}
```

This is using Mocha and Chai, and it passes.

Let's customize our component and add a bit more substantial of an assertion to test it. First, let's just drop a "Hello, world!" message into the component's template:

```html
<div class="welcome">Hello, world!</div>
```

Now, make the following changes to the component test:

```diff
{% raw %} import { expect } from 'chai';
 import { describe, it } from 'mocha';
 import { setupRenderingTest } from 'ember-mocha';
-import { render } from '@ember/test-helpers';
+import { render, find } from '@ember/test-helpers';
 import hbs from 'htmlbars-inline-precompile';

 describe('Integration | Component | my-component', fu
nction() {
   setupRenderingTest();


   it('renders', async function() {
-    // Set any properties with this.set('myProperty', 'value');
-    // Handle any actions with this.set('myAction', function(val) { ... });
-
     await render(hbs`{{my-component}}`);
-
-    expect(this.element.textContent.trim()).to.equal('');
-
-    // Template block usage:
-    await render(hbs`
-      {{#my-component}}
-        template block text
-      {{/my-component}}
-    `);
-
-    expect(this.element.textContent.trim()).to.equal('template block text');
+    expect(find('.welcome').textContent).to.include('Hello, world!');
   });
 });{% endraw %}
```

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
$ yarn add --dev sinon-chai
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

Next, generate an acceptance test:

```sh
$ ember generate acceptance-test mirage
```

To set up the test to work with Mirage, make the following changes:

```diff
 import { describe, it } from 'mocha';
 import { expect } from 'chai';
 import { setupApplicationTest } from 'ember-mocha';
-import { visit, currentURL } from '@ember/test-helpers';
+import { visit, find } from '@ember/test-helpers';
+import setupMirage from 'ember-cli-mirage/test-support/setup-mirage';

 describe('Acceptance | mirage', function() {
-  setupApplicationTest();
+  let hooks = setupApplicationTest();
+  setupMirage(hooks);
```

Notice how we save the return value of `setupApplicationTest()` to a variable, then pass it as an argument to `setupMirage()`.

Then replace the existing `it()` with the following:

```js
it('allows creating models', async function() {
  let widget = server.create('widget', {
    name: 'Awesome Widget',
  });

  await visit('/');

  let widgets = find('.widget');
  expect(widgets).to.contain.text(widget.name);
});
```
Finally, set up the Mirage route to serve the widget endpoint. Add the following to `mirage/config.js`:

```diff
 export default function() {
+  this.get('/widgets');
 }
```

Run `ember test` again and your acceptance test should pass.

With that, all our testing tools are set up and working together: Mocha, Chai, Sinon, Mirage, and Ember's new testing APIs. Give them a try and see what you think!

[chai]: https://www.chaijs.com
[chai-dom]: https://github.com/nathanboktae/chai-dom
[mirage]: https://www.ember-cli-mirage.com
[mocha]: https://mochajs.org
[new-ember-tests]: https://youtu.be/8D-O4cSteRk
[sinon]: https://sinonjs.org
[sinon-chai]: https://github.com/domenic/sinon-chai
