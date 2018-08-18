---
title: Setting Up ESLint in a React Native Project
tags: [javascript]
---

*Updated 2018-06-16: updated my rule overrides, improved config file formatting, and added a link to my eslint preset.*

I was a bit surprised that the React Native project created by `react-native-cli` doesn't include `eslint` support by default. I usually like the code consistency that linters provide, and in JavaScript linting is even more important to keep you away from hard-to-spot errors. Here are the steps I went through to add ESLint to a React Native project.

I installed ESLint by running `yarn add --dev eslint babel-eslint`

Then I set up the initial ESLint settings by running `./node_modules/.bin/eslint --init` and giving the following answers:

- Use the [Airbnb style guide](https://github.com/airbnb/javascript), because I like that it’s detailed and includes some React recommendations. (It's a bit restrictive, though. If you get frustrated with it making it tedious for you to make changes to your code, try my own preset, [`eslint-config-codingitwrong`](https://github.com/CodingItWrong/eslint-config-codingitwrong).)
- Indicated I use React.
- Went with JavaScript for the config file format. (Hooray for comments!)

Then, set up an NPM script in `package.json` to run the linter:

```diff
   "scripts": {
     "start": "node node_modules/react-native/local-cli/cli.js start",
     "test": "jest",
+    "lint": "eslint *.js **/*.js"
   },
```

After this I had to `yarn install` again. This is likely because `eslint --init` installs stuff with `npm` instead, so Yarn's lockfile was not updated to indicate it's accessible.

I had to make the following manual additions to the generated `.eslintrc.js` file:

- `'parser': 'babel-eslint'`: enables the latest ES features, like [class properties](https://babeljs.io/docs/plugins/transform-class-properties/).
- `'env': { 'jest': true }`: let ESLint know I was using Jest, so it wasn’t confused about the `it()` function

Then I can run `yarn lint` to lint my code. Many of the errors it showed were inconsistencies for me to fix. But to match the style I currently like for my code, I had to make the following rule changes to `.eslintrc.js`. For now, I preferred not to have eslint fix them automatically; I wanted to think through them to see if I liked the rule, and to learn how to write my code more consistently.

- `'no-use-before-define': 'off',` - allows me to make the component the topmost element in a module file, and define a `styles` object further down. Without this setting I have to put the `styles` object first, which makes it harder to see the component.
- `'react/jsx-filename-extension': 'off',` - uh yeah no, component files still have a `.js` extension.
- `'react/prop-types': 'off',` - I'm not currently using props validation; I haven’t felt a need for it yet.

In the end, this was my `.eslintrc.js` file:

```javascript
module.exports = {
  'extends': 'airbnb',
  'parser': 'babel-eslint',
  'env': {
    'jest': true,
  },
  'rules': {
    'no-use-before-define': 'off',
    'react/jsx-filename-extension': 'off',
    'react/prop-types': 'off',
  },
};
```
