---
title: Building for Web and Mobile with Expo
---

<https://docs.expo.dev/get-started/create-a-new-app/>

{% raw %}
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

Add the dependencies for React Navigation, including both the drawer and stack navigator:

```bash
$ yarn add @react-navigation/native \
           @react-navigation/drawer \
           @react-navigation/native-stack
$ npx expo install react-native-gesture-handler \
                   react-native-reanimated \
                   react-native-screens \
                   react-native-safe-area-context
```

We will also need to configure Babel to support the Reanimated library:

```diff
   return {
     presets: ['babel-preset-expo'],
+    plugins: ['react-native-reanimated/plugin'],
   };
```

Create a file `src/Navigation.js`. First, let's create a few sample screens to navigate between:

```jsx
import {useNavigation} from '@react-navigation/native';
import {Pressable, Text} from 'react-native';

function HomeRoot() {
  const navigation = useNavigation();
  return (
    <>
      <Text>HomeRoot</Text>
      <Pressable onPress={() => navigation.navigate('HomeDetail')}>
        <Text>Go to Detail</Text>
      </Pressable>
    </>
  );
}

function HomeDetail() {
  const navigation = useNavigation();
  return (
    <>
      <Text>HomeDetail</Text>
      <Pressable onPress={() => navigation.pop()}>
        <Text>Back to HomeRoot</Text>
      </Pressable>
    </>
  );
}

const OtherRoot = () => (
  <>
    <Text>OtherRoot</Text>
  </>
);
```

Now let's create two stack navigators, one for each set:

```jsx
import {createNativeStackNavigator} from '@react-navigation/native-stack';
//...
const HomeStack = createNativeStackNavigator();
const Home = () => (
  <HomeStack.Navigator>
    <HomeStack.Screen
      name="HomeRoot"
      component={HomeRoot}
      options={{title: 'Home'}}
    />
    <HomeStack.Screen
      name="HomeDetail"
      component={HomeDetail}
      options={{title: 'Home Detail'}}
    />
  </HomeStack.Navigator>
);

const OtherStack = createNativeStackNavigator();
const Other = () => (
  <OtherStack.Navigator>
    <OtherStack.Screen
      name="OtherRoot"
      component={OtherRoot}
      options={{title: 'Other'}}
    />
  </OtherStack.Navigator>
);
```

Now, we combine them in a Drawer:

```jsx
import {createDrawerNavigator} from '@react-navigation/drawer';
//...
const Drawer = createDrawerNavigator();

function NavigationContents() {
  return (
    <Drawer.Navigator useLegacyImplementation>
      <Drawer.Screen name="Home" component={Home} />
      <Drawer.Screen name="Other" component={Other} />
    </Drawer.Navigator>
  );
}
```

To finish up this file for now, we wrap it in a `NavigationContainer` in a separate component:

```diff
-import {useNavigation} from '@react-navigation/native';
+import {NavigationContainer, useNavigation} from '@react-navigation/native';
...
+export default function Navigation() {
+  // IMPORTANT: NavigationContainer needs to not rerender too often or
+  // else Safari and Firefox error on too many history API calls. Put
+  // any hooks in NavigationContents so this parent doesn't rerender.
+  return (
+    <NavigationContainer>
+      <NavigationContents />
+    </NavigationContainer>
+  );
+}
```

Now we render this `Navigation` in our App component. Replace the contents of `App.js` with:

```jsx
import {StatusBar} from 'expo-status-bar';
import Navigation from './src/Navigation';

export default function App() {
  return (
    <>
      <StatusBar style="auto" />
      <Navigation />
    </>
  );
}
```

## URLs

On the web we expect each screen to correspond to a URL. URLs can be useful for deep linking into screens of the app as well.

Set up the linking mapping in `Navigation.js`:

```js
const linking = {
  config: {
    screens: {
      Home: {
        path: '/',
        screens: {
          HomeRoot: '',
          HomeDetail: 'home/detail',
        },
      },
      Other: {
        path: '/other',
        screens: {
          OtherRoot: '',
        },
      },
    },
  },
};
```

Configure the `NavigationContainer` with it:

```diff
   return (
-    <NavigationContainer>
+    <NavigationContainer linking={linking}>
       <NavigationContents />
     </NavigationContainer>
   );
```

TODO is `initialRouteName` needed in component or linking config?

# RN Paper

Add the Paper dependency:

```bash
$ yarn add react-native-paper@5.0.0-rc.4
```

Next, wrap the app with the Paper provider:

```diff
 import Navigation from './src/Navigation';
+import {Provider as PaperProvider} from 'react-native-paper';

 export default function App() {
   return (
-    <>
+    <PaperProvider>
       <StatusBar style="auto" />
       <Navigation />
-    </>
+    </PaperProvider>
   );
 }
```

To start to style our app, we can configure the navigation bar and drawer to use RN Paper.

The drawer will be a bit simpler to style, so let's start there.

Create the folder `src/components` and create a file `src/components/CustomNavigationDrawer.js`. Add the following:

```jsx
import {DrawerContentScrollView} from '@react-navigation/drawer';
import {Drawer} from 'react-native-paper';

export default function CustomNavigationDrawer({...navProps}) {
  const {state, navigation} = navProps;

  const isSelected = index => index === state.index;

  return (
    <DrawerContentScrollView {...navProps}>
      {state.routes.map((route, index) => (
        <Drawer.Item
          key={route.key}
          label={route.name}
          accessibilityLabel={route.name}
          active={isSelected(index)}
          onPress={() => navigation.navigate(route.name)}
        />
      ))}
    </DrawerContentScrollView>
  );
}
```

Now configure the drawer to use it:

```diff
+import CustomNavigationDrawer from './components/CustomNavigationDrawer';

 function HomeRoot() {
   const navigation = useNavigation();
...
       screenOptions={{
         headerShown: false,
       }}
+      drawerContent={props => <CustomNavigationDrawer {...props} />}
     >
       <Drawer.Screen name="Home" component={Home} />
```

Next, the header. Create a file `src/components/CustomNavigationBar.js`. Add the following:

```jsx
import {Appbar} from 'react-native-paper';

export default function CustomNavigationBar({navigation, options, back}) {
  return (
    <Appbar.Header>
      {back ? (
        <Appbar.BackAction
          testID="back-button"
          onPress={navigation.goBack}
          accessibilityLabel="Back"
        />
      ) : null}
      <Appbar.Content title={options.title} />
      <Appbar.Action
        testID="toggle-navigation-button"
        accessibilityLabel="Menu"
        icon="menu"
        onPress={navigation.toggleDrawer}
      />
    </Appbar.Header>
  );
}
```

Now configure each Stack Navigator to use this header component:

```diff
+import CustomNavigationBar from './components/CustomNavigationBar';
...
 const HomeStack = createNativeStackNavigator();
 const Home = () => (
-  <HomeStack.Navigator>
+  <HomeStack.Navigator
+    screenOptions={{
+      header: props => <CustomNavigationBar {...props} />,
+    }}
+  >
     <HomeStack.Screen
       name="HomeRoot"
       component={HomeRoot}
@@ -51,7 +56,11 @@ const Home = () => (

 const OtherStack = createNativeStackNavigator();
 const Other = () => (
-  <OtherStack.Navigator>
+  <OtherStack.Navigator
+    screenOptions={{
+      header: props => <CustomNavigationBar {...props} />,
+    }}
+  >
     <OtherStack.Screen
```

We also configure the drawer's own header not to show, since our custom navigation bar handles it:

```diff
   return (
-    <Drawer.Navigator useLegacyImplementation initialRouteName="Home">
+    <Drawer.Navigator
+      useLegacyImplementation
+      initialRouteName="Home"
+      screenOptions={{
+        headerShown: false,
+      }}
+    >
       <Drawer.Screen name="Home" component={Home} />
       <Drawer.Screen name="Other" component={Other} />
     </Drawer.Navigator>
```

## Dark Mode

React Native Paper supports dark mode, but by default Expo doesn't allow switching the app to dark mode. We can change that in `app.json`:

```diff
     "icon": "./assets/icon.png",
-    "userInterfaceStyle": "light",
+    "userInterfaceStyle": "automatic",
     "splash": {
```

Our RN Paper components change to dark mode, but some of our other components do not. First, backgrounds. Let's add a `ScreenBackground` component that supports dark mode:

```jsx
import {View} from 'react-native';
import {withTheme} from 'react-native-paper';

function ScreenBackground({theme, style, children}) {
  const baseStyle = {flex: 1, backgroundColor: theme.colors.background};
  return <View style={[baseStyle, style]}>{children}</View>;
}

export default withTheme(ScreenBackground);
```

Let's use this for all our screens:

```diff
+import ScreenBackground from './components/ScreenBackground';

 function HomeRoot() {
   const navigation = useNavigation();
   return (
-    <>
+    <ScreenBackground>
       <Text>HomeRoot</Text>
       <Pressable onPress={() => navigation.navigate('HomeDetail')}>
         <Text>Go to Detail</Text>
       </Pressable>
-    </>
+    </ScreenBackground>
   );
 }

 function HomeDetail() {
   const navigation = useNavigation();
   return (
-    <>
+    <ScreenBackground>
       <Text>HomeDetail</Text>
       <Pressable onPress={() => navigation.pop()}>
         <Text>Back to HomeRoot</Text>
       </Pressable>
-    </>
+    </ScreenBackground>
   );
 }

 const OtherRoot = () => (
-  <>
+  <ScreenBackground>
     <Text>OtherRoot</Text>
-  </>
+  </ScreenBackground>
 );
```

Now our backgrounds change color but our text doesn't. The `Text` component from Paper will help:

```diff
 import {createNativeStackNavigator} from '@react-navigation/native-stack';
-import {Pressable, Text} from 'react-native';
+import {Pressable} from 'react-native';
+import {Text} from 'react-native-paper';
 import CustomNavigationBar from './components/CustomNavigationBar';
```

But now our drawer doesn't change. Let's add that support:

```diff
 import {DrawerContentScrollView} from '@react-navigation/drawer';
-import {Drawer} from 'react-native-paper';
+import {Drawer, withTheme} from 'react-native-paper';

-function CustomNavigationDrawer({...navProps}) {
+function CustomNavigationDrawer({theme, ...navProps}) {
   const {state, navigation} = navProps;

   const isSelected = index => index === state.index;

+  const scrollViewStyle = {
+    backgroundColor: theme.colors.background,
+  };
+
   return (
-    <DrawerContentScrollView {...navProps}>
+    <DrawerContentScrollView style={scrollViewStyle} {...navProps}>
       {state.routes.map((route, index) => (
         <Drawer.Item
           key={route.key}
...
   );
 }

-export default CustomNavigationDrawer;
+export default withTheme(CustomNavigationDrawer);
```

## Theme Color

This looks good, but the theme also looks exactly like every other RN Paper app out there. To give our app a more unique identity, we can set a custom primary theme color.

There's one complication: if we want to customize the theme, then we need to handle light and dark mode switching ourselves. Here's how. Create a file `src/useTheme.js` and add the following:

```js
import {useColorScheme} from 'react-native';
import {MD2DarkTheme, MD2LightTheme} from 'react-native-paper';

export default function useTheme() {
  const colorScheme = useColorScheme() ?? 'light';
  const baseTheme = colorScheme === 'dark' ? MD2DarkTheme : MD2LightTheme;
  return baseTheme;
}
```

Now use this to set the theme on the provider:

```diff
 import {Provider as PaperProvider} from 'react-native-paper';
+import useTheme from './src/useTheme';

 export default function App() {
+  const theme = useTheme();
+
   return (
-    <PaperProvider>
+    <PaperProvider theme={theme}>
       <StatusBar style="auto" />
```

So far, everything looks just the same. But now let's set that theme color. You can pick any vibrant color. Here's what I chose:

```diff
 import {MD2DarkTheme, MD2LightTheme} from 'react-native-paper';

+const THEME_COLOR = '#4caf50';
+
 export default function useTheme() {
   const colorScheme = useColorScheme() ?? 'light';
   const baseTheme = colorScheme === 'dark' ? MD2DarkTheme : MD2LightTheme;
-  return baseTheme;
+
+  const theme = {
+    ...baseTheme,
+    colors: {
+      ...baseTheme.colors,
+      primary: THEME_COLOR,
+    },
+  };
+
+  return theme;
 }
```

Now note that we have a custom color and we still support light and dark mode.

## Responsive Design

Let's take a look at a few ways we can make our app look good at different sizes.

First, the content area is pretty wide. Let's make it a centered column that hsa a maximum width. Let's create a component for this, `src/components/CenterColumn.js`. Add the following:

```jsx
import {StyleSheet, View} from 'react-native';

export default function CenterColumn({columnStyle, children}) {
  return (
    <View style={styles.columnWrapper}>
      <View style={[styles.column, columnStyle]}>{children}</View>
    </View>
  );
}

const styles = StyleSheet.create({
  columnWrapper: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'center',
  },
  column: {
    flex: 1,
    maxWidth: 640,
  },
});
```

Let's center the content on all the screens:

```diff
 import ScreenBackground from './components/ScreenBackground';
+import CenterColumn from './components/CenterColumn';

 function HomeRoot() {
   const navigation = useNavigation();
   return (
     <ScreenBackground>
+      <CenterColumn>
         <Text>HomeRoot</Text>
         <Pressable onPress={() => navigation.navigate('HomeDetail')}>
           <Text>Go to Detail</Text>
         </Pressable>
+      </CenterColumn>
     </ScreenBackground>
   );
 }
...
   const navigation = useNavigation();
   return (
     <ScreenBackground>
+      <CenterColumn>
         <Text>HomeDetail</Text>
         <Pressable onPress={() => navigation.pop()}>
           <Text>Back to HomeRoot</Text>
         </Pressable>
+      </CenterColumn>
     </ScreenBackground>
   );
 }

 const OtherRoot = () => (
   <ScreenBackground>
+    <CenterColumn>
       <Text>OtherRoot</Text>
+    </CenterColumn>
   </ScreenBackground>
 );
```

What about more complex situations? Let's say we want to put buttons above one another on small screens but put them next to each other on large screens. On the web you can accomplish this with media queries and breakpoints, but that's not built in to React Native. But I've created this library to help with that:

```bash
$ yarn add react-native-style-queries
```

Let's create a file `src/breakpoints.js` where we'll configure the breakpoints for our app. For now we just need one value:

```js
export const breakpointMedium = 429;
```

Now let's create a `src/components/ButtonGroup.js` that is responsive:

```jsx
import {View} from 'react-native';
import {screenWidthMin, useStyleQueries} from 'react-native-style-queries';
import {breakpointMedium} from '../breakpoints';

export default function ButtonGroup({children}) {
  const styles = useStyleQueries(styleQueries);
  return <View style={styles.buttonContainer}>{children}</View>;
}

const styleQueries = {
  buttonContainer: [
    {
      flexDirection: 'column',
    },
    screenWidthMin(breakpointMedium, {
      flexDirection: 'row',
      justifyContent: 'flex-end',
    }),
  ],
};
```

STOPPED HERE

{% endraw %}
