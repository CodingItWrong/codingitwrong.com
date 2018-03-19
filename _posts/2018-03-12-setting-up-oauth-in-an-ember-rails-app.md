---
title: "Setting Up OAuth in an Ember/Rails App"
---

Let’s see how to set up OAuth authentication between an Ember frontend and a Rails API backend. We’ll be using two popular packages to help with this integration: [Ember Simple Auth][ember-simple-auth] on the frontend and [Doorkeeper][doorkeeper] on the backend.

## Rails Setup

Doorkeeper doesn't create a `User` model for you; it expects you to already have one in your Rails app. If you don't, you can create one like so:

```sh
bin/rails generate model user email:string password_digest:string
```

We'll use Rails' built-in [`has_secure_password`][has_secure_password] functionality to store the password securely as a digest. Add it to the generated `User` model:

```diff
 class User < ApplicationRecord
+  has_secure_password
 end
```

`has_secure_password` requires the `bcrypt` gem, so check your `Gemfile` to make sure it's included. If not, add it:

```ruby
gem 'bcrypt'
```

## Doorkeeper Setup

Next, to add Doorkeeper to your Rails app, add it to your Gemfile:

```ruby
gem 'doorkeeper'
```

Then run `bundle install`.

Run the Doorkeeper installation generator:

```sh
bin/rails generate doorkeeper:install
```

This will install several files you'll need to run Doorkeeper. Next, run the Doorkeeper Active Record migration generator:

```sh
rails generate doorkeeper:migration
```

This will generate a new migration which creates the following tables:

- `oauth_applications` - Tracks different client applications that can access your API. We won't need to use this for this tutorial.
- `oauth_access_grants` - Tracks users' access to use different `oauth_applications`. We won't need to use this for this tutorial.
- `oauth_access_tokens` - The temporary access tokens that are generated when a user logs in.

Interestingly, none of these tables reference a `users` table directly. Instead, the tables have a generic `resource_owner_id` column. These could be tied to a concept other than a user, but for typical cases like ours, they *will* point to a user.

To enforce this connection at the database level, add the following foreign keys to the migration. These will ensure each `resource_owner_id` column points to a real record in the `users` table:

```ruby
add_foreign_key :oauth_access_grants, :users, column: :resource_owner_id
add_foreign_key :oauth_access_tokens, :users, column: :resource_owner_id
```

Next, we need to enable the correct grant type. OAuth 2 includes several grant types, which are different ways to authenticate. The one we want to use for our Ember app is the ["Resource Owner Password Credentials Grant"][doorkeeper-password-grant], or "Password Grant" for short.

In `config/initializers/doorkeeper.rb`, find a commented-out line containing `grant_flows`. Uncomment it and replace the contents of `%w[]` with "password":

```diff
-  # grant_flows %w[authorization_code client_credentials]
+  grant_flows %w[password]
```

Finally, we need to tell Doorkeeper how to check the username and password passed in. It does this by looking for a `resource_owner_from_credentials` block in your Doorkeeper initializer. Let's add it:

```ruby
  resource_owner_from_credentials do
    User.find_by(email: params[:username])
        .try(:authenticate, params[:password])
  end
```

We use the values in `params` to try to find the user. Note that Doorkeeper uses a parameter named `username`, even though on our User model the column is called `email`.

If the User is found, we use the `authenticate` method created for us by `has_secure_password` to check if the password is correct. This method will return the user if the password is correct, and `false` if it's incorrect. This is exactly the return value that Doorkeeper expects from `resource_owner_from_credentials`. Conventions are great!

The last thing we should do is confirm that our OAuth endpoint is available. Run `bin/rails routes` and you should see a number of them including `oauth` in the path. Here are the ones we care about:

```
...
 oauth_token POST   /oauth/token(.:format)    doorkeeper/tokens#create
oauth_revoke POST   /oauth/revoke(.:format)   doorkeeper/tokens#revoke
...
```

(If you'd like to nest these routes somewhere, like under an `/api/` scope for example, you can change where `use_doorkeeper` is called in your `routes.rb` file.)

With this, we're done the Rails setup, and we're ready to move over to Ember.

## Ember Secure Auth setup

In your Ember app, install Ember Simple Auth:

```sh
ember install ember-simple-auth
```

Next, create an `app/authenticators` directory and an authenticator inside it. It can be called whatever you want; we'll call ours `oauth2`. Implement it by extending the [`OAuth2PasswordGrantAuthenticator`][esa-password-grant] class:

```js
import OAuth2PasswordGrant from 'ember-simple-auth/authenticators/oauth2-password-grant';

export default OAuth2PasswordGrant.extend({
  serverTokenEndpoint: 'http://localhost:3000/oauth/token',
});
```

Note that we specify the `serverTokenEndpoint` to use, pointing to the host and port our Rails app is running on, and using the path we saw in the Rails routes.

To finish our setup, we just need to add a simple login form:

```handlebars
{% raw %}<form {{action 'authenticate' on='submit'}}>
  <label for="email">Email</label>
  {{input id='email' placeholder='Enter Email' value=email}}
  <label for="password">Password</label>
  {{input id='password' placeholder='Enter Password' type='password' value=password}}
  <button type="submit">Login</button>
  {{#if errorMessage}}
    <p>{{errorMessage}}</p>
  {{/if}}
</form>{% endraw %}
```

Implement the corresponding action in the controller:

```js
import Controller from '@ember/controller';
import { inject as service } from '@ember/service';

export default Controller.extend({
  session: service(),

  actions: {
    async authenticate() {
      let { email, password } = this.getProperties('email', 'password');

      try {
        await this.get('session').authenticate('authenticator:oauth2', email, password);
      } catch(e) {
        this.set('errorMessage', e.error || e);
      }
    }
  }
});
```

A few notes:

- The `session` service is what provides us the `authenticate()` method to log in. We inject the service into the controller with `@ember/service/inject`.
- Here I'm using the ECMAScript `async/await` syntax to handle asynchronous calls nicely. `async/await` is enabled by default in Ember 3.0. If you're using an older version or don't want to use it, you can use Promise-based syntax instead.

Load up your app and try to log in. If there is an error it will be displayed. If login succeeds, you won't see any visual feedback, but checking the Network tab of your developer tools should show the 200 status call.

## What's Left

This gets us a successful login, but we aren't actually doing anything to protect our app with this login. There are two parts to this protection:

1. Stop requests to backend endpoints a user isn't authorized to use. To learn how to do this using the Pundit gem, see [my blog post series on authorizing jsonapi_resources][authorizing-jsonapi-resources] on the Big Nerd Ranch blog.
2. Hide features in the UI a user isn't authorized to use. The [Ember Simple Auth walkthrough][esa-walkthrough] describes this well.

[authorizing-jsonapi-resources]: https://www.bignerdranch.com/blog/authorizing-jsonapi-resources-part-1-visibility/
[doorkeeper]: https://github.com/doorkeeper-gem/doorkeeper
[doorkeeper-password-grant]: https://github.com/doorkeeper-gem/doorkeeper/wiki/Using-Resource-Owner-Password-Credentials-flow
[ember-simple-auth]: https://github.com/simplabs/ember-simple-auth
[esa-password-grant]: http://ember-simple-auth.com/api/classes/OAuth2PasswordGrantAuthenticator.html
[esa-walkthrough]: https://github.com/simplabs/ember-simple-auth#walkthrough
[has_secure_password]: http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html
