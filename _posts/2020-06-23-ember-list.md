---
title: "Tweaking UI Behavior with Ember Data Queries"
---

There are a lot of things I like about Ember.js, but Ember Data is my favorite part. Ember Data with a JSON:API backend are an order of magnitude less work than any other data layer I've seen. A lot works out of the box--but it still sometimes requires tweaking to get it to work just the way you like. Let's look at how a simple todo list can present some surprising requirements, and how we can implement them with Ember Data.

You can also check out [the full application repo](https://github.com/CodingItWrong/surely-ember) if you like.

## The Query

I'm building a todo list application called Surely.

The main route of the application is the Available list. It should show all the todos that are available to be worked on now: they aren't completed, deleted, or deferred to the future. It also provides a text field to quickly add new todos, which should appear in the list.

![screenshot of todo app layout](/img/posts/ember-queries/01-layout.png)

We need to decide how we'll load the data for this screen. Our list of available todos will hopefully always be short. But, as time goes on, we will have more and more *completed* todos, and we won't want to load all of *those* from the server. Instead, we'll query the server for just the records we want.

I've set up a filter on my JSON:API server, so that I can ask for the avaialble todos with the query string parameter:

```sh
?filter[status]=available
```

We can send this query to the server in the route model like this:

```js
export default class TodosAvailableRoute extends Route {
  model() {
    return this.store.query('todo', {
      filter: { status: 'available' },
    });
  }
}
```

In our route template, we pass the model to a todo list component:

```html
{% raw %}<TodoList
  @todos={{this.model}}
/>{% endraw %}
```

We load up our route, the data is returned from the server, and it displays fine.

## Adding a Record to the Query

We run into a problem when we try to create a new todo using the form on the same route. The call to the server to create the todo succeeds, but it doesn't show up in the list.

![animation of new todo not appearing in the list](/img/posts/ember-queries/02-new-todo-doesnt-appear.gif)

Why not? Because our form and list are on the same route, the route model is not reloaded after we add a todo. Our route model is a query, which relies on the server to tell us which records to show in the list. We've added a new record that should be included as well, but our client doesn't know that. All it knows is to display the records the server told it to.

One option would be to rerun the query from the server after adding a record, to let the server give us an updated list of records, including the new one. But this case is so simple that it would be nice to have an option that didn't require reloading so much data from the server.

To approach this, we can separate out the concept of what data we query from the server, from what data we return in the route. Currently, we query from the server using the filter parameter, and then return those results. Instead, what we can do is:

1. Query the server for available todos, which adds them to the store.
1. Return *all* the todos we have in the local store to the route.
1. Filter that array to show only the available todos.

This seems unintuitive at first. Why return all the todos from the model method when we've just queried for the ones we need? But this is what allows us to solve the problem. We query the server for the records *it* knows should be displayed on this screen, but then we use all the records the *client* knows about to decide what is ultimately shown.

We can implement this by replacing the model method with the following:

```js
async model() {
  await this.store.query('todo', {
    filter: { status: 'available' },
  });
  return this.store.peekAll('todo');
}
```

Don't miss the `async` keyword added in front of the `model()` method. This allows us to `await` the results of the query before we return. After the query returns, we use the `store.peekAll()` function to return all the records that are present in the store on the client.

Now all the records are returned in the route model, so we need a way to filter them before rendering them. There's some nontrivial logic involved in determining if a todo is available. Instead of putting that in a component or controller, let's put it in the model itself. That way any part of our app can just ask a model if it is available.

We're starting with a simple Ember Data model that just declares its attributes:

```js
import Model, { attr } from '@ember-data/model';

export default class TodoModel extends Model {
  @attr name;
  @attr('date') completedAt;
  @attr('date') deletedAt;
  @attr('date') deferredUntil;
}
```

We add a computed getter:

```diff
 import Model, { attr } from '@ember-data/model';
+import { computed } from '@ember/object';

 export default class TodoModel extends Model {
   @attr name;
   @attr('date') completedAt;
   @attr('date') deletedAt;
   @attr('date') deferredUntil;
+
+  @computed('completedAt', 'deletedAt', 'deferredUntil')
+  get isAvailable() {
+    return (
+      !this.completedAt &&
+      !this.deletedAt &&
+      (!this.deferredUntil || this.deferredUntil < new Date())
+    );
+  }
 }
```

The `isAvailable` getter returns a boolean. A todo is available if it is not completed, not deleted, and it does not have a future `deferredUntil` date. (My real application has a more complex series of getters to handle other statuses, but this simpler version keeps the example focused for this post.)

Note that I found I needed to use the `@computed` decorator to get this getter to recompute when the data changes. For some reason, Ember Octane's auto-tracking didn't always catch changes to the model otherwise.

Now we can use this attribute to filter the todos. We could do it in a component or a controller; in my app I decided to do it in the controller:

```js
import Controller from '@ember/controller';
import { filter } from '@ember/object/computed';

export default class TodosAvailableController extends Controller {
  @filter('model.@each.isAvailable', function (todo) {
    return todo.isAvailable;
  })
  filteredTodos;
}
```

We use the `@filter` decorator to create a property that filters out the list. As the dependent key argument, we pass `'model.@each.isAvailable'`. This means the property should be recomputed each time the `isAvailable` property of any model changes. As the filter function, we provide a function that returns the todo's `isAvailable` property, which means the todo will be included only when `isAvailable` is true.

Now that we aren't relying on the server to tell us the order of todos, we should also add a sort to make the order intuitive. Let's sort alphabetically by the todo name. We can use the `@sort` decorator to sort the results of the filter operation:

```diff
 import Controller from '@ember/controller';
-import { filter } from '@ember/object/computed';
+import { filter, sort } from '@ember/object/computed';

 export default class TodosAvailableController extends Controller {
+  sortPropertiesAlphabetical = ['name:asc'];

   @filter('model.@each.isAvailable', function (todo) {
     return todo.isAvailable;
   })
   filteredTodos;
+
+  @sort('filteredTodos', 'sortPropertiesAlphabetical')
+  sortedTodos;
}
```

Now we update our template to display the `sortedTodos` property:

```diff
{% raw %} <TodoList
-  @todos={{this.model}}
+  @todos={{this.sortedTodos}}
 />{% endraw %}
```

When we load up the Available route, our list still shows only the available todos, as before. But now, when we add a new todo, it shows in the list. And the list is sorted both before and after the addition.

![animation of new todo appearing in the list with a delay](/img/posts/ember-queries/03-new-todo-delay.gif)

## Hiding Unsaved Records

There's a minor timing issue, though, and you can see it in the animation above. If the server response is slow, for a brief second after submitting a new todo, the todo's name displays both in the list and in the input field. Then, after a second, the name in the input field is cleared.

To understand why, let's look at the action in the `NewTodoForm` component that creates the record:

```js
@tracked newTodoName;

@action
async createTodo() {
  const todo = this.store.createRecord('todo', {
    name: this.newTodoName,
  });
  await todo.save();
  this.newTodoName = '';
}
```

The sequence is:

1. We create the new record in the store.
2. We save it, and `await` the API request to return.
3. We clear out the `newTodoName` field.

What's going on is that the text in the field is not cleared out until step 3. But the todo appears in the list immediately step 1, while still waiting for the web service request in step 2 to complete.

Why does the todo appear in the list right away? This is because of the fact that Ember Data uses an identity map in memory to track the records we create. Before we call `todo.save()`, the new todo record already exists in the identity map; it's just not saved to the server yet. The route model dynamically updates to include that new todo, and it shows up in the list.

To solve this problem, we just need to adjust our mental model. We thought we wanted to display *all* available todos in the list, but in Ember Data, "all records" includes records that are not yet saved to the server. So, actually, we want to display all *persisted* available todos.

To handle this, we just need to add another filter condition:

```diff
-@filter('model.@each.isAvailable', function (todo) {
+@filter('model.@each.{id,isAvailable}', function (todo) {
-  return todo.isAvailable;
+  return todo.id && todo.isAvailable;
 })
 filteredTodos;
```

In the dependent key property, we use "brace expansion" to indicate that there are two different attributes of each model that we are dependent on: `id` and `isAvailable`. In our filter function, we return true only if `id` is truthy (that is, it is not blank) and if `isAvailable` is true.

Now when we add a new todo it does not appear in the list until it finishes saving to the server, at which time the text field is cleared out as well.

![animation of new todo appearing in the list instantly](/img/posts/ember-queries/04-new-todo-instant.gif)

## Reloading

It would be nice to add reload functionality to our Available list. To see why this would be useful, we can open the app in two different tabs. If we a todo in one tab, it doesn't appear in the other tab. To get it to appear, we have to reload the browser tab.

![animation of refreshing the page to get updated todos](/img/posts/ember-queries/05-page-refresh.gif)

This isn't ideal. It'd be a nicer experience to allow the user to reload the model.

We add a Reload button to the `TodoList` component, which will trigger an action we pass in to it. What action can we pass in? Typically in Ember Octane we reference actions on the current component or controller like this:

```html
{% raw %}<TodoList
  @todo={{this.filteredTodos}}
  @onReload={{this.myCoolAction}}
/>{% endraw %}
```

Unfortunately, we can't define an action on our controller that will do what we need. Ember's Route class has a `reload()` method that will reload the model for us. But there isn't an easy way to get access to the Route object from the controller. Instead, we have to use an older Ember actions technique to do so: the `send()` method.

To start, let's define an action on the route:

```diff
 import Route from '@ember/routing/route';
+import { action } from '@ember/object';

 export default class TodosAvailableRoute extends Route {
   async model() {
     await this.store.query('todo', {
       filter: { status: 'available' },
     });
     return this.store.peekAll('todo');
   }
+
+  @action
+  refreshModel() {
+    this.refresh();
+  }
 }
```

We define a `refreshModel()` action method that just calls the `refresh()` method. Since `refresh()` isn't an action, we can't access it directly from outside the route; we have to create our own method that is an action.

To call this action on the route, we can use the `send()` method to specify the name of an action as a string. When that action is not found on the controller, it will "bubble" up to the route and execute it there.

```html
{% raw %}<TodoList
  @todo={{this.filteredTodos}}
  @onReload={{action send "refreshModel"}}
/>{% endraw %}
```

Now we can try it out. We add a record in one tab, then click Reload in the other tab and the widget list will reload.

![animation of getting new todo with refresh button](/img/posts/ember-queries/06-new-todo-refresh-button.gif)

## Reloading Removed Records

This reloading works for adding todos, but if a todo disappears from the list--by being completed, for example--reloading doesn't remove it.

![animation of refresh button not removing completed todo](/img/posts/ember-queries/07-completed-todo-does-not-refresh.gif)

Why doesn't reloading remove todos? Refreshing re-runs the query to the server, retrieving all available records. But if a record is marked as complete, it's no longer available, so it isn't returned in that result set. Because of this, that record isn't updated in the local store. But this means that in the local store the record still shows as available. And since we are doing client-side filtering, that record will still show in our list.

How can we get completed todos to disappear from our Available list? We could implement some server logic that would return not only the available records, but also records that had recently been marked as no longer available. That could be tricky to implement, however, and we might not get it right.

In this case, since we're doing a "refresh" operation and getting all the records for the list anyway, another option is to clear out the local store before refreshing the data. This way, there will not be any out-of-date records.

Clearing it out is a simple method call on the store:

```diff
 @action
 refreshModel() {
+  this.store.unloadAll('todo');
   this.refresh();
 }
```

Open the app in two tabs, then complete a todo in one tab. In the other tab, click "Reload". The record will disappear from the list.

![animation of refresh button removing completed todo](/img/posts/ember-queries/08-completed-todo-refreshes.gif)

## Reloading Upon Navigate

We have one final improvement to make. Clicking the "Reload" button isn't the only time data is loaded from the server; whenever we enter the route, it's loaded as well. This means that if we leave the Available route and then return to it, we'll run into the same problem where todos completed on the server are not removed from the list.

![animation of navigation not removing completed todo](/img/posts/ember-queries/09-completed-todo-stays-on-navigate.gif)

To get route navigation behavior to be consistent with reloading behavior, we can just move the unload operation out of the `refreshModel()` action and into the `model()` method:

```diff
 async model() {
+  this.store.unloadAll('todo');
   await this.store.query('todo', {
     filter: { status: 'available' },
   });
   return this.store.peekAll('todo');
 }

 @action
 refreshModel() {
-  this.store.unloadAll('todo');
   this.refresh();
 }
```

Let's test it out. In one tab, we add a todo and complete another todo. In a second tab, we navigate away from the Available list and back to it. When the model reloads, the added todo appears and the completed one disappears.

![animation of navigation removing completed todo](/img/posts/ember-queries/10-completed-todo-removed-on-navigate.gif)

## Review

Let's review the final code. Although we built it up over a number of steps, the final state is pretty straightforward.

Our route provides a model method that clears out the store, queries the appropriate records from the server, but then returns all records so we can do client-side filtering. It also provides a mechanism for refreshing the model:

```js
export default class TodosAvailableRoute extends Route {
  async model() {
    this.store.unloadAll('todo');
    await this.store.query('todo', {
      filter: { status: 'available' },
    });
    return this.store.peekAll('todo');
  }

  @action
  refreshModel() {
    this.refresh();
  }
}
```

Our Todo model provides a getter that make it easy to determine the state of the todo:

```js
import Model, { attr } from '@ember-data/model';
import { computed } from '@ember/object';

export default class TodoModel extends Model {
  @attr name;
  @attr('date') completedAt;
  @attr('date') deletedAt;
  @attr('date') deferredUntil;

  @computed('completedAt', 'deletedAt', 'deferredUntil')
  get isAvailable() {
    return (
      !this.completedAt &&
      !this.deletedAt &&
      (!this.deferredUntil || this.deferredUntil < new Date())
    );
  }
}
```

Our controller filters the todos to include only the ones already saved to the server, and the ones the client knows are available. Then it sorts those todos:

```js
import Controller from '@ember/controller';
import { filter, sort } from '@ember/object/computed';

export default class TodosAvailableController extends Controller {
  sortPropertiesAlphabetical = ['name:asc'];

  @filter('model.@each.{id,isAvailable}', function (todo) {
    return todo.id && todo.isAvailable;
  })
  filteredTodos;

  @sort('filteredTodos', 'sortPropertiesAlphabetical')
  sortedTodos;
}
```

Our route template provides the form to add a new todo, and passes the todos that have been both filtered and sorted to the `TodoList` component. We also pass an action to the component allowing it to refresh the route's model:

```html
{% raw %}<NewTodoForm />

<TodoList
  @todos={{this.sortedTodos}}
  @onRefresh={{action send "refreshModel"}}
/>{% endraw %}
```

With all these changes, Ember Data is still allowing us to work at a level of abstraction higher than other data layers. Once we got the right mental model, we're able to express how we want our data flow to work, and it works great.
