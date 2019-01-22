---
title: React Navigation Quick Reference
---

*Updated 2019-01-22: Updated to React Navigation 3.0*


When it comes to navigation libraries for React Native, I like [React Navigation](https://reactnavigation.org/) a lot. Their docs are great, but because I'm not in it every day I often find myself combing through their docs repeatedly to figure out how to use the same basic features. To save me (and hopefully you!) some time in the future, here's a combination of the code snippets from their docs I consult the most to get basic stack navigation working.

Note that according to [this issue](https://github.com/react-navigation/react-navigation/issues/3326#issuecomment-392380151) it seems as though the navigator and screen components need to be defined in separate files.

`App.js`

```js
import { createStackNavigator, createAppContainer } from 'react-navigation';
import HomeScreen from './HomeScreen';
import DetailsScreen from './DetailsScreen';

const RootStack = createStackNavigator({
  Home: HomeScreen,
  Details: DetailsScreen,
});

export default createAppContainer(RootStack);
```

`HomeScreen.js`

```jsx
import React from 'react';
import { View, Text, Button } from 'react-native';

export default class HomeScreen extends React.Component {
  static navigationOptions = {
    title: 'Home',
  };

  render() {
    return (
      <View>
        <Text>Home Screen</Text>
        <Button
          title="Go to Details"
          onPress={() => {
            this.props.navigation.navigate('Details', {
              itemId: 86,
              otherParam: 'anything you want here',
            });
          }}
        />
      </View>
    );
  }
}
```

`DetailsScreen.js`

```jsx
import React from 'react';
import { View, Text, Button } from 'react-native';

export default class DetailsScreen extends React.Component {
  static navigationOptions = ({ navigation }) => {
    return {
      title: navigation.getParam('otherParam', 'A Nested Details Screen'),
    };
  };

  render() {
    return (
      <View>
        <Text>Details Screen</Text>
        <Button
          title="Go back"
          onPress={() => this.props.navigation.goBack()}
        />
      </View>
    );
  }
}
```
