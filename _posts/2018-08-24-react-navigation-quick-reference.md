---
title: React Navigation Quick Reference
---

When it comes to navigation libraries for React Native, I like [React Navigation](https://reactnavigation.org/) a lot. Their docs are great, but because I'm not in it every day I often find myself combing through their docs repeatedly to figure out how to use the same basic features. To save me (and hopefully you!) some time in the future, here's a combination of the code snippets from their docs I consult the most to get basic stack navigation working. (Imports omitted.)

```jsx
export default class App extends React.Component {
  render() {
    return <RootStack />;
  }
}

const RootStack = createStackNavigator({
  Home: HomeScreen,
  Details: DetailsScreen,
});

class HomeScreen extends React.Component {
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

class DetailsScreen extends React.Component {
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
