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

**(tabs)/_layout.tsx**: add `lazy: true` to options to make the back routing work from a cold start.

```diff
<Tabs
      screenOptions={{
        headerShown: false,
        tabBarHideOnKeyboard: true,
        tabBarStyle: [$tabBar, { height: bottom + 70 }],
        tabBarActiveTintColor: colors.text,
        tabBarInactiveTintColor: colors.text,
        tabBarLabelStyle: $tabBarLabel,
        tabBarItemStyle: $tabBarItem,
+        lazy: false
      }}
    >
```

This is needed because the routing messes up due to all the tabs not being loaded at start. This is why lazy tab loading often makes me upset :).

## Module 05

- `Bound renderChildren: Support for defaultProps ...` warning when navigating to Podcast detail screen. Turns out this is just an open issue with `react-native-render-html` (https://github.com/meliorence/react-native-render-html/issues/661). It's more annoying than an actual issue at the moment. So add `"Support for defaultProps"` to **ignoreWarnings.ts**.
  - Ah, but wait? Why isn't it working? In the conversion for Expo Router, we might have missed an import of **ignoreWarnings.ts**. Add this in the root layout: `import "src/utils/ignoreWarnings"`.
- Creating your own config plugin: workshop snipped from App.js Conf where we basically try to recreate part of `@bacons/expo-targets`: https://docs.expo.dev/get-started/set-up-your-environment/?platform=android&device=physical&mode=development-build&buildEnv=local#set-up-an-android-device-with-a-development-build
- [Cool blog about using Expo Apple Targets in Pillar Valley](https://evanbacon.dev/blog/apple-home-screen-widgets)... also refers to Pillar Valley source code, good reference for this in action.

### Notes about building signed apps with widget extensions for iOS

We're using simulators and ignoring Apple teams and provisioning profiles fow now to keep things simple, focusing on the feature itself. However, for an actual production version (or even an ad-hoc testing version), you'll need real Apple signing stuff for both your main app and the app extension.

You'll notice we skipped on setting the "development team" in the iOS plugin code. If you set that, it'll assign the correct development team for the extension.

EAS Build will automatically apply your credentials for your main app, but doesn't automatically know to do that for your app extension. However, there is a secret property that can help with this. Set this in the `extra` in your app config:

```json
"eas": {
  "build": {
    "experimental": {
      "ios": {
        "appExtensions": [
          {
            "bundleIdentifier": "com.expo.appjs24-workflows-workshop",
            "targetName": "HelloWidget",
            "entitlements": {
              "com.apple.security.application-groups": ["group.appjs24-workflows-workshop"]
            }
          }
        ]
      }
    }
  }
}
```

### Making Xcode widget preview not take forever to load (partial)
The concept (haven't verified this yet)
1. Create a Prebuild template without the React Native cocoapods linked
2. Use that template when running prebuild
3. Open Xcode
4. Work on your widget swift file with live preview.

One part: [how to create an alternative template for prebuild](https://github.com/expo/fyi/blob/main/prebuild-without-npm-access.md)
 
## Other
- How to add a little bit of Expo to an RNC CLI app: https://github.com/keith-kurak/AddSomeExpo2023
- Android environment setup (specifically JDK version): https://docs.expo.dev/get-started/set-up-your-environment/?platform=android&device=physical&mode=development-build&buildEnv=local#set-up-an-android-device-with-a-development-build
