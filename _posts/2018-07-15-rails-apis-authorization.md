---
title: Rails API Authorization
---

In my previous post we [set up a simple Rails web service](http://codingitwrong.com/2018/07/02/rails-the-easiest-way-to-create-a-web-service.html) for todos. But to use it in production, we need some kind of authentication, or else anyone could mess up our data. Luckily, the [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper) gem makes it easy to add authentication to Rails APIs using the OAuth 2 standard.

# Setting Up Password Authentication
First we need to create a `User` model:

```sh
$ rails generate model user email:string password_digest:string
```

We'll use Rails' built-in `has_secure_password` functionality to store the password securely in the `password_digest` column. Add it to `app/models.user.rb`:

```diff
 class User < ApplicationRecord
+  has_secure_password
 end
```

`has_secure_password` requires the the `bcrypt` gem, so uncomment in your `Gemfile`. Add the Doorkeeper gem while you're at it:

```diff
 # Use ActiveModel has_secure_password
-# gem 'bcrypt', '~> 3.1.7'
+gem 'bcrypt', '~> 3.1.7'
+gem 'doorkeeper'
```

Then run `bundle install`.

Run the Doorkeeper installation and migration generators:

```sh
$ rails generate doorkeeper:install
$ rails generate doorkeeper:migration
```

These will install several files you'll need to use Doorkeeper. It also sets up routes for you. Let's see them by running `rails routes`. You should see a number of them including `oauth` in the path. Here's the one we'll use to log in:

```
 oauth_token POST   /oauth/token(.:format)    doorkeeper/tokens#create
```

The commands you ran also created a migration file which creates tables used by Doorkeeper. Run the migration then create a new User in the Rails console:

```sh
$ rails db:migrate
$ rails console
irb(main):001:0> User.create!(email: 'me@example.com', password: 'rosebud')
```

Next, we need to tell Doorkeeper what mechanism we want to use for authentication; in our case, simple password authentication.

In `config/initializers/doorkeeper.rb`, paste the following inside the `Doorkeeper.configure do` block:

```ruby
  grant_flows %w[password]

  resource_owner_from_credentials do
    User.find_by(email: params[:username])
        &.authenticate, params[:password])
  end
```

This tells Doorkeeper to accept password-based authentication, and provides the code to validate a password. We look up the user using the passed-in username, then we call the `authenticate` method to check if the password is correct.

A few notes:

- `params` is an object that acts like a hash or dictionary, allowing looking up values by a key using square brackets.
- `&.` is Ruby's safe navigation operator, like `?.` in Swift or Kotlin. If the object it's used on is `nil` it will return `nil` instead of attempting to call the method, avoiding a `NoMethodError`.

Start your `rails server`, or if it's already running, restart it. Doorkeeper should be working now. To test this out, in Postman, create a `POST` request to `http://localhost:3000/oauth/token`, with a "Body" type of `form-data` and key-value pairs:

- grant_type=password
- username=me@example.com (or whatever you entered)
- password=rosebud (or whatever you entered)

When you send the request, you should see something like this back:

```json
{
    "access_token": "6b28549ccbaf1e0713d9ff7e4f596b9bcc2892e8c9a638f2bb05eada40f4c1a4",
    "token_type": "bearer",
    "expires_in": 7200,
    "created_at": 1530490943
}
```

The `access_token` value is what you'll need to provide to access protected routes; save it somewhere handy.

# Protecting Routes
Now let's make our API read-only for guests, by ensuring that create, update, and delete methods require you to be logged in. Add the following line to `app/controllers/todos_controller.rb`:

```diff
 class TodosController < ApplicationController
+  before_action :doorkeeper_authorize!, except: [:index, :show]
   before_action :set_todo, only: [:show, :update, :destroy]
```

Here's what's going on here:

- `before_action` is a Rails controller method that lets you specify a certain method should be called before all actions in the controller. If the method throws an exception, the controller action won’t be executed afterward. This is how the `doorkeeper_authorize!` method is meant to be used: it throws an exception if the request doesn’t include a valid token.
- We indicate the name of the method to call by passing a symbol. In this case, we want to call the method `doorkeeper_authorize!`, so we add a colon to the front of that name to make it a symbol.
- What object does the `doorkeeper_authorize!`  method exist on? The controller itself. Notice that we didn't need to inherit from a Doorkeeper controller or add a Doorkeeper mixin. Ruby is such a flexible language that allows methods to be added to existing classes. Doorkeeper actually adds the `doorkeeper_authorize!` method onto a Rails parent controller class, so it's automatically available on all your controllers.
- The named parameter `except:` allows us to tell `before_action` which methods we *don’t* want it to apply to. In our case, we want to allow anonymous requests to the `index` and `show` actions, but protect all the others.

Let's try out this protection. Send a `GET` request to `http://localhost:3000/todos` and it should succeed. 

Now, send a `POST` request to it and it should fail with a `401 Unauthorized` response. Add the following header, including the `access_token` you received earlier in place of this access token:

```
Authorization: Bearer 6b28549ccbaf1e0713d9ff7e4f596b9bcc2892e8c9a638f2bb05eada40f4c1a4
```

Rerun the `POST` and it should succeed.

If you deployed the app to Heroku last time, you can deploy the update:

```sh
$ git add .
$ git commit -m "Add authentication"
$ git push heroku master
$ heroku run rails db:migrate
$ heroku run rails console
irb(main):001:0> User.create!(email: 'me@example.com', password: 'rosebud')
```

# Next Steps
Now you've got a working, authenticated Rails API. You barely had to write any code! Now that authentication is set up, all you have to do to create a new authenticated resource is to generate the scaffold and add the `before_action :doorkeeper_authorize!`.

If this series has gotten you interested in how productive you can be with a Rails API, here are some next steps you can learn:

- Rails features for [validations](http://guides.rubyonrails.org/active_record_validations.html) and [associations between models](http://guides.rubyonrails.org/association_basics.html).
- More nuanced authorization rules with [the Pundit gem](https://github.com/varvet/pundit).
- Testing Rails APIs with [rspec-rails](https://github.com/rspec/rspec-rails).
- Generating APIs that conform to standards like [JSON API](https://github.com/cerebris/jsonapi-resources) or [GraphQL](https://github.com/rmosolgo/graphql-ruby).
- Learn more about [the Ruby language](http://ruby-doc.com/docs/ProgrammingRuby/).
- Learn more about [how to use Heroku for Ruby apps](https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction).