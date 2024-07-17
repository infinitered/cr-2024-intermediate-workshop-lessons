# Miscellaneous stuff
We'll be put interesting stuff up here as it comes up right before or during the workshop

## Module 01

### EAS Build: also an option!
The same concepts we covered about when to run `prebuild` / `npx expo run:ios|android` also applies to when we could run EAS Build. We want to build locally today, especially later, because we'll be iterating on native features. But you could actually get through module 3 with just one build on EAS Build:
1. run `npm install -g eas-cli` to install the EAS CLI
2. run `eas build:configure` to setup your project. Login/ create an Expo account if you don't already have one.
3. Follow the directions onscreen to add the projectId to **app.json**.
4. Run `eas build --profile development --platform ios|android`. This will make a build you can drag to your Android emulator / iOS simulator

This wouldn't scale well to the iOS widget creation part of Module 5... there's just too much native code iteration going on.

### Other workshop
- Build a Firebase-enabled chat app with the Ignite starter: [Freaky Fast Full Stack with the FERNI Stack](https://github.com/keith-kurak/ferni-chat-2023)

## Module 02

## Module 03

## Module 04

## Module 05

- `Bound renderChildren: Support for defaultProps ...` warning when navigating to Podcast detail screen. Turns out this is just an open issue with `react-native-render-html` (https://github.com/meliorence/react-native-render-html/issues/661). It's more annoying than an actual issue at the moment. So add `"Support for defaultProps"` to **ignoreWarnings.ts**.
  - Ah, but wait? Why isn't it working? In the conversion for Expo Router, we might have missed an import of **ignoreWarnings.ts**. Add this in the root layout: `import "src/utils/ignoreWarnings"`.
 
## Other
- How to add a little bit of Expo to an RNC CLI app: https://github.com/keith-kurak/AddSomeExpo2023
