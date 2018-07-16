---
title: "Rails: The Easiest Way to Create a Web Service"
---

Whether you create frontends in JavaScript, Swift, Java, or Kotlin, there's a server-side framework for creating web service APIs in the same programming language you already use. It would seem like it’d be easiest for you to write your web service in the same language you use for your frontend.

But "seems" can be deceiving! The fastest and easiest way to create a web service API is Ruby on Rails—even if you don't know Ruby. Rails' batteries-included approach means that you don't have to write boilerplate or implement plumbing yourself. Instead of being locked in to features a hosted API service provides, or having to write them all yourself, the Rails ecosystem is so mature that there is a package for just about every feature you'll need.

To see how quickly Rails allows us to create and deploy a web service to production, let's create a web service for tracking todos.

# Getting Past the CRUD
[Install the latest version of Ruby](https://www.ruby-lang.org/en/documentation/installation/) (2.5.1 as of this writing), then install Rails:

```sh
$ gem install rails
```

Create a new Rails app with `rails new`. We’ll pass in a few flags to remove features we don’t need, to keep our app as simple as possible:

```sh
$ rails new simple-rails-api --api \
                             --skip-action-cable \
                             --skip-active-storage \
                             --skip-test 
$ cd simple-rails-api
```

Now let’s create our todo resource using the `rails generate scaffold` command, passing it the fields and types we want:

```sh
$ rails generate scaffold todo title:string complete:boolean
```

This creates everything we need to get started with todos:

- `app/models/todo.rb` — A model with a `title` string field and a `complete` boolean field.
- `app/controllers/todos_controller.rb` — A controller with default implementations of functionality to create, read, update, and delete todos.
- An entry in `config/routes.rb`  mapping URLs to the actions in that controller.
- A file in `db/migrate` with instructions to create a table for todos in our database.

Let's run that migration to update our database:

```sh
$ rails db:migrate
```

Next, let's insert a Todo record by hand. You don’t need to use a SQL database tool directly; you can use the `rails console`:

```sh
$ rails console
```

The Rails console lets you execute any arbitrary Ruby code you want. Run the following to create a todo:

```sh
irb(main):001:0> Todo.create!(title: 'Create an API', complete: false)
```

You’ll see some log output and then the following:

```sh
=> #<Todo id: 1, title: "Create an API", complete: false,
created_at: "2018-07-01 10:42:49", updated_at: "2018-07-01 10:42:49">
```

A few notes on what we typed:

- `Todo` is the name of the model class.
- `create!` is one of many methods available on Rails models. It inserts the record and will throw an error if we get something wrong, like pass an incorrect field name. Ruby method names can end with a `!` or `?`.
- Ruby supports named parameters just like Swift and Kotlin.

Now let’s see what URLs were created for us by `generate resource`. Run `rails routes`:

```sh
$ rails routes

Prefix Verb   URI Pattern          Controller#Action
 todos GET    /todos(.:format)     todos#index
       POST   /todos(.:format)     todos#create
  todo GET    /todos/:id(.:format) todos#show
       PATCH  /todos/:id(.:format) todos#update
       PUT    /todos/:id(.:format) todos#update
       DELETE /todos/:id(.:format) todos#destroy
```

This is all we need to create, read, update, or delete todos. Let’s give them a try! Let’s start up our server:

```sh
$ rails server
```

Now go to  `http://localhost:3000/todos` in your browser. You should see JSON data returned (the way it’s formatted may vary depending on your browser):

```json
[
  {
    "id": 1,
    "title": "Create an API",
    "complete": false,
    "created_at": "2018-07-01T10:42:49.188Z",
    "updated_at": "2018-07-01T10:42:49.188Z"
  }
]
```

Notice that, although we didn’t specify them, Rails created fields for `id`, `created_at`, and `updated_at`. All of these will be automatically assigned for you when records are created or updated.

Next let’s access an individual record. Go to `http://localhost:3000/todos/1` and you should see just an individual record returned.

To test out creating a record, download the [Postman app](https://www.getpostman.com/apps). Create a new request to `POST` to `http://localhost:3000/todos`. In the Body tab, choose “raw” and “JSON (application/json)”, then enter the following body:

```json
{
    "title": "Test out create endpoint",
    "complete": true
}
```

Click “Send” and you should receive a `201 Created` response with the following content:

```json
{
    "id": 2,
    "title": "Test out create endpoint",
    "complete": true,
    "created_at": "2018-07-01T11:36:35.793Z",
    "updated_at": "2018-07-01T11:36:35.793Z"
}
```

We saw other routes earlier for updating and deleting todos; you can feel free to try those out if you like.

# Validation
Next, let’s add some validation. We probably want to ensure the `title` field isn’t empty. Open `app/models/todo.rb` and you’ll see the following:

```ruby
class Todo < ApplicationRecord
end
```

This is pretty typical class declaration syntax, other than using `<` to indicate inheritance. It creates a `Todo` class that inherits from `ApplicationRecord`, a class Rails sets up for us. Note that there is no other code in this model yet, but it still works. We don’t need to configure it with the fields on the object; Rails can tell the fields by inspecting the database table.

To make the title required, add this:

```diff
 class Todo < ApplicationRecord
+  validates :title, presence: true
 end
```

Here’s what’s going on with this line of code:

- `validates` is a method provided on `ApplicationRecord` that we are running in the middle of the class definition. Notice that parentheses are optional for method calls in Ruby; this is the same as writing `validates(:title, presence: true)`.  Typically, “normal” method calls in our code are written with parens, but calls to Rails class-level methods like `validates` are called without parens.
- An identifier that starts with `:` is a symbol; it’s just like a string but it doesn’t have string manipulation methods like substring. Here we’re just saying to validate the field named `:title`.

To test this out, in your Postman request, change the `title` to an empty string:

```json
 {
     "title": "",
     "complete": true
 }
```

Rerun the request. You’ll get a `422 Unprocessable Entity` response, with the following body:

```json
{
    "title": [
        "can't be blank"
    ]
}
```

# Default Values
Instead of making the `complete` field required, let’s set it to default to `false` if it isn’t specified. We can create another migration to update our database scheme to set the default value:

```sh
$ rails generate migration set_default_todo_completed_to_false
```

This creates a new empty migration file. Add this line to it:

```diff
 class SetDefaultTodoCompletedToFalse < ActiveRecord::Migration[5.2]
   def change
+    change_column_default :todos, :complete, from: nil, to: false
   end
 end
```

(The reason we specify a `from:` value is so the migration is reversible. If you ever need to undo a migration, just run `rails db:rollback`.)

Run the migration with `rails db:migrate`.  Now send a `POST` request without a `complete` value specified:

```json
 {
     "title": "Defaulting to incomplete"
 }
```

The response you get should have `complete` set to `false`:

```json
{
    "id": 7,
    "title": "Defaulting to incomplete",
    "complete": false,
    "created_at": "2018-07-02T09:02:08.884Z",
    "updated_at": "2018-07-02T09:02:08.884Z"
}
```

# Enabling CORS
If you're planning on accessing this API from a web app on another "origin"—even a different port on `localhost`—browsers will block the request by default for security. We can allow these requests by configuring Cross-Origin Resource Sharing (CORS) headers.

Open the `Gemfile` at the root of your project. This file specifies dependencies like `package.json` for JavaScript or `gradle.build` for Android.

In your Gemfile,  find this line:

```ruby
$ gem 'rack-cors'
```

A gem is a Ruby package.  The `#` is Ruby's comment marker, so this dependency is commented out. Uncomment this line by removing `#`.

Next, run `bundle install` to install the `rack-cors` gem.  `bundle` is the Ruby package manager, equivalent to `npm`, `yarn`, or `gradle`. Running the `install` subcommand will install the gems and update the `Gemfile.lock`, which tracks exact versions that were installed.

Next, open `config/initializers/cors.rb`. Uncomment the following block:

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

Then change `origins` to allow any origin:

```diff
-    origins 'example.com'
+    origins '*'
```

Next, restart your `rails server`. Rails can automatically load changes to your classes without requiring a restart. But if you add a gem or change an initializer, as we did here, you need to restart the server to catch the changes.

Now you should be able to make requests from web apps running on other ports.

# Enabling a Production-Ready Database
We're almost ready to go to production, but first we need to change our database. By default Rails uses a SQLite database, which is easy for development but not robust enough for production. Let’s tell our Rails app to install support for Postgres in production, but continue to use SQLite for development.

Open your `Gemfile` and find the line `gem 'sqlite3'`. Move this line down inside the `group :development, :test do` block:

```diff
 group :development, :test do
   # Call 'byebug' anywhere in the code to stop execution and get a debugger console
   gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
+  gem 'sqlite3'
 end
```

This ensures SQLite will only be installed in the development and test environments, not production.

Next, create a `:production` group with the `pg` gem in it:

```ruby
group :production do
  gem 'pg'
end
```

This will install the Postgres gem only in production.

Run `bundle install` to update your `Gemfile.lock`.

# Deploying to Production

Now we're ready to deploy to production. [Heroku](https://www.heroku.com/) is a super simple platform for deploying applications in many languages, and allows you to run "hobby" applications for free. Heroku has particularly good support for Rails apps: it will automatically detect that you have a Rails app and run it appropriately. It will also automatically create a Postgres database and provide the connection info to your Rails app, so you don't need to change any config files to use it.

Heroku allows you to deploy code using Git pushes. If you haven't already been committing your code to a Git repo, go ahead and do so:

```sh
$ git init
$ git add .
$ git commit -m "Create app"
```

[Sign up for a Heroku account](https://signup.heroku.com/?c=70130000001x9jFAAQ), [install the Heroku command line tools](https://devcenter.heroku.com/articles/heroku-cli#download-and-install), and log in:

```sh
$ heroku login
```

Now it's time to create our Heroku app.

```sh
$ heroku create
```

This will create an app on Heroku, assign it a (very) random name, and connect your local Rails app to it. To deploy:

```sh
$ git push heroku master
```

This is just a normal git command; you're pushing your local master branch to the git remote named `heroku` that the `heroku create` command set up for you.

You'll see a bunch of output, and if it succeeds, you'll see the following at the end:

```sh
https://your-app-name.herokuapp.com/ deployed to Heroku
```

Heroku doesn't automatically migrate your database, so we need to run the migrate command as we've been doing locally. To run a command in a Heroku app, prefix it with `heroku run`:

```sh
$ heroku run rails db:migrate
```

Now your app should be ready to access. Run your POST command against your production app URL and you should see a successful response!

# Next Steps

In just a few minutes Rails allowed us to create a fully-functioning web service and deploy it to production. But there's one major thing missing: we need to secure it so not just anyone can change data. In my next post we’ll add password authentication to our API.
