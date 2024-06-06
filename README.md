# Intermediate Workshop - Chain React 2024

Workshop exercises for the Chain React 2024 Intermediate Workshop.

## How to use this repo

1. Clone the [companion repo](https://github.com/infinitered/cr-2024-intermediate-workshop-template). You'll start working right on `main`.
2. Start at the first module by opening up the file starting with "01-".
3. Do the rest of the modules in order.

## Prerequisites

- A local development environment ready for native iOS and Android React Native / Expo development, capable of running the `npx expo run:ios` and `npx expo run:android`, including recent versions of:
  - Xcode (version 15+)
  - Watchman
  - Cocoapods
  - JDK 17
  - Android Studio
  - iOS simulator
  - Android emulator
  - [Fastlane](https://docs.fastlane.tools/getting-started/ios/setup/) - a small part of the workshop will use [EAS local builds](https://docs.expo.dev/build-reference/local-builds/), though it if you cannot get this working, it will not prevent you from doing the rest of the workshop.
  - If you're not sure if you have all of these or if you have the right versions, check the [Expo Local App Development requirements](https://docs.expo.dev/guides/local-app-development/) for details on how to install these tools in order to enable local native development with the Expo CLI.
- Other general development tools:
  - Node 18.
  - Visual Studio Code
  - Git (Github Desktop works great)
- Hardware:
  - A Mac is highly recommended for the full experience (though all exercises have an Android-only track, so it's possible to do most of the exercises without a Mac).
- Online accounts:
  - An Expo account (go to https://expo.dev to sign up)
  - A Github account
- Yarn

## Test your setup before the workshop

Do these steps to ensure you'll be able to complete the workshop exercises.

1. Fork and clone the [template repo](https://github.com/infinitered/cr-2024-intermediate-workshop-template).

2. Restore dependencies with

```
yarn
```

3. Build and run the app on your iOS simulator:

```
npx expo run:ios
```

4. Build and run the app on your Android emulator:

```
npx expo run:android
```

If these steps don't work, refer to the [Expo Local Development guide](https://docs.expo.dev/guides/local-app-development/).

**Want to run on a device?** We will be focusing on emulator/simulator usage during the workshop, as it's especially easier for iOS. If you want to do some or all of the workshop on a device, you can also test with `npx expo run:ios --device` and/or `npx expo run:android --device`. Some later sections of the workshop may not work on an iOS device without additional configuration.