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

- Add any helpful links here

# Exercises

## Exercise 0: Drop in some dark mode compatible assets

TODO: should we put this in the template ahead of time? Use `files/01` dir

## Exercise 1: Font scaling that looks good!

### Device settings prep

Turn on larger text sizes
iOS Simulator:

1. Settings app
2. Display & Text Size
3. Larger Text
4. Drag slider all the way to the larger letter `A`

iOS device:

1. Settings app
2. Accessibility
3. Display & Text Size
4. Larger Text
5. Drag slider all the way to the larger letter `A`

Android emulator:

1. TODO

Android device:

1. TODO

### Observe the Sign In screen

TODO flesh out text: Notice how we now have to scroll for the button, etc. We know that the heading and subheading are already large from our style implementation. We can choose to not scale the font on these text components so that the screen has a better UX.

`src/app/log-in.tsx`

```
<Text testID="login-heading" tx="loginScreen.signIn" preset="heading" style={$signIn} />
<Text tx="loginScreen.enterDetails" preset="subheading" style={$enterDetails} />
```

With these changes, the input labels, inputs and button text are all larger - aiding the user in reading the text. The heading and subheading are still large by design and now everything fits on one screen, so they'll know to click the sign in button.

## Exercise 2: Add haptics for tactile feedback around the app

`src/app/(app)/(tabs)/profile.tsx`

```diff
+ import * as Haptics from 'expo-haptics'

// ...

// find <Slider />
onValueChange={(value) => {
+ Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)
  setProp("rnFamiliarity", value)
}}
```

`src/app/log-in.tsx`

```diff

// inside function login() {
if (validationError) {
+  Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)
  return
}
```

## Exercise 3: Start building our themeable component library

### Color Palette

New file `src/theme/colorsDark.ts` (or come up with your own color scheme!)

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

#### Theme Types and Helpers

Update `src/theme/index.ts`

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

Note the export here for `colorsLight` and `spacingLight`

### `useAppTheme`

#### ThemeContext

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

#### The Hook

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

#### Applying the Theme

```diff
import { useStores } from "src/models"
import { useFonts } from "expo-font"
- import { colors, customFontsToLoad } from "src/theme"
+ import { customFontsToLoad } from "src/theme"
+import { useAppTheme, useThemeProvider } from "src/utils/useAppTheme"

export default observer(function Layout() {
  const {
    authenticationStore: { isAuthenticated },
  } = useStores()
+  const {
+    theme: { colors },
+  } = useAppTheme()
+  const { themeScheme, setThemeContextOverride, ThemeProvider } = useThemeProvider()

  const [fontsLoaded, fontError] = useFonts(customFontsToLoad)

  React.useEffect(() => {
    if (fontsLoaded || fontError) {
      // Hide the splash screen after the fonts have loaded and the UI is ready.
      SplashScreen.hideAsync()
    }
  }, [fontsLoaded, fontError])

  if (!fontsLoaded && !fontError) {
    return null
  }

  if (!isAuthenticated) {
    return <Redirect href="/log-in" />
  }

+ return (
+  <ThemeProvider value={{ themeScheme, setThemeContextOverride }}>
      <Stack screenOptions={{ headerShown: false }} />
+  </ThemeProvider>
  )
})
```

## Exercise 3: Bzzzt! Adding haptics

## Side Quests

- Add any extra stuff to do here

## See the solution

Switch to branch: `01-blending-in-solution`
