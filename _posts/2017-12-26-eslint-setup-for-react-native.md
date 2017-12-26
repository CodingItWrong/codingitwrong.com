---
title: Setting Up ESLint in a React Native Project
---

I was a bit surprised that the React Native project created by `react-native-cli` doesn't include `eslint` support by default. I usually like the guidance of code linters, and I'm new enough to modern JS that my code formatting ends up inconsistent without some guidance. Here are the steps I went through to add ESLint to a React Native project.

I installed ESLint by running `yarn add --dev eslint babel-eslint`

Then I set up the initial ESLint settings by running `./node_modules/.bin/eslint --init` and giving the following answers:

- Use the [Airbnb style guide](https://github.com/airbnb/javascript), because I’ve heard good things about it.
- Indicated I use React.
- Went with JavaScript for the config file format. (Hooray for comments!)

Then, set up an NPM script in `package.json` to run the linter:

```diff
   "scripts": {
     "start": "node node_modules/react-native/local-cli/cli.js start",
     "test": "jest",
+    "lint": "eslint **/*.js"
   },
```

(You'll have to find a pattern to match all your JS files. This only matches ones in subdirectories, not the root.)

For some reason after this I had to `yarn install` again. Maybe because `eslint --init` installs stuff with `npm` instead, so Yarn's lockfile was not updated to indicate it's accessible.

I had to make the following manual additions to the generated `.eslintrc.js` file:

- `"parser": "babel-eslint"`: enables the latest ES features, like [class properties](https://babeljs.io/docs/plugins/transform-class-properties/).
- `"env": { "jest": true }`: let ESLint know I was using Jest, so it wasn’t confused about the `it()` function

Then I can run `yarn lint` to lint my code. Many of the errors it showed were inconsistencies for me to fix. But to match the style I currently like for my code, I had to make the following rule changes to `.eslintrc.js`. For now, I preferred not to have eslint fix them automatically; I wanted to think through them to see if I liked the rule, and to learn how to write my code more consistently.

- `"semi": 0,` - disallow semicolons. I'm trying this out lately.
- `"react/jsx-wrap-multilines": 0,` - along the same lines of playing fast-and-loose with ASI, I’m avoiding parents for multiline JSX expressions unless there’s a parsing error (as in the case of a `return` statement.
- `"react/jsx-filename-extension": 0,` - uh yeah no, component files still have a `.js` extension
- `"no-use-before-define": 0,` - allows me to make the component the topmost element in a module file, and define a `styles` object further down. Without this setting I have to put the `styles` object first, which makes it harder to see the component.
- `"react/prop-types": 0,` - I'm not currently using props validation; I haven’t felt a need for it yet.

In the end, this was my `.eslintrc.js` file:

```javascript
module.exports = {
  "extends": "airbnb",
  "parser": "babel-eslint",
  "env": {
    "jest": true,
  },
  "rules": {
    "semi": 0,
    "react/jsx-wrap-multilines": 0,
    "react/jsx-filename-extension": 0,
    "no-use-before-define": 0,
    "react/prop-types": 0,
  },
};
```
