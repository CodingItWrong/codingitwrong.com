---
title: "Decorating Guide: Commonly-Used Ember Decorators"
tags: [ember]
---

*Updated 8/22: added brief explanations of each decorator.*

`ember-decorators` offers a lot of different decorators to allow you to indicate metadata to Ember about your ES6 classes, properties, and methods. I find myself checking the documentation for these a lot, so I thought I'd combine all the common decorators I use in one place. First is a quick reference of all of their imports for easy copying-and-pasting, and then is an example of usage of each.

## Import Reference

```js
import { tagName } from '@ember-decorators/component';
import { attr, belongsTo, hasMany } from '@ember-decorators/data';
import { action, computed } from '@ember-decorators/object';
import { filterBy, sort } from '@ember-decorators/object/computed';
import { service } from '@ember-decorators/service';
```

## Examples

Note that these decorators are not necessarily *limited* to use with the class types I use in these examples; these were just convenient classes to illustrate them.

- [`@tagName`][tagName] lets you designate the outer wrapping tag for a component, or `''` to not use an outer wrapping tag.
- [`@action`][action] lets you designate a method as an action available to be called from the template.
- [`@computed`][computed] creates a computed property. Note that the property needs to specified as an ES5 getter, with the `get` keyword. You pass the properties the computed property depends on to the decorator.

```js
import Component from '@ember/component';
import { tagName } from '@ember-decorators/component';
import { action, computed } from '@ember-decorators/object';

@tagName('')
export default class LinkRow extends Component {
  editing = false;

  @computed('link.read')
  get showLink() {
    return this.showIfRead === this.link.read;
  }

  @action
  edit() {
    this.set('editing', true);
  }

  @action
  finishEditing() {
    this.set('editing', false);
  }
}
```

- [`@filterBy`][filterBy] filters an array by a property/value pair.
- [`@sort`][sort] allows you to sort an array using an array of fields to sort by, or a predicate function.

```js
import Controller from '@ember/controller';
import { filterBy, sort } from '@ember-decorators/object/computed';

export default class PastReadingsController extends Controller {
  completionOrder = Object.freeze(['completed_at:desc']);

  @filterBy('model', 'complete', true)
  pastReadings;

  @sort('pastReadings', 'completionOrder')
  pastReadingsInCompletionOrder;
}
```

- [`@attr`][attr] allows you to designate Ember Data model attributes, optionally specifying a transformation.
- [`@belongsTo`][belongsTo] and [`@hasMany`][hasMany] allows you to specify Ember Data relationships, passing the name of the related model.

```js
import DS from 'ember-data';
const { Model } = DS;
import { attr, belongsTo, hasMany } from '@ember-decorators/data';

export default class Bookmark extends Model {
  @attr url;
  @attr('date') published_at;

  @belongsTo('user') user;
  @hasMany('tag') tags;
}
```

- [`@service`][service] allows you to inject a service, by default using the name of the property as the service name to look up.

```js
import Route from '@ember/routing/route';
import { service } from '@ember-decorators/service';

export default class IndexRoute extends Route {
  @service session;

  model() {
    if (this.session.isAuthenticated) {
      return this.store.findAll('reading');
    }
  }
}
```

[action]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/object#action
[attr]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/data#attr
[belongsTo]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/data#belongsTo
[computed]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/object#computed
[filterBy]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/object/computed#filterBy
[hasMany]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/data#hasMany
[service]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/service#service
[sort]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/object/computed#sort
[tagName]: https://ember-decorators.github.io/ember-decorators/latest/docs/api/modules/@ember-decorators/component#tagName
