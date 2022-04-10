---
title: "Building for Mobile and Web with Expo"
---

In this eight-part series we rewrite a link-saving app in Expo, a framework built on top of React Native. This allows us to run the same codebase on Android, iOS, and the web. You can find the repo here: https://github.com/CodingItWrong/firehose-expo

The app we're rewriting the frontend for is [Firehose](https://github.com/CodingItWrong/firehose-api), an open-source link-saving app. The backend of the app is a JSON:API written in Ruby on Rails.

Here's a 1-minute intro video for the series:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/UNGFDVrGQtk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## [Session 1 - Feb 25, 2022](https://www.youtube.com/watch?v=VSZEfQx-byg&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh)

In this session we made it as far as setting up:

- The Expo project itself
- Linting and autoformatting
- The navigation library, [React Navigation](https://reactnavigation.org/)
- The UI library, [React Native Paper](https://reactnativepaper.com/)
- The "chrome" of the app, including getting most of the way through setting up the navigation drawer and dark mode support

## [Session 2 - Mar 4, 2022](https://www.youtube.com/watch?v=utW8qME38mQ&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=2)

In this session we:

- Looked at [the GitHub issue for a bug](https://github.com/software-mansion/react-native-reanimated/issues/3026) in react-native-reanimated we found last time, and confirmed the latest version of the library fixes it
- Updated the status bar color to match the color scheme
- Styled our screen background to respond to dark mode
- Configured the sidebar to be persistent on large screen sizes and collapse on small
- Deployed the web app successfully to Netlify
- Submitted the iOS build to Apple
- Got URLs updating to match the screen we've navigated to

## [Session 3 - Mar 11, 2022](https://www.youtube.com/watch?v=BNHXwHAec18&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=3)

In this session we:

- Finished deploying the app to iOS with TestFlight
- Customized app icons and metadata on web and iOS
- Made good progress toward sign in, including the screen, access token storage, and sidebar changes.

## [Session 4 - Mar 18, 2022](https://www.youtube.com/watch?v=qVIAcX7GG0Y&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=4)

In this session we:

- Fixed a bug where the app flashed the Sign In screen when we're already signed in
- Set up the app to connect to the production backend
- Finished styling the Sign In screen
- Set up the API client for the JSON:API backend
- Set up the Unread page displaying links from the backend and opening them in the browser

## [Session 5 - Mar 25, 2022](https://www.youtube.com/watch?v=wln9_z9LEfU&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=5)

In this session we:

- Reviewed "end-to-end" testing options and made a case for "screen tests" in React Native Testing Library
- Tested our pre-existing code that displays links from the backend
- Set up Dependabot to do automatic dependency updates
- Talked about screen reader and Voice Control accessibility
- Made progress on implementing and testing our next feature: marking a link as read

## [Session 6 - Mar 30, 2022](https://www.youtube.com/watch?v=xCcZJ7bQFQA&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=6)

In this session we:

- Finished our mark-read feature by removing a read bookmark from the list, confirming the menu is dismissed, and fixing some test warnings
- Implemented deleting a bookmark via test-driven development
- Refactored by extracting components, removing duplication, and improving names

## [Session 7 - Apr 1, 2022](https://www.youtube.com/watch?v=rQ6zNXlAYHs&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=7)

In this session we:

- Implemented refreshing the list of links, including handling a mobile/web behavior difference
- Implemented navigating to a detail screen where we'll add editing next time, and showed how to test navigation within RNTL
- Extracted some components into files with the help of our thorough tests

## [Session 8 - Finale! - Apr 8, 2022](https://www.youtube.com/watch?v=2Dd8vtfVmTs&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=8)

In this session we:

- Implemented basic functionality of the edit screen
- Showed how to write a screen test for a screen that receives a navigation param
- Refactored edit screen visuals with confidence from passing tests
- Implemented responsive button layouts using the react-native-style-queries library
- Removed duplicative provider setup in tests

