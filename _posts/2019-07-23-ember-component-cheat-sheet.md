---
title: Ember Component Cheat Sheet
---

In the 3.x series Ember's component model has undergone a number of incremental improvements. When Ember Octane lands the new programming model will be fully available. But until then, if you'd like to know what's available in the version of Ember you're using--or if you'd like to trace the changes--here's a cheat sheet for you.

This post shows a simple component for displaying a form for entering a person's name. It shows most of the basic features of components. It traces each version of the Ember 3.x series where code related to that component changed.

Note that since Ember follows semver and so breaking changes are only made in a major release, none of the old APIs for components have gone away; newer APIs have been added while the old are retained as well. This post shows the latest-and-greatest APIs as they're added.

- [Ember 3.0](#ember-30)
- [Ember 3.1](#ember-31)
- [Ember 3.4](#ember-34)
- [Ember 3.10](#ember-310)
- [Ember 3.11](#ember-311)
- [Ember Octane Preview](#ember-octane-preview)

## Ember 3.0

#### Component Invocation

```hbs
{% raw %}{{new-person-form
  prompt="Enter a new person's name"
  onSave=(action 'handleSave')
  formClasses="my-stylish-form"
}}{% endraw %}
```

#### Component JS File

```js
import Component from '@ember/component';
import { computed } from '@ember/object';
import { inject as service } from '@ember/service';

export default Component.extend({
  store: service(),

  firstName: '',
  lastName: '',

  fullName: computed('firstName', 'lastName', function() {
    return `${this.get('lastName')}, ${this.get('firstName')}`;
  }),

  actions: {
    async save(event) {
      event.preventDefault();

      const person = this.get('store').createRecord('person', {
        firstName: this.get('firstName'),
        lastName: this.get('lastName'),
      });

      this.set('firstName', '');
      this.set('lastName', '');

      await person.save();
      this.onSave();
    },
  },
});
```

#### Component Template

```hbs
{% raw %}<form onsubmit={{action "save"}} class={{formClasses}}>
  <p>{{prompt}}</p>
  {{input type="text" placeholder="First Name" value=firstName}}
  {{input type="text" placeholder="Last Name" value=lastName}}
  <p>{{fullName}}</p>
  <button type="submit">Save</button>
</form>{% endraw %}
```

## Ember 3.1

- ES5 getters
- @args

#### Component Invocation

```hbs
{% raw %}{{new-person-form
  prompt="Enter a new person's name"
  onSave=(action "handleSave")
  formClasses="my-stylish-form"
}}{% endraw %}
```

#### Component JS File

<div class="language-js highlighter-rouge">
<div class="highlight">
<pre class="highlight language-js" data-line="12,19-21"><code class="language-js">import Component from '@ember/component';
import { computed } from '@ember/object';
import { inject as service } from '@ember/service';

export default Component.extend({
  store: service(),

  firstName: '',
  lastName: '',

  fullName: computed('firstName', 'lastName', function() {
    return `${this.lastName}, ${this.firstName}`;
  }),

  actions: {
    async save(event) {
      event.preventDefault();

      const person = this.store.createRecord('person', {
        firstName: this.firstName,
        lastName: this.lastName,
      });

      this.set('firstName', '');
      this.set('lastName', '');

      await person.save();
      this.onSave();
    },
  },
});
</code></pre>
</div>
</div>

#### Component Template

<div class="language-hbs highlighter-rouge">
<div class="highlight">
<pre class="highlight language-hbs" data-line="2"><code class="language-hbs">{% raw %}<form onsubmit={{action "save"}} class={{formClasses}}>
  <p>{{@prompt}}</p>
  {{input type="text" placeholder="First Name" value=firstName}}
  {{input type="text" placeholder="Last Name" value=lastName}}
  <p>{{fullName}}</p>
  <button type="submit">Save</button>
</form>{% endraw %}
</code></pre>
</div>
</div>

## Ember 3.4

- Angle bracket component invocation

#### Component Invocation

<div class="language-hbs highlighter-rouge">
<div class="highlight">
<pre class="highlight language-hbs" data-line="1-5"><code class="language-hbs">{% raw %}<NewPersonForm
  @prompt="Enter a new person's name"
  @onSave={{action "handleSave"}}
  class="my-stylish-form"
/>{% endraw %}
</code></pre>
</div>
</div>

#### Component JS File

```js
import Component from '@ember/component';
import { computed } from '@ember/object';
import { inject as service } from '@ember/service';

export default Component.extend({
  store: service(),

  firstName: '',
  lastName: '',

  fullName: computed('firstName', 'lastName', function() {
    return `${this.lastName}, ${this.firstName}`;
  }),

  actions: {
    async save(event) {
      event.preventDefault();

      const person = this.store.createRecord('person', {
        firstName: this.firstName,
        lastName: this.lastName,
      });

      this.set('firstName', '');
      this.set('lastName', '');

      await person.save();
      this.onSave();
    },
  },
});
```

#### Component Template

```hbs
{% raw %}<form onsubmit={{action "save"}} ...attributes>
  <p>{{@prompt}}</p>
  {{input type="text" placeholder="First Name" value=firstName}}
  {{input type="text" placeholder="Last Name" value=lastName}}
  <p>{{fullName}}</p>
  <button type="submit">Save</button>
</form>{% endraw %}
```

## Ember 3.10

- Decorators, enabling class components

Note: generators do not create components as ES6 classes by default; they continue to create them with `Component.extend()`. But this guide illustrates the new class syntax that's available.

#### Component Invocation

```hbs
{% raw %}<NewPersonForm
  @prompt="Enter a new person's name"
  @onSave={{action this.handleSave}}
  class="my-stylish-form"
/>{% endraw %}
```

#### Component JS File

<div class="language-js highlighter-rouge">
<div class="highlight">
<pre class="highlight language-js" data-line="1-31"><code class="language-js">import Component from '@ember/component';
import { computed, action } from '@ember/object';
import { inject as service } from '@ember/service';

export default class NewPersonFormClass extends Component {
  @service store;

  firstName = '';
  lastName = '';

  @computed('firstName', 'lastName')
  get fullName() {
    return `${this.lastName}, ${this.firstName}`;
  }

  @action
  async save(event) {
    event.preventDefault();

    const person = this.store.createRecord('person', {
      firstName: this.firstName,
      lastName: this.lastName,
    });

    this.set('firstName', '');
    this.set('lastName', '');

    await person.save();
    this.onSave();
  }
}
</code></pre>
</div>
</div>

#### Component Template

```hbs
{% raw %}<form onsubmit={{action this.save}} ...attributes>
  <p>{{@prompt}}</p>
  <Input type="text" placeholder="First Name" @value={{this.firstName}} />
  <Input type="text" placeholder="Last Name" @value={{this.lastName}} />
  <p>{{this.fullName}}</p>
  <button type="submit">Save</button>
</form>{% endraw %}
```

## Ember 3.11

- on and fn helpers

#### Component Invocation

<div class="language-hbs highlighter-rouge">
<div class="highlight">
<pre class="highlight language-hbs" data-line="3"><code class="language-hbs">{% raw %}<NewPersonForm
  @prompt="Enter a new person's name"
  @onSave={{fn this.handleSave}}
  class="my-stylish-form"
/>{% endraw %}
</code></pre>
</div>
</div>

#### Component JS File

```js
import Component from '@ember/component';
import { computed, action } from '@ember/object';
import { inject as service } from '@ember/service';

export default class NewPersonFormClass extends Component {
  @service store;

  firstName = '';
  lastName = '';

  @computed('firstName', 'lastName')
  get fullName() {
    return `${this.lastName}, ${this.firstName}`;
  }

  @action
  async save(event) {
    event.preventDefault();

    const person = this.store.createRecord('person', {
      firstName: this.firstName,
      lastName: this.lastName,
    });

    this.set('firstName', '');
    this.set('lastName', '');

    await person.save();
    this.onSave();
  }
}
```

#### Component Template

<div class="language-hbs highlighter-rouge">
<div class="highlight">
<pre class="highlight language-hbs" data-line="1"><code class="language-hbs">{% raw %}<form {{on "submit" this.save}} ...attributes>
  <p>{{@prompt}}</p>
  <Input type="text" placeholder="First Name" @value={{this.firstName}} />
  <Input type="text" placeholder="Last Name" @value={{this.lastName}} />
  <p>{{this.fullName}}</p>
  <button type="submit">Save</button>
</form>{% endraw %}
</code></pre>
</div>
</div>

## Ember Octane Preview

- Glimmer Components (this.args, @tracked)

#### Component Invocation

```hbs
{% raw %}<NewPersonForm
  @prompt="Enter a new person's name"
  @onSave={{fn this.handleSave}}
  class="my-stylish-form"
/>{% endraw %}
```

#### Component JS File

<div class="language-js highlighter-rouge">
<div class="highlight">
<pre class="highlight language-js" data-line="1-2,9-10,12,26,28-29"><code class="language-js">import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { action } from '@ember/object';
import { inject as service } from '@ember/service';

export default class NewPersonForm extends Component {
  @service store;

  @tracked firstName = '';
  @tracked lastName = '';

  get fullName() {
    return `${this.lastName}, ${this.firstName}`;
  }

  @action
  async save(event) {
    event.preventDefault();

    const person = this.store.createRecord('person', {
      firstName: this.firstName,
      lastName: this.lastName,
    });

    await person.save();
    this.args.onSave();

    this.firstName = '';
    this.lastName = '';
  }
}
</code></pre>
</div>
</div>

#### Component Template

```hbs
{% raw %}<form {{on "submit" this.save}} ...attributes>
  <p>{{@prompt}}</p>
  <Input type="text" placeholder="First Name" @value={{this.firstName}} />
  <Input type="text" placeholder="Last Name" @value={{this.lastName}} />
  <p>{{this.fullName}}</p>
  <button type="submit">Save</button>
</form>{% endraw %}
```
