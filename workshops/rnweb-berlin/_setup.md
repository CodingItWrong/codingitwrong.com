## Requirements

- [git](https://git-scm.com/)
- [Node](https://nodejs.org/) 18.x or above
- [Yarn 1.x](https://classic.yarnpkg.com/en/docs/install) - 1.22.19 or above recommended
- At least one of the following ways to run a React Native app:
  - A physical Android or iOS device
  - [Android Studio](https://developer.android.com/studio/) for the Android Emulator
  - [Xcode](https://developer.apple.com/xcode/) for the iOS Simulator
- A code editor configured for React development

## Setting Up the Repo

We will be building off of the GitHub repo [rnweb-workshop-app](https://github.com/CodingItWrong/rnweb-workshop-app). It is a basic [Expo](https://expo.dev/) app with linting and autoformatting added.

Clone the repo locally:

```bash
git clone https://github.com/CodingItWrong/rnweb-workshop-app.git
```

Install the dependencies:

```bash
$ cd rnweb-workshop-app
$ yarn install
```

Start the development server:

```bash
$ yarn start
```

Next, open the app on a physical phone, Android Emulator, or iOS Simulator. For instructions, see [Opening the app on your phone/tablet - Expo Docs](https://docs.expo.dev/get-started/create-a-new-app/#opening-the-app-on-your-phonetablet). Note that the app is not yet ready to run on the web.

You should see the app launch on the device and the message "Open up App.js to start working on your app!" displayed.
