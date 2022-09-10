---
title: Building for Web and Mobile with Expo
---

<https://docs.expo.dev/get-started/create-a-new-app/>

```bash
$ yarn create expo-app expo-workshop-app
```

## RN Web

If you try to run web you get warned to install the following:

```bash
$ npx expo install react-native-web@~0.18.7 \
                   react-dom@18.0.0 \
                   @expo/webpack-config@^0.17.0
```

## ESLint and Prettier

```bash
$ yarn add --dev eslint \
                 @react-native-community/eslint-config \
                 prettier
```

```js
// .eslintrc.js
module.exports = {
  root: true,
  extends: '@react-native-community',
  rules: {
    'react/jsx-uses-react': 'off',
    'react/react-in-jsx-scope': 'off',
  },
};
```

```js
// .prettierrc.js
module.exports = {
  arrowParens: 'avoid',
  bracketSpacing: false,
  singleQuote: true,
  trailingComma: 'all',
};
```

## React Navigation

```bash
$ yarn add @react-navigation/native \
           @react-navigation/drawer \
           @react-navigation/native-stack
$ npx expo install react-native-gesture-handler \
                   react-native-reanimated \
                   react-native-screens \
                   react-native-safe-area-context
```

# RN Paper

```bash
$ yarn add react-native-paper@5.0.0-rc.4
```

## Responsive Design

```bash
$ yarn add react-native-style-queries
```
