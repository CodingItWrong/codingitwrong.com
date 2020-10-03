---
title: Ember Data Live Updates with WebSockets
---

WebSockets provide a way for client apps to receive live updates from the server as data is changed by other users, or by the same user in a different tab or on a different device. WebSockets are a low-level technology, but by building on top of high-level abstractions like JSON:API we can get started adding live updates to our app fairly quickly.

Let's take a look at adding WebSockets to an Ember app backed by Rails. We'll use Ember Data on the frontend, [JSONAPI::Resources](https://jsonapi-resources.com) on the backend, and Rails' [Action Cable](https://guides.rubyonrails.org/action_cable_overview.html) for the WebSocket communication.

You'll need the following installed:

- Node
- Ruby
- [Postgres](https://postgresapp.com/)

## Setting Up the Backend

Create a new Rails app in API mode with a Postgres database:

```bash
$ rails new --api --database=postgresql live_updates_api
$ cd live_updates_api
```

Next, we need to configure the Postgres database to use UUID fields. This will make it easier to keep the frontend in sync. Generate a migration file:

```bash
$ rails g migration enable_uuids
```

Open the migration file and enable the following two extensions:

```ruby
class EnableUuids < ActiveRecord::Migration[6.0]
  def change
    enable_extension 'uuid-ossp'
    enable_extension 'pgcrypto'
  end
end
```

Create the database and run the migration:

```bash
$ rails db:create
$ rails db:migrate
```

Now, generate a Todo model:

```
$ rails g model todo name:string
```

Before running the migration, update the migration file to use UUIDs for the ID field:

```diff
 class CreateTodos < ActiveRecord::Migration[6.0]
   def change
-    create_table :todos do |t|
+    create_table :todos, id: :uuid do |t|
       t.string :name
```

Now run the migration:


```bash
$ rails db:migrate
```

Add the JSONAPI::Resources gem to your Gemfile:

```ruby
gem 'jsonapi-resources'
```

And install it:

```bash
$ bundle install
```

Generate a Resource class for the todo:

```bash
$ rails g jsonapi:resource todo
```

Open `app/resources/todo.rb` and configure it to expose the `name` attribute:


```ruby
class TodoResource < JSONAPI::Resource
  attribute :name
end
```

Generate the controller for the todos:

```bash
$ rails g jsonapi:controller todos
```

Configure the controller to not require an authenticity token, since that is more difficult to do with single-page apps like our Ember app:

```ruby
class TodosController < JSONAPI::ResourceController
  skip_before_action :verify_authenticity_token
end
```

Then add routes for that controller in `app/config/routes.rb`:

```ruby
Rails.application.routes.draw do
  jsonapi_resources :todos
end
```

Now we need to enable CORS so our Ember frontend on another port can access the API. In the Gemfile, uncomment:

```ruby
gem 'rack-cors'
```

Then run:

```bash
$ bundle install
```

Open `config/initializers/cors.rb` and uncomment the following block:

```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins 'example.com'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

Then change the `origins` line to allow all origins:

```diff
 allow do
-  origins 'example.com'
+  origins '*'

   resource '*',
```

Now the backend should be ready to go!

## Setting Up the Frontend

Create a new Ember app:

```bash
$ ember new --no-welcome live-updates-frontend
$ cd live-updates-frontend
```

Generate an Ember Data adapter to configure our connection to the server:

```bash
$ ember g adapter application
```

Open `app/adapters/application.js` and replace the contents with the following:

```js
import JSONAPIAdapter from '@ember-data/adapter/json-api';

export default class ApplicationAdapter extends JSONAPIAdapter {
  host = 'http://localhost:3000';
}
```

Generate a todo model:

```bash
$ ember g model todo
```

Open the generated `app/models/todo.js` and add the `name` attribute to it:

```diff
-import Model from '@ember-data/model';
+import Model, { attr } from '@ember-data/model';

 export default class TodoModel extends Model {
+  @attr name;
 }
```

Generate a `TodoList` component:

```bash
$ ember g component TodoList
```

Open the generated `app/components/todo-list.hbs` and add the following code:

```handlebars
{% raw %}<ul>
  {{#each @todos as |todo|}}
    <li>{{todo.name}}</li>
  {{/each}}
</ul>{% endraw %}
```

Now generate a `NewTodoForm` component and class:

```bash
$ ember g component NewTodoForm
$ ember g component-class NewTodoForm
```

Add the following to `app/components/new-todo-form.hbs`:

```handlebars
{% raw %}<form {{on 'submit' this.add}}>
  <Input
    @value={{this.name}}
    type="text"
    placeholder="New Todo"
  />
  <button type="submit">Add</button>
</form>{% endraw %}
```

And replace the contents of `app/components/new-todo-form.js` with this:

```js
import Component from '@glimmer/component';
import { inject as service } from '@ember/service';
import { tracked } from '@glimmer/tracking';
import { action } from '@ember/object';

export default class NewTodoFormComponent extends Component {
  @service store;

  @tracked name;

  @action async add(evt) {
    evt.preventDefault();

    const todo = this.store.createRecord('todo', { name: this.name });

    try {
      await todo.save();
      this.name = '';
    } catch (err) {
      console.error(err);
    }
  }
}
```

Generate an index route to put these components on:

```bash
$ ember g route index
```

Then open `app/routes/index.js` and load the Todo models:

```js
import Route from '@ember/routing/route';

export default class IndexRoute extends Route {
  model() {
    return this.store.findAll('todo');
  }
}
```

In `app/templates/index.js`, create an instance of each component:

```handlebars
{% raw %}<NewTodoForm />

<TodoList @todos={{@model}} />{% endraw %}
```

## Trying It Out

With that, we should be ready to run our app!

In one tab, start the Rails API:

```bash
$ rails s
```

And in another tab, start the Ember frontend:

```bash
$ ember s
```

Visit the Ember app in a browser at `http://localhost:4200`. You should see the New Todo form. Add a few todos and confirm that they show up in the list below.

Now, let's see the limitation we have, and how live updates can help us. Open the Ember app in a second tab and add a todo to it. Note how the todo list in the first list is not updated. Reload the first tab and see that the new todo shows up.

For applications where we want live updating, how can we accomplish that?

## Adding Live Updates with Action Cable

First, we generate a `TodosChannel` that will handle live update communication for todos:

```bash
$ rails g channel todos
```

Open `app/channels/todos_channel.rb`. In The `TodosChannel#subscribed` method, uncomment the line with `#stream_from` and give the channel a name:

```diff
 class TodosChannel < ApplicationCable::Channel
   def subscribed
-    # stream_from "some_channel"
+    stream_from "todos"
  end
```

Next, when we create a todo, we need to configure the app to send that todo over the channel. Where can we do that? We could put it in a model `#after_create` callback, but that would mean it would attempt to run any time we create a model, even in a seeder or the console.

Instead, let's add this logic to an `#after_create` callback in the `TodoResource` class. We only use the resources through our web interface, so this should only run when we make a web request.

In `app/resources/todo_resource.rb`, add the following:

```diff
 class TodoResource < JSONAPI::Resource
+  after_create :notify_clients
+
   attribute :name
+
+  private
+
+  def notify_clients
+    ActionCable.server.broadcast 'todos', serialized_model
+  end
+
+  def serialized_model
+    serializer = JSONAPI::ResourceSerializer.new(TodoResource)
+    serializer.object_hash(self, nil)
+  end
 end
```

Here's what's going on:

- In the class body, we call the `#after_create` macro to set up a callback, passing the `:notify_clients` symbol to indicate that a `#notify_clients` method should be called
- We define `#notify_clients` as a private method. In it, we call `ActionCable.server.broadcast` to send the message to the `'todos'` channel. The data we pass is the return value of the `#serialized_model` method
- We define `#serialized_model` to instantiate a serializer for the `TodoResource` and then serialize the current resource

This is all we need to do in the API; let's move over to the frontend.

Action Cable has an `npm` package for the client JavaScript code; let's install it:

```bash
$ npm install --save-dev actioncable
```

Now, we need somewhere in our code to set up the Action Cable connection. An `ApplicationController` will work well enough for this sample project. Generate it:

```bash
$ ember g controller application
```

Open `app/controllers/application.js` and see the following contents:

```js
import Controller from '@ember/controller';

export default class ApplicationController extends Controller {
}
```

Now let's set up Action Cable one step at a time. First, let's add a `constructor` that we can add additional functionality to:

```diff
 export default class ApplicationController extends Controller {
+  constructor(...args) {
+    super(...args);
+  }
 }
```

Our constructor calls the parent class constructor.

Next, let's create an Action Cable connection:

```diff
 import Controller from '@ember/controller';
+import ActionCable from 'actioncable';

 export default class ApplicationController extends Controller {
   constructor(...args) {
     super(...args);
+
+    this.cable = ActionCable.createConsumer(
+      'ws://localhost:3000/cable'
+    );
   }
 }
```

We connect to our `localhost:3000` server over the WebSocket protocol (`ws`), passing the default Action Cable path of `/cable`.

Next, we need to subscribe to the `TodosChannel` specifically:

```diff
 constructor(...args) {
   super(...args);

   this.cable = ActionCable.createConsumer(
     'ws://localhost:3000/cable'
   );

+  this.cable.subscriptions.create('TodosChannel', {
+    connected: () => {
+      console.log('connected');
+    },
+
+    disconnected: () => {
+      console.log('disconnected');
+    },
+
+    received: todo => {
+      console.log('received', todo);
+    }
+  });
 }
```

We go ahead and add some logging to confirm when Action Cable connects and disconnects.

Reload the Ember application and you should see "connected" in the web developer console, showing that we've connected. Make sure you have the app open in two tabs, then add a todo. Note that in _both_ tabs we get a log entry that says "received", and then a JSON:API-formatted data record.

So we're getting the created record pushed to the frontend--what can we do with it? The Ember Data store has a `.pushPayload()` method that will add this record into the store. To use it, inject the store service into the controller, then call the method:

```diff
 import Controller from '@ember/controller';
+import { inject as service } from '@ember/service';
 import ActionCable from 'actioncable';

 export default class ApplicationController extends Controller {
+  @service store;

   constructor(...args) {
...
      received: todo => {
        console.log('received', todo);
+       this.store.pushPayload({ data: todo });
      }
```

After the Ember app reloads, add another todo. You will see it automatically appear in the list in the second tab!

In the first tab, though, there's a problem: the same todo appears _twice_! Why is that? The problem is a timing issue:

- We send the request to the API to create the new todo.
- Action Cable sends the new todo to all connected clients.
- The client sees the new todo and pushes it into the store.
- THEN, the web service call returns a response with the new todo. Ember Data doesn't suspect we might have added that todo into the store ourselves already, so it dutifully adds it into the store again, resulting in a duplicate record.

There are a few different ways we can solve this. The approach we'll take is to assign UUIDs in the frontend, rather than relying on the backend to do it. That way, as soon as the record is created, we can uniquely identify it. Whether we are adding it into the store from Action Cable or a create response, Ember Data will know it is the same record and will overwrite it instead of duplicating it.

To set up the frontend to create UUIDs, first, add the `uuid` package as a dependency:

```bash
$ npm install --save-dev uuid
```

Next, configure the `ApplicationAdapter` to assign a UUID as the default ID for a record:

```diff
 import JSONAPIAdapter from '@ember-data/adapter/json-api';
+import { v4 } from 'uuid';

 export default class ApplicationAdapter extends JSONAPIAdapter {
   host = 'http://localhost:3000';
+
+  generateIdForRecord() {
+    return v4();
+  }
 }
```

We'll need to make a change to the backend, as well, to allow the ID to be passed in. Add the following class method to `TodoResource`:

```ruby
def self.creatable_fields(context)
  super + %i[id]
end
```

This adds `id` as one of the fields allowed to be created.

Try adding a new todo again. You'll see that only one instance of the todo appears in the first tab, and it still automatically shows up in the second tab as well.

## Conclusion

Live updates are a new way of thinking about data flow in your app, but the libraries we used allowed us to add them without too much code. We were able to get live updates working with the help of the `ActionCable.server.broadcast` method on the server and Ember Data's `pushPayload()` method on the client. We had an issue with duplicate records, and we solved it with UUIDs here—but other approaches would be possible if you want or need to do something different. For a more complex app you would need to add more logic to handle live updating of different model types, related records, and user-specific data. But this approach should get you a good start!
