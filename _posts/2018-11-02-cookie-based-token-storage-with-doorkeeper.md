---
title: Cookie-Based Token Storage with Doorkeeper
tags: [ruby]
---

When a frontend webapp authenticates against an API, where should the access token be stored?

When we're talking about API keys for a third-party service, the most secure solution is to store the API keys on the server, and control access to them with an access token for your application that is user-specific. But this just moves the problem back a level. What do we do about our *application-specific* access tokens?

It's common practice to store it in browser local storage. But this is generally considered insecure because of the risk of Cross-Site Scripting (XSS) attacks. If a malicious third party can get JavaScript executed on your domain, it can steal the access tokens. Instead, it’s generally recommended to store access tokens in `HttpOnly` cookies that aren’t accessible via JavaScript. While it’s true that these cookies are potentially vulnerable to Cross-Site Request Forgery (CSRF) attacks, the risk of that can be mitigated. See Auth0's blog post ["Where to Store Tokens"](https://auth0.com/docs/security/store-tokens) for more information.

Let’s walk through setting up cookie-based access tokens in a Ruby API using [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper). Note that by making this change our response is no longer following [the OAuth2 access token response format](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/), which would be a problem if you were planning on using any other OAuth2 libraries to integrate with it.

**WARNING: I am not a security professional and this code has not been deployed to production.** I can’t guarantee that it’s free from security errors. Do your own security analysis before deploying a solution like this to production.

First, we’ll change the auth response to include the access token in a cookie instead of in the body. Create a new file `lib/cookie_token_response.rb` and add the following:

```ruby
module CookieTokenResponse
  def body
    super.except('access_token', 'token_type')
  end
end
```

This removes the `access_token` and `token_type` fields from the response body.

Now we’ll instruct the browser to store a cookie using the `Set-Cookie` response header. Add the following method to `CookieTokenResponse`:

```ruby
  def headers
    cookie_args = [
      "access_token=#{token.token}",
      'Path=/',
    ]
    cookie = cookie_args.join('; ')
    super.merge({'Set-Cookie' => cookie})
  end
```

Setting `Path=/` is necessary to get the token to be sent back with future requests. I believe the default behavior for cookies when `Path` is not specified is to only send the cookie back for the same path, so it would not be sent back for other endpoints in the API.

This is enough to get the cookie working, but let’s add a few security settings to it.

Usually, cookies accessible to a given page are accessible in JavaScript via reading `document.cookies`. This puts cookies at risk of Cross-Site Scripting (XSS) attacks. If we are storing the access token in a cookie for security, we need to add the `HttpOnly` argument to the cookie or else we won’t be accomplishing anything. The `HttpOnly` argument prevents the cookie from being accessed via JavaScript; the cookie will only be used for sending in appropriate server requests.

```ruby
cookie_args = [
  "access_token=#{token.token}",
  'Path=/',
  'HttpOnly',
]
```

You can check that this is working by running `document.cookie` in your web developer tools console and confirming it doesn’t output the cookie.

Another important setting is `Secure`, which will prevent the cookie from being sent over snoopable HTTP requests; it will only be sent via HTTPS requests. Ideally your frontend and backend will be configured to use HTTPS exclusively, but adding this flag protects from misconfigurations. Oftentimes we don’t run HTTPS locally in development, though. In that case, we can add the `Secure` flag only in production.

```ruby
cookie_args = [
  "access_token=#{token.token}",
  'Path=/',
  'HttpOnly',
]
if Rails.env.production?
  cookie_args.push('Secure')
end
```

When we implement access token checking shortly we’ll put in a few preventions for CSRF. There’s one we can add in the cookie assignment, too, though. If we are running our frontend and backend on the same server, we can set `SameSite=strict` to prevent the cookie being sent on a request initiated on a page that’s on a different site:

```ruby
cookie_args = [
  "access_token=#{token.token}",
  'Path=/',
  'HttpOnly',
  'SameSite=strict',
]
if Rails.env.production?
  cookie_args.push('Secure')
end
```

You may not easily be able to set up your frontend and backend on the same server, though, in which case you’ll need to leave this setting off.

Next we need to configure Doorkeeper to use this module. In `config/initializers/doorkeeper.rb` add the following *outside* of the `Doorkeeper.configure` block:

```ruby
Doorkeeper::OAuth::TokenResponse.send :prepend, CookieTokenResponse
```

This will add the module to the `TokenResponse` class so its methods will be run when sending a token back to the user.

We will also need to autoload the `lib` folder if we aren’t doing so already. Add the following in `config/application.rb` inside the `Application` class:

```ruby
config.autoload_paths << "#{Rails.root}/lib"
```

Now we need to change how the access token is detected for authentication. We can do this using the `access_token_methods` method in the Doorkeeper config. Read the cookie like so:

```ruby
access_token_methods lambda { |request|
  request.cookies['access_token']
}
```

This is where we can add some extra CSRF safety as well. We should check the Origin and Referer headers to confirm the request is coming from a valid domain. In our case, our frontend is running locally on port 3000, so we can write:

```ruby
access_token_methods lambda { |request|
  return nil if request.headers['HTTP_ORIGIN'] != 'http://localhost:3000'
  return nil if request.headers['HTTP_REFERER'] != 'http://localhost:3000/'
  request.cookies['access_token']
}
```

(Note that the Referer header has an extra `/` at the end.)

Next, let's update our frontend app. We just need to make one config change: to configure our Ajax request to send authentication cookies. We're using the Axios library, so we can configure it like this:

```diff
 const api = axios.create({
   baseURL: 'https://api.example.com',
+  withCredentials: true,
 });
```

We're using the `rack-cors` gem to allow cross-origin requests. We can configure it to accept credentials from other origins like this:

```diff
 allow do
   origins '*'

   resource '*',
     headers: :any,
-    methods: [:get, :post, :put, :patch, :delete, :options, :head]
+    methods: [:get, :post, :put, :patch, :delete, :options, :head],
+    credentials: true
 end
```

Once we do this, `rack-cors` will prevent our app from starting with the following error:

> Allowing credentials for wildcard origins is insecure. Please specify more restrictive origins or set 'credentials' to false in your CORS configuration.

This is fine because we're already checking for specific domains in the Origin and Referer headers. Let's check for the same domain here:

```diff
 allow do
-  origins '*'
+  origins 'localhost:3000'
```

Now when we send our request in the app it should succeed.

One other note: in Chrome developer tools, if we are running both our frontend and our API on localhost, we’ll see the `Set-Cookie` header in the auth response, and the `Cookie` header in the data requests. This is because both are on the same domain, `localhost`—the different ports don’t matter from this standpoint. If we update our app to point to an API on a different domain (say, local frontend pointing to production backend), we won’t see either the `Set-Cookie` response header or `Cookie` request headers. This is because they’re running on different domains.
