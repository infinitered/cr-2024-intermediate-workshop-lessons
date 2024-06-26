# Creating the Android Podcast Widget

## Background

These instructions continue where Module 05 left off, describing how to transform your "hello" widget into one that displays information about favorited podcasts.

## Renaming things
Let's rename `HelloWidget` to `FavoriteEpisodeWidget`:
- Rename **HelloWidget.tsx** to **FavoriteEpisodeWidget.tsx**,
- change the component name, and
- change the component reference in **widget-task-handler.tsx**.
- change the `name` in the config plugin in **app.json** to `FavoriteEpisode`

## Basic data flow
Let's output the episodes to the widget and finish implementing refresh. That way, all our data flow will be working, and everything else will just be styling and some in-widget functionality.

1. Update **FavoriteEpisodeWidget.tsx** to read episodes and output them to a list:

```tsx
/* eslint-disable react-native/no-color-literals */
/* eslint-disable react-native/no-inline-styles */
import React from "react"
import { ListWidget, TextWidget } from "react-native-android-widget"
import { Episode } from "src/models/Episode"
import { colors } from "src/theme"

interface FavoriteEpisodeWidgetProps {
  episodes: Episode[]
}

export function FavoriteEpisodeWidget({ episodes }: FavoriteEpisodeWidgetProps) {
  return (
    <ListWidget
      style={{
        height: "match_parent",
        width: "match_parent",
        backgroundColor: colors.background,
      }}
    >
    {episodes.map(episode => (
      <TextWidget
        key={episode.guid}
        text={episode.parsedTitleAndSubtitle.subtitle}
        style={{
          fontSize: 16,
          fontFamily: "Inter",
          color: colors.text,
        }}
      />
    )}
```

2. Read episodes from the `EpisodeStore` in **widget-task-handler.tsx**:

```diff
import React from "react"
import type { WidgetTaskHandlerProps } from "react-native-android-widget";
import { FavoriteEpisodeWidget } from "./FavoriteEpisodeWidget";
+import { setupRootStore, RootStoreModel } from 'src/models';

export async function widgetTaskHandler(props: WidgetTaskHandlerProps) {

+  const { rootStore } = await setupRootStore(RootStoreModel.create({}))

  switch (props.widgetAction) {
    case "WIDGET_ADDED":
    case "WIDGET_UPDATE":
    case "WIDGET_RESIZED":
      props.renderWidget(
        <FavoriteEpisodeWidget
+          episodes={rootStore.episodeStore.favorites.slice()}
        />
      );
      break;
    default:
      break;
  }
}
```

<details>
  <summary>Expand to just get the whole file's new code for easy copying</summary>

  ```tsx
import React from "react"
import type { WidgetTaskHandlerProps } from "react-native-android-widget";
import { FavoriteEpisodeWidget } from "./FavoriteEpisodeWidget";
import { setupRootStore, RootStoreModel } from 'src/models';

export async function widgetTaskHandler(props: WidgetTaskHandlerProps) {

  const { rootStore } = await setupRootStore(RootStoreModel.create({}))

  switch (props.widgetAction) {
    case "WIDGET_ADDED":
    case "WIDGET_UPDATE":
    case "WIDGET_RESIZED":
      props.renderWidget(
        <FavoriteEpisodeWidget
          episodes={rootStore.episodeStore.favorites.slice()}
        />
      );
      break;
    default:
      break;
  }
}
```

</details>

Notice how we setup the root store separately from the main app. This is done because the task handler is actually getting spun up in an entirely different context from the main app. Accessing the root store helps us get at the episodes stored in local storage with their favorite status.

3. Add the on-demand widget refresh code in **widget-refresher.tsx**:

```diff
import React from "react"
import { Platform } from "react-native"
+import { requestWidgetUpdate } from "react-native-android-widget"
import { Episode } from "src/models/Episode"
+import { FavoriteEpisodeWidget } from "./android/FavoriteEpisodeWidget"

export const updateEpisodesWidget = (episodes: Episode[]) => {
  if (Platform.OS === "android") {
+    requestWidgetUpdate({
+      widgetName: "FavoriteEpisode",
+      renderWidget: () => <HelloWidget episodes={episodes} />,
+      widgetNotFound: () => {
+        // Called if no widget is present on the home screen
+      },
    })
  }
```

<details>
  <summary>Expand to just get the whole file's new code for easy copying</summary>

  ```tsx

import React from "react"
import { Platform } from "react-native"
import { requestWidgetUpdate } from "react-native-android-widget"
import { Episode } from "src/models/Episode"
import { FavoriteEpisodeWidget } from "./android/FavoriteEpisodeWidget"

export const updateEpisodesWidget = (episodes: Episode[]) => {
  if (Platform.OS === "android") {
    requestWidgetUpdate({
      widgetName: "FavoriteEpisode",
      renderWidget: () => <HelloWidget episodes={episodes} />,
      widgetNotFound: () => {
        // Called if no widget is present on the home screen
      },
    })
  }
  if (Platform.OS === "ios") {
    // iOS widget refresh code
  }
}
```

</details>

🏃**Try it.** Run `npx expo prebuild --clean` and `npx expo run:android`. You may need to delete an re-add your widget once. Favorite and unfavorite episodes and then leave your app for the home screen. Your widget should update.

## Make the widget look good

Let's style the repeating list of make it look like nice, with a thumbnail, spacing, and perhaps some segmentation of the rows.

[<img src="./assets/05/android-widget.png" width="250" />](./assets/05/android-widget.png)

1. In **FavoriteEpisodeWidget.tsx**, swap out the simple `TextWidget` inside the `episodes.map()` expression for a styled row:

```tsx
<FlexWidget
  key={episode.guid}
  style={{
    flexDirection: "row",
    padding: spacing.md,
    width: "match_parent",
  }}
>
  <FlexWidget
    style={{
      flexDirection: "row",
      alignItems: "center",
    }}
  >
    <ImageWidget
      imageHeight={50}
      imageWidth={50}
      image={episode.thumbnail as `https://${string}`}
      radius={25}
    />
    <TextWidget
      text={episode.parsedTitleAndSubtitle.subtitle}
      style={{
        marginLeft: spacing.md,
        fontSize: 16,
        fontFamily: "Inter",
        color: colors.text,
      }}
    />
  </FlexWidget>
</FlexWidget>
```

Feel free to play around with the spacing and sizing.

🏃**Try it.** Force your widget to update by opening the app and backgrounding it again. It looks better, hopefully. Try favoriting more than one podcast. What do you think? Doesn't quite hit the spot, eh?

We need something to separate the episodes. By now, you can probably see that the `Widget` controls work a lot like typical React Native controls. So, you could probably fashion a separator. To keep things simple, let's try using color to separate the cells.

2. Let's alternate colors to provide some separation. Modify the rows as such:

```diff
-{episodes.map(episode => (
+{episodes.map((episode, index) => (
  style={{
    flexDirection: "row",
    padding: spacing.md,
+    backgroundColor:
+      index % 2 === 0 ? colors.palette.neutral200 : colors.palette.neutral300,
    width: "match_parent",
  }}
```

🏃**Try it.** That's better! OK, now try to unfavorite all the podcasts...oh, not so great.

### Empty state

Add an empty condition just before `episodes.map`:

```tsx
{episodes.length === 0 && (
  <FlexWidget
    style={{
      height: "match_parent",
      width: "match_parent",
      justifyContent: "center",
      alignItems: "center",
    }}
    clickAction="OPEN_URI"
    clickActionData={{ uri: "cr2024-im://podcasts" }}
  >
    <TextWidget
      text="No episodes favorited yet. Tap here to start!"
      style={{
        fontSize: 16,
        fontFamily: "Inter",
        color: colors.text,
      }}
    />
  </FlexWidget>
)}
```

🏃**Try it.** A little simple, but it'll do the trick. Favorite some podcasts and watch it dissappear again.

## Deep link to podcasts from the widget

Right now, tapping on the widget doesn't do anything. It'd be really smart if tapping on each episode took you to that actual episode.

We can lean on Expo Router and automatic deep linking to help with that. Widgets can have `clickAction`'s and `clickActionData` that can contain an app-specific URI, which will include the deep link into our app.

1. Add the click action props to the episode row, deep linking to the specific podcast:

```diff
{episodes.map((episode, index) => (
<FlexWidget
  key={episode.guid}
  style={{
    flexDirection: "row",
    padding: spacing.md,
    backgroundColor:
      index % 2 === 0 ? colors.palette.neutral200 : colors.palette.neutral300,
    width: "match_parent",
  }}
+  clickAction="OPEN_URI"
+  clickActionData={{ uri: `cr2024-im://podcasts/${episode.guid}` }}
>
```

2. Don't forget the empty state! That can open just the podcasts tab:

```diff
{episodes.length === 0 && (
<FlexWidget
  style={{
    height: "match_parent",
    width: "match_parent",
    justifyContent: "center",
    alignItems: "center",
  }}
+  clickAction="OPEN_URI"
+  clickActionData={{ uri: "cr2024-im://podcasts" }}
>
```

🏃**Try it.** Tap on a podcast. It should take you to that podcast. Try the back button, too.

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

This will tell Expo Router that the top level route of the podcasts group is **index**, so it will be able to go back there even if it starts at a specific podcast.

## Side Quests
You could probably figure out something better for styling the podcast rows or the divider between them. If you have extra time, go for it!