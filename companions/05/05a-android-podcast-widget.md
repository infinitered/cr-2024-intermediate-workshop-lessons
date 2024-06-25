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
    {episodes.map((episode, index) => (
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

## Deep link to podcasts from the widget