---
title: "Await Off My Shoulders: Enabling Async/Await in Webpack"
tags: [javascript]
---

I had a little trouble setting up `async`/`await` support in Webpack via the Babel Loader, so I wanted to write out what I found in case it helps anyone else.

Let's start with a simple Webpack config to do Babel transpilation:

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');

module.exports = {
  entry: {
    app: './src/index.js',
  },
  plugins: [
    new HtmlWebpackPlugin(),
  ],
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['env'],
          },
        },
      },
    ],
  },
};

```

And the following simple `async`/`await` code:

```js
async function fetchPost() {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts/1');
  return response.json();
}

fetchPost('https://google.com')
  .then(content => console.log(content));
```

The `env` preset results in the `async`/`await` code transpiling successfully. But when you try to run it in-browser, you get this error:

```
ReferenceError: regeneratorRuntime is not defined
```

## Polyfill

The recommended way to provide a `regeneratorRuntime` for apps is to use [`babel-polyfill`](https://babeljs.io/docs/usage/polyfill/):

```sh
npm install --save babel-polyfill
```

At first I tried importing it at the top of my script:

```diff
+import('babel-polyfill');
+
async function fetchPost() {
...
```

But I still got the same error. What fixed it was including 'babel-polyfill' in the list of endpoints, rather than importing it. This may be related to the requirement for the polyfill to be loaded as early as possible, before *any* other code:

```diff
 entry: {
+  polyfill: 'babel-polyfill',
   app: './src/index.js',
 },
```

## Transform Runtime

An alternative to `babel-polyfill` is using [babel-plugin-transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/). Apparently this is more useful for libraries than for apps.

```sh
npm install --save-dev babel-plugin-transform-runtime
```

Then adding to my Babel config:

```diff
 options: {
   presets: ['env'],
+  plugins: [
+    ["transform-runtime", {
+      "regenerator": true,
+    }],
+  ],
},
```

## Other Options

There are a few other alternatives for providing a `regeneratorRuntime` listed under [`syntax-async-functions`](https://babeljs.io/docs/plugins/syntax-async-functions), but none of them seemed to help me. Here they are for completeness:

- [`transform-regenerator`](https://babeljs.io/docs/plugins/transform-regenerator/): This says it still requires the Babel polyfill or the regenerator runtime, so it doesn't gain us anything.

- [`transform-async-to-generator`](https://babeljs.io/docs/plugins/transform-async-to-generator): I still get the same `regeneratorRuntime` error.

- [`transform-async-to-module-method`](https://babeljs.io/docs/plugins/transform-async-to-module-method/): This requires the Bluebird library for promises. If I'm going to need a runtime library, I'd prefer to use `babel-polyfill` rather than a third-party library.
