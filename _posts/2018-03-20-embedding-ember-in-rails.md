---
title: "Embedding Ember in Rails"
tags: [ember, rails, apis]
---

Webpacker allows you to run an Angular, React, or Vue app within Rails. But what about Ember? Since it's not built with Webpack we don't get integration out of the box. But here's the setup I've come to to run Ember within my Rails app.

Oftentimes I would want to keep Ember and Rails repos separate and hosted separately--it's a nice separation of concerns. But considerations are different for [Firehose](https://github.com/CodingItWrong/firehose), my open-source link-tracking app. I want people to be able to easily self-host it, so I don't want them to have to set up separate API and frontend hosting. I'd like them to be able to deploy one app that runs both the API and the frontend.

## The Goal

We want different setups in development and production.

**In production** we want the Ember app to be served by Rails. This means building the Ember app into the `/public/` folder. It also means ensuring that any URL the user enters that isn't a valid API URL will display the Ember app's `index.html` page.

**In development** we want to let Rails and Ember servers run as normal, so we get live reloading of our Ember app. In general, differences between environments aren't ideal, but for frontend development having live reloading is such a major productivity boost that it's been embraced widely. We'll test out our "production" setup locally to make sure it works.

## Installation

Since our apps will be deployed together, let's install them in the same repo. We'll create a new directory for the Ember app at the root level of the Rails app. If you have an existing Ember app you can just move its directory there. If you're creating a new Ember app, you can use the `ember new` command, with an extra flag:

```sh
cd my-cool-app
ember new --directory ember my-cool-app
```

Usually Ember names your directory the name of the app, but here we use the `--directory` flag to specify a different directory name. This is because we want the Ember app's name to be the same as our overapp app name, but it's more descriptive to name the directory `ember`. (You could call it `frontend` instead if you like.)

`ember new` also initializes a git repo in the app directory, so let's delete that repo so it's tracked as part of the Rails app repo just like any other directory:

```sh
rm -fr ember/.git
```

Note that since Ember is in a subdirectory, you'll need to `cd` into it to run any Ember commands, but Rails commands will need to be run at the root.

## Host Name

First let's configure Ember with the correct host to use in development and production. In development they will be on separate hosts (really, separate ports on `localhost`). In production they will be on the same host, so we can just not specify a hostname.

First, we'll set up an `ENV` variable with the host name. Add the following to `config/environment.js`:

```diff
   if (environment === 'development') {
     //...
+    ENV.apiHost = 'http://localhost:3000';
   }
```

We use the default Rails port of 3000; enter a different value if your setup is different.

Let's assume our app is using Ember Data for data, and [Ember Simple Auth](https://github.com/simplabs/ember-simple-auth#readme) for authentication. If so, we need to configure the host in two places.

## Ember Data

For Ember Data, we add the host name to our application adapter. If you don't already have an `adapters/application.js` file, generate one:

```sh
ember generate adapter application
```

Make the following changes to it:

```diff
 import DS from 'ember-data';
+import ENV from '../config/environment';

-export default DS.JSONAPIAdapter.extend({
+let options = {
   // any options you previously sent to extend() here, e.g. namespace
-});
+};
+
+if (ENV.apiHost) {
+  options.host = ENV.apiHost;
+}
+
+export default DS.JSONAPIAdapter.extend(options);
```

If `ENV.apiHost` is set (as in the development environment) then we add a `host` property to the `options` object set to that value. Otherwise, we leave the `host` property off.

Incidentally, if your Rails API isn't already set to serve data from a subdirectory like `/api/`, it's a good idea to set that up. Ember apps use all kinds of different routes, so if both your Ember app and API have a `/blog-posts` path, they're going to conflict. By putting all your API routes under `/api/` it prevents overlap.

## Ember Simple Auth

Ember Simple Auth has its own config we need to update. Make the following changes to your authenticator, under `app/authenticators/`. For example, I'm using the `OAuth2PasswordGrantAuthenticator`, and it takes a `serverTokenEndpoint` property. Because it has a single property for both the host and path, we need to join them together ourselves:

```diff
 import OAuth2PasswordGrant from 'ember-simple-auth/authenticators/oauth2-password-grant';
+import ENV from '../config/environment';

+const serverTokenPath = '/api/oauth/token';
+const serverTokenEndpoint = ENV.apiHost
+  ? ENV.apiHost + serverTokenPath
+  : serverTokenPath;

 export default OAuth2PasswordGrant.extend({
-  serverTokenEndpoint: '/api/oauth/token',
+  serverTokenEndpoint,
 });
```

## CORS

If you're already running Ember and an API on different hosts then you're likely familiar with [Cross-Origin Resource Sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS). If not, CORS is a mechanism to allow frontend apps to make web service requests to different hosts. In production our Rails API and Ember app will be on the same host, so we don't need CORS, and it's safer to leave it out. But in development we need CORS to allow the Ember app on `localhost:4200` to access the API on `localhost:3000`.

Check your `Gemfile` for the [`rack-cors`](https://github.com/cyu/rack-cors) gem and add it if it isn't present:

```diff
+ gem 'rack-cors'
```

If you added it, run `bundle install` to install it.

Now add your CORS configuration. The `rack-cors` gem README shows it added to `config/application.rb`, but since we only want it enabled in development, let's add it to `config/environments/development.rb` instead. You can add it right near the end of the file, before the `end` keyword closing the `Rails.application.configure` block:

```diff
   # Use an evented file watcher to asynchronously detect changes in source code,
   # routes, locales, etc. This feature depends on the listen gem.
   config.file_watcher = ActiveSupport::EventedFileUpdateChecker
+
+  config.middleware.insert_before 0, Rack::Cors do
+    allow do
+      origins '*'
+      resource '*', :headers => :any, :methods => [:get, :post, :options]
+    end
+  end
 end
```

See the [rack-cors documentation](https://github.com/cyu/rack-cors) for details on how to customize this configuration. But this is good enough for now.

## Running the Servers in Dev

[Foreman](https://github.com/ddollar/foreman) is a useful tool for starting multiple services at once. Install it globally if you haven't before:

```sh
gem install foreman
```

Then create a `Procfile.dev` in your app root directory with the following:

```
api: bundle exec rails server -p 3000
frontend: cd ember && ember serve -p 4200
```

We run both Rails and Ember using their normal server commands. But note that we explicitly pass in the apps' default ports of 3000 for Rails and 4200 for Ember. This is because Foreman automatically assigns its own ports to processes, starting at 5000 and increasing by 100 for each separate process type. I prefer to keep Rails and Ember running on their default ports; that way if we want to run them individually outside of Foreman they'll still work.

I also like to create a `bin` script to make it easy to run my servers. Create a `bin/serve` file and add the following:

```sh
#!/usr/bin/env bash

foreman start -f Procfile.dev
```

Then make it executable:

```sh
chmod +x bin/serve
```

Now you can run your servers by running `bin/serve`.

One note is that Ember can take a while to build, but Foreman's output doesn't show Ember's "building" animation. Just be patient and wait until the logs indicate that both Rails and Ember are ready.

## Building for Production

For Rails to serve our frontend assets in production, the built assets need to be in our Rails app's `/public/` folder. That folder is usually tracked by version control, but in Ember we usually don't commit our built assets to version control. Because we want Ember to completely own the public assets that are shown for our site, we can delete the `/public/` folder out of source control entirely.

```sh
rm -fr public
git add public
git commit -m "Remove public folder from source control"
```

We should also add `public` to our `.gitignore` file to prevent it from being re-added in the future.

Next, let's configure Ember to build our assets to the `public` folder. Add the following to `.ember-cli`:

```diff
 {
-  "disableAnalytics": false
+  "disableAnalytics": false,
+  "output-path": "../public"
 }
```

Next, let's set up a script to run the build. Create a `bin/production` with the following:

```sh
#!/usr/bin/env bash

cd ember
ember build
cd ..
bundle exec rails db:migrate
bundle exec rails server
```

Note that we also run our DB migration, because that'll be helpful in production.

Next, we need to configure Rails to server our Ember `index.html` page for all routes that aren't otherwise matched by Rails routes. This is so that users can enter paths other than the root of the site and still get the Ember app.

Let's create a `frontend_controller.rb` and add the following:

```ruby
# frozen_string_literal: true

class FrontendController < ApplicationController
  def index
    path = File.join(Rails.root, 'public', 'index.html')
    html = File.read(path)
    render html: html.html_safe
  end
end
```

This will serve the contents of the `index.html` file.

Now the magic happens — we set up our routes to use this controller action as a default, if no other route is matched. Add the following to your `routes.rb` file:

```diff
 Rails.application.routes.draw do
   # ...
+  get '*frontend_path', to: 'frontend#index'
 end
```

The `*` means that any routes not otherwise matched will be matched by this route. The route is saved to the variable `frontend_path`, but note that we don't actually need to do anything with that on the server side. The Ember app running in the browser will detect the browser's URL and use that to serve up the right Ember route.

Now let's run our app and try it! Run `bin/production`, then go to `http://localhost:3000`. Your Ember app should come up and run just fine. Now try manually typing in a path other than the root of the site; it should work too.
