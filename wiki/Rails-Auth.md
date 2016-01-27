---
title: Rails Auth
---

The following are popular gems for Rails authentication and authorization.

As a reminder, authentication means determining whether the person making the request is really a certain user, and authorization means determining what that user is allowed to do.

* [**Devise**](https://github.com/plataformatec/devise) - a high-level Rails authentication framework, including full Rails engine features (model mixins, views, controllers)
* [**Doorkeeper**](https://github.com/doorkeeper-gem/doorkeeper) - allows your Rails app to be an OAuth 2 provider; i.e. your own user store is made available through the OAuth API
* [**OmniAuth**](https://github.com/intridea/omniauth) - allows authentication via a number of third-party services, some OAuth and some otherwise
* [**Pundit**](https://github.com/elabs/pundit) - allows easily configuring authorization via policy classes
* [**Warden**](https://github.com/hassox/warden) - a low-level Rack authentication framework, suitable for web services

Some relationships between these gems include:

* Pundit can be used with any of the others, because it's the only authorization library listed; the others are all authentication.
* If you're making a web service:
  * Use Doorkeeper if you want people to authenticate to it with OAuth
  * Use Warden if you want them to authenticate in a different way
* If you're making a webapp:
  * Use Devise alone if you are only logging in to your own user table
  * Use OmniAuth alone if you're only authenticating against third parties, and you're okay building screens
  * Use Devise with its OmniAuth integration if you're allowing signup with email or with third parties, or if you don't want to build screens
