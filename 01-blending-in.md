# Module 01: Blending in with the surroundings: dynamic text, dark mode, and more

### Goal

Let's make our app feel more cohesive with the operating system by respecting the user's visual settings, including light/dark mode and font scaling.

### Concepts

- Using font scaling wherever we can without breaking the UI and selectively turning it off for text where it doesn't make sense.
- Refactoring a component library to implement a separate dark mode theme.

### Features to build

- Support for font scaling
- Add haptics give our app a more tactile feel
- Switch to dark mode based on the user's operating system settings
  - Also allow overriding of the OS theme setting for the app

### Resources

- React Native docs
  - [allowFontScaling](https://reactnative.dev/docs/text#allowfontscaling)
  - [maxFontSizeMultiplier](https://reactnative.dev/docs/text#maxfontsizemultiplier)
  - [useWindowDimensions().fontScale](https://reactnative.dev/docs/usewindowdimensions#fontscale)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Kadi Kraman's video on _Enhancing your React Native App with Haptics, Sounds and Micro-Animations_](https://www.youtube.com/watch?v=hDGASxkKEXE)
- [Mark Rickert's PR to Ignite for Theming and Documentation](https://github.com/infinitered/ignite/pull/2636)

# Exercises

## Exercise -1: Build the app (if you haven't already)
1. `yarn`
2. `npx expo run:ios` or `npx expo run:android`

## Exercise 0: Drop in some dark mode compatible assets

We'll need some dark mode assets later in this lesson. Copy the `sad-face-dark*.png` files from `files/01` into the project's `assets/images` directory

On macOS or Linux you can download the files with the following commands:

```
curl https://raw.githubusercontent.com/infinitered/cr-2024-intermediate-workshop-lessons/main/files/01/sad-face-dark.png > assets/images/sad-face-dark.png
curl https://raw.githubusercontent.com/infinitered/cr-2024-intermediate-workshop-lessons/main/files/01/sad-face-dark@2x.png > assets/images/sad-face-dark@2x.png
curl https://raw.githubusercontent.com/infinitered/cr-2024-intermediate-workshop-lessons/main/files/01/sad-face-dark@3x.png > assets/images/sad-face-dark@3x.png
```

## Exercise 1: Font scaling that looks good!

One of the greatest benefits to using React Native (and even more so with Expo) is the velocity we can achieve during development. The product team asks, you whip it up and deliver. It works among the team of 5 doing wonders for your next start up.

You'll hand it to the first group of outsiders and potentially will receive some feedback about some button unable to be pressed or some text being unreadable.

Let's explore an aspect of font scaling to help alleviate these pains.

### Device settings prep

Turn on larger text sizes with the instructions below. The steps may vary by OS version or manufacturer (such as Samsung versus Google if using an Android device).

iOS:

1. Settings app
2. Accessibility
3. Display & Text Size
4. Larger Text
5. Drag slider all the way to the larger letter `A`

Android:

1. Settings app
2. Display (or Display Size and Text)
3. Font Size
4. Drag slider to the largest setting

🏃**Try it.** Open up the app after changing the settings. How well can you navigate around? Log in (if not already), scroll down on the lists, switch tabs.

You should notice some obvious challenges just tapping around the app. Let's continue on and learn how we can fix some of these font scaling issues.

### Fix the Log In screen

Notice how we now have to scroll for the sign in button. It's not the worst thing, but it's an example of how to proceed may not be apparent to the user.

We know that the heading and subheading are already large from our style implementation. We can choose to not scale the font on these text components so the screen has a better UX.

`src/app/log-in.tsx`

```typescript
<Text testID="login-heading" tx="loginScreen.signIn" preset="heading" style={$logIn} allowFontScaling={false} />
<Text tx="loginScreen.enterDetails" preset="subheading" style={$enterDetails} allowFontScaling={false} />
```

With these changes, the input labels, inputs and button text are all larger - aiding the user in reading the text. The heading and subheading are still large by design and now everything fits on one screen, so they'll know to click the sign in button.

### Even larger sizes

Sometimes, the user will have larger accessibility sizes (or a zoom) enabled. If you noticed on iOS, you have the ability in the Settings app to unlock some larger text sizes. Change the `Larger Accessibility Sizes` toggle and again drag the slider further to the right.

🏃**Try it.** Flip back to the application and you'll notice the screen has changed again (for the worse, of course, just developer life things)!

We'll fix it but this time we'll use the `maxFontSizeMultiplier` prop to the screen under control. Try the following:

1. Apply `maxFontSizeMultiplier` to the `<TextField />` labels via `LabelTextProps={{ maxFontSizeMultiplier: 2 }}`

Still very readable, but notice the text is still not visible in the textfield itself. If you type, it looks as if nothing is being entered into the fields.

2. Apply `maxFontSizeMultiplier` to the `<TextField />` input via `maxFontSizeMultiplier={2}`

Getting better! The button text still feels extremely large, but you're now fully equipped to deal with this issue. Using a value of 2 for the `maxFontSizeMultiplier` prop is just an example, you can come up with a factor based on the device size, dimensions, font scale and so on.

Feel free to restore your device settings before moving on to the next exercise.

## Exercise 2: Start building our themeable component library

Adding dark mode to your application has its benefits - reduced eye strain, improved readability, increased accessibility for users with light sensitivity. Who wouldn't want to give their users the opportunity to use that in their application?

Let's build out a theming system in the application, which can be used not only for dark mode but your own custom themes if desired (think seasonal themes with custom color palettes for example).

### Color Palette

First, we'll need another color palette entirely for this dark theme. When the theme is active, these are the color values your components and screens will use, as opposed to the light theme (currently the only set of colors).

Create `src/theme/colorsDark.ts` with the following values (or come up with your own color scheme!):

```tsx
const palette = {
  neutral900: "#FFFFFF",
  neutral800: "#F4F2F1",
  neutral700: "#D7CEC9",
  neutral600: "#B6ACA6",
  neutral500: "#978F8A",
  neutral400: "#564E4A",
  neutral300: "#3C3836",
  neutral200: "#191015",
  neutral100: "#000000",

  primary600: "#F4E0D9",
  primary500: "#E8C1B4",
  primary400: "#DDA28E",
  primary300: "#D28468",
  primary200: "#C76542",
  primary100: "#A54F31",

  secondary500: "#DCDDE9",
  secondary400: "#BCC0D6",
  secondary300: "#9196B9",
  secondary200: "#626894",
  secondary100: "#41476E",

  accent500: "#FFEED4",
  accent400: "#FFE1B2",
  accent300: "#FDD495",
  accent200: "#FBC878",
  accent100: "#FFBB50",

  angry100: "#F2D6CD",
  angry500: "#C03403",

  overlay20: "rgba(25, 16, 21, 0.2)",
  overlay50: "rgba(25, 16, 21, 0.5)",
} as const;

export const colors = {
  palette,
  transparent: "rgba(0, 0, 0, 0)",
  text: palette.neutral800,
  textDim: palette.neutral600,
  background: palette.neutral200,
  border: palette.neutral400,
  tint: palette.primary500,
  tintInactive: palette.neutral300,
  separator: palette.neutral300,
  error: palette.angry500,
  errorBackground: palette.angry100,
} as const;
```

Spacing is a first class citizen in Ignite - `src/thee/spacing.ts` has a set of values so your padding, margin and gap values can all be consistent throughout the application. We can consider these values part of the theme like we do colors, this way if you did want separate spacing for a particular theme, you can achieve that.

Create `src/theme/spacingDark.ts` with the following values (again, feel free to mimic the same values as light mode or come up with your own dark mode spacing values):

```tsx
/**
  Use these spacings for margins/paddings and other whitespace throughout your app.
 */
export const spacing = {
  xxxs: 2,
  xxs: 4,
  xs: 8,
  sm: 12,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
  xxxl: 64,
} as const;

export type Spacing = keyof typeof spacing;
```

#### Theme Types and Helpers

Update `src/theme/index.ts` to support both light and dark theme configurations and some helper functions to use later in our styling implementation.

1. First we'll create the type definitions:

```tsx
import type { StyleProp } from "react-native";
import { colors as colorsLight } from "./colors";
import { colors as colorsDark } from "./colorsDark";
import { spacing as spacingLight } from "./spacing";
import { spacing as spacingDark } from "./spacingDark";
import { timing } from "./timing";
import { typography } from "./typography";

// This supports "light" and "dark" themes by default. If undefined, it'll use the system theme
export type ThemeContexts = "light" | "dark" | undefined;

// Because we have two themes, we need to define the types for each of them.
// colorsLight and colorsDark should have the same keys, but different values.
export type Colors = typeof colorsLight | typeof colorsDark;
// The spacing type needs to take into account the different spacing values for light and dark themes.
export type Spacing = typeof spacingLight | typeof spacingDark;

// These two are consistent across themes.
export type Timing = typeof timing;
export type Typography = typeof typography;
```

Here we have the idea of `ThemeContexts`, which in this case will be `light` or `dark` and we'll reserve `undefined` to reference the system theme.

2. Next we'll create the structure of a `Theme` itself, which consists of colors, spacing, typography and timing values.

```tsx
// The overall Theme object should contain all of the data you need to style your app.
export interface Theme {
  colors: Colors;
  spacing: Spacing;
  typography: Typography;
  timing: Timing;
}

// Here we define our themes.
export const lightTheme: Theme = {
  colors: colorsLight,
  spacing: spacingLight,
  typography,
  timing,
};
export const darkTheme: Theme = {
  colors: colorsDark,
  spacing: spacingDark,
  typography,
  timing,
};
```

3. A few helper types that will translate a more primitive React Native style type such as `ViewStyle` into a `ThemedStyle`. Basically, given our current theme objects (colors, spacing, etc), here is what my `ViewStyle` will look like:

```tsx
/**
 * Represents a function that returns a styled component based on the provided theme.
 * @template T The type of the style.
 * @param theme The theme object.
 * @returns The styled component.
 *
 * @example
 * const $container: ThemedStyle<ViewStyle> = (theme) => ({
 *   flex: 1,
 *   backgroundColor: theme.colors.background,
 *   justifyContent: "center",
 *   alignItems: "center",
 * })
 * // Then use in a component like so:
 * const Component = () => {
 *   const { themed } = useAppTheme()
 *   return <View style={themed($container)} />
 * }
 */
export type ThemedStyle<T> = (theme: Theme) => T;
export type ThemedStyleArray<T> = (
  | ThemedStyle<T>
  | StyleProp<T>
  | (StyleProp<T> | ThemedStyle<T>)[]
)[];
```

4. And finally export all these so they're available for use within our project.

```tsx
// Export the two Theme objects:
export {
  colorsLight as colors,
  colorsDark,
  spacingLight as spacing,
  spacingDark,
  typography,
};

export { customFontsToLoad } from "./typography";
```

Note the export here for `colorsLight` and `spacingLight`. We're exporting them as the original `colors` and `spacing` so that the app doesn't just red box everywhere (since they're currently in use, remember, we're applying this to an existing application).

Exporting like this will allow for us to continue using our application even if we haven't finished theming all of the components and routes yet. We can gradually make the changes over time, which is a nicer developer experience.

### `useAppTheme`

Next we'll create the context provider which will give the application access to the current theme information. We'll also create a hook to consume in your components and screens to help create the dynamic styles.

#### ThemeContext

1. Create `src/utils/useAppTheme.ts` and start with the context provider. It will just be a type to keep the `themeScheme` (type of `ThemeContexts`) and the setter function to change this value.

```tsx
import {
  createContext,
  useCallback,
  useContext,
  useMemo,
  useState,
} from "react";
import { Alert, StyleProp, useColorScheme } from "react-native";
import {
  DarkTheme,
  DefaultTheme,
  useTheme as useNavTheme,
} from "@react-navigation/native";
import {
  type Theme,
  type ThemeContexts,
  type ThemedStyle,
  type ThemedStyleArray,
  lightTheme,
  darkTheme,
} from "src/theme";

type ThemeContextType = {
  themeScheme: ThemeContexts;
  setThemeContextOverride: (newTheme: ThemeContexts) => void;
};

// create a React context and provider for the current theme
export const ThemeContext = createContext<ThemeContextType>({
  themeScheme: undefined, // default to the system theme
  setThemeContextOverride: (_newTheme: ThemeContexts) => {
    Alert.alert("setThemeContextOverride not implemented");
  },
});
```

2. Create the `useThemeProvider` hook. This will be used in one of our more top level `_layout.tsx` files just like any other provider with some type of global state.

This way the entire application will have access to them current theme value stored in the context.

It will return the Context Provider, the setter function and the current theme value.

```tsx
export const useThemeProvider = (initialTheme: ThemeContexts = undefined) => {
  const colorScheme = useColorScheme();
  const [overrideTheme, setTheme] = useState<ThemeContexts>(initialTheme);

  const setThemeContextOverride = useCallback((newTheme: ThemeContexts) => {
    setTheme(newTheme);
  }, []);

  const themeScheme = overrideTheme || colorScheme || "light";
  const navigationTheme = themeScheme === "dark" ? DarkTheme : DefaultTheme;

  return {
    themeScheme,
    navigationTheme,
    setThemeContextOverride,
    ThemeProvider: ThemeContext.Provider,
  };
};
```

This Context Provider will keep track of the theme in use. It defaults to `overrideTheme` which can be passed into the hook optionally, followed by the user's preference on the OS (obtained by the `useColorScheme` hook from `react-native`) and finally just defaults to light.

#### The Hook

In the same file, `src/utils/useAppTheme.ts`, add the hook.

1. Let's start with the interface for the value being returned by the hook. It will return an object

```tsx
interface UseAppThemeValue {
  // The theme object from react-navigation
  navTheme: typeof DefaultTheme;
  // A function to set the theme context override (for switching modes)
  setThemeContextOverride: (newTheme: ThemeContexts) => void;
  // The current theme object
  theme: Theme;
  // The current theme context "light" | "dark"
  themeContext: ThemeContexts;
  // A function to apply the theme to a style object.
  themed: <T>(
    styleOrStyleFn: ThemedStyle<T> | StyleProp<T> | ThemedStyleArray<T>
  ) => T;
}
```

2. Implement the `useAppTheme` hook that can be leveraged in your components throughout the application.

Two of the more important properties returned here are the `theme` and the callback `themed` - which will be used to wrap any `ThemeStyle<T>` object before passing it into a component's `style property`

```tsx
/**
 * Custom hook that provides the app theme and utility functions for theming.
 *
 * @returns {UseAppThemeReturn} An object containing various theming values and utilities.
 * @throws {Error} If used outside of a ThemeProvider.
 */
export const useAppTheme = (): UseAppThemeValue => {
  const navTheme = useNavTheme();
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }

  const { themeScheme: overrideTheme, setThemeContextOverride } = context;

  const themeContext: ThemeContexts = useMemo(
    () => overrideTheme || (navTheme.dark ? "dark" : "light"),
    [overrideTheme, navTheme]
  );

  const themeVariant: Theme = useMemo(
    () => (themeContext === "dark" ? darkTheme : lightTheme),
    [themeContext]
  );

  const themed = useCallback(
    <T,>(
      styleOrStyleFn: ThemedStyle<T> | StyleProp<T> | ThemedStyleArray<T>
    ) => {
      const flatStyles = [styleOrStyleFn].flat(3) as (
        | ThemedStyle<T>
        | StyleProp<T>
      )[];
      const stylesArray = flatStyles.map((f) => {
        if (typeof f === "function") {
          return (f as ThemedStyle<T>)(themeVariant);
        } else {
          return f;
        }
      });

      // Flatten the array of styles into a single object
      return Object.assign({}, ...stylesArray) as T;
    },
    [themeVariant]
  );

  return {
    navTheme,
    setThemeContextOverride,
    theme: themeVariant,
    themeContext,
    themed,
  };
};
```

All the set up is just about there to start seeing our theming in action. We're now ready to apply the theme to our existing components and screens!

#### Applying the Theme

First, wrap the app in the Theme Provider we just created inside `src/app/(app)/_layout.tsx`:

```diff
import { useStores } from "src/models"
import { useFonts } from "expo-font"
+ import {  useThemeProvider } from "src/utils/useAppTheme"

export default observer(function Layout() {
  const {
    authenticationStore: { isAuthenticated },
+   profileStore: { profile: { darkMode } }
  } = useStores()

+ const { themeScheme, setThemeContextOverride, ThemeProvider } = useThemeProvider(darkMode ? "dark" : "light")

  if (!isAuthenticated) {
    return <Redirect href="/log-in" />
  }

return (
+ <ThemeProvider value={{ themeScheme, setThemeContextOverride }}>
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="(tabs)" />
    </Stack>
+ </ThemeProvider>
  )
})
```

This will make the theme value available to the entire application. Something to note here, we pass in the preference from the `profileStore` into `useThemeProvider`. This will allow us to retrieve the persisted value during app reloads (or in the real world, when the app gets killed, phone is restarted, etc).

#### Toggling the theme at runtime

We'll enable the switching between light and dark theme on the `profile` route. This will allow us to toggle the theme at runtime and we can see our dark theming in action as we begin to style our components, followed by the `podcast` screen.

Update `src/app/(app)/(tabs)/profile.tsx`:

1. Call the `useAppTheme` hook

```diff
//...
- import {TextStyle, View, ViewStyle } from "react-native"
+ import { LayoutAnimation, TextStyle, View, ViewStyle } from "react-native"
+ import { useAppTheme } from "src/utils/useAppTheme"
// ...

export default observer(function ProfileScreen() {
  const {
    profileStore: { profile },
    authenticationStore: { logout },
  } = useStores()
+ const { setThemeContextOverride, themeContext } = useAppTheme()
```

2. Define a callback function to toggle the theme before the returned JSX.

```tsx
const toggleTheme = React.useCallback(() => {
  LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut); // Animate the transition
  setThemeContextOverride(themeContext === "dark" ? "light" : "dark");
}, [themeContext, setThemeContextOverride]);
```

3. Fire the callback in the `onPress` of the `<Toggle />`

```diff
<Toggle
  labelTx="demoProfileScreen.darkMode"
  variant="switch"
  labelPosition="left"
  containerStyle={$textField}
- value={darkMode}
- onPress={() => setProp("darkMode", !darkMode)}
+ value={themeContext === "dark"}
+  onPress={() => {
+   setProp("darkMode", !darkMode)
+   toggleTheme()
+ }}
/>
```

Now the theme value is being updated when the user switches their selection. It is also persisted to the MST store, so we can initialize the proper value when the app is restarted.

🏃**Try it.** Navigate to the profile tab and toggle your dark mode selection. Refresh the app and see that it sticks!

Notice, nothing magically changes, we'll implement that magic next! :magic_wand:

#### Update the component library

Next we'll want to update our component library to utilize the theming values in our styles. For this lesson, we're going to enhance the Podcast index screen to support our dark mode.

To do so, we'll need Button, Card, EmptyState, Screen, Text and Toggle to be themeable. First, we'll walk through the changes to `Button`. Below that will be the diffs for each of the other completed components. Go for it without peeking 🙈 or walk through the code to spot the differences yourself and understand where the changes are being made.

If you get stuck, you can copy the full file from `files/01/components`.

### Themed Button

With the hooks and utility functions we created earlier, we will update the `Button` component to receive colors and spacing from the current theme.

1. Import the hook

```diff
import React, { ComponentType } from "react";
import {
  Pressable,
  PressableProps,
  PressableStateCallbackType,
  StyleProp,
  TextStyle,
  ViewStyle,
} from "react-native";
+import type { ThemedStyle, ThemedStyleArray } from "src/theme";
import { Text, TextProps } from "./Text";
+import { useAppTheme } from "src/utils/useAppTheme";
```

2. Call the hook under the `props` destructuring

```diff
export function Button(props: ButtonProps) {
  const {
    tx,
    text,
    txOptions,
    style: $viewStyleOverride,
    pressedStyle: $pressedViewStyleOverride,
    textStyle: $textStyleOverride,
    pressedTextStyle: $pressedTextStyleOverride,
    disabledTextStyle: $disabledTextStyleOverride,
    children,
    RightAccessory,
    LeftAccessory,
    disabled,
    disabledStyle: $disabledViewStyleOverride,
    TextProps,
    ...rest
  } = props;

+  const { themed } = useAppTheme();
```

3. Update dynamic styles defined as functions

```diff
function $viewStyle({
  pressed,
}: PressableStateCallbackType): StyleProp<ViewStyle> {
  return [
+    themed($viewPresets[preset]),
    $viewStyleOverride,
    !!pressed &&
+      themed([$pressedViewPresets[preset], $pressedViewStyleOverride]),
    !!disabled && $disabledViewStyleOverride,
  ];

function $textStyle({
  pressed,
}: PressableStateCallbackType): StyleProp<TextStyle> {
  return [
+    themed($textPresets[preset]),
    $textStyleOverride,
    !!pressed &&
+      themed([$pressedTextPresets[preset], $pressedTextStyleOverride]),
    !!disabled && $disabledTextStyleOverride,
  ];
}
```

4. Update the style objects

```diff
+const $baseViewStyle: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  minHeight: 56,
  borderRadius: 4,
  justifyContent: "center",
  alignItems: "center",
  flexDirection: "row",
  paddingVertical: spacing.sm,
  paddingHorizontal: spacing.sm,
  overflow: "hidden",
+});

+const $baseTextStyle: ThemedStyle<TextStyle> = ({ typography }) => ({
  fontSize: 16,
  lineHeight: 20,
  fontFamily: typography.primary.medium,
  textAlign: "center",
  flexShrink: 1,
  flexGrow: 0,
  zIndex: 2,
+});

+const $rightAccessoryStyle: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  marginStart: spacing.xs,
  zIndex: 1,
+});

+const $leftAccessoryStyle: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  marginEnd: spacing.xs,
  zIndex: 1,
+});

+const $viewPresets: Record<Presets, ThemedStyleArray<ViewStyle>> = {
  default: [
    $baseViewStyle,
+   ({ colors }) => ({
      borderWidth: 1,
      borderColor: colors.palette.neutral400,
      backgroundColor: colors.palette.neutral100,
+  }),
  ],
  filled: [
    $baseViewStyle,
+    ({ colors }) => ({ backgroundColor: colors.palette.neutral300 }),
  ],
  reversed: [
    $baseViewStyle,
+    ({ colors }) => ({ backgroundColor: colors.palette.neutral800 }),
  ],
};

+const $textPresets: Record<Presets, ThemedStyleArray<TextStyle>> = {
  default: [$baseTextStyle],
  filled: [$baseTextStyle],
  reversed: [
    $baseTextStyle,
+    ({ colors }) => ({ color: colors.palette.neutral100 }),
  ],
};

+const $pressedViewPresets: Record<Presets, ThemedStyle<ViewStyle>> = {
+  default: ({ colors }) => ({ backgroundColor: colors.palette.neutral200 }),
+  filled: ({ colors }) => ({ backgroundColor: colors.palette.neutral400 }),
+  reversed: ({ colors }) => ({ backgroundColor: colors.palette.neutral700 }),
};

+const $pressedTextPresets: Record<Presets, ThemedStyle<ViewStyle>> = {
  default: () => ({ opacity: 0.9 }),
  filled: () => ({ opacity: 0.9 }),
  reversed: () => ({ opacity: 0.9 }),
};

```

</details>

<br />

🏃**Try it.** Now that you've styled the `<Button />` component, toggle the theme back and forth from the `profile` route. Observe the button text changes from dark to light and the background from light to dark.

You're on your way now! Complete the rest of the necessary components for the `podcast` screen. You'll see that as you tackle these components, the rest of the app will begin changing as an added bonus!

### Themed Card, EmptyState, Screen, Text and Toggle

Work on the remaining components, expand the diffs if you need!

<details>
  <summary>src/components/Card.tsx</summary>

```diff
import React, {
  Fragment,
  ReactElement,
  RefAttributes,
  forwardRef,
} from "react";
import {
  StyleProp,
  TextStyle,
  TouchableOpacity,
  TouchableOpacityProps,
  View,
  ViewProps,
  ViewStyle,
} from "react-native";
+ import type { ThemedStyle, ThemedStyleArray } from "src/theme";
import { Text, TextProps } from "./Text";
+ import { useAppTheme } from "src/utils/useAppTheme";

type Presets = "default" | "reversed";

interface CardProps extends TouchableOpacityProps {
  /**
   * One of the different types of text presets.
   */
  preset?: Presets;
  /**
   * How the content should be aligned vertically. This is especially (but not exclusively) useful
   * when the card is a fixed height but the content is dynamic.
   *
   * `top` (default) - aligns all content to the top.
   * `center` - aligns all content to the center.
   * `space-between` - spreads out the content evenly.
   * `force-footer-bottom` - aligns all content to the top, but forces the footer to the bottom.
   */
  verticalAlignment?:
    | "top"
    | "center"
    | "space-between"
    | "force-footer-bottom";
  /**
   * Custom component added to the left of the card body.
   */
  LeftComponent?: ReactElement;
  /**
   * Custom component added to the right of the card body.
   */
  RightComponent?: ReactElement;
  /**
   * The heading text to display if not using `headingTx`.
   */
  heading?: TextProps["text"];
  /**
   * Heading text which is looked up via i18n.
   */
  headingTx?: TextProps["tx"];
  /**
   * Optional heading options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  headingTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for heading text.
   */
  headingStyle?: StyleProp<TextStyle>;
  /**
   * Pass any additional props directly to the heading Text component.
   */
  HeadingTextProps?: TextProps;
  /**
   * Custom heading component.
   * Overrides all other `heading*` props.
   */
  HeadingComponent?: ReactElement;
  /**
   * The content text to display if not using `contentTx`.
   */
  content?: TextProps["text"];
  /**
   * Content text which is looked up via i18n.
   */
  contentTx?: TextProps["tx"];
  /**
   * Optional content options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  contentTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for content text.
   */
  contentStyle?: StyleProp<TextStyle>;
  /**
   * Pass any additional props directly to the content Text component.
   */
  ContentTextProps?: TextProps;
  /**
   * Custom content component.
   * Overrides all other `content*` props.
   */
  ContentComponent?: ReactElement;
  /**
   * The footer text to display if not using `footerTx`.
   */
  footer?: TextProps["text"];
  /**
   * Footer text which is looked up via i18n.
   */
  footerTx?: TextProps["tx"];
  /**
   * Optional footer options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  footerTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for footer text.
   */
  footerStyle?: StyleProp<TextStyle>;
  /**
   * Pass any additional props directly to the footer Text component.
   */
  FooterTextProps?: TextProps;
  /**
   * Custom footer component.
   * Overrides all other `footer*` props.
   */
  FooterComponent?: ReactElement;
}

type RefProps = RefAttributes<View>;

/**
 * Cards are useful for displaying related information in a contained way.
 * If a ListItem displays content horizontally, a Card can be used to display content vertically.
 * @see [Documentation and Examples]{@link https://docs.infinite.red/ignite-cli/boilerplate/components/Card/}
 * @param {CardProps} props - The props for the `Card` component.
 * @returns {JSX.Element} The rendered `Card` component.
 */
export const Card = forwardRef(function Card(props: CardProps, ref) {
  const {
    content,
    contentTx,
    contentTxOptions,
    footer,
    footerTx,
    footerTxOptions,
    heading,
    headingTx,
    headingTxOptions,
    ContentComponent,
    HeadingComponent,
    FooterComponent,
    LeftComponent,
    RightComponent,
    verticalAlignment = "top",
    style: $containerStyleOverride,
    contentStyle: $contentStyleOverride,
    headingStyle: $headingStyleOverride,
    footerStyle: $footerStyleOverride,
    ContentTextProps,
    HeadingTextProps,
    FooterTextProps,
    ...WrapperProps
  } = props;

+  const {
+    theme: { colors },
+    themed,
+  } = useAppTheme()

  const preset: Presets = props.preset ?? "default";
  const isPressable = !!WrapperProps.onPress;
  const isHeadingPresent = !!(HeadingComponent || heading || headingTx);
  const isContentPresent = !!(ContentComponent || content || contentTx);
  const isFooterPresent = !!(FooterComponent || footer || footerTx);

  const Wrapper = forwardRef<View, typeof WrapperProps & RefProps>(
    function Wrapper(wrapperProps, innerRef) {
      const Component = isPressable
        ? TouchableOpacity
        : (View as React.ComponentType<TouchableOpacityProps | ViewProps>);
      return <Component ref={innerRef} {...wrapperProps} />;
    }
  );

  const HeaderContentWrapper =
    verticalAlignment === "force-footer-bottom" ? View : Fragment;

  const $containerStyle: StyleProp<ViewStyle> = [
+    themed($containerPresets[preset]),
    $containerStyleOverride,
  ];
  const $headingStyle = [
+    themed($headingPresets[preset]),
    (isFooterPresent || isContentPresent) && { marginBottom: spacing.xxxs },
    $headingStyleOverride,
    HeadingTextProps?.style,
  ];
  const $contentStyle = [
+    themed($contentPresets[preset]),
    isHeadingPresent && { marginTop: spacing.xxxs },
    isFooterPresent && { marginBottom: spacing.xxxs },
    $contentStyleOverride,
    ContentTextProps?.style,
  ];
  const $footerStyle = [
+    themed($footerPresets[preset]),
    (isHeadingPresent || isContentPresent) && { marginTop: spacing.xxxs },
    $footerStyleOverride,
    FooterTextProps?.style,
  ];
  const $alignmentWrapperStyle = [
    $alignmentWrapper,
    { justifyContent: $alignmentWrapperFlexOptions[verticalAlignment] },
    LeftComponent && { marginStart: spacing.md },
    RightComponent && { marginEnd: spacing.md },
  ];

  return (
    <Wrapper
      ref={ref}
      style={$containerStyle}
      activeOpacity={0.8}
      accessibilityRole={isPressable ? "button" : undefined}
      {...WrapperProps}
    >
      {LeftComponent}

      <View style={$alignmentWrapperStyle}>
        <HeaderContentWrapper>
          {HeadingComponent ||
            (isHeadingPresent && (
              <Text
                weight="bold"
                text={heading}
                tx={headingTx}
                txOptions={headingTxOptions}
                {...HeadingTextProps}
                style={$headingStyle}
              />
            ))}

          {ContentComponent ||
            (isContentPresent && (
              <Text
                weight="normal"
                text={content}
                tx={contentTx}
                txOptions={contentTxOptions}
                {...ContentTextProps}
                style={$contentStyle}
              />
            ))}
        </HeaderContentWrapper>

        {FooterComponent ||
          (isFooterPresent && (
            <Text
              weight="normal"
              size="xs"
              text={footer}
              tx={footerTx}
              txOptions={footerTxOptions}
              {...FooterTextProps}
              style={$footerStyle}
            />
          ))}
      </View>

      {RightComponent}
    </Wrapper>
  );
});

+const $containerBase: ThemedStyle<ViewStyle> = (theme) => ({
  borderRadius: theme.spacing.md,
  padding: theme.spacing.xs,
  borderWidth: 1,
  shadowColor: theme.colors.palette.neutral800,
  shadowOffset: { width: 0, height: 12 },
  shadowOpacity: 0.08,
  shadowRadius: 12.81,
  elevation: 16,
  minHeight: 96,
  flexDirection: "row",
+});

const $alignmentWrapper: ViewStyle = {
  flex: 1,
  alignSelf: "stretch",
};

+const $alignmentWrapperFlexOptions = {
  top: "flex-start",
  center: "center",
  "space-between": "space-between",
  "force-footer-bottom": "space-between",
+} as const;

+const $containerPresets: Record<Presets, ThemedStyleArray<ViewStyle>> = {
  default: [
    $containerBase,
+    (theme) => ({
      backgroundColor: theme.colors.palette.neutral100,
      borderColor: theme.colors.palette.neutral300,
+    }),
  ],
  reversed: [
    $containerBase,
+    (theme) => ({
      backgroundColor: theme.colors.palette.neutral800,
      borderColor: theme.colors.palette.neutral500,
+    }),
  ],
};

+const $headingPresets: Record<Presets, ThemedStyleArray<TextStyle>> = {
  default: [],
+  reversed: [(theme) => ({ color: theme.colors.palette.neutral100 })],
+};

+const $contentPresets: Record<Presets, ThemedStyleArray<TextStyle>> = {
  default: [],
+  reversed: [(theme) => ({ color: theme.colors.palette.neutral100 })],
+};

+const $footerPresets: Record<Presets, ThemedStyleArray<TextStyle>> = {
  default: [],
+  reversed: [(theme) => ({ color: theme.colors.palette.neutral100 })],
+};
```

</details>

<details>
  <summary>src/components/EmptyState.tsx</summary>

```diff
import React from "react";
import {
  Image,
  ImageProps,
  ImageStyle,
  StyleProp,
  TextStyle,
  View,
  ViewStyle,
} from "react-native";
import { translate } from "../i18n";
import { Button, ButtonProps } from "./Button";
import { Text, TextProps } from "./Text";
+import { useAppTheme } from "src/utils/useAppTheme";
+import type { ThemedStyle } from "src/theme";

const sadFace = require("../../assets/images/sad-face.png");
const sadFaceDark = require("../../assets/images/sad-face-dark.png");

interface EmptyStateProps {
  /**
   * An optional prop that specifies the text/image set to use for the empty state.
   */
  preset?: "generic";
  /**
   * Style override for the container.
   */
  style?: StyleProp<ViewStyle>;
  /**
   * An Image source to be displayed above the heading.
   */
  imageSource?: ImageProps["source"];
  /**
   * Style overrides for image.
   */
  imageStyle?: StyleProp<ImageStyle>;
  /**
   * Pass any additional props directly to the Image component.
   */
  ImageProps?: Omit<ImageProps, "source">;
  /**
   * The heading text to display if not using `headingTx`.
   */
  heading?: TextProps["text"];
  /**
   * Heading text which is looked up via i18n.
   */
  headingTx?: TextProps["tx"];
  /**
   * Optional heading options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  headingTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for heading text.
   */
  headingStyle?: StyleProp<TextStyle>;
  /**
   * Pass any additional props directly to the heading Text component.
   */
  HeadingTextProps?: TextProps;
  /**
   * The content text to display if not using `contentTx`.
   */
  content?: TextProps["text"];
  /**
   * Content text which is looked up via i18n.
   */
  contentTx?: TextProps["tx"];
  /**
   * Optional content options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  contentTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for content text.
   */
  contentStyle?: StyleProp<TextStyle>;
  /**
   * Pass any additional props directly to the content Text component.
   */
  ContentTextProps?: TextProps;
  /**
   * The button text to display if not using `buttonTx`.
   */
  button?: TextProps["text"];
  /**
   * Button text which is looked up via i18n.
   */
  buttonTx?: TextProps["tx"];
  /**
   * Optional button options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  buttonTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for button.
   */
  buttonStyle?: ButtonProps["style"];
  /**
   * Style overrides for button text.
   */
  buttonTextStyle?: ButtonProps["textStyle"];
  /**
   * Called when the button is pressed.
   */
  buttonOnPress?: ButtonProps["onPress"];
  /**
   * Pass any additional props directly to the Button component.
   */
  ButtonProps?: ButtonProps;
}

interface EmptyStatePresetItem {
  imageSource: ImageProps["source"];
  heading: TextProps["text"];
  content: TextProps["text"];
  button: TextProps["text"];
}

/**
 * A component to use when there is no data to display. It can be utilized to direct the user what to do next.
 * @see [Documentation and Examples]{@link https://docs.infinite.red/ignite-cli/boilerplate/components/EmptyState/}
 * @param {EmptyStateProps} props - The props for the `EmptyState` component.
 * @returns {JSX.Element} The rendered `EmptyState` component.
 */
export function EmptyState(props: EmptyStateProps) {
+  const {
+    themeContext,
+    themed,
+    theme: { spacing },
+  } = useAppTheme();

  const EmptyStatePresets = {
    generic: {
+      imageSource: themeContext === "dark" ? sadFaceDark : sadFace,
      heading: translate("emptyStateComponent.generic.heading"),
      content: translate("emptyStateComponent.generic.content"),
      button: translate("emptyStateComponent.generic.button"),
    } as EmptyStatePresetItem,
  } as const;

  const preset = EmptyStatePresets[props.preset ?? "generic"];

  const {
    button = preset.button,
    buttonTx,
    buttonOnPress,
    buttonTxOptions,
    content = preset.content,
    contentTx,
    contentTxOptions,
    heading = preset.heading,
    headingTx,
    headingTxOptions,
    imageSource = preset.imageSource,
    style: $containerStyleOverride,
    buttonStyle: $buttonStyleOverride,
    buttonTextStyle: $buttonTextStyleOverride,
    contentStyle: $contentStyleOverride,
    headingStyle: $headingStyleOverride,
    imageStyle: $imageStyleOverride,
    ButtonProps,
    ContentTextProps,
    HeadingTextProps,
    ImageProps,
  } = props;

  const isImagePresent = !!imageSource;
  const isHeadingPresent = !!(heading || headingTx);
  const isContentPresent = !!(content || contentTx);
  const isButtonPresent = !!(button || buttonTx);

  const $containerStyles = [$containerStyleOverride];
  const $imageStyles = [
    $image,
    (isHeadingPresent || isContentPresent || isButtonPresent) && {
      marginBottom: spacing.xxxs,
    },
    $imageStyleOverride,
    ImageProps?.style,
  ];
  const $headingStyles = [
+    themed($heading),
    isImagePresent && { marginTop: spacing.xxxs },
    (isContentPresent || isButtonPresent) && { marginBottom: spacing.xxxs },
    $headingStyleOverride,
    HeadingTextProps?.style,
  ];
  const $contentStyles = [
+    themed($content),
    (isImagePresent || isHeadingPresent) && { marginTop: spacing.xxxs },
    isButtonPresent && { marginBottom: spacing.xxxs },
    $contentStyleOverride,
    ContentTextProps?.style,
  ];
  const $buttonStyles = [
    (isImagePresent || isHeadingPresent || isContentPresent) && {
      marginTop: spacing.xl,
    },
    $buttonStyleOverride,
    ButtonProps?.style,
  ];

  return (
    <View style={$containerStyles}>
      {isImagePresent && (
        <Image source={imageSource} {...ImageProps} style={$imageStyles} />
      )}

      {isHeadingPresent && (
        <Text
          preset="subheading"
          text={heading}
          tx={headingTx}
          txOptions={headingTxOptions}
          {...HeadingTextProps}
          style={$headingStyles}
        />
      )}

      {isContentPresent && (
        <Text
          text={content}
          tx={contentTx}
          txOptions={contentTxOptions}
          {...ContentTextProps}
          style={$contentStyles}
        />
      )}

      {isButtonPresent && (
        <Button
          onPress={buttonOnPress}
          text={button}
          tx={buttonTx}
          txOptions={buttonTxOptions}
          textStyle={$buttonTextStyleOverride}
          {...ButtonProps}
          style={$buttonStyles}
        />
      )}
    </View>
  );
}

const $image: ImageStyle = { alignSelf: "center" };
+const $heading: ThemedStyle<TextStyle> = ({ spacing }) => ({
  textAlign: "center",
  paddingHorizontal: spacing.lg,
+});
+const $content: ThemedStyle<TextStyle> = ({ spacing }) => ({
  textAlign: "center",
  paddingHorizontal: spacing.lg,
+});
```

</details>

<details>
  <summary>src/components/Screen.tsx</summary>

```diff
import { useScrollToTop } from "@react-navigation/native";
import { StatusBar, StatusBarProps, StatusBarStyle } from "expo-status-bar";
import React, { useRef, useState } from "react";
import {
  KeyboardAvoidingView,
  KeyboardAvoidingViewProps,
  LayoutChangeEvent,
  Platform,
  ScrollView,
  ScrollViewProps,
  StyleProp,
  View,
  ViewStyle,
} from "react-native";
import {
  ExtendedEdge,
  useSafeAreaInsetsStyle,
} from "../utils/useSafeAreaInsetsStyle";
+import { useAppTheme } from "src/utils/useAppTheme";

interface BaseScreenProps {
  /**
   * Children components.
   */
  children?: React.ReactNode;
  /**
   * Style for the outer content container useful for padding & margin.
   */
  style?: StyleProp<ViewStyle>;
  /**
   * Style for the inner content container useful for padding & margin.
   */
  contentContainerStyle?: StyleProp<ViewStyle>;
  /**
   * Override the default edges for the safe area.
   */
  safeAreaEdges?: ExtendedEdge[];
  /**
   * Background color
   */
  backgroundColor?: string;
  /**
   * Status bar setting. Defaults to dark.
   */
  statusBarStyle?: StatusBarStyle;
  /**
   * By how much should we offset the keyboard? Defaults to 0.
   */
  keyboardOffset?: number;
  /**
   * Pass any additional props directly to the StatusBar component.
   */
  StatusBarProps?: StatusBarProps;
  /**
   * Pass any additional props directly to the KeyboardAvoidingView component.
   */
  KeyboardAvoidingViewProps?: KeyboardAvoidingViewProps;
}

interface FixedScreenProps extends BaseScreenProps {
  preset?: "fixed";
}
interface ScrollScreenProps extends BaseScreenProps {
  preset?: "scroll";
  /**
   * Should keyboard persist on screen tap. Defaults to handled.
   * Only applies to scroll preset.
   */
  keyboardShouldPersistTaps?: "handled" | "always" | "never";
  /**
   * Pass any additional props directly to the ScrollView component.
   */
  ScrollViewProps?: ScrollViewProps;
}

interface AutoScreenProps extends Omit<ScrollScreenProps, "preset"> {
  preset?: "auto";
  /**
   * Threshold to trigger the automatic disabling/enabling of scroll ability.
   * Defaults to `{ percent: 0.92 }`.
   */
  scrollEnabledToggleThreshold?: { percent?: number; point?: number };
}

export type ScreenProps =
  | ScrollScreenProps
  | FixedScreenProps
  | AutoScreenProps;

const isIos = Platform.OS === "ios";

type ScreenPreset = "fixed" | "scroll" | "auto";

/**
 * @param {ScreenPreset?} preset - The preset to check.
 * @returns {boolean} - Whether the preset is non-scrolling.
 */
function isNonScrolling(preset?: ScreenPreset) {
  return !preset || preset === "fixed";
}

/**
 * Custom hook that handles the automatic enabling/disabling of scroll ability based on the content size and screen size.
 * @param {UseAutoPresetProps} props - The props for the `useAutoPreset` hook.
 * @returns {{boolean, Function, Function}} - The scroll state, and the `onContentSizeChange` and `onLayout` functions.
 */
function useAutoPreset(props: AutoScreenProps): {
  scrollEnabled: boolean;
  onContentSizeChange: (w: number, h: number) => void;
  onLayout: (e: LayoutChangeEvent) => void;
} {
  const { preset, scrollEnabledToggleThreshold } = props;
  const { percent = 0.92, point = 0 } = scrollEnabledToggleThreshold || {};

  const scrollViewHeight = useRef<null | number>(null);
  const scrollViewContentHeight = useRef<null | number>(null);
  const [scrollEnabled, setScrollEnabled] = useState(true);

  function updateScrollState() {
    if (
      scrollViewHeight.current === null ||
      scrollViewContentHeight.current === null
    )
      return;

    // check whether content fits the screen then toggle scroll state according to it
    const contentFitsScreen = (function () {
      if (point) {
        return (
          scrollViewContentHeight.current < scrollViewHeight.current - point
        );
      } else {
        return (
          scrollViewContentHeight.current < scrollViewHeight.current * percent
        );
      }
    })();

    // content is less than the size of the screen, so we can disable scrolling
    if (scrollEnabled && contentFitsScreen) setScrollEnabled(false);

    // content is greater than the size of the screen, so let's enable scrolling
    if (!scrollEnabled && !contentFitsScreen) setScrollEnabled(true);
  }

  /**
   * @param {number} w - The width of the content.
   * @param {number} h - The height of the content.
   */
  function onContentSizeChange(w: number, h: number) {
    // update scroll-view content height
    scrollViewContentHeight.current = h;
    updateScrollState();
  }

  /**
   * @param {LayoutChangeEvent} e = The layout change event.
   */
  function onLayout(e: LayoutChangeEvent) {
    const { height } = e.nativeEvent.layout;
    // update scroll-view  height
    scrollViewHeight.current = height;
    updateScrollState();
  }

  // update scroll state on every render
  if (preset === "auto") updateScrollState();

  return {
    scrollEnabled: preset === "auto" ? scrollEnabled : true,
    onContentSizeChange,
    onLayout,
  };
}

/**
 * @param {ScreenProps} props - The props for the `ScreenWithoutScrolling` component.
 * @returns {JSX.Element} - The rendered `ScreenWithoutScrolling` component.
 */
function ScreenWithoutScrolling(props: ScreenProps) {
  const { style, contentContainerStyle, children } = props;
  return (
    <View style={[$outerStyle, style]}>
      <View style={[$innerStyle, contentContainerStyle]}>{children}</View>
    </View>
  );
}

/**
 * @param {ScreenProps} props - The props for the `ScreenWithScrolling` component.
 * @returns {JSX.Element} - The rendered `ScreenWithScrolling` component.
 */
function ScreenWithScrolling(props: ScreenProps) {
  const {
    children,
    keyboardShouldPersistTaps = "handled",
    contentContainerStyle,
    ScrollViewProps,
    style,
  } = props as ScrollScreenProps;

  const ref = useRef<ScrollView>(null);

  const { scrollEnabled, onContentSizeChange, onLayout } = useAutoPreset(
    props as AutoScreenProps
  );

  // Add native behavior of pressing the active tab to scroll to the top of the content
  // More info at: https://reactnavigation.org/docs/use-scroll-to-top/
  useScrollToTop(ref);

  return (
    <ScrollView
      {...{ keyboardShouldPersistTaps, scrollEnabled, ref }}
      {...ScrollViewProps}
      onLayout={(e) => {
        onLayout(e);
        ScrollViewProps?.onLayout?.(e);
      }}
      onContentSizeChange={(w: number, h: number) => {
        onContentSizeChange(w, h);
        ScrollViewProps?.onContentSizeChange?.(w, h);
      }}
      style={[$outerStyle, ScrollViewProps?.style, style]}
      contentContainerStyle={[
        $innerStyle,
        ScrollViewProps?.contentContainerStyle,
        contentContainerStyle,
      ]}
    >
      {children}
    </ScrollView>
  );
}

/**
 * Represents a screen component that provides a consistent layout and behavior for different screen presets.
 * The `Screen` component can be used with different presets such as "fixed", "scroll", or "auto".
 * It handles safe area insets, status bar settings, keyboard avoiding behavior, and scrollability based on the preset.
 * @see [Documentation and Examples]{@link https://docs.infinite.red/ignite-cli/boilerplate/app/components/Screen/}
 * @param {ScreenProps} props - The props for the `Screen` component.
 * @returns {JSX.Element} The rendered `Screen` component.
 */
export function Screen(props: ScreenProps) {
+  const {
+    theme: { colors },
+    themeContext,
+  } = useAppTheme();
  const {
    backgroundColor,
    KeyboardAvoidingViewProps,
    keyboardOffset = 0,
    safeAreaEdges,
    StatusBarProps,
    statusBarStyle,
  } = props;

  const $containerInsets = useSafeAreaInsetsStyle(safeAreaEdges);

  return (
    <View
      style={[
        $containerStyle,
        { backgroundColor: backgroundColor || colors.background },
        $containerInsets,
      ]}
    >
      <StatusBar
+        style={statusBarStyle || (themeContext === "dark" ? "light" : "dark")}
        {...StatusBarProps}
      />

      <KeyboardAvoidingView
        behavior={isIos ? "padding" : "height"}
        keyboardVerticalOffset={keyboardOffset}
        {...KeyboardAvoidingViewProps}
        style={[$keyboardAvoidingViewStyle, KeyboardAvoidingViewProps?.style]}
      >
        {isNonScrolling(props.preset) ? (
          <ScreenWithoutScrolling {...props} />
        ) : (
          <ScreenWithScrolling {...props} />
        )}
      </KeyboardAvoidingView>
    </View>
  );
}

const $containerStyle: ViewStyle = {
  flex: 1,
  height: "100%",
  width: "100%",
};

const $keyboardAvoidingViewStyle: ViewStyle = {
  flex: 1,
};

const $outerStyle: ViewStyle = {
  flex: 1,
  height: "100%",
  width: "100%",
};

const $innerStyle: ViewStyle = {
  justifyContent: "flex-start",
  alignItems: "stretch",
};
```

</details>

<details>
  <summary>src/components/Text.tsx</summary>

```diff
import i18n from "i18n-js";
import React from "react";
import {
  StyleProp,
  Text as RNText,
  TextProps as RNTextProps,
  TextStyle,
} from "react-native";
import { isRTL, translate, TxKeyPath } from "../i18n";
+import type { ThemedStyle, ThemedStyleArray } from "src/theme";
+import { useAppTheme } from "src/utils/useAppTheme";
import { typography } from "src/theme/typography";

type Sizes = keyof typeof $sizeStyles;
type Weights = keyof typeof typography.primary;
type Presets =
  | "default"
  | "bold"
  | "heading"
  | "subheading"
  | "formLabel"
  | "formHelper";

export interface TextProps extends RNTextProps {
  /**
   * Text which is looked up via i18n.
   */
  tx?: TxKeyPath;
  /**
   * The text to display if not using `tx` or nested components.
   */
  text?: string;
  /**
   * Optional options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  txOptions?: i18n.TranslateOptions;
  /**
   * An optional style override useful for padding & margin.
   */
  style?: StyleProp<TextStyle>;
  /**
   * One of the different types of text presets.
   */
  preset?: Presets;
  /**
   * Text weight modifier.
   */
  weight?: Weights;
  /**
   * Text size modifier.
   */
  size?: Sizes;
  /**
   * Children components.
   */
  children?: React.ReactNode;
}

/**
 * For your text displaying needs.
 * This component is a HOC over the built-in React Native one.
 * @see [Documentation and Examples]{@link https://docs.infinite.red/ignite-cli/boilerplate/components/Text/}
 * @param {TextProps} props - The props for the `Text` component.
 * @returns {JSX.Element} The rendered `Text` component.
 */
export function Text(props: TextProps) {
  const {
    weight,
    size,
    tx,
    txOptions,
    text,
    children,
    style: $styleOverride,
    ...rest
  } = props;
+  const { themed } = useAppTheme();

  const i18nText = tx && translate(tx, txOptions);
  const content = i18nText || text || children;

  const preset: Presets = props.preset ?? "default";
  const $styles: StyleProp<TextStyle> = [
    $rtlStyle,
+    themed($presets[preset]),
    weight && $fontWeightStyles[weight],
    size && $sizeStyles[size],
    $styleOverride,
  ];

  return (
    <RNText {...rest} style={$styles}>
      {content}
    </RNText>
  );
}

const $sizeStyles = {
  xxl: { fontSize: 36, lineHeight: 44 } satisfies TextStyle,
  xl: { fontSize: 24, lineHeight: 34 } satisfies TextStyle,
  lg: { fontSize: 20, lineHeight: 32 } satisfies TextStyle,
  md: { fontSize: 18, lineHeight: 26 } satisfies TextStyle,
  sm: { fontSize: 16, lineHeight: 24 } satisfies TextStyle,
  xs: { fontSize: 14, lineHeight: 21 } satisfies TextStyle,
  xxs: { fontSize: 12, lineHeight: 18 } satisfies TextStyle,
};

const $fontWeightStyles = Object.entries(typography.primary).reduce(
  (acc, [weight, fontFamily]) => {
    return { ...acc, [weight]: { fontFamily } };
  },
  {}
) as Record<Weights, TextStyle>;

+const $baseStyle: ThemedStyle<TextStyle> = (theme) => ({
  ...$sizeStyles.sm,
  ...$fontWeightStyles.normal,
  color: theme.colors.text,
+});

+const $presets: Record<Presets, ThemedStyleArray<TextStyle>> = {
  default: [$baseStyle],
  bold: [$baseStyle, { ...$fontWeightStyles.bold }],
  heading: [
    $baseStyle,
    {
      ...$sizeStyles.xxl,
      ...$fontWeightStyles.bold,
    },
  ],
  subheading: [$baseStyle, { ...$sizeStyles.lg, ...$fontWeightStyles.medium }],
  formLabel: [$baseStyle, { ...$fontWeightStyles.medium }],
  formHelper: [$baseStyle, { ...$sizeStyles.sm, ...$fontWeightStyles.normal }],
+};
const $rtlStyle: TextStyle = isRTL ? { writingDirection: "rtl" } : {};
```

</details>

<details>
  <summary>src/components/Toggle.tsx</summary>

```diff
import React, { ComponentType, FC, useMemo } from "react";
import {
  Animated,
  GestureResponderEvent,
  Image,
  ImageStyle,
  Platform,
  StyleProp,
  SwitchProps,
  TextInputProps,
  TextStyle,
  TouchableOpacity,
  TouchableOpacityProps,
  View,
  ViewProps,
  ViewStyle,
} from "react-native";

+import { ThemedStyle, colors } from "../theme";
import { iconRegistry, IconTypes } from "./Icon";
import { Text, TextProps } from "./Text";
import { isRTL } from "src/i18n";
+import { useAppTheme } from "src/utils/useAppTheme";

type Variants = "checkbox" | "switch" | "radio";

interface BaseToggleProps extends Omit<TouchableOpacityProps, "style"> {
  /**
   * The variant of the toggle.
   * Options: "checkbox", "switch", "radio"
   * Default: "checkbox"
   */
  variant?: unknown;
  /**
   * A style modifier for different input states.
   */
  status?: "error" | "disabled";
  /**
   * If false, input is not editable. The default value is true.
   */
  editable?: TextInputProps["editable"];
  /**
   * The value of the field. If true the component will be turned on.
   */
  value?: boolean;
  /**
   * Invoked with the new value when the value changes.
   */
  onValueChange?: SwitchProps["onValueChange"];
  /**
   * Style overrides for the container
   */
  containerStyle?: StyleProp<ViewStyle>;
  /**
   * Style overrides for the input wrapper
   */
  inputWrapperStyle?: StyleProp<ViewStyle>;
  /**
   * Optional input wrapper style override.
   * This gives the inputs their size, shape, "off" background-color, and outer border.
   */
  inputOuterStyle?: ViewStyle;
  /**
   * Optional input style override.
   * This gives the inputs their inner characteristics and "on" background-color.
   */
  inputInnerStyle?: ViewStyle;
  /**
   * The position of the label relative to the action component.
   * Default: right
   */
  labelPosition?: "left" | "right";
  /**
   * The label text to display if not using `labelTx`.
   */
  label?: TextProps["text"];
  /**
   * Label text which is looked up via i18n.
   */
  labelTx?: TextProps["tx"];
  /**
   * Optional label options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  labelTxOptions?: TextProps["txOptions"];
  /**
   * Style overrides for label text.
   */
  labelStyle?: StyleProp<TextStyle>;
  /**
   * Pass any additional props directly to the label Text component.
   */
  LabelTextProps?: TextProps;
  /**
   * The helper text to display if not using `helperTx`.
   */
  helper?: TextProps["text"];
  /**
   * Helper text which is looked up via i18n.
   */
  helperTx?: TextProps["tx"];
  /**
   * Optional helper options to pass to i18n. Useful for interpolation
   * as well as explicitly setting locale or translation fallbacks.
   */
  helperTxOptions?: TextProps["txOptions"];
  /**
   * Pass any additional props directly to the helper Text component.
   */
  HelperTextProps?: TextProps;
}

interface CheckboxToggleProps extends BaseToggleProps {
  variant?: "checkbox";
  /**
   * Optional style prop that affects the Image component.
   */
  inputDetailStyle?: ImageStyle;
  /**
   * Checkbox-only prop that changes the icon used for the "on" state.
   */
  checkboxIcon?: IconTypes;
}

interface RadioToggleProps extends BaseToggleProps {
  variant?: "radio";
  /**
   * Optional style prop that affects the dot View.
   */
  inputDetailStyle?: ViewStyle;
}

interface SwitchToggleProps extends BaseToggleProps {
  variant?: "switch";
  /**
   * Switch-only prop that adds a text/icon label for on/off states.
   */
  switchAccessibilityMode?: "text" | "icon";
  /**
   * Optional style prop that affects the knob View.
   * Note: `width` and `height` rules should be points (numbers), not percentages.
   */
  inputDetailStyle?: Omit<ViewStyle, "width" | "height"> & {
    width?: number;
    height?: number;
  };
}

export type ToggleProps =
  | CheckboxToggleProps
  | RadioToggleProps
  | SwitchToggleProps;

interface ToggleInputProps {
  on: boolean;
  status: BaseToggleProps["status"];
  disabled: boolean;
  outerStyle: ViewStyle;
  innerStyle: ViewStyle;
  detailStyle: Omit<ViewStyle & ImageStyle, "overflow">;
  switchAccessibilityMode?: SwitchToggleProps["switchAccessibilityMode"];
  checkboxIcon?: CheckboxToggleProps["checkboxIcon"];
}

/**
 * Renders a boolean input.
 * This is a controlled component that requires an onValueChange callback that updates the value prop in order for the component to reflect user actions. If the value prop is not updated, the component will continue to render the supplied value prop instead of the expected result of any user actions.
 * @see [Documentation and Examples]{@link https://docs.infinite.red/ignite-cli/boilerplate/components/Toggle/}
 * @param {ToggleProps} props - The props for the `Toggle` component.
 * @returns {JSX.Element} The rendered `Toggle` component.
 */
export function Toggle(props: ToggleProps) {
  const {
    variant = "checkbox",
    editable = true,
    status,
    value,
    onPress,
    onValueChange,
    labelPosition = "right",
    helper,
    helperTx,
    helperTxOptions,
    HelperTextProps,
    containerStyle: $containerStyleOverride,
    inputWrapperStyle: $inputWrapperStyleOverride,
    ...WrapperProps
  } = props;

  const { switchAccessibilityMode } = props as SwitchToggleProps;
  const { checkboxIcon } = props as CheckboxToggleProps;

+  const {
+    theme: { colors },
+    themed,
+  } = useAppTheme();
  const disabled =
    editable === false || status === "disabled" || props.disabled;

  const Wrapper = useMemo(
    () =>
      (disabled ? View : TouchableOpacity) as ComponentType<
        TouchableOpacityProps | ViewProps
      >,
    [disabled]
  );
  const ToggleInput = useMemo(
    () => ToggleInputs[variant] || (() => null),
    [variant]
  );

  const $containerStyles = [$containerStyleOverride];
  const $inputWrapperStyles = [$inputWrapper, $inputWrapperStyleOverride];
+  const $helperStyles = themed([
    $helper,
    status === "error" && { color: colors.error },
    HelperTextProps?.style,
+  ]);

  /**
   * @param {GestureResponderEvent} e - The event object.
   */
  function handlePress(e: GestureResponderEvent) {
    if (disabled) return;
    onValueChange?.(!value);
    onPress?.(e);
  }

  return (
    <Wrapper
      activeOpacity={1}
      accessibilityRole={variant}
      accessibilityState={{ checked: value, disabled }}
      {...WrapperProps}
      style={$containerStyles}
      onPress={handlePress}
    >
      <View style={$inputWrapperStyles}>
        {labelPosition === "left" && (
          <FieldLabel {...props} labelPosition={labelPosition} />
        )}

        <ToggleInput
          on={!!value}
          disabled={!!disabled}
          status={status}
          outerStyle={props.inputOuterStyle ?? {}}
          innerStyle={props.inputInnerStyle ?? {}}
          detailStyle={props.inputDetailStyle ?? {}}
          switchAccessibilityMode={switchAccessibilityMode}
          checkboxIcon={checkboxIcon}
        />

        {labelPosition === "right" && (
          <FieldLabel {...props} labelPosition={labelPosition} />
        )}
      </View>

      {!!(helper || helperTx) && (
        <Text
          preset="formHelper"
          text={helper}
          tx={helperTx}
          txOptions={helperTxOptions}
          {...HelperTextProps}
          style={$helperStyles}
        />
      )}
    </Wrapper>
  );
}

const ToggleInputs: Record<Variants, FC<ToggleInputProps>> = {
  checkbox: Checkbox,
  switch: Switch,
  radio: Radio,
};

/**
 * @param {ToggleInputProps} props - The props for the `Checkbox` component.
 * @returns {JSX.Element} The rendered `Checkbox` component.
 */
function Checkbox(props: ToggleInputProps) {
  const {
    on,
    status,
    disabled,
    checkboxIcon,
    outerStyle: $outerStyleOverride,
    innerStyle: $innerStyleOverride,
    detailStyle: $detailStyleOverride,
  } = props;

  const [opacity] = React.useState(new Animated.Value(0));

  React.useEffect(() => {
    Animated.timing(opacity, {
      toValue: on ? 1 : 0,
      duration: 300,
      useNativeDriver: true,
    }).start();
  }, [on]);

  const offBackgroundColor = [
    disabled && colors.palette.neutral400,
    status === "error" && colors.errorBackground,
    colors.palette.neutral200,
  ].filter(Boolean)[0];

  const outerBorderColor = [
    disabled && colors.palette.neutral400,
    status === "error" && colors.error,
    !on && colors.palette.neutral800,
    colors.palette.secondary500,
  ].filter(Boolean)[0];

  const onBackgroundColor = [
    disabled && colors.transparent,
    status === "error" && colors.errorBackground,
    colors.palette.secondary500,
  ].filter(Boolean)[0];

  const iconTintColor = [
    disabled && colors.palette.neutral600,
    status === "error" && colors.error,
    colors.palette.accent100,
  ].filter(Boolean)[0];

  return (
    <View
      style={[
        $inputOuterVariants.checkbox,
        { backgroundColor: offBackgroundColor, borderColor: outerBorderColor },
        $outerStyleOverride,
      ]}
    >
      <Animated.View
        style={[
          $checkboxInner,
          { backgroundColor: onBackgroundColor },
          $innerStyleOverride,
          { opacity },
        ]}
      >
        <Image
          source={
            checkboxIcon ? iconRegistry[checkboxIcon] : iconRegistry.check
          }
          style={[
            $checkboxDetail,
            !!iconTintColor && { tintColor: iconTintColor },
            $detailStyleOverride,
          ]}
        />
      </Animated.View>
    </View>
  );
}

/**
 * @param {ToggleInputProps} props - The props for the `Radio` component.
 * @returns {JSX.Element} The rendered `Radio` component.
 */
function Radio(props: ToggleInputProps) {
  const {
    on,
    status,
    disabled,
    outerStyle: $outerStyleOverride,
    innerStyle: $innerStyleOverride,
    detailStyle: $detailStyleOverride,
  } = props;

  const [opacity] = React.useState(new Animated.Value(0));

  React.useEffect(() => {
    Animated.timing(opacity, {
      toValue: on ? 1 : 0,
      duration: 300,
      useNativeDriver: true,
    }).start();
  }, [on]);

  const offBackgroundColor = [
    disabled && colors.palette.neutral400,
    status === "error" && colors.errorBackground,
    colors.palette.neutral200,
  ].filter(Boolean)[0];

  const outerBorderColor = [
    disabled && colors.palette.neutral400,
    status === "error" && colors.error,
    !on && colors.palette.neutral800,
    colors.palette.secondary500,
  ].filter(Boolean)[0];

  const onBackgroundColor = [
    disabled && colors.transparent,
    status === "error" && colors.errorBackground,
    colors.palette.neutral100,
  ].filter(Boolean)[0];

  const dotBackgroundColor = [
    disabled && colors.palette.neutral600,
    status === "error" && colors.error,
    colors.palette.secondary500,
  ].filter(Boolean)[0];

  return (
    <View
      style={[
        $inputOuterVariants.radio,
        { backgroundColor: offBackgroundColor, borderColor: outerBorderColor },
        $outerStyleOverride,
      ]}
    >
      <Animated.View
        style={[
          $radioInner,
          { backgroundColor: onBackgroundColor },
          $innerStyleOverride,
          { opacity },
        ]}
      >
        <View
          style={[
            $radioDetail,
            { backgroundColor: dotBackgroundColor },
            $detailStyleOverride,
          ]}
        />
      </Animated.View>
    </View>
  );
}

/**
 * @param {ToggleInputProps} props - The props for the `Switch` component.
 * @returns {JSX.Element} The rendered `Switch` component.
 */
function Switch(props: ToggleInputProps) {
  const {
    on,
    status,
    disabled,
    outerStyle: $outerStyleOverride,
    innerStyle: $innerStyleOverride,
    detailStyle: $detailStyleOverride,
  } = props;

  const {
    theme: { colors },
    themed,
  } = useAppTheme();
  const animate = React.useRef(new Animated.Value(on ? 1 : 0)).current; // Initial value is set based on isActive
  const [opacity] = React.useState(new Animated.Value(0));

  React.useEffect(() => {
    Animated.timing(animate, {
      toValue: on ? 1 : 0,
      duration: 300,
      useNativeDriver: true, // Enable native driver for smoother animations
    }).start();
  }, [on]);

  React.useEffect(() => {
    Animated.timing(opacity, {
      toValue: on ? 1 : 0,
      duration: 300,
      useNativeDriver: true,
    }).start();
  }, [on]);

  const knobSizeFallback = 2;

  const knobWidth = [
    $detailStyleOverride?.width,
    $switchDetail?.width,
    knobSizeFallback,
  ].find((v) => typeof v === "number");

  const knobHeight = [
    $detailStyleOverride?.height,
    $switchDetail?.height,
    knobSizeFallback,
  ].find((v) => typeof v === "number");

  const offBackgroundColor = [
    disabled && colors.palette.neutral400,
    status === "error" && colors.errorBackground,
    colors.palette.neutral300,
  ].filter(Boolean)[0];

  const onBackgroundColor = [
    disabled && colors.transparent,
    status === "error" && colors.errorBackground,
    colors.palette.secondary500,
  ].filter(Boolean)[0];

  const knobBackgroundColor = (function () {
    if (on) {
      return [
        $detailStyleOverride?.backgroundColor,
        status === "error" && colors.error,
        disabled && colors.palette.neutral600,
        colors.palette.neutral100,
      ].filter(Boolean)[0];
    } else {
      return [
        $innerStyleOverride?.backgroundColor,
        disabled && colors.palette.neutral600,
        status === "error" && colors.error,
        colors.palette.neutral200,
      ].filter(Boolean)[0];
    }
  })();

  const rtlAdjustment = isRTL ? -1 : 1;
  const $themedSwitchInner = React.useMemo(
    () => themed($switchInner),
    [themed]
  );

  const offsetLeft = ($innerStyleOverride?.paddingStart ||
    $innerStyleOverride?.paddingLeft ||
    $themedSwitchInner?.paddingStart ||
    $themedSwitchInner?.paddingLeft ||
    0) as number;

  const offsetRight = ($innerStyleOverride?.paddingEnd ||
    $innerStyleOverride?.paddingRight ||
    $themedSwitchInner?.paddingEnd ||
    $themedSwitchInner?.paddingRight ||
    0) as number;

  const outputRange =
    Platform.OS === "web"
      ? isRTL
        ? [+(knobWidth || 0) + offsetRight, offsetLeft]
        : [offsetLeft, +(knobWidth || 0) + offsetRight]
      : [
          rtlAdjustment * offsetLeft,
          rtlAdjustment * (+(knobWidth || 0) + offsetRight),
        ];

  const $animatedSwitchKnob = animate.interpolate({
    inputRange: [0, 1],
    outputRange,
  });

  return (
    <View
      style={[
        $inputOuter,
        { backgroundColor: offBackgroundColor },
        $outerStyleOverride,
      ]}
    >
      <Animated.View
        style={[
          $themedSwitchInner,
          { backgroundColor: onBackgroundColor },
          $innerStyleOverride,
          { opacity },
        ]}
      />

      <SwitchAccessibilityLabel {...props} role="on" />
      <SwitchAccessibilityLabel {...props} role="off" />

      <Animated.View
        style={[
          $switchDetail,
          $detailStyleOverride,
          { transform: [{ translateX: $animatedSwitchKnob }] },
          { width: knobWidth, height: knobHeight },
          { backgroundColor: knobBackgroundColor },
        ]}
      />
    </View>
  );
}

/**
 * @param {ToggleInputProps & { role: "on" | "off" }} props - The props for the `SwitchAccessibilityLabel` component.
 * @returns {JSX.Element} The rendered `SwitchAccessibilityLabel` component.
 */
function SwitchAccessibilityLabel(
  props: ToggleInputProps & { role: "on" | "off" }
) {
  const {
    on,
    disabled,
    status,
    switchAccessibilityMode,
    role,
    innerStyle,
    detailStyle,
  } = props;

  if (!switchAccessibilityMode) return null;

  const shouldLabelBeVisible = (on && role === "on") || (!on && role === "off");

  const $switchAccessibilityStyle: StyleProp<ViewStyle> = [
    $switchAccessibility,
    role === "off" && { end: "5%" },
    role === "on" && { left: "5%" },
  ];

  const color = (function () {
    if (disabled) return colors.palette.neutral600;
    if (status === "error") return colors.error;
    if (!on) return innerStyle?.backgroundColor || colors.palette.secondary500;
    return detailStyle?.backgroundColor || colors.palette.neutral100;
  })();

  return (
    <View style={$switchAccessibilityStyle}>
      {switchAccessibilityMode === "text" && shouldLabelBeVisible && (
        <View
          style={[
            role === "on" && $switchAccessibilityLine,
            role === "on" && { backgroundColor: color },
            role === "off" && $switchAccessibilityCircle,
            role === "off" && { borderColor: color },
          ]}
        />
      )}

      {switchAccessibilityMode === "icon" && shouldLabelBeVisible && (
        <Image
          style={[$switchAccessibilityIcon, { tintColor: color }]}
          source={role === "off" ? iconRegistry.hidden : iconRegistry.view}
        />
      )}
    </View>
  );
}

/**
 * @param {BaseToggleProps} props - The props for the `FieldLabel` component.
 * @returns {JSX.Element} The rendered `FieldLabel` component.
 */
function FieldLabel(props: BaseToggleProps) {
  const {
    status,
    label,
    labelTx,
    labelTxOptions,
    LabelTextProps,
    labelPosition,
    labelStyle: $labelStyleOverride,
  } = props;
  const {
    theme: { colors },
    themed,
  } = useAppTheme();

  if (!label && !labelTx && !LabelTextProps?.children) return null;

  const $labelStyle = themed([
    $label,
    status === "error" && { color: colors.error },
    labelPosition === "right" && $labelRight,
    labelPosition === "left" && $labelLeft,
    $labelStyleOverride,
    LabelTextProps?.style,
  ]);

  return (
    <Text
      preset="formLabel"
      text={label}
      tx={labelTx}
      txOptions={labelTxOptions}
      {...LabelTextProps}
      style={$labelStyle}
    />
  );
}

const $inputWrapper: ViewStyle = {
  flexDirection: "row",
  alignItems: "center",
};

const $inputOuterBase: ViewStyle = {
  height: 24,
  width: 24,
  borderWidth: 2,
  alignItems: "center",
  overflow: "hidden",
  flexGrow: 0,
  flexShrink: 0,
  justifyContent: "space-between",
  flexDirection: "row",
};

const $inputOuterVariants: Record<Variants, StyleProp<ViewStyle>> = {
  checkbox: [$inputOuterBase, { borderRadius: 4 }],
  radio: [$inputOuterBase, { borderRadius: 12 }],
  switch: [
    $inputOuterBase,
    { height: 32, width: 56, borderRadius: 16, borderWidth: 0 },
  ],
};

const $checkboxInner: ViewStyle = {
  width: "100%",
  height: "100%",
  alignItems: "center",
  justifyContent: "center",
  overflow: "hidden",
};

const $checkboxDetail: ImageStyle = {
  width: 20,
  height: 20,
  resizeMode: "contain",
};

const $radioInner: ViewStyle = {
  width: "100%",
  height: "100%",
  alignItems: "center",
  justifyContent: "center",
  overflow: "hidden",
};

const $radioDetail: ViewStyle = {
  width: 12,
  height: 12,
  borderRadius: 6,
};

const $inputOuter: StyleProp<ViewStyle> = [
  $inputOuterBase,
  { height: 32, width: 56, borderRadius: 16, borderWidth: 0 },
];

+const $switchInner: ThemedStyle<ViewStyle> = ({ colors }) => ({
  width: "100%",
  height: "100%",
  alignItems: "center",
  borderColor: colors.transparent,
  overflow: "hidden",
  position: "absolute",
  paddingStart: 4,
  paddingEnd: 4,
+});

const $switchDetail: SwitchToggleProps["inputDetailStyle"] = {
  borderRadius: 12,
  position: "absolute",
  width: 24,
  height: 24,
};

const $switchAccessibility: TextStyle = {
  width: "40%",
  justifyContent: "center",
  alignItems: "center",
};

const $switchAccessibilityIcon: ImageStyle = {
  width: 14,
  height: 14,
  resizeMode: "contain",
};

const $switchAccessibilityLine: ViewStyle = {
  width: 2,
  height: 12,
};

const $switchAccessibilityCircle: ViewStyle = {
  borderWidth: 2,
  width: 12,
  height: 12,
  borderRadius: 6,
};

+const $helper: ThemedStyle<TextStyle> = ({ spacing }) => ({
  marginTop: spacing.xs,
+});

const $label: TextStyle = {
  flex: 1,
};

+const $labelRight: ThemedStyle<TextStyle> = ({ spacing }) => ({
  marginStart: spacing.md,
+});

+const $labelLeft: ThemedStyle<TextStyle> = ({ spacing }) => ({
  marginEnd: spacing.md,
+});
```

</details>
<br />

🏃**Try it.** Toggle the theme back and forth again while checking out the Podcasts route (honestly, any part of the app since we're working on our component library). You'll see we're not quite there yet, but making good progress!

#### Finally, update the Podcasts route

Ok, the final push to getting our Podcasts route styled for dark theme! Still hanging in? 😅

Now that we've set up the context provider and restyled the necessary component primitives we utilize on our screen, we have one task left to accomplish our goal. Within the Podcasts screen itself, we have to also update any styles with colors and spacing from the active theme and apply any `themed` helpers where necessary.

More of the same, just in a different spot - you got this! If you get stuck along the way, the final solution can be found in `files/01/app/(app)/(tabs)/podcasts/index.tsx`.

Update the Podcasts screen for dynamic theming in `src/app/(app)/(tabs)/podcasts/index.tsx`:

1. First, tackle the `DemoPodcastListScreen` component

```diff
import { isRTL, translate } from "src/i18n"
import { useStores } from "src/models"
import { Episode } from "src/models/Episode"
+import type { ThemedStyle } from "src/theme"
import { delay } from "src/utils/delay"
+import { useAppTheme } from "src/utils/useAppTheme"
import { Link } from "expo-router"

// ...

export default observer(function DemoPodcastListScreen(_props) {
  const { episodeStore } = useStores()
+  const { themed } = useAppTheme()

  const [refreshing, setRefreshing] = React.useState(false)
  const [isLoading, setIsLoading] = React.useState(false)

  // initially, kick off a background refresh without the refreshing UI
  useEffect(() => {
    ;(async function load() {
      setIsLoading(true)
      await episodeStore.fetchEpisodes()
      setIsLoading(false)
    })()
  }, [episodeStore])

  // simulate a longer refresh, if the refresh is too fast for UX
  async function manualRefresh() {
    setRefreshing(true)
    await Promise.all([episodeStore.fetchEpisodes(), delay(750)])
    setRefreshing(false)
  }

  return (
    <Screen preset="fixed" safeAreaEdges={["top"]} contentContainerStyle={$screenContentContainer}>
      <ListView<Episode>
+        contentContainerStyle={themed($listContentContainer)}
        data={episodeStore.episodesForList.slice()}
        extraData={episodeStore.favorites.length + episodeStore.episodes.length}
        refreshing={refreshing}
        estimatedItemSize={177}
        onRefresh={manualRefresh}
        ListEmptyComponent={
          isLoading ? (
            <ActivityIndicator />
          ) : (
            <EmptyState
              preset="generic"
+              style={themed($emptyState)}
              headingTx={
                episodeStore.favoritesOnly
                  ? "demoPodcastListScreen.noFavoritesEmptyState.heading"
                  : undefined
              }
              contentTx={
                episodeStore.favoritesOnly
                  ? "demoPodcastListScreen.noFavoritesEmptyState.content"
                  : undefined
              }
              button={episodeStore.favoritesOnly ? "" : undefined}
              buttonOnPress={manualRefresh}
              imageStyle={$emptyStateImage}
              ImageProps={{ resizeMode: "contain" }}
            />
          )
        }
        ListHeaderComponent={
+          <View style={themed($heading)}>
            <Text preset="heading" tx="demoPodcastListScreen.title" />
            {(episodeStore.favoritesOnly || episodeStore.episodesForList.length > 0) && (
+              <View style={themed($toggle)}>
                <Toggle
                  value={episodeStore.favoritesOnly}
                  onValueChange={() =>
                    episodeStore.setProp("favoritesOnly", !episodeStore.favoritesOnly)
                  }
                  variant="switch"
                  labelTx="demoPodcastListScreen.onlyFavorites"
                  labelPosition="left"
                  labelStyle={$labelStyle}
                  accessibilityLabel={translate("demoPodcastListScreen.accessibility.switch")}
                />
              </View>
            )}
          </View>
        }
        renderItem={({ item }) => (
          <EpisodeCard
            episode={item}
            isFavorite={episodeStore.hasFavorite(item)}
            onPressFavorite={() => episodeStore.toggleFavorite(item)}
          />
        )}
      />
    </Screen>
  )
})
```

2. Now for the `EpisodeCard` child component (same file)

```diff
const EpisodeCard = observer(function EpisodeCard({
  episode,
  isFavorite,
  onPressFavorite,
}: {
  episode: Episode
  onPressFavorite: () => void
  isFavorite: boolean
}) {
+  const {
+    theme: { colors },
+    themed,
+  } = useAppTheme()

  const liked = useSharedValue(isFavorite ? 1 : 0)

  const imageUri = useMemo<ImageSourcePropType>(() => {
    return rnrImages[Math.floor(Math.random() * rnrImages.length)]
  }, [])

  // ... some animation and accessibility code ...

  const handlePressFavorite = () => {
    onPressFavorite()
    liked.value = withSpring(liked.value ? 0 : 1)
  }

  const ButtonLeftAccessory: ComponentType<ButtonAccessoryProps> = useMemo(
    () =>
      function ButtonLeftAccessory() {
        return (
          <View>
            <Animated.View
+              style={[themed($iconContainer), StyleSheet.absoluteFill, animatedLikeButtonStyles]}
            >
              <Icon
                icon="heart"
                size={ICON_SIZE}
                color={colors.palette.neutral800} // dark grey
              />
            </Animated.View>
+            <Animated.View style={[themed($iconContainer), animatedUnlikeButtonStyles]}>
              <Icon
                icon="heart"
                size={ICON_SIZE}
                color={colors.palette.primary400} // pink
              />
            </Animated.View>
          </View>
        )
      },
+   [themed],
  )

  return (
    <Link href={`/podcasts/${episode.guid}`} asChild>
      <Card
+        style={themed($item)}
        verticalAlignment="force-footer-bottom"
        onLongPress={handlePressFavorite}
        HeadingComponent={
+          <View style={themed($metadata)}>
            <Text
+             style={themed($metadataText)}
              size="xxs"
              accessibilityLabel={episode.datePublished.accessibilityLabel}
            >
              {episode.datePublished.textLabel}
            </Text>
            <Text
+             style={themed($metadataText)}
              size="xxs"
              accessibilityLabel={episode.duration.accessibilityLabel}
            >
              {episode.duration.textLabel}
            </Text>
          </View>
        }
        content={`${episode.parsedTitleAndSubtitle.title} - ${episode.parsedTitleAndSubtitle.subtitle}`}
        {...accessibilityHintProps}
+        RightComponent={<Image source={imageUri} style={themed($itemThumbnail)} />}
        FooterComponent={
          <Button
            onPress={handlePressFavorite}
            onLongPress={handlePressFavorite}
+           style={themed([$favoriteButton, isFavorite && $unFavoriteButton])}
            accessibilityLabel={
              isFavorite
                ? translate("demoPodcastListScreen.accessibility.unfavoriteIcon")
                : translate("demoPodcastListScreen.accessibility.favoriteIcon")
            }
            LeftAccessory={ButtonLeftAccessory}
          >
            <Text
              size="xxs"
              accessibilityLabel={episode.duration.accessibilityLabel}
              weight="medium"
              text={
                isFavorite
                  ? translate("demoPodcastListScreen.unfavoriteButton")
                  : translate("demoPodcastListScreen.favoriteButton")
              }
            />
          </Button>
        }
      />
    </Link>
  )
})
```

3. Now just touch up the styles

```diff
const $screenContentContainer: ViewStyle = {
  flex: 1,
}

+const $listContentContainer: ThemedStyle<ContentStyle> = ({ spacing }) => ({
  paddingHorizontal: spacing.lg,
  paddingTop: spacing.lg + spacing.xl,
  paddingBottom: spacing.lg,
+})

+const $heading: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  marginBottom: spacing.md,
+})

+const $item: ThemedStyle<ViewStyle> = ({ colors, spacing }) => ({
  padding: spacing.md,
  marginTop: spacing.md,
  minHeight: 120,
  backgroundColor: colors.palette.neutral100,
+})

+const $itemThumbnail: ThemedStyle<ImageStyle> = ({ spacing }) => ({
  marginTop: spacing.sm,
  borderRadius: 50,
  alignSelf: "flex-start",
+})

+const $toggle: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  marginTop: spacing.md,
+})

const $labelStyle: TextStyle = {
  textAlign: "left",
}

+const $iconContainer: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  height: ICON_SIZE,
  width: ICON_SIZE,
  flexDirection: "row",
  marginEnd: spacing.sm,
+})

+const $metadata: ThemedStyle<TextStyle> = ({ colors, spacing }) => ({
  color: colors.textDim,
  marginTop: spacing.xs,
  flexDirection: "row",
+})

+const $metadataText: ThemedStyle<TextStyle> = ({ colors, spacing }) => ({
  color: colors.textDim,
  marginEnd: spacing.md,
  marginBottom: spacing.xs,
+})

+const $favoriteButton: ThemedStyle<ViewStyle> = ({ colors, spacing }) => ({
  borderRadius: 17,
  marginTop: spacing.md,
  justifyContent: "flex-start",
  backgroundColor: colors.palette.neutral300,
  borderColor: colors.palette.neutral300,
  paddingHorizontal: spacing.md,
  paddingTop: spacing.xxxs,
  paddingBottom: 0,
  minHeight: 32,
  alignSelf: "flex-start",
+})

+const $unFavoriteButton: ThemedStyle<ViewStyle> = ({ colors }) => ({
  borderColor: colors.palette.primary100,
  backgroundColor: colors.palette.primary100,
+})

+const $emptyState: ThemedStyle<ViewStyle> = ({ spacing }) => ({
  marginTop: spacing.xxl,
+})

const $emptyStateImage: ImageStyle = {
  transform: [{ scaleX: isRTL ? -1 : 1 }],
}
```

You'll notice a bunch of the styles became functions of the theme. Many of them are just spacing, which might not be too noticeable. The reason for that is due to the contents of `src/theme/spacingDark.ts`. The values here are the same as the light theme.

🏃**Try it.** Open up `src/theme/spacingDark.ts` and play with some of the spacing values and toggle your dark theme to see your views adjust!

Congrats! ✨ You've made it (for this screen, with this set of components, at least - hah!). You're now fully capable on telling your boss why the task _"just add dark mode"_ isn't a half day task.

## Side Quests

- [Bzzzt! Adding haptics](./companions/01/expo-haptics.md)
- Update the rest of the `src/components` for theming
- Support dark mode for any of the other screens
- Create a third theme with your own color palette

## See the solution

Switch to branch: [`01-blending-in-solution`](https://github.com/infinitered/cr-2024-intermediate-workshop-template/tree/01-blending-in-solution)
