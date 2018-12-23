---
title: Modern Ember
tags: [ember]
---

*Updated 2018-12-13: updated to use ember-cli-create, and reference Octane.*

Ember has been undergoing a lot of development in the past year to add features to make it easier to understand and allow it to take advantage of emerging JS ecosystem conventions. This includes:

- A filesystem layout more conducive to component-based development
- Angle bracket syntax for components
- Easy ES6 imports of NPM modules
- Decorators for clear and expressive component implementations

Some of these features are already available in the stable release of Ember, and others will land as part of the upcoming [Ember Octane](https://github.com/emberjs/rfcs/blob/26c4d83fb66568e1087a05818fb39a307ebf8da8/text/0000-roadmap-2018.md#ember-octane) edition, planned to be released at EmberConf 2019. But we can get a preview of these features today! As part of the [#EmberJS2018](https://www.emberjs.com/blog/2018/05/02/ember-2018-roadmap-call-for-posts.html) initiative, I wanted to share a look at what it's like to develop in Ember with all of these modern features in place. You can also [download the completed project](https://github.com/CodingItWrong/modern-ember) if you like.

(You might also be interested in TypeScript; if so, check out [ember-cli-typescript](https://github.com/typed-ember/ember-cli-typescript) to see how easy it is to use with Ember.)

## Project Setup

To get the latest Ember features, we can use `ember-cli-create` to create an "Octane" project, using features from the upcoming edition of Ember. First, install `ember-cli-create` version 0.0.4 or above:

```sh
$ npm install -g ember-cli-create
```

Then, create a new project:

```sh
$ ember-cli-create modern-ember
```

You'll be presented with a menu:

```sh
? What kind of ember project you want to create? (Use arrow keys)
❯ Beginner
  -> If you are new to ember, use this!
  Ember App
  -> Regular ember app wo/ welcome page
  Ember Addon
  -> Regular ember addon
  Octane
  -> MU, Decorators, Sparkles, ...
  Octane + TS
  -> Fuel up your Octane with Type-Power!
  Glimmer
  -> Create a glimmer project
  Glimmer WebComponent
  -> Create a web component with glimmer
  Manual...
  -> Run this wizard on your own, select your needs
```

For our purposes we'll choose `Octane`. Next, accept the default folder name. The installation script will take a few minutes to run.

Finally, we also want to manually add one more addon. `ember-auto-import` will allow us to import NPM packages that aren't Ember addons:

```sh
$ ember install ember-auto-import
```

## Components

With Ember you can jump right in to creating components without needing to look into other concepts yet. `ember-cli` will allow us to easily generate a component's files:

```sh
$ ember generate component my-component
```

This will create a `src/ui/components/my-component` folder with the following files:

- `component-test.js` - an automated test
- `component.js` - the logic for the component
- `template.hbs` - the markup for the component

Having separate files has the advantage that each file is a normal .js or .hbs file, but since they are in the same folder it's easy to switch between them.

Let's add a message to the component markup. Replace the contents of `template.hbs` with:

```hbs
{% raw %}Hello!{% endraw %}
```

Now we want to display the component. The main template file that renders the app is `src/routes/application/template.hbs`. We can just add the component in there. Delete the contents of the file and replace it with:

```hbs
{% raw %}<MyComponent />{% endraw %}
```

Note that components don't need to be imported anywhere; they're automatically available in templates.

We can see this working by starting up the server with `ember serve`. Go to `http://localhost:4200` and you should see the "Hello!" message. You can leave the server running.

## Arguments and Properties

Next, let's pass in an argument to our component. Arguments are passed with an `@` prefix:

```hbs
{% raw %}<MyComponent @name="world" />{% endraw %}
```

(Arguments are equivalent to props in React or Vue.)

In the component itself, we access that argument in the template using the `@` sigil as well. This makes it clear at the point of use that the data comes in as an argument:

```hbs
{% raw %}Hello, {{@name}}!{% endraw %}
```

We can also store properties on the component itself and display them. (Properties are equivalent to state in React or data in Vue. They're called "properties" because they're just normal JavaScript object properties!) Open `src/ui/components/my-component/component.js` and add a property:

```diff
 import Component from '@ember/component';

 export default class MyComponentComponent extends Component {
+  count = 0;
 }
```

Ember uses plain object properties for data storage, rather than a framework-specific mechanism like a `state` or `data` object.

And reference it in the template with a `this.` prefix, just as you would in JavaScript:

```hbs
{% raw %}Hello, {{@name}}!

{{this.count}}{% endraw %}
```

## Actions

To update the data, we can implement an action on the component. We use a decorator to indicate that a method is available as an action.

```diff
 import Component from '@ember/component';
+import {
+  action,
+} from '@ember-decorators/object';

 export default class MyComponent extends Component {
   count = 0;
+
+  @action
+  increment() {
+    this.set('count', this.count + 1);
+  }
 }
```

Note that we use a `this.set()` method to update the value of the property. A little bit like React's `this.setState()`, this indicates to Ember that a data item has changed, so it can rerender the template.

To call this action, add a button to the template:

```diff
{% raw %} Hello, {{@name}}!

 {{this.count}}
+
+<button onclick={{action this.increment}}>Increment</button>{% endraw %}
```

## Computed Properties

Components can also have computed properties. These are cached, and will be automatically recomputed when their dependent data changes.

```diff
 import Component from '@ember/component';
 import {
   action,
+  computed,
 } from '@ember-decorators/object';

 export default class MyComponent extends Component {
   count = 0;

+  @computed('name')
+  get uppercaseName() {
+    return this.name.toUpperCase();
+  }
+
   @action
   increment() {
```

Note that computed properties use standard ES5 getters, along with a decorator to provide extra information to Ember on dependent data.

```diff
{% raw %}-Hello, {{@name}}!
+Hello, {{this.uppercaseName}}!{% endraw %}
```

## Nested Components

Components can be nested inside one another in the filesystem, at which point they are only available to be used inside their parent component.

To see this in action, generate a child component:

```sh
$ ember generate component my-component/child-component
```

Note that under the `src/ui/components/my-component` folder, it creates a subfolder `child-component`, containing the same test, component, and template files.

Add some content to its hbs file:

```hbs
{% raw %}Hello, child!{% endraw %}
```

Then add it to the parent hbs file:

```diff
{% raw %} {{this.count}}

 <button onclick={{action this.increment}}>Increment</button>
+
+<ChildComponent />{% endraw %}
```

If you remove `ChildComponent` from this file and try adding it to `application/template.hbs` instead, you'll see an error in the browser console "Cannot find component child-component".

## Using NPM Packages

In Ember you can load data the same way you would in any other framework. To see this in action, let's import the popular Axios web service client library:

```sh
$ npm install --save axios
```

You will need to stop and rerun `ember serve` to get the package loaded.

Next, in `src/ui/components/my-component/child-component/component.js`, initialize a `records` property and populate it in `didInsertElement()` - this is a lifecycle hook that runs when the component is added to the DOM. We'll use [JSONPlaceholder](https://jsonplaceholder.typicode.com/) for some convenient JSON data:

```diff
 import Component from '@ember/component';
+import axios from 'axios';

 export default class ChildComponent extends Component {
+  records = [];
+
+  async didInsertElement() {
+    const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
+    this.set('records', response.data);
+  }
 }
```

Note that we can use `async/await` - it's enabled by default in Ember's Babel config.

Display it in the template:

```hbs
{% raw %}Hello, child!

{{#each this.records as |record|}}
  <p>
    {{record.title}}
  </p>
{{/each}}{% endraw %}
```

`#each` is a helper that loops through an array, and provides a variable to the nested markup.

## Moving Content to a Route Template

Now, say we want to display the contents of a post. Usually users would expect this to be on a separate route. Ember includes routing out of the box.

Earlier we added `<MyComponent />` to our `application/template.hbs` file. This file is the template that is always displayed no matter which route your app is on. To get `MyComponent` to display only on the "home page", generate an index route:

```sh
$ ember generate route index
```

This should create a folder `src/ui/routes/index` containing:

- `route-test.js` - a test
- `route.js` - the route class
- `template.hbs` - the route markup

Copy the component invocation from `application/template.hbs` and paste it into `index/template.hbs`:

```hbs
{% raw %}<MyComponent @name="world"/>{% endraw %}
```

Now replace the contents of `application/template.hbs` with the following:

```hbs
{% raw %}<h1>My App!</h1>

{{outlet}}{% endraw %}
```

`outlet` is a helper that will output the contents of the specific route you're on. Reload your app and you should see the "My App" header as well as your parent and child component content.

## Creating Another Route

To create a route for child pages, use Ember CLI again. As you can tell, in Ember we use `ember generate` a lot; you can use `ember g` as a shortcut:

```sh
$ ember g route post
```

The `generate route` command modified the file `src/router.js`. Check the changes:

```js
Router.map(function() {
  this.route('post');
});
```

This configures a route at `/post`. But we want to modify it to be at `/post/27` for post ID 27, for example. To do this, we can customize the path of this route to include a dynamic segment, indicated with a `:` prefix:

```js
Router.map(function() {
  this.route('post', { path: 'post/:id' });
});
```

You can't access the `id` variable from just anywhere; the place that has access to it is a route class, `route.js`. Open it. A route's dynamic segments are passed to a method called `model()` if one is defined. Add it:

```diff
 export default class PostRoute extends Route {
+  model({ id }) {
+  }
 }
```

Note that we can use destructuring to get the `id`.

For now, let's generate some fake post data. We'll come back to share data between the routes later:

```diff
export default class PostRoute extends Route {
   model({ id }) {
+    return {
+      title: `Post ${id}`,
+      body: `This is post ${id}!`,
+    };
   }
}
```

In the route's template, let's render a `PostDetail` component:

```hbs
{% raw %}<PostDetail @post={{this.model}} />{% endraw %}
```

Earlier we used quotes around an argument when passing a string. Now that we are passing an object instead, we need to use double-curlies.

Generate this `PostDetail` component:

```sh
$ ember g component post-detail
```

Then fill in its template:

```hbs
{% raw %}<h3>{{@post.title}}</h3>

<p>{{@post.body}}</p>{% endraw %}
```

Now, back in our child component template, we can link our post titles to that route:

```diff
{% raw %}{{#each this.records as |record|}}
   <p>
-    {{record.title}}
+    {{#link-to "post" record.id}}
+      {{record.title}}
+    {{/link-to}}
   </p>
 {{/each}}{% endraw %}
```

`link-to` is a helper that sets up a link to another route. We can pass a separate argument to it and it's filled in as the dynamic segment.

The links should be clickable and take you to the route with the right ID, and our fake data will be shown.

## Loading Data in a Service

To share data between components, we can put it in a service. The service can be injected into any component that needs it.

Generate a new service:

```sh
$ ember g service posts
```

This creates a `src/services` folder with the following files:

- `posts-test.js` - a test for the service
- `posts.js` - the service class

Add a `getAll()` method to the service that implements loading the data:

```diff
 import Service from '@ember/service';
+import axios from 'axios';

 export default class Posts extends Service {
+  async getAll() {
+    const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
+    return response.data;
+  }
}
```

Now update the `ChildComponent` to inject the `Posts` service to get the data from there instead of using Axios directly:

```diff
 import Component from '@ember/component';
-import axios from 'axios';
+import { service } from '@ember-decorators/service';

 export default class ChildComponent extends Component {
+  @service posts;
+
   records = [];

   async didInsertElement() {
-    const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
-    this.set('records', response.data);
+    this.set('records', await this.posts.getAll());
   }
 }
```

Note that just by using the `@service` decorator on a property, Ember knows to inject the Posts service when it creates the component. There's no multi-step provider component process. Dependency injection also makes testing easier, because in a component test you can inject a fake service instead of the real one.

## Service State

To avoid having to re-retrieve the post when we go to the detail page, let's cache the post data in the service.

```diff
 export default class Posts extends Service {
+  posts = null;
+
   async getAll() {
-    const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
-    return response.data
+    if (this.posts === null) {
+      const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
+      this.posts = response.data;
+    }
+
+    return this.posts;
   }
 }
```

Now that the posts are cached, we can easily add a method to retrieve a single post by ID:

```diff
     return this.posts;
   }
+
+  async findById(postId) {
+    const posts = await this.getAll();
+    return posts.find(p => p.id === postId);
+  }
 }
```

Note that we call `getAll()` instead of accessing `this.posts` directly. This way, even if `getAll()` hasn't been called before, we'll be sure to retrieve them.

Inject the service into the post route and call it in the `model()` method to return the data:

```diff
 import Route from '@ember/routing/route';
+import { service } from '@ember-decorators/service';

 export default class PostRoute extends Route {
+  @service posts;

   model({ id }) {
-    return {
-      title: `Post ${post_id}`,
-      body: `This is post ${post_id}!`,
-    };
+    return this.posts.findById(Number(id));
   }
 }
```

Now, go to the root of your app, click a blog post link, and you should be taken to a detail page with the data.

## Conclusion

We've looked at Ember's module unification file layout, angle bracket syntax, ES6 classes and decorators, and auto importing of NPM modules. These changes remove some of the incidental differences between Ember and other frameworks, as well as some legacy cruft from before components were the mental model and before ES6 classes existed. Put together, they give Ember a very modern feel.

If you're interested in a framework that gives you a reliable upgrade path as the web evolves, that takes care of implementation details so you can focus on your application's business logic, and that values creating addons so you don't have to reinvent the wheel, there are a lot of great reasons to check out Ember.
