---
title: "Building for Mobile and Web with Expo"
---

We're rewriting the frontend of a link-saving app in [Expo](https://expo.dev/), a framework built on top of React Native. This will allow us to run the same codebase on Android, iOS, and the web. You can find the work-in-progress repo here: [firehose-expo](https://github.com/CodingItWrong/firehose-expo).

<h2>
  <a href="https://www.twitch.tv/codingitwrong">
    <img src="/img/logos/twitch.png" alt="Twitch logo" class="stream-logo" />
    Next Stream</a>
  - Session 3 - Fri, Mar 11, 2022 - 4 pm ET
</h2>

In this stream we'll:

- Finish configuring our iOS app for testing on devices with Apple TestFlight
- Implement login to our existing backend
- Implement our first feature, displaying a list of links that have been saved and allowing opening them in the browser

## [Session 2 - Mar 4, 2022](https://www.youtube.com/watch?v=utW8qME38mQ&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh&index=2)

In this session we:

- Looked at the GitHub issue for a bug in react-native-reanimated we found last time, and confirmed the latest version of the library fixes it
- Updated the status bar color to match the color scheme
- Styled our screen background to respond to dark mode
- Configured the sidebar to be persistent on large screen sizes and collapse on small
- Deployed the web app successfully to Netlify
- Submitted the iOS build to Apple
- Got URLs updating to match the screen we've navigated to

## [Session 1 - Feb 25, 2022](https://www.youtube.com/watch?v=VSZEfQx-byg&list=PLXXnezSEtvNPlwbFvG3NzJAW5ikYsG2Lh)

In this session we made it as far as setting up:

- The Expo project itself
- Linting and autoformatting
- The navigation library, [React Navigation](https://reactnavigation.org/)
- The UI library, [React Native Paper](https://reactnativepaper.com/)
- The "chrome" of the app, including getting most of the way through setting up the navigation drawer and dark mode support

We ran into a bug getting React Navigation working on web that kept us stuck for a while. Turns out it's [a known issue with react-native-reanimated](https://github.com/software-mansion/react-native-reanimated/issues/3026) reported just this past week. But I don't mind that it happened: I said I wanted to demonstrate both the pros and cons of this architecture, and an error in a transitive dependency is one of the risks of a React Native project. I hope it's encouraging to y'all that even streamers run into tricky errors, and I hope the troubleshooting techniques were helpful to you!

## More Info

The app we're rewriitng the frontend for is [Firehose](https://github.com/CodingItWrong/firehose-api), an open-source link-saving app. The backend of the app is a JSON:API written in Ruby on Rails.

We're building the app using the same architecture I successfully used to write the frontend for [Surely](https://surelytodo.com), a todo list app. I use it every day on iOS and web, and it works great!

Here's a 1-minute intro video for the series:

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/UNGFDVrGQtk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>