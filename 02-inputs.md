# Module 02: The joy of inputting: complex controls and keyboard interactions

### Goal

We have a basic scrollable input form that technically works, but doesn't feel great to use on the small screen. Let's build some controls that make it much easier to accept complex inputs and fix keyboard handling so you can smoothly jump between fields.

### Concepts

- Keyboard avoidance and changing focus from field to field without things feeling too jumpy or having inputs hidden.
- Writing a custom control that handles complex data more effectively than default options

<!--
Thoughts:
- In terms of how the data itself is stored, the form doesn't really need a submit button, but providing one could help frame future exercise... e.g., validation UX and accessibility.

Maybe we could still save/persist the data instantly in MST, but the Submit button is to simulate saving to a server, so it triggers validation?

- Now I'm wondering more if it would be better if the initial template used a picker that's not well suited to the task, like https://github.com/react-native-picker
(doesn't support multiselect, doesn't really work well with large lists).

- If we have extra time, maybe over form validation, we could do something with swapping a control? The toggle or the slider. Depends if form validation plays into the accessibility timebox. Though, the number of prospective exercises already feels like plenty.

- Wondering if it would be better to do the keyboard handling section before the bottom sheet selection, since it's something that makes the whole form feel better right away? Also gets native build stuff out of the way. (assuming we don't include all the dependencies in the project at the outset, which is actually often a good idea)
-->

### Features to build

- Keep text inputs always visible with `react-native-keyboard-controller`
  - Smoothly transition from field to field using the submit/next functionality on the keyboard, with the view scrolling to show the next text field
  - Properly handle keyboard dismissal for all scenarios, including numeric keypad on iOS
- Add a multi-select downdown for the skills field, with the list contained in `react-native-bottom-sheet`
  - Add text search to the list to make it easier to pick stuff without tons of scrolling

### Resources

- [react-native-keyboard-controller](https://github.com/kirillzyusko/react-native-keyboard-controller)
- [react-native-bottom-sheet](https://github.com/gorhom/react-native-bottom-sheet)
- [Ignite Cookbook - Select field with bottom sheet recipe](https://ignitecookbook.com/docs/recipes/SelectFieldWithBottomSheet/)

# Exercises

## Exercise 1: Basic keyboard avoidance with `react-native-keyboard-controller`

Let's start by ensuring we can always see our input fields on the profile screen, especially when the keyboard is engaged. With the app running, give the profile screen a scroll and try switching between inputs. Is the keyboard in the way? Let's fix that.

### Add Keyboard Provider

We'll start by wrapping our app with KeyboardProvider in the root layout.

Do the following in **src/app/\_layout.tsx**:

1. Import `KeyboardProvider` from `react-native-keyboard-controller`

```tsx
import { KeyboardProvider } from "react-native-keyboard-controller";
```

2. Wrap the provider around the `Slot` component

```diff
return (
+  <KeyboardProvider>
    <Slot />
+  </KeyboardProvider>
);
```

Now let's head to our Screen component and update the existing scrolling screen to use a KeyboardAvoidingScrollview instead of a basic ScrollView.

### Update our Screen component

In **src/components/Screen.tsx**

Notice that we're already using a KeyboardAvoidingView in the main screen component.

```tsx
export function Screen(props: ScreenProps) {
  const {
    backgroundColor = colors.background,
    KeyboardAvoidingViewProps,
    keyboardOffset = 0,
    safeAreaEdges,
    StatusBarProps,
    statusBarStyle = "dark",
  } = props;

  const $containerInsets = useSafeAreaInsetsStyle(safeAreaEdges);

  return (
    <View style={[$containerStyle, { backgroundColor }, $containerInsets]}>
      <StatusBar style={statusBarStyle} {...StatusBarProps} />

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
```

> This works great for some views, keeping floating buttons at the bottom of a view when the keyboard is open for example, but has trouble managing on views with scrolling so we need another solution.

<details><summary>Bad Keyboard Avoidance Demo</summary>
<video width=300 src=./files/02/KeyboardAvoidingViewFail.mp4 />
</details>

<br/>

To fix this, let's head into the `ScreenWithScrolling` component (in the same file, just up above) and replace the existing `Scrollview` with a `KeyboardAwareScrollView`. We can keep all the existing props, and see how this helps.

```diff
-    <ScrollView>
+    <KeyboardAwareScrollView>
...
-    </ScrollView>
+    </KeyboardAwareScrollView>
```

<details><summary>Somewhat Better Keyboard Avoidance</summary>
<video width=300 src=./files/02/KeyboardAwareScrollView1.mp4 />
</details>
<br/>

### Pad for extra viewing comfort

We're getting somewhere now, but lets add some padding to offset the bottom of the textfields just a little so we don't have any cutoff.

1. Add an optional prop to `ScrollScreenProps` called `bottomOffset` of type number
2. Destructure that prop in our `ScreenWithScrolling` component, preferrably with a default of 0
3. Pass `bottomOffset` in to the `KeyboardAwareScrollView`

There we go! The keyboard isn't blocking any of our fields.

### Fix double padding issue on scroll

Uh oh! Try focusing on one of the text fields and scrolling to the bottom of the screen with the keyboard open. There's so much extra padding at the bottom of the screen when the keyboard is open.

This is a known issue when nesting `KeyboardAvoidingView` and `KeyboardAwareScrollView` (https://github.com/kirillzyusko/react-native-keyboard-controller/issues/451 )

We don't really need both when we're using the `KeyboardAwareScrollView`, but we definitely want to keep avoiding the keyboard for non-scrolling screens. Let's enable the `KeyboardAvoidingView` only if it's a fixed screen.

Reuse the `isNonScrolling` function and add an `enable` prop to the KeyboardAvoidingView.

```diff
 <KeyboardAvoidingView
        behavior={isIos ? "padding" : "height"}
        keyboardVerticalOffset={keyboardOffset}
        {...KeyboardAvoidingViewProps}
        style={[$keyboardAvoidingViewStyle, KeyboardAvoidingViewProps?.style]}
+        enabled={isNonScrolling(props.preset)}
      >
```

Now let's check it again...and no doubled up padding when the keyboard is open!

#### Android Only

If you're working on an android you might have noticed some weird behavior forcing the screen up when focusing on text inputs. The screen seems to be scrolling too far and focusing the input even though it is off screen.

To fix this, in our `Screen` component update the `KeyboarAvoidingView` behavior property to "undefined" if not iOS. This should solve it, but needs a rebuild so make sure to kill your app and re-run `yarn android` to get it behaving correctly!

```diff
 <KeyboardAvoidingView
-   behavior={isIos ? "padding" : "height"}
+   behavior={isIos ? "padding" : undefined}
```

## Exercise 2: Smooth transitions between fields from the keyboard

Not that keyboard avoiding isn't great, but it's quite a pain to have to scroll to see the other text fields. On top of that, if you scroll past, or haven't scrolled enough, a user might not even know there's an input to see!

A toolbar with inputs to navigate between textfields and close the keyboard when we're done is a great native feeling solution.

### Add toolbar to Profile Screen

Since we don't necessarily want a toolbar on every scrolling screen (ie. even the ones without inputs), we can add it directly to our Profile screen.

In _src/app/(app)/(tabs)/profile.tsx_:

1. Wrap the Screen component in a fragment
2. Import `KeyboardToolbar` from `react-native-keyboard-controller`

```tsx
import { KeyboardToolbar } from "react-native-keyboard-controller";
```

3. Add the `KeyboardToolbar` below the `Screen` component within the fragment

```tsx
<>
  <Screen>...</Screen>
  <KeyboardToolbar />
</>
```

> Verify that this works as expected, with arrows to move between fields and closing the keyboard with the done button instead of simply pressing outside an input or hitting return on the keyboard.

### Update bottomOffset

Now that we've got the toolbar added, you might notice that there is a slight overlap with the two text fields lower down on our screen. You might be thinking _"But I fixed this earlier when I added the bottom offset to the screen!"_ and you would be right.

Since we added the toolbar outside of the `Screen` component, theIn adding the toolbar, we need to also offset that height as the KeyboardAwareScrollView isn't aware of it since it's outside of the `Screen` component.

- Try inspecting the toolbar and checking the height!
- You'll find that the toolbar has a height of 42px on both iOS and android so lets add that to our existing `bottomOffset` prop on our profile screen.

```tsx
<Screen
        preset="scroll"
        contentContainerStyle={$container}
        keyboardShouldPersistTaps="handled"
        bottomOffset={spacing.md + 42} // height of the toolbar + existing padding
      >
```

And there you have it, keyboard avoidance and moving from field to field without covering inputs for a form that is much easier to use.

## Exercise 3: New dropdown component

- Build select field.
- Replace text field for skills with select field
- Add onPress to set the values
- Add observer around component so the bottom sheet has access to store updates
- update it to show number of skills selected instead of listing them with renderValue

## Exercise 4: Full text search in the dropdown

## Side Quests

- Form validation?
- Improve the slider
- ???

## See the solution

Switch to branch: `02-inputs-solution`
