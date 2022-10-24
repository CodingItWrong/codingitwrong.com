---
title: Building for Web and Mobile with Expo - Exercise
---

This page will take you through the full workshop exercise. You can also download [the finished repository](https://github.com/CodingItWrong/rnweb-workshop-app), but I recommend working through the exercise to get a better feel for how the libraries and customizations work together.

{% include_relative _setup.md %}

{% raw %}

## RN Web

Expo is actually *almost* preconfigured to run on the web out of the box.

Press W on the keyboard to open it on the web. You will get a warning:

```text
It looks like you're trying to use web support but don't have the required
dependencies installed.

Please install react-native-web@~0.18.7, react-dom@18.0.0,
@expo/webpack-config@^0.17.0 by running:

npx expo install react-native-web@~0.18.7 react-dom@18.0.0 \
  @expo/webpack-config@^0.17.0
```

Let's install those. Press ctrl-C to quit Expo, then run the following command:

```bash
$ npx expo install react-native-web@~0.18.7 \
                   react-dom@18.0.0 \
                   @expo/webpack-config@^0.17.0
```

Once those are installed, run `yarn start` again, then press W. The app should open on the web.

## React Navigation

The first thing we'll set up is navigation. We'll use React Navigation, the most popular navigation library for React Native. It has great web support as well.

Add the dependencies for React Navigation, including both the drawer and stack navigator:

```bash
$ yarn add @react-navigation/drawer \
           @react-navigation/native \
           @react-navigation/native-stack
$ npx expo install react-native-gesture-handler \
                   react-native-reanimated \
                   react-native-screens \
                   react-native-safe-area-context
```

We will also need to configure Babel to support the Reanimated library. Make the following change in `babel.config.js`:

```diff
   return {
     presets: ['babel-preset-expo'],
+    plugins: ['react-native-reanimated/plugin'],
   };
```

Now that our project is set up for navigation, let's begin configuring our navigation structure.

Create a `src` folder and, inside it, a `Navigation.js` file. First, let's create a few sample screens to navigate between:

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

function OtherRoot() {
  return <Text>OtherRoot</Text>;
}
```

Now let's create two stack navigators, one for each set:

```jsx
import {createNativeStackNavigator} from '@react-navigation/native-stack';
//...
const HomeStack = createNativeStackNavigator();
function Home() {
  return (
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
}

const OtherStack = createNativeStackNavigator();
function Other() {
  return (
    <OtherStack.Navigator>
      <OtherStack.Screen
        name="OtherRoot"
        component={OtherRoot}
        options={{title: 'Other'}}
      />
    </OtherStack.Navigator>
  );
}
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
       <Drawer.Screen name="Other" component={Other} />
     </Drawer.Navigator>
   );
 }
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

Open the app on mobile and on web. See how we can drill down into screens and switch between different stacks. Now, there are two title bars which doesn't look good. We'll address that once we start customizing our navigation components.

## URLs

Before we move on to theming, though, let's talk about URLs. Notice how on the web the app has the same URL no matter what screen we're on. On the web we expect each screen to correspond to a URL. URLs can be useful for deep linking into screens of the app as well.

In `Navigation.js`, add the following configuration object:

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

Click around the app and notice the URLs change as we expect.

Try reloading the browser tab when you're on a screen other than the home screen. Note that you're taken to the screen you expect.

Try reloading when you're on `/home/detail`, though. There's a problem: there's no back button! There's no way to get to the home screen. Why is this?

The reason is that stacks don't automatically have a "starting" screen. If you navigate to the details screen, that's where you begin in that stack.

To fix this, configure an `initialRouteName` in the linking config for the home stack:

```diff
 screens: {
   Home: {
     path: '/',
+    initialRouteName: 'HomeRoot',
     screens: {
       HomeRoot: '',
       HomeDetail: 'home/detail',
```

Try reloading `/home/detail` again. Now you should see the back button and be able to get back to the home screen.

# RN Paper

With our navigation working, it's time to make our app look awesome. There are a lot of different ways to style a React Native app.

In this workshop we're going to use React Native Paper. I've found that it has great support for both mobile and web. Version 5 of Paper is currently in release candidate stage; we're going to go ahead and use it to make sure you're set for the future. It's nice and stable.

Add the Paper dependency:

```bash
$ yarn add react-native-paper@5.0.0-rc.6
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

Now configure the drawer to use it in `Navigation.js`:

```diff
+import CustomNavigationDrawer from './components/CustomNavigationDrawer';

 const linking = {
...
 function NavigationContents() {
   return (
-    <Drawer.Navigator useLegacyImplementation>
+    <Drawer.Navigator
+      useLegacyImplementation
+      drawerContent={CustomNavigationDrawer}
+    >
       <Drawer.Screen name="Home" component={Home} />
       <Drawer.Screen name="Other" component={Other} />
     </Drawer.Navigator>
   );
 }
```

Reload the app and open the drawer. Check out how we have a material design look and feel. We're going to adjust it later once we customize our Material Design theme.

Next, the header. Create a file `src/components/CustomNavigationBar.js`. Add the following:

```jsx
import {Appbar} from 'react-native-paper';

export default function CustomNavigationBar({navigation, options, back}) {
  return (
    <Appbar.Header>
      {back ? (
        <Appbar.BackAction
          onPress={navigation.goBack}
          accessibilityLabel="Back"
        />
      ) : null}
      <Appbar.Content title={options.title} />
      <Appbar.Action
        icon="menu"
        accessibilityLabel="Menu"
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
+  <HomeStack.Navigator screenOptions={{header: CustomNavigationBar}}>
     <HomeStack.Screen
       name="HomeRoot"
       component={HomeRoot}
...
 const OtherStack = createNativeStackNavigator();
 function Other() {
   return (
-    <OtherStack.Navigator>
+    <OtherStack.Navigator screenOptions={{header: CustomNavigationBar}}>
       <OtherStack.Screen
```

We also configure the drawer's own header not to show, since our custom navigation bar handles it:

```diff
 <Drawer.Navigator
   useLegacyImplementation
+  screenOptions={{headerShown: false}}
   drawerContent={CustomNavigationDrawer}
 >
```

Load up the app. See how there is only one header bar now, and it has a Material look. We can press the right-hand icon to toggle the drawer. And if we're on a detail screen there is a back arrow to go back to the initial screen.

## Dark Mode

Switch your computer and Simulator/Emulator to dark mode. Note that on the web the navigation bar and drawer buttons switch to dark mode, but not on mobile. By default Expo doesn't allow switching the mobile apps to dark mode. We can change that in `app.json`:

```diff
     "icon": "./assets/icon.png",
-    "userInterfaceStyle": "light",
+    "userInterfaceStyle": "automatic",
     "splash": {
```

Save and reload the app on mobile. You should see the same dark mode styling there.

Our next problem is that our RN Paper components change to dark mode, but some of our other components do not. We need to customize them to respect dark mode settings.

First, backgrounds. Let's add a `ScreenBackground` component that supports dark mode:

```jsx
import {View} from 'react-native';
import {useTheme} from 'react-native-paper';

export default function ScreenBackground({style, children}) {
  const theme = useTheme();
  const baseStyle = {flex: 1, backgroundColor: theme.colors.background};
  return <View style={[baseStyle, style]}>{children}</View>;
}
```

Let's use this for all our screens in `Navigation.js`:

```diff
+import ScreenBackground from './components/ScreenBackground';
...
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

 function OtherRoot() {
-  return <Text>OtherRoot</Text>;
+  return (
+    <ScreenBackground>
+      <Text>OtherRoot</Text>
+    </ScreenBackground>
+  );
 }
```

Now our backgrounds change color but our text doesn't. The `Text` component from Paper will help:

```diff
 import {NavigationContainer, useNavigation} from '@react-navigation/native';
-import {Pressable, Text} from 'react-native';
+import {Pressable} from 'react-native';
+import {Text} from 'react-native-paper';
 import {createDrawerNavigator} from '@react-navigation/drawer';
```

Now our text looks good.

Next, take a look at the drawer. Its background doesn't change either. Let's add that support:

```diff
 import {DrawerContentScrollView} from '@react-navigation/drawer';
-import {Drawer} from 'react-native-paper';
+import {Drawer, useTheme} from 'react-native-paper';

 function CustomNavigationDrawer({...navProps}) {
   const {state, navigation} = navProps;
+  const theme = useTheme();
+
+  const scrollViewStyle = {
+    backgroundColor: theme.colors.background,
+  };

   const isSelected = index => index === state.index;

   return (
-    <DrawerContentScrollView {...navProps}>
+    <DrawerContentScrollView style={scrollViewStyle} {...navProps}>
       {state.routes.map((route, index) => (
         <Drawer.Item
           key={route.key}
```

When we make this change, we get an error:

> Render Error
>
> Invalid hook call. Hooks can only be called inside of the body of a function component.

This is subtle, but I think this is because the `Drawer.Navigator` isn't actually calling the `drawerContent` as a component, but as a plain function. But I've found we can fix this by wrapping the component in a function, so the component is actually called within JSX:

```diff
 const Drawer = createDrawerNavigator();

+function renderCustomNavigationDrawer(props) {
+  return <CustomNavigationDrawer {...props} />;
+}
+
 function NavigationContents() {
   return (
     <Drawer.Navigator
       useLegacyImplementation
       screenOptions={{headerShown: false}}
-      drawerContent={CustomNavigationDrawer}
+      drawerContent={renderCustomNavigationDrawer}
     >
```

Save, and the error should be fixed.

## Theme Color

This looks good, but the theme also looks exactly like every other RN Paper app out there. Here's how we can do it.

React Native Paper v5 supports two different versions of the Material Design spec: the older Material Design 2, and the newer Material You. It defaults to Material You.

If we want to stick with Material You, we have to customize a number of different colors in the theme. But if we're willing to fall back to Material Design 2, we can just set one primary color, and our design will be set. For this exercise, we'll do the latter.

If we use the built-in theme we get light and dark mode switching automatically, but since we're customizing the theme we will need to handle the switching ourselves.

Create a file `src/useCustomTheme.js` and add the following:

```js
import {useColorScheme} from 'react-native';
import {MD2DarkTheme, MD2LightTheme} from 'react-native-paper';

export default function useCustomTheme() {
  const colorScheme = useColorScheme() ?? 'light';
  const baseTheme = colorScheme === 'dark' ? MD2DarkTheme : MD2LightTheme;
  return baseTheme;
}
```

Now use this to set the theme on the provider in `App.js`:

```diff
 import {Provider as PaperProvider} from 'react-native-paper';
+import useCustomTheme from './src/useCustomTheme';

 export default function App() {
+  const theme = useCustomTheme();
+
   return (
-    <PaperProvider>
+    <PaperProvider theme={theme}>
       <StatusBar style="auto" />
```

If you look in the app, the look has changed. This is Material Design 2.

But now let's set that theme color. You can pick any vibrant color. Here's what I chose:

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

Pull up the app again and try switching between light and dark mode to see our custom color in action.

## Responsive Design

Let's take a look at a few ways we can make our app look good at different sizes.

First, in a large viewport like the web on desktop, the content area is pretty wide. Let's make it a centered column that hsa a maximum width. Let's create a component for this, `src/components/CenterColumn.js`. Add the following:

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

 function OtherRoot() {
   return (
     <ScreenBackground>
+      <CenterColumn>
         <Text>OtherRoot</Text>
+      </CenterColumn>
     </ScreenBackground>
   );
 }
```

What about more complex situations? Let's say we want to put buttons above one another on small screens but put them next to each other on large screens. On the web you can accomplish this with media queries and breakpoints, but that's not built in to React Native. There are a lot of styling libraries for React Native, and some of them may provide built in breakpoint support.

If you don't already have a breakpoint option, I created a tiny library to help with this:

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

Now let's update `HomeRoot` to use it:

```diff
 import {createNativeStackNavigator} from '@react-navigation/native-stack';
 import {Pressable} from 'react-native';
-import {Text} from 'react-native-paper';
+import {Button, Text} from 'react-native-paper';
 import CustomNavigationBar from './components/CustomNavigationBar';
 import CustomNavigationDrawer from './components/CustomNavigationDrawer';
 import ScreenBackground from './components/ScreenBackground';
 import CenterColumn from './components/CenterColumn';
+import ButtonGroup from './components/ButtonGroup';

 function HomeRoot() {
   const navigation = useNavigation();
...
     <ScreenBackground>
       <CenterColumn>
         <Text>HomeRoot</Text>
-        <Pressable onPress={() => navigation.navigate('HomeDetail')}>
-          <Text>Go to Detail</Text>
-        </Pressable>
+        <ButtonGroup>
+          <Button mode="outlined">Other Button</Button>
+          <Button mode="outlined">Third Button</Button>
+          <Button
+            mode="contained"
+            onPress={() => navigation.navigate('HomeDetail')}
+          >
+            <Text>Go to Detail</Text>
+          </Button>
+        </ButtonGroup>
       </CenterColumn>
     </ScreenBackground>
   );
```

Note how the buttons are laid out. We could add padding to them to space them apart.

Another responsive feature that would be the nice relates to the drawer. With all the screen space on a large screen, it would be nice to keep it visible all the time. Here's how we can do that.

First, let's make a function that tells us which breakpoint we're at. In `breakpoints.js`:

```diff
+import {useWindowDimensions} from 'react-native';
+
 export const breakpointMedium = 429;
+export const breakpointLarge = 600;
+
+export const small = 'small';
+export const medium = 'medium';
+export const large = 'large';
+
+function breakpointForWidth(width) {
+  if (width >= breakpointLarge) {
+    return large;
+  } else if (width >= breakpointMedium) {
+    return medium;
+  } else {
+    return small;
+  }
+}
+
+export function useBreakpoint() {
+  const {width} = useWindowDimensions();
+  return breakpointForWidth(width);
+}
```

Next, we'll configure the drawer to be either permanent or not depending on the breakpoint. In `Navigation.js`:

```diff
 import ButtonGroup from './components/ButtonGroup';
+import {large, useBreakpoint} from './breakpoints';

 function HomeRoot() {
   const navigation = useNavigation();
...
 const Drawer = createDrawerNavigator();

 function NavigationContents() {
+  const breakpoint = useBreakpoint();
+  const drawerType = breakpoint === large ? 'permanent' : 'back';
+
   // not sure why useLegacyImplementation is needed; isn't in my other apps
   return (
     <Drawer.Navigator
...
       initialRouteName="Home"
       screenOptions={{
         headerShown: false,
+        drawerType,
       }}
       drawerContent={props => <CustomNavigationDrawer {...props} />}
     >
```

This keeps the drawer present on large viewports. Try narrowing the browser window and notice how the drawer disappears when you go narrower and reappears when you go wider.

As a final touch, let's hide the drawer toggle button when it's permanently shown. In `CustomNavigationBar.js`:

```diff
 import {Appbar} from 'react-native-paper';
+import {large, useBreakpoint} from '../breakpoints';

 export default function CustomNavigationBar({navigation, options, back}) {
+  const breakpoint = useBreakpoint();
+  const showDrawerToggle = breakpoint !== large;
+
   return (
     <Appbar.Header>
...
       <Appbar.Content title={options.title} />
+      {showDrawerToggle && (
         <Appbar.Action
           icon="menu"
           accessibilityLabel="Menu"
           onPress={navigation.toggleDrawer}
         />
+      )}
     </Appbar.Header>
   );
 }
```

## Platform-Specific Functionality

To learn more about conditionals based on whether you are on web or mobile, or separate module implementations for web, see [Web-specific code in the React Native Web docs](https://necolas.github.io/react-native-web/docs/multi-platform/#web-specific-code).

{% endraw %}

## Resources

{% include_relative _resources.md %}

{% include_relative _contact.md %}
