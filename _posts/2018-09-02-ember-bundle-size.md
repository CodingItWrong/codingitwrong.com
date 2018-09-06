---
title: "The Size of Ember"
---

There's sometimes a perception that Ember is huge and bloated. Is that correct? In Tom and Yehuda's 2018 EmberConf keynote they indicated that Ember can be slimmed down to 120 KB. But when running my Ember apps it seemed larger to me. I wanted to see for myself.

![](/img/posts/ember-bundle-size/ember-120kb.jpg)

Let's see how we can measure the size of our Ember app bundle, find out what's contributing to it, and slim it down as much as possible.

## Measuring Bundle Size

Make sure you have `ember-cli` 3.4.1 or above installed.

```sh
$ npm install -g ember-cli
$ ember --version
ember-cli: 3.4.1
```

We'll also install a tool called [broccoli-concat-analyser](https://github.com/stefanpenner/broccoli-concat-analyser) to investigate our bundle size. [Broccoli](http://broccolijs.com/) is the build tool Ember uses. [broccoli-concat](https://github.com/broccolijs/broccoli-concat) is a Broccoli plugin that bundles multiple files together. And "analyser" is how British people spell "analyzer" ;-)

```sh
$ npm install -g broccoli-concat-analyser
```

Now let’s create a new Ember project:

```sh
$ ember new ember-bundle-size
```

An Ember project includes everything out of the box to run an Ember app, so we can go ahead and do a build to see how big it is.

Run an Ember build with a few extra options:

```sh
$ CONCAT_STATS=true ember build --environment production
```

- `CONCAT_STATS=true` outputs statistics on the bundle that will allow us to analyze it.
- `—environment production` specifies that this should be a production build (TODO what is the specific impact?)

When the build finishes you should see sizes for the different files created. It should be something like this, although your numbers may differ slightly:

```
Built project successfully. Stored in "dist/".
File sizes:
 - dist/assets/ember-bundle-size-781ce0d07d2d05e5a86dfb1a7f131f16.js: 4.62 KB (1.33 KB gzipped)
 - dist/assets/ember-bundle-size-d41d8cd98f00b204e9800998ecf8427e.css: 0 B
 - dist/assets/vendor-a45221a9eeb07b3b5a8039746fd68d27.js: 721.79 KB (189.95 KB gzipped)
 - dist/assets/vendor-d41d8cd98f00b204e9800998ecf8427e.css: 0 B
```

`vendor.js` is what we care about here because it’s where the library code lives. It's 721 KB ungzipped (whew!), 189 KB gzipped. Now let's see what's contributing to that size.

At the root of your project you should see a `concat-stats-for` folder. Run `broccoli-concat-analyser ./concat-stats-for`. When it finishes processing, it should give you a `file:///` URL you can open in your browser.

![](/img/posts/ember-bundle-size/start.png)

This says:

- My overall `vendor.js` file is 188.79 KB gzipped
- 119.02 KB of that is Ember
- 22.1 KB of that is jQuery
- 36.87 KB of that is ember-data

Let’s see if we can slim some of this down!

## Removing Dependencies

One thing contributing to the size is Ember Data. You don't have to use Ember Data though; you can make basic `fetch` or `axios` requests, use `ember-redux`, or even use Firebase or Apollo. Because your data layer is so flexible, there’s no real way to compare it across frameworks, and you aren’t locked in—you can always change it. So for a more parallel comparison to React we should remove Ember Data and just count `ember.js` itself.

Removing Ember Data is easy; we can just run `npm remove ember-data`. Rerun the build and our gzipped size is only 157 KB. Rerun the analysis and we can confirm that Ember Data is no longer included.

![](/img/posts/ember-bundle-size/no-ember-data.png)

The other big piece in the bundle is jQuery. When Ember was released in 2011 cross-browser compatibility was terrible, and jQuery handled a lot of those issues. But Ember has been updated to no longer depend on jQuery as of version 3.4, so it can be removed.

To do so, we need to set an Ember feature flag. Run `ember feature:list` and you can see that `jquery-integration` is enabled. To disable it, run `ember feature:disable jquery-integration`.

Now we can remove the jQuery module. Run `npm remove @ember/jquery`. Run the build again and our bundle size is only 128 KB. Run the analyzer and we can see that ember.js itself is the bulk of the size.

![](/img/posts/ember-bundle-size/no-jquery.png)

So, as of today you can get the core of your app bundle down to 128 KB. Only you can decide if that's plenty small enough for your needs or if it's too big—do your research and test. And don't forget to include all the libraries you'll need to include in your app; Ember includes a lot out of the box that you'd need to add other packages for with other frameworks.

There are a few initiatives underway to decrease the size of the initial Ember bundle even further. Svelte builds will strip out unused code. Code splitting will allow code not needed for the initial page render to be deferred until later.
