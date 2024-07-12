# Module 04: “I was just resting my thumb!” - quick home screen interactivity with Quick Actions

### Goal

Let's use `expo-quick-actions` to add App Shortcuts (Android) / Quick Actions (iOS) to our app icon, letting users quickly access key features without even opening the app.

### Concepts

- How config plugins work with CNG to replace “manually update **AndroidManifest.xml**, entitlements, MainActivity, etc.” steps that are common when tapping into native capabilities that require more than just code.
- Meanwhile, Expo Modules API is a way to wrap native code and access it via JS. Modules are also integrated with CNG, but with autolinking.

### Features to build

- Add app shortcuts / quick actions that do the following:
  - open the user profile form
  - open the latest podcast episode

### Resources

- [expo-quick-actions](https://github.com/EvanBacon/expo-quick-actions/blob/c9f54fb948026b75053082660695e0e78f7493b4/example/app.json#L54)

# Exercises

## Exercise 1. Quick actions "Hello World"

> From here on out, regardless of platform, we're going to call these "quick actions" regardless of whether its Android or iOS, because using both terms all the time is just exhausting!

Let's do the most simple thing possible: add an action to the long-press menu, and make it execute some code.

### Add `expo-quick-actions` to your development build

1. Run `yarn add expo-quick-actions` to install the module.
2. Run `npx expo prebuild --clean` to regenerate the native projects.
3. Run `npx expo run:ios` or `npx expo run:android` to build the app (just one is fine for this section);

> Now that you have a build containing the new native module, you can continue development just by running `npx expo start` the next time (for at least a little while).

### Add a simple quick action and listener

Do the following in **src/app/_layout.tsx**:

4. Import `expo-quick-actions`:

```tsx
import * as QuickActions from "expo-quick-actions"
```

5. At the top of your file (doesn't have to be inside a component), setup a quick action. This is what actually puts the quick action in the long press menu:

```tsx
QuickActions.setItems([
  {
    id: "0",
    title: "Do something!",
    subtitle: "It could be anything!",
    icon: "heart",
    params: { stuff: "whatever" },
  },
])
```

6. Add a listener (also could be anywhere in the file) that will execute when the quick action is tapped:

```tsx
const subscription = QuickActions.addListener((action) => {
  console.log(action);
})
```

🏃**Try it.** Open the app, leave it, long press, and see what happens!

> Notice that the `params` can be anything. From the perspective of the operating system, it's entirely up to your app code what it does with those parameters.

## Exercise 2: Quick actions, now with routing

Technically (in native-land), the quick action parameters can be anything, but, `expo-quick-actions` has its own special listener that can read `href` params and map those to Expo Router routes. Then, all quick actions will open specific screens in your app. So, let's switch to that.

### Comment out / remove your current quick actions setter and listener.

1. We don't need the code we just made in Exercise 1 anymore. So get rid of it!

### Add "quick action routing"

Add all of this code in **src/app/(app)/_layout.tsx** (this will help delay the setup until the user is "logged in"):

2. Add imports:

```tsx
import { Platform } from "react-native"
// ...
import * as QuickActions from "expo-quick-actions"
import { useQuickActionRouting, RouterAction } from "expo-quick-actions/router"
```

3. Inside the component, add the following:

```tsx
useQuickActionRouting()

React.useEffect(() => {
  QuickActions.setItems<RouterAction>([
    {
      title: "Update your profile",
      subtitle: "Keep your deets up-to-date!",
      icon: Platform.OS === "android" ? undefined : "contact", // we'll come back to the Android icon
      id: "0",
      params: { href: "/(app)/(tabs)/profile" },
    },
  ])
}, [])
```

<!-- TODO: find an appropriate SF icon -->

🏃**Try it.** Try out your new quick action. Does it open the profile?

## Exercise 3: Quick action to the latest podcast episode

A quick action to the Podcasts tab would be... fine, but what if it could take us to the latest episode? There is no "latest episode" route, per se, just routes based on episode ID. Let's think like a router and add a placeholder route just for our quick action that will then redirect to the latest episode.

### Add the quick action

1. Add another list item to the `setItems` function called in **src/app/(app)/_layout.tsx**:

```tsx
{
  title: "Check out the latest podcast",
  subtitle: "What's everyone saying on RN Radio?",
  icon: Platform.OS === "android" ? undefined : "play",
  id: "1",
  params: { href: "/(app)/(tabs)/podcasts/latest" },
},
```

🏃**Try it.** Try out your new quick action. 404 much?

Spoiler alert: we're going to create that new route next.

<!-- TODO: find an appropriate SF icon -->

### "Latest" podcast route

2. Create the file: **src/app/(app)/(tabs)/podcasts/latest.tsx**:

3. Setup your file like this:

```tsx
export default () => {
  return null;
}
```

🏃**Try it.** Try out your new quick action. Does it should at least open this page ... which doesn't have much (or anything, really).

### Redirect to the latest podcast.

To make this route actually do something, we'll use a `Redirect` component, which will send us to the correct route, once we know what that is.
Write all of the following code in **src/app/(app)/(tabs)/podcasts/latest.tsx**:

4. Query the latest podcasts, much like we do in **podcasts/index.tsx**. Switch out your `latest` screen for this:

```tsx
import React from "react"
import { observer } from "mobx-react-lite"
import { Redirect } from 'expo-router';
import { useStores } from 'src/models';

export default observer(() => {
  const { episodeStore } = useStores();
  if (episodeStore.episodes.length > 0) {
    return <Redirect href={`/(app)/(tabs)/podcasts/${episodeStore.episodes.slice()[0].guid}`} />;
  }
  return <Redirect href={`/(app)/(tabs)/podcasts/`} />;;
});
```

> Of course, we're relying on the podcasts already being loaded here; that's not a good assumption. In a real-life app, you could show a spinner if no podcasts are loaded, and then show an error if loading fails.

🏃**Try it.** Try out your new quick action. Hopefully it actually takes you to the latest podcast.

### Fix Expo Router back behavior

You may have noticed your back button not working. This can happen when going directly to a deep link, as your app will not know where to go back to. Expo Router has a setting to fix this.

In **src/app/(app)/(tabs)/podcast/_layout.tsx**, add this:

```tsx
// eslint-disable-next-line camelcase
export const unstable_settings = {
  // Ensure any route can link back to `/`
  initialRouteName: 'index',
};
```

## Exercise 4: Customizing quick action icons with the `expo-quick-actions` config plugin.

Most of what we can do with `expo-quick-actions` is the result of the Expo Module API code inside of the package exposing the App Shortcuts / Quick Actions native API's. However, the package also ships with a config plugin that can make changes to the native project itself, specifically adding assets that can be used for custom icons to the native project.

With iOS, it's typical to use SF Symbols for these icons, since they match the Operating System. Currently, we're using the "system" icons, of which there is just a few (check out the Side Quests below for advice on quickly switching to SF Symbols - there's a ton to choose from). However, with Android, any App Shortcuts can be dragged to your home screen and treated as separate icons, so these icons should be adaptive icons, just like your app's home screen icon itself. Let's add icons for each of our two app shortcuts.

1. Add the `expo-quick-actions` plugin and configuration to **app.json** inside the `plugins` key:
<!-- TODO: update these icons to the correct ones -->

```json
plugins: [
  [
    "expo-quick-actions",
    {
      "androidIcons": {
        "profile_icon": {
          "foregroundImage": "./assets/icons/adaptive-icon-profile.png",
          "backgroundColor": "#29cfc1",
        },
         "podcast_icon": {
          "foregroundImage": "./assets/icons/adaptive-icon-podcast.png",
          "backgroundColor": "#29cfc1",
        },
      },
    },
  ],
],
```

2. Those icons aren't actually in your project yet, so grab them from [the Module 04 files](/files/04/) and copy them into **assets/icons**.

3. Run `npx expo prebuild --clean --platform android` to regenerate the Android native project.

4. Run `npx expo run:android` to build the app.

5. Update the `setItems` function to incorporate these icons:

```diff
QuickActions.setItems<RouterAction>([
    {
      title: "Update your profile",
      subtitle: "Keep your deets up-to-date!",
-      icon: Platform.OS === "android" ? undefined : "contact", // we'll come back to the Android icon
+      icon: Platform.OS === "android" ? "profile_icon" : "contact",
      id: "0",
      params: { href: "/(app)/(tabs)/profile" },
    },
    {
      title: "Check out the latest podcast",
      subtitle: "What's everyone saying on RN Radio?",
-      icon: Platform.OS === "android" ? undefined : "play",
+      icon: Platform.OS === "android" ? "podcast_icon" : "play",
      id: "1",
      params: { href: "/(app)/(tabs)/podcasts/latest" },
    },
  ])
}, [])
```

🏃**Try it.** Run your app, letting the quick actions refresh, then try them out. Are the icons updated on Android? What happens when you drag an icon and drop it on your home screen?

## Side Quests

### 1. Use SF Symbols icons

[Read more on how to find good SF Symbols and change out your quick actions icons on iOS for them.](https://github.com/EvanBacon/expo-quick-actions?tab=readme-ov-file#sf-symbols).

### 2. Static quick actions in iOS

The config plugin also supports adding static actions on iOS- quick actions that will show up before the app is run. Convert the "profile" quick action into a static action using the [example here](https://github.com/EvanBacon/expo-quick-actions?tab=readme-ov-file#config-plugin).

### 3. SF Symbols everywhere

SF Symbols can be used inside your app, as well! Take some steps towards a UI that's more cohesive with the OS on iOS:

1. Change the font to the iOS default (San Francisco) <!-- IMO it's not 100% intuitive to fallback to system font in Ignite. I've done it, but it felt hacky ... interested in the right way -->
2. Use [Expo Symbols](https://docs.expo.dev/versions/latest/sdk/symbols/) to swap out a few icons. Try the Podcasts tab icon (match the Quick action icon) or even the favorite icon.

## See the solution

Switch to branch: `04-quick-actions-solution`
