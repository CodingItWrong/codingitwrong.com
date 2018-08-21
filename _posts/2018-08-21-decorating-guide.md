---
title: "Decorating Guide: Commonly-Used Ember Decorators"
---

`ember-decorators` offers a lot of different decorators to allow you to indicate metadata to Ember about your ES6 classes, properties, and methods. I find myself checking the documentation for these a lot, so I thought I'd combine all the common decorators I use in one place. First is a quick reference of all of their imports for easy copying-and-pasting, and then is an example of usage of each.

### Import Reference

```js
import { tagName } from '@ember-decorators/component';
import { attr, belongsTo, hasMany } from '@ember-decorators/data';
import { action, computed } from '@ember-decorators/object';
import { filterBy, sort } from '@ember-decorators/object/computed';
import { service } from '@ember-decorators/service';
```

### Examples

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
