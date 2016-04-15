---
title: A Definitive Guide to Asset Pipeline Settings
---

I ran across an issue in production recently where my Rails app assets weren't being served correctly through the Asset Pipeline. It brought back to my attention the fact that I didn't really understand how all of the Asset-Pipeline-related configuration settings work together.

I got so frustrated that I decided to write out a truth table and test out all possible combinations, and I'd like to share my findings. Hopefully this will help you better understand error messages you may receive, and maybe even show you a setup you'd like to use.

Here's the tl;dr for if you're getting a certain error:

- If you're getting a 404 for assets at an `/assets/` path, make sure you have `config.serve_static_files = true`, or that you are running inside a web server that will serve these files.
- If you're getting a 404 for assets at a `/stylesheets/` or `/javascripts/` paths, this is because you've set Rails not to compile assets at runtime (`config.assets_compile = false`), and some other settings are incorrect. If you want assets to be precompiled, make sure all your settings match row 6 above.

Here are the settings I investigated:

- **`config.assets.compile`**: allows Rails to generate assets at runtime.
- **`config.assets.debug`**: disables concatenating files, instead creating separate `<link>` and `<script>` tags for each.
- **`config.assets.digest`**: appends fingerprints to asset URLs to bust cache.
- **`config.serve_static_files`**: allows Rails to serve up files in `/public/`, including precompiled assets.
- **Running `rake assets:precompile`**: generates assets at build time and stores them in `/public/`.

Here are the effects of enabling and disabling these settings in different combinations, as to whether the asset loads (HTTP status 200) or can't be found (404):

<table>
  <thead>
    <tr>
      <th>#</th>
      <th title="config.assets.compile">compile</th>
      <th title="config.assets.debug">debug</th>
      <th title="config.assets.digest">digest</th>
      <th title="config.serve_static_files">serve</th>
      <th title="run rake assets:precompile">precompile</th>
      <th title="whether the browser is able to load the assets">status</th>
      <th title="the path(s) to the asset(s) generated">path(s)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>true</td>
      <td>true</td>
      <td>true</td>
      <td>(either)</td>
      <td>(either)</td>
      <td class="works">200</td>
      <td><code>/assets/jquery.self-FINGERPRINT.js?body=1<br />/assets/jquery_ujs.self-FINGERPRINT.js?body=1<br />/assets/turbolinks.self-FINGERPRINT.js?body=1<br />/assets/application.self-FINGERPRINT.js?body=1</code></td>
    </tr>
    <tr>
      <td>2</td>
      <td>true</td>
      <td>true</td>
      <td>false</td>
      <td>(either)</td>
      <td>(either)</td>
      <td class="works">200</td>
      <td><code>/assets/jquery.self.js?body=1<br />/assets/jquery_ujs.self.js?body=1<br />/assets/turbolinks.self.js?body=1</code></td>
    </tr>
    <tr>
      <td>3</td>
      <td>true</td>
      <td>false</td>
      <td>true</td>
      <td>(either)</td>
      <td>(either)</td>
      <td class="works">200</td>
      <td><code>/assets/application-FINGERPRINT.js</code></td>
    </tr>
    <tr>
      <td>4</td>
      <td>true</td>
      <td>false</td>
      <td>false</td>
      <td>(either)</td>
      <td>(either)</td>
      <td class="works">200</td>
      <td><code>/assets/application.js</code></td>
    </tr>
    <tr>
      <td>5</td>
      <td>false</td>
      <td>true</td>
      <td>(either)</td>
      <td>(either)</td>
      <td>(either)</td>
      <td class="does-not-work">404</td>
      <td><code>/stylesheets/application.js</code></td>
    </tr>
    <tr>
      <td>6</td>
      <td>false</td>
      <td>false</td>
      <td>true</td>
      <td>true</td>
      <td>true</td>
      <td class="works">200</td>
      <td><code>/assets/application-FINGERPRINT.js</code></td>
    </tr>
    <tr>
      <td>7</td>
      <td>false</td>
      <td>false</td>
      <td>true</td>
      <td>true</td>
      <td>false</td>
      <td class="does-not-work">404</td>
      <td><code>/stylesheets/application.js</code></td>
    </tr>
    <tr>
      <td>8</td>
      <td>false</td>
      <td>false</td>
      <td>true</td>
      <td>false</td>
      <td>true</td>
      <td class="depends">depends</td>
      <td><code>/assets/application-FINGERPRINT.js</code></td>
    </tr>
    <tr>
      <td>9</td>
      <td>false</td>
      <td>false</td>
      <td>true</td>
      <td>false</td>
      <td>false</td>
      <td class="does-not-work">404</td>
      <td><code>/stylesheets/application.js</code></td>
    </tr>
    <tr>
      <td>10</td>
      <td>false</td>
      <td>false</td>
      <td>false</td>
      <td>(either)</td>
      <td>(either)</td>
      <td class="does-not-work">404</td>
      <td><code>/stylesheets/application.js</code></td>
    </tr>
  </tbody>
</table>

A few observations:

- If you have `config.assets.compile = true`, every other combination of settings works. `config.serve_static_files` and `rake assets:precompile` don't make a difference because the application will always fall back to compiling the assets on the fly. But you can change `config.assets.debug` and `config.assets.digest` to control concatenation and fingerprinting, respectively.
- If you have `config.assets.compile = false`, there are only two combinations of settings that work. `config.assets.debug` must be set to `false`, `config.assets.digest` must be set to `true`, and you must run `rake assets:precompile`. Your setting for `config.serve_static_files` is the only one that can vary, and the setting you want depends on how you're running Rails. If it's running behind a web server like Apache or Nginx, you can set `config.serve_static_files` to false, because the web server will serve up those files. If Rails is not running behind a web server, you will need to set `config.serve_static_files` to true, so that Rails will serve these files.
- Note that [Heroku precompiles your assets](https://devcenter.heroku.com/articles/rails-asset-pipeline#the-rails-3-asset-pipeline) if it doesn't already see the compiled assets committed to Git. If you move off of Heroku to another host, you may need to start precompiling yourself.

If you've seen the Asset Pipeline behave differently or there's another setting I should be testing against, let me know!
