---
title: "vuex-jsonapi: A Zero-Config Data Layer"
tags: [apis]
---

Do you love writing similar web service client code over and over again for all the endpoints you access? Yeah me either. How many times have you written a Vuex module or a Redux reducer that looks more or less like this? (Warning: long code sample because that is the point!)

```js
const state = {
  data: [],
  ready: false,
  error: false,
};

const mutations = {
  SET_DATA(state, data) {
    this.data = data;
  },

  SET_READY(state, ready) {
    this.ready = ready;
  },

  SET_ERROR(state, error) {
    this.error = error;
  },
};

const actions = {
  loadData({ commit }, params) {
    commit('SET_READY', false);
    commit('SET_ERROR', false);

    return api.fetch({
      url: 'https://api.example.com/path/to/resource',
      params,
    }).then(response => {
      commit('SET_DATA', response.data);
      commit('SET_READY', true);
      commit('SET_ERROR', false);
    }).catch(error => {
      commit('SET_READY', true);
      commit('SET_ERROR', true);
    });
  }
};
```

I can understand the value of explicitness, but a structure like this is boilerplate in the worst sense of the term: repetitive code that provides little business value, and is prone to errors when a bug is fixed but not in all copies of the code.

Don’t you wish there was a way you could just skip all of that? It’s possible, and I’ve seen it work: in [Ember Data][ember-data], the built-in data layer for the Ember.js framework. For the kind of apps I usually write that are backed by app-specific web services, it doesn’t get any simpler than Ember Data. By default, Ember Data uses the [JSON API][jsonapi] format to communicate with web services. By assuming the server will follow JSON API's conventions, Ember Data is able to handle all the URLs and response parsing automatically. All you have to do is:

**1. Configure the base URL to connect to**

```js
// adapters/application.js
export default DS.JSONAPIAdapter.extend({
  host: 'https://api.example.com',
});
```

**2. Define a model and its fields**

```js
// models/widget.js
export default DS.Model.extend({
  title: DS.attr(),
  description: DS.attr(),
});
```

**3. Use it**

```js
const modelPromise = this.store.findAll('widget');
```

What if we could have an effectively "zero-configuration" data layer like that in Vue? What if we could set up a Vuex module to access a resource as simply as this:

```js
new Vuex.Store({
  modules: {
    widgets: resourceModule('widgets', httpClient),
  },
});
```

I've started working on an experiment to try to create such an interface, and the result is [`vuex-jsonapi`][vuex-jsonapi]. Currently it's not production-ready, but it's promising. I was able to get many of the basic features I need in just 200 lines of code. Let's take a look at how it's used.

You may be wondering why I'm looking into JSON API when GraphQL has some similar concerns. I’ve only done basic research into GraphQL so far. One of the first things I noticed was the requirement to explicitly declare data types, as well as which fields to be returned in a given query. I can see how that’s helpful in many circumstances. But in my use case here, I prefer to have my fields automatically available in whatever formats they’re returned in. If you have thoughts on how close GraphQL tools can get to a zero-config experience like this, let me know!

# The Architecture: Identity Map

The Vuex module for a resource implements an [identity map][identity-map]: it stores a single instance of each record in memory, indexed by ID field. There are a few advantages to this approach.

First, an identity map keeps your data consistent. You won't run into a situation where a list screen has an old copy of a record and a detail screen has a newer copy of a record. Because only a single instance of each record is stored in memory, when it's updated in one place it's updated everywhere. Combined with Vue's reactivity, this means that other parts of your UI will be updated automatically.

Second, an identity map can prevent unnecessary network traffic. If you already have a record in memory, you have the option of using that version instead of going back to the network to request it again. You can make a network request when it's important to have the freshest data, but otherwise you can use the copy from the identity map.

The idea of controlling when you make network requests is supported by Vuex’s separation between actions and getters. Actions are used to make requests to the server to load data or to make changes. The downloaded and changed data is stored in the Vuex state, and is accessed by the app using getters.

# Design Goals

From a design standpoint, for the sake of keeping this experiment simple, I decided not to try to hide the structure of JSON API records from the rest of the app. For example, the attributes of the records are available under the `attributes` property as in the JSON API spec, rather than directly on the record object. This ended up having a nice benefit in terms of being able to save relationship data, as we’ll see later. If I continue to develop `vuex-jsonapi` I might eventually change it to expose data in a format that hides the details of JSON API.

# Setup

As I mentioned above, setting up a Vuex module to correspond to a resource is as simple as:

```js
new Vuex.Store({
  modules: {
    widgets: resourceModule({ name: 'widgets', httpClient }),
  },
});
```

`httpClient` needs to be an object matching the basic interface to the popular [Axios][axios] HTTP client, so you can just pass in an Axios instance configured with any headers and authorization you need.

Since it's likely you'll be setting up modules for several resources, there's a function for that too:

```js
new Vuex.Store({
  modules: {
    ...mapResourceModules({
      names: ['widgets', 'categories'],
      httpClient,
    }),
  },
});
```

# Reading Data

In the simplest case, requesting all records, you dispatch the module's `loadAll` action and then access the response via its `all` getter:

```js
this.$store.dispatch('widgets/loadAll')
  .then(() => {
    const widgets = this.$store.getters['widgets/all'];
    console.log(widgets);
  });
```

If you're accessing these from within a Vue component, you can use Vuex's `mapActions` and `mapGetters` as usual:

```js
import { mapActions, mapGetters } from 'vuex';

export default {
  // ...
  methods: {
    ...mapActions({
      loadWidgets: 'widgets/loadAll',
    }),
  },
  computed: {
    ...mapGetters({
      widgets: 'widgets/all',
    }),
  },
  // ...
};
```

The `loadAll` action will send a request to the server in the following format:

```
GET https://api.example.com/widgets
```

How about when you want to display a single record? Well, if you know the record is already in the Vuex store, you can simply use the find getter:

```js
const widget = this.$store.getters['widgets/byId']({ id: 42 });
console.log(widget);
```

If the record is not yet in the store, you can request a single record with the `loadById` action, then use the `byId` getter:

```js
this.$store.dispatch('widgets/loadById', { id: 42 })
  .then(() => {
    const widget = this.$store.getters['widgets/byId']({ id: 42 });
    console.log(widget);
  });
```

The `loadById` action will send the following request:

```
GET https://api.example.com/widgets/42
```

Even if you do already have a local copy of the record, you can still dispatch the `loadById` action to ensure you have the latest data. Jake Archibald calls this [the “cache then network” strategy][cache-then-network]. Incidentally, this is the default approach Ember Data takes for most of its requests: if you have cached data then go ahead and display it, but always make a request to the server to get fresher data, and update it reactively when it is available.

If you want to send filter parameters in the server request, use the `loadWhere` action. Provide it an object with properties and values; they will be set as filter properties on the request.

```js
const filter = {
  category: 'whizbang',
};
this.$store.dispatch('widgets/loadWhere', { filter });
```

This will make a request with the filter options passed in the standard JSON API way:

```
GET https://api.example.com/widgets?filter[category]=whizbang
```

To retrieve the matches results, pass the same filter criteria to the where getter:

```js
const filter = {
  category: 'whizbang',
};
this.$store.dispatch('widgets/loadWhere', { filter });
  .then(() => {
    const widgets = this.$store.getters['widgets/where']({ filter });
    console.log(widgets);
  });
```

This doesn’t perform any filtering logic on the client side; it simply keeps track of which IDs were returned by the server side request and retrieves those records.

The last way to query is by retrieving records related to another record. You can load the records by dispatching the `loadRelated` action, passing in the parent record (or, really, just the type and ID is fine). Once the records are downloaded locally, we retrieve them passing the parent record to the related getter:

```js
const parent = {
  type: 'categories',
  id: 27,
};

this.$store.dispatch('widgets/loadRelated', { parent })
  .then(() => {
    const widgets = this.$store.getters['widgets/related']({ parent });
    console.log(widgets);
  });
```

Related records can be retrieved in a few different ways. In our case, we access them through a nested resource:

```
GET https://api.example.com/categories/27/widgets
```

In this case, the name of the relationship on `categories` is the same as the name of the other model: `widgets`. In cases where the names are not the same, you can explicitly pass the relationship name:

```js
const params = {
  parent: {
    type: 'categories',
    id: 27,
  },
  relationship: 'purchased-widgets',
};

this.$store.dispatch('widgets/loadRelated', params)
  .then(() => {
    const widgets = this.$store.getters['widgets/related'](params);
    console.log(widgets);
  });
```

# Writing Data

For create, update, and delete operations, there are corresponding actions you can dispatch.

To create a record, you don’t need to pass a full JSON API record. You don’t need the type field because the module knows what type it corresponds to. And you won’t yet have an ID field if you’re letting the server generate IDs. Just pass an object with an attributes property:

```js
const recordData = {
  attributes: {
    title: 'My Widget',
  },
};
this.$store.dispatch('widgets/create', recordData);
```

This sends the following POST request:

```
POST https://api.example.com/widgets
```

```json
{
  "type": "widgets",
  "attributes": {
    "title": "My Widget"
  }
}
```

Why have an `attributes` property instead of just passing the attributes directly? It’s so you can pass a relationships property as well:

```js
const recordData = {
  attributes: {
    title: 'My Widget',
  },
  relationships: {
    category: {
      data: {
        type: 'categories',
        id: 42,
      },
    },
  },
};
this.$store.dispatch('widgets/create', recordData);
```

The `update` action takes a complete JSON API record; just update the attributes and relationships in place.

```js
const widget = this.$store.getters['widgets/byId']({ id: 42 });
widget.attributes.title = 'Updated Title';
this.$store.dispatch('widgets/update', widget);
```

Note: if you modify a record, keep in mind that `vuex-jsonapi` doesn't yet perform defensive copies of the values it returns, so that record will be updated in the store whether or not you dispatch an `update` operation.

Dispatching `update` sends the following PATCH request:

```
PATCH https://api.example.com/widgets/42
```

```json
{
  "type": "widgets",
  "id": "42",
  "attributes": {
    "title": "Updated Title"
  }
}
```

The `delete` action only requires an object with an id property, so you can construct it yourself, or pass in a complete record you've retrieved.

```js
const widgetIdObject = { id: 42 };
this.$store.dispatch('widgets/delete', widget);
```

As you might expect, this sends a DELETE request:

```
DELETE https://api.example.com/widgets/42
```

# Results

So, how did the experiment turn out? I find it really nice to be able to configure Vuex modules for different JSON API resources with a single line. Vuex’s architecture introduces a separation between loading remote records into the local store (actions) and retrieving them from that store (getters). This separation results in duplication in terms of passing the same arguments to the action and the getter; this is duplication you don't get in a framework like Ember Data where your data is accessed directly in the resolved Promise. But this separation also ends up having some benefits in terms of making your data loading strategy more visible. I don’t mind this so much: it’s still raising the level of abstraction over the details of constructing requests and navigating responses.

Overall, the results of this experiment are very promising for the possibility of working with RESTful APIs in Vuex in a more convention-based way. I’m hoping to write a Vue frontend for a side project of mine soon to really put `vuex-jsonapi` through its paces. If you want to give it a try, let me know what features you'd like me to add, or submit a PR!

# Prior Art

There are a few other JSON API clients out there for Vuex and Redux; let's compare them.

- [redux-json-api](https://github.com/redux-json-api/redux-json-api/blob/master/docs/api.md#readendpoint-endpoint-string--promise) - a similar approach. I would give it a try in a Redux context.
- [redux-bees](https://github.com/cantierecreativo/redux-bees#defining-your-api-endpoints) - requires defining individual endpoints for get/create/update/destroy/etc. operations, instead of creating them for you following conventions
- [vue-api-query](https://github.com/robsontenorio/vue-api-query) - handles making API requests but not storing them in Vuex

[axios]: https://github.com/axios/axios
[cache-then-network]: https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/#cache-then-network
[ember-data]: https://guides.emberjs.com/release/models/
[jsonapi]: http://jsonapi.org/
[identity-map]: https://martinfowler.com/eaaCatalog/identityMap.html
[vuex-jsonapi]: https://github.com/CodingItWrong/vuex-jsonapi
