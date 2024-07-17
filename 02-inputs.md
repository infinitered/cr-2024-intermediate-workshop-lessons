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
- Add a multi-select dropdown for the skills field, with the list contained in `react-native-bottom-sheet`
  - Add text search to the list to make it easier to pick stuff without tons of scrolling

### Resources

- [react-native-keyboard-controller](https://github.com/kirillzyusko/react-native-keyboard-controller)
- [react-native-bottom-sheet](https://github.com/gorhom/react-native-bottom-sheet)
- [Ignite Cookbook - Select field with bottom sheet recipe](https://ignitecookbook.com/docs/recipes/SelectFieldWithBottomSheet/)

# Exercises

## Exercise 0: Poke Around A Little

In the next 4 exercises we're going to be updating our profile form so that it's easier to use. Let's start by testing out the current behavior and try to identify what's not working. Poke around in the code a little. Try filling out the form on your simulator or device.

What is already implemented to make the form more user friendly? What could be improved?

Here are some things to get your mind going:

- Are the different types of inputs clear?
- Can we access / see the text fields when we're typing?
- How easy is it to get from input to input?
- Does the type of input suit the data it's asking for?
- What happens when I try to close the keyboard?

</br>

The existing `KeyboardAvoidingView` doesn't seem to be doing much when we go to use the text inputs. Let's get started fixing that behavior in our first exercise!

## Exercise 1: Basic keyboard avoidance with `react-native-keyboard-controller`

### Add Keyboard Provider

We'll start by wrapping our app with `KeyboardProvider` in the entry file.

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

Now let's head to our `Screen` component and update the existing scrolling screen to use a `KeyboardAvoidingScrollview` instead of a basic `ScrollView`.

### Update our `Screen` component

In **src/components/Screen.tsx**

Notice that we're already using a `KeyboardAvoidingView` in the main screen component.

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

> [!NOTE]
> This works great for some views, keeping floating buttons at the bottom of a view when the keyboard is open for example, but has trouble managing on views with scrolling so we need another solution.

To fix this, let's head into the `ScreenWithScrolling` component (in the same file, just up above) and replace the existing `Scrollview` with a `KeyboardAwareScrollView`. We can keep all the existing props, and see how this helps.

```tsx
import { KeyboardAwareScrollView } from "react-native-keyboard-controller";
```

```diff
-    <ScrollView>
+    <KeyboardAwareScrollView>
...
-    </ScrollView>
+    </KeyboardAwareScrollView>
```

🏃**Try it.** How does it look? Is it perfect? Probably not. What issues are you seeing?

### Pad for extra viewing comfort

We're getting somewhere now, but let's add some padding to offset the bottom of the text fields just a little so we don't have any cutoff.

3. Add an optional prop to `ScrollScreenProps` called `bottomOffset` of type number
4. Destructure that prop in our `ScreenWithScrolling` component, preferably with a default of 0
5. Pass `bottomOffset` in to the `KeyboardAwareScrollView`
6. In **src/app/(app)/(tabs)/profile.tsx** add a 16px `bottomOffset` prop to the `Screen` component.

```diff
<Screen
        preset="scroll"
        contentContainerStyle={$container}
        keyboardShouldPersistTaps="handled"
+        bottomOffset={spacing.md}
        >
```

🏃**Try it.** There we go! The keyboard isn't blocking any of our fields.

### Fix double padding issue on scroll

Uh oh! Try focusing on one of the text fields and scrolling to the bottom of the screen with the keyboard open. There's so much extra padding at the bottom of the screen when the keyboard is open.

This is a known issue when nesting `KeyboardAvoidingView` and `KeyboardAwareScrollView` (https://github.com/kirillzyusko/react-native-keyboard-controller/issues/451 )

We don't really need both when we're using the `KeyboardAwareScrollView`, but we definitely want to keep avoiding the keyboard for non-scrolling screens. Let's enable the `KeyboardAvoidingView` only if it's a fixed screen.

Reuse the `isNonScrolling` function and add an `enable` prop to the `KeyboardAvoidingView`.

```diff
 <KeyboardAvoidingView
        behavior={isIos ? "padding" : "height"}
        keyboardVerticalOffset={keyboardOffset}
        {...KeyboardAvoidingViewProps}
        style={[$keyboardAvoidingViewStyle, KeyboardAvoidingViewProps?.style]}
+        enabled={isNonScrolling(props.preset)}
      >
```

🏃**Try it.** Now let's check it again...and no doubled up padding when the keyboard is open!

#### Android Only

If you're working on an android you might have noticed some weird behavior forcing the screen up when focusing on text inputs. The screen seems to be scrolling too far and focusing the input even though it is off screen.

To fix this, in our `Screen` component update the `KeyboardAvoidingView` behavior property to "undefined" if not iOS. This should solve it, but needs a rebuild so make sure to kill your app and re-run `yarn android` to get it behaving correctly!

```diff
 <KeyboardAvoidingView
-   behavior={isIos ? "padding" : "height"}
+   behavior={isIos ? "padding" : undefined}
```

## Exercise 2: Smooth transitions between fields from the keyboard

Not that keyboard avoiding isn't great, but it's quite a pain to have to scroll to see the other text fields. On top of that, if you scroll past, or haven't scrolled enough, a user might not even know there's an input to see!

A toolbar with arrows to navigate between text fields and a clear button to close the keyboard when we're done is a great native feeling solution.

### Add toolbar to Profile Screen

Since we don't necessarily want a toolbar on every scrolling screen (ie. even the ones without inputs), let's add it directly to our Profile screen.

In **src/app/(app)/(tabs)/profile.tsx**:

1. Wrap the `Screen` component in a fragment
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

🏃**Try it.** Verify that this works as expected, with arrows to move between fields and closing the keyboard with the done button instead of simply pressing outside an input or hitting return on the keyboard.

### Update bottomOffset

Now that we've got the toolbar added, you might notice that there is a slight overlap with the two text fields lower down on our screen. You might be thinking _"But I fixed this earlier when I added the bottom offset to the screen!"_ and you would be right.

Since we added the `KeyboardToolbar` outside of the `Screen` component, we need to also offset that height as the `KeyboardAwareScrollView`

- Try inspecting the toolbar and checking the height!
- You'll find that the toolbar has a height of 42px on both iOS and android so let's add that to our existing `bottomOffset` prop on our profile screen.

```tsx
<Screen
        preset="scroll"
        contentContainerStyle={$container}
        keyboardShouldPersistTaps="handled"
        bottomOffset={spacing.md + 42} // height of the toolbar + existing padding
      >
```

🏃**Try it.** And there you have it, keyboard avoidance and moving from field to field without covering inputs for a form that is much easier to use.

## Exercise 3a: New dropdown component

For the next exercise we're going to switch gears and improve one of our form fields. The skills input is mediocre, especially if you type out more than a couple skills. A single text string with a comma separated list just isn't the right type or display for this data.

Instead, let's consider a select field, with a predefined list of skills that a user can select from. We don't have this component yet so let's build it and then update our stores to hold the data as we expect.

### Build Select Field

> [!TIP]
> The Ignite Cookbook by Infinite Red has a great recipe for building this component so we're going to leverage parts of that and incorporate it into our app! [Check it out here.](https://ignitecookbook.com/docs/recipes/SelectFieldWithBottomSheet)

1. Create the `SelectField.tsx` component file.

Instead of extending the `TextField` component with more props and functionality, we'll be creating a wrapper for the `TextField` component that contains additional functionality.

We'll start by creating a new file in the components directory.

```bash
touch ./src/components/SelectField.tsx
```

Let's add some preliminary code to the file. Since the `TextField` has its own touch handlers for focus, we'll want to disable that by wrapping it in a `View` with no pointer-events. The new `TouchableOpacity` will trigger our options sheet.

```tsx
import React, { forwardRef, Ref, useImperativeHandle, useRef } from "react";
import { View, TouchableOpacity } from "react-native";
import { observer } from "mobx-react-lite";
import { TextField, TextFieldProps } from "./TextField";

export interface SelectFieldProps
  extends Omit<
    TextFieldProps,
    "ref" | "onValueChange" | "onChange" | "value"
  > {}
export interface SelectFieldRef {}

export const SelectField = observer(
  forwardRef(function SelectField(
    props: SelectFieldProps,
    ref: Ref<SelectFieldRef>
  ) {
    const { ...TextFieldProps } = props;

    const disabled =
      TextFieldProps.editable === false || TextFieldProps.status === "disabled";

    useImperativeHandle(ref, () => ({}));

    return (
      <>
        <TouchableOpacity activeOpacity={1}>
          <View pointerEvents="none">
            <TextField {...TextFieldProps} />
          </View>
        </TouchableOpacity>
      </>
    );
  })
);
```

2. Export the component from **src/components/index.ts**

3. Add a SelectField to our Profile screen so we can monitor our progress as we build it.

In **src/app/(app)/(tabs)/profile.tsx**:

- Add a `SelectField` right below the existing skills `TextField`.
- Copy over the label from the `Text` component into the `labelTx` prop

4. Add New Props and Customize the TextField

Now, we can start modifying the code we added in the previous step to support multiple options as well as making the `TextField` look like a `SelectField`.

#### Add a Caret Icon Accessory

Let's add an accessory to the input to make it look like a `SelectField`.

```diff
<TextField
  {...TextFieldProps}
+ RightAccessory={(props) => (
+   <Icon icon="caretRight" containerStyle={props.style} />
+ )}
/>
```

#### Add props

- The options prop can be any structure that you want (e.g. flat array of values, object where the key is the option value and the value is the label, etc). For our `SelectField` guide, we'll be doing an array of label and value pairs.

- We will support multi-select (by default) as well as a single select.

- We will override the value prop.

- A new renderValue prop can be used to format and display a custom text value. This can be useful when the `TextField` is not multiline, but your `SelectField` is.

- Additionally, we'll add a new event callback called `onSelect` since that makes more sense for a `SelectField`. However, feel free to override `TextField`'s `onChange` if you prefer.

```tsx
export interface SelectFieldProps
  extends Omit<TextFieldProps, "ref" | "onValueChange" | "onChange"> {
  value?: string[];
  renderValue?: (value: string[]) => string;
  onSelect?: (newValue: string[]) => void;
  multiple?: boolean;
  options: { label: string; value: string }[];
}

// ...

const {
  value = [],
  renderValue,
  onSelect,
  options = [],
  multiple = true,
  ...TextFieldProps
} = props;
```

#### Add Logic to Display Selected Options

We'll add some code to display the selected options inside the `TextField`. This will attempt to use the `renderValue` formatter function and fallback to a joined string.

```tsx
const valueString =
  renderValue?.(value) ??
  value
    .map((v) => options.find((o) => o.value === v)?.label)
    .filter(Boolean)
    .join(", ");
```

5. Update the `TextField` to use this as the displayed value.

### Update Profile Screen SelectField

Now that we've added a bit to the `SelectField` (ie. our souped up `TextField`) let's add some more data!

In **src/app/(app)/(tabs)/profile.tsx**:

6. We need some data, copy the following into a constant at the top of the file.

<details><summary>Skills List</summary>

```tsx
const skillsList: {
  label: string;
  value: string;
}[] = [
  { label: "JavaScript", value: "javascript" },
  { label: "React Native", value: "react_native" },
  { label: "Redux", value: "redux" },
  { label: "TypeScript", value: "typescript" },
  { label: "API Integration", value: "api_integration" },
  { label: "RESTful Services", value: "restful_services" },
  { label: "GraphQL", value: "graphql" },
  { label: "Node.js", value: "node_js" },
  { label: "Firebase", value: "firebase" },
  { label: "AWS", value: "aws" },
  { label: "Google Cloud", value: "google_cloud" },
  { label: "CI/CD", value: "ci_cd" },
  { label: "Jest", value: "jest" },
  { label: "Mocha", value: "mocha" },
  { label: "Enzyme", value: "enzyme" },
  { label: "Unit Testing", value: "unit_testing" },
  { label: "Integration Testing", value: "integration_testing" },
  { label: "UI/UX Design", value: "ui_ux_design" },
  { label: "Agile Methodologies", value: "agile_methodologies" },
  { label: "Scrum", value: "scrum" },
  { label: "React Navigation", value: "react_navigation" },
  { label: "Expo", value: "expo" },
  { label: "Expo CLI", value: "expo_cli" },
  { label: "Expo SDK", value: "expo_sdk" },
  { label: "Styled Components", value: "styled_components" },
  { label: "Reanimated", value: "reanimated" },
  { label: "Native Base", value: "native_base" },
  { label: "React Native Paper", value: "react_native_paper" },
  { label: "React Native Elements", value: "react_native_elements" },
  { label: "React Native Vector Icons", value: "react_native_vector_icons" },
  { label: "Lottie", value: "lottie" },
  { label: "React Native Maps", value: "react_native_maps" },
  { label: "CodePush", value: "codepush" },
  { label: "Fastlane", value: "fastlane" },
  { label: "Realm", value: "realm" },
];
```

</details>
<br/>

7. Pass our new `skillsList` in for the `options` property
8. The value should be set to skills from our `Profile` model.
9. Add an `onSelect` prop that takes in `selected` and passes that through the `setProp` action for skills.
10. Remove the old skill `TextField` and `Text` label above it.

So far your select field should look like this:

```tsx
<SelectField
  options={skillsList}
  labelTx="demoProfileScreen.skills"
  onSelect={(selected) => setProp("skills", selected)}
  value={skills}
/>
```

But now our `Profile` model is out of sync with the data we are providing it. Let's go update that!

In **src/models/Profile.ts**:

11. Update the `skills` prop to be an optional array of strings

```tsx
types.optional(types.array(types.string), []);
```

## Exercise 3b: Bottom Sheet Selection component

### Add the Sheet Components

Let's go back to building our `SelectField` now that we've got it added to our Profile screen and we'll add the functionality to view and select options.

In this step, we'll be adding the `BottomSheetModal` and related components and setting up the touch-events to show/hide it.

#### Add the `BottomSheetModalProvider`

Since we will be using the `BottomSheetModal` component instead of `BottomSheet`, we will need to add a provider to your entry file.

We're going to need our keyboard avoiding behavior in a later step, so make sure to add the provider nested within the existing `KeyboardProvider`.

We also need to wrap everything in `GestureHandlerRootView` so let's do that as well.

In **src/app/\_layout.tsx**:

1. Add the `ReactNativeGestureHandler` and `KeyboardProvider` wrappers

```tsx
<GestureHandlerRootView>
  <KeyboardProvider>
    <BottomSheetModalProvider>
      <Slot />
    </BottomSheetModalProvider>
  </KeyboardProvider>
</GestureHandlerRootView>
```

#### Add the Necessary Components to `SelectField`

Now we will add the UI components that will display our options. This will be a basic example and we'll customize it a bit later.

2. Update the imports:

```diff
import React, { forwardRef, Ref, useImperativeHandle, useRef } from "react";
import { TouchableOpacity, View, ViewStyle } from "react-native";
import { observer } from "mobx-react-lite"
import { Icon } from "./Icon";
import { TextField, TextFieldProps } from "./TextField";
+import { BottomSheetBackdrop, BottomSheetFlatList,  BottomSheetFooter,  BottomSheetModal } from "@gorhom/bottom-sheet";
+import { useSafeAreaInsets } from "react-native-safe-area-context";
+import { spacing } from "../theme";
+import { Button } from "./Button";
+import { ListItem } from "./ListItem";
```

3. Update the SelectFieldRef interface:

```diff
export interface SelectFieldRef {
+  presentOptions: () => void;
+  dismissOptions: () => void;
}
```

4. Add the BottomSheetModal below our display `TextField`:

(We know this is a doozy of a diff, the full file is available to copy at the end of this section)

```diff
export const SelectField = observer(forwardRef(function SelectField(
  props: SelectFieldProps,
  ref: Ref<SelectFieldRef>
) {
  const {
    value = [],
    onSelect,
    renderValue,
    options = [],
    multiple = true,
    ...TextFieldProps
  } = props;
+  const sheet = useRef<BottomSheetModal>(null);
+  const { bottom } = useSafeAreaInsets();

  const disabled =
    TextFieldProps.editable === false || TextFieldProps.status === "disabled";

+  useImperativeHandle(ref, () => ({ presentOptions, dismissOptions }));

  const valueString =
    renderValue?.(value) ??
    value
      .map((v) => options.find((o) => o.value === v)?.label)
      .filter(Boolean)
      .join(", ");

+  function presentOptions() {
+    if (disabled) return;
+    sheet.current?.present();
+ }

+  function dismissOptions() {
+    sheet.current?.dismiss();
+  }

  return (
    <>
      <TouchableOpacity
        activeOpacity={1}
+        onPress={presentOptions}
      >
        <View pointerEvents="none">
          <TextField
            {...TextFieldProps}
            value={valueString}
            RightAccessory={(props) => (
              <Icon icon="caretRight" containerStyle={props.style} />
            )}
          />
        </View>
      </TouchableOpacity>

+     <BottomSheetModal
+       ref={sheet}
+       snapPoints={["50%"]}
+       stackBehavior="replace"
+       enableDismissOnClose
+       backdropComponent={(props) => (
+         <BottomSheetBackdrop
+           {...props}
+           appearsOnIndex={0}
+           disappearsOnIndex={-1}
+         />
+       )}
+       footerComponent={
+         !multiple
+           ? undefined
+           : (props) => (
+               <BottomSheetFooter
+                 {...props}
+                 style={$bottomSheetFooter}
+                 bottomInset={bottom}
+               >
+                 <Button
+                   text="Dismiss"
+                   preset="reversed"
+                   onPress={dismissOptions}
+                 />
+               </BottomSheetFooter>
+             )
+       }
+     >
+       <BottomSheetFlatList
+         style={{ marginBottom: bottom + (multiple ? spacing.xl * 2 : 0) }}
+         data={options}
+         keyExtractor={(o) => o.value}
+         renderItem={({ item, index }) => (
+           <ListItem
+             text={item.label}
+             topSeparator={index !== 0}
+             style={$listItem}
+           />
+         )}
+         keyboardShouldPersistTaps="always"
+       />
+     </BottomSheetModal>
    </>
  );
}))
```

Add the styling:

```diff

+const $bottomSheetFooter: ViewStyle = {
+  paddingHorizontal: spacing.lg,
+  paddingBottom: spacing.xs,
+};

+const $listItem: ViewStyle = {
+  paddingHorizontal: spacing.lg,
+};
```

<details><summary>Full SelectField.tsx File</summary>

```tsx
import {
  BottomSheetBackdrop,
  BottomSheetFlatList,
  BottomSheetFooter,
  BottomSheetModal,
} from "@gorhom/bottom-sheet";
import React, { forwardRef, Ref, useImperativeHandle, useRef } from "react";
import { TouchableOpacity, View, ViewStyle } from "react-native";
import { observer } from "mobx-react-lite";
import { useSafeAreaInsets } from "react-native-safe-area-context";
import { spacing } from "../theme";
import { Button } from "./Button";
import { Icon } from "./Icon";
import { ListItem } from "./ListItem";
import { TextField, TextFieldProps } from "./TextField";

export interface SelectFieldProps
  extends Omit<TextFieldProps, "ref" | "onValueChange" | "onChange" | "value"> {
  value?: string[];
  renderValue?: (value: string[]) => string;
  onSelect?: (newValue: string[]) => void;
  multiple?: boolean;
  options: { label: string; value: string }[];
}
export interface SelectFieldRef {
  presentOptions: () => void;
  dismissOptions: () => void;
}

export const SelectField = observer(
  forwardRef(function SelectField(
    props: SelectFieldProps,
    ref: Ref<SelectFieldRef>
  ) {
    const {
      value = [],
      onSelect,
      renderValue,
      options = [],
      multiple = true,
      ...TextFieldProps
    } = props;
    const sheet = useRef<BottomSheetModal>(null);
    const { bottom } = useSafeAreaInsets();

    const disabled =
      TextFieldProps.editable === false || TextFieldProps.status === "disabled";

    useImperativeHandle(ref, () => ({ presentOptions, dismissOptions }));

    const valueString =
      renderValue?.(value) ??
      value
        .map((v) => options.find((o) => o.value === v)?.label)
        .filter(Boolean)
        .join(", ");

    function presentOptions() {
      if (disabled) return;
      sheet.current?.present();
    }

    function dismissOptions() {
      sheet.current?.dismiss();
    }

    return (
      <>
        <TouchableOpacity activeOpacity={1} onPress={presentOptions}>
          <View pointerEvents="none">
            <TextField
              {...TextFieldProps}
              value={valueString}
              RightAccessory={(props) => (
                <Icon icon="caretRight" containerStyle={props.style} />
              )}
            />
          </View>
        </TouchableOpacity>

        <BottomSheetModal
          ref={sheet}
          snapPoints={["50%"]}
          stackBehavior="replace"
          enableDismissOnClose
          backdropComponent={(props) => (
            <BottomSheetBackdrop
              {...props}
              appearsOnIndex={0}
              disappearsOnIndex={-1}
            />
          )}
          footerComponent={
            !multiple
              ? undefined
              : (props) => (
                  <BottomSheetFooter
                    {...props}
                    style={$bottomSheetFooter}
                    bottomInset={bottom}
                  >
                    <Button
                      text="Dismiss"
                      preset="reversed"
                      onPress={dismissOptions}
                    />
                  </BottomSheetFooter>
                )
          }
        >
          <BottomSheetFlatList
            style={{ marginBottom: bottom + (multiple ? spacing.xl * 2 : 0) }}
            data={options}
            keyExtractor={(o) => o.value}
            renderItem={({ item, index }) => (
              <ListItem
                text={item.label}
                topSeparator={index !== 0}
                style={$listItem}
              />
            )}
            keyboardShouldPersistTaps="always"
          />
        </BottomSheetModal>
      </>
    );
  })
);

const $bottomSheetFooter: ViewStyle = {
  paddingHorizontal: spacing.lg,
  paddingBottom: spacing.xs,
};

const $listItem: ViewStyle = {
  paddingHorizontal: spacing.lg,
};
```

</details>
<br/>

#### Add Selected State to Options and Hook Up Callback

The last step is to add the selected state to our options inside the sheet as well as hook up the callback to change the value.

5. Add a function to return an array with a specific value removed:

```tsx
function without<T>(array: T[], value: T) {
  return array.filter((v) => v !== value);
}
```

6. Use that function in a new updateValue function that we will call when a user selects one of the list items

```tsx
function updateValue(optionValue: string) {
  if (value.includes(optionValue)) {
    onSelect?.(multiple ? without(value, optionValue) : []);
  } else {
    onSelect?.(multiple ? [...value, optionValue] : [optionValue]);
    if (!multiple) dismissOptions();
  }
}
```

7. Update the `ListItem` within the `BottomSheetFlatList` to call `updateValue` on press and some UI sprinkles to show which items are selected.

```diff
+import {spacing, color} from '../theme'
```

```diff
<ListItem
text={item.label}
topSeparator={index !== 0}
style={$listItem}
+rightIcon={value.includes(item.value) ? "check" : undefined}
+rightIconColor={colors.palette.angry500}
+onPress={() => updateValue(item.value)}
/>
```

And we're done building the basic select field component with a bottom sheet modal to display the options! Let's test it out and add some finishing touches.

### Finishing Touches

At this point it looks great, but notice when you select a large number of options, the comma separated list in the `TextField` isn't ideal for reading and is information that is better consumer in the bottom sheet.

Let's update that to display the number of skills selected instead of listing them by passing in a custom `renderValue` prop

Back in **src/app/(app)/(tabs)/profile.tsx**:

8. The search field is a little close to its neighbors, so let's add the existing style we're using on the other inputs as well.

```diff
<SelectField
options={skillsList}
labelTx="demoProfileScreen.skills"
onSelect={(selected) => setProp("skills", selected)}
value={skills}
renderValue={(value) => t("demoProfileScreen.skillsSelected", { count: value.length })}
+containerStyle={$textField}
/>
```

9. Add a renderValue prop with the following function for displaying the number of items selected.

```diff
<SelectField
options={skillsList}
labelTx="demoProfileScreen.skills"
onSelect={(selected) => setProp("skills", selected)}
value={skills}
+renderValue={(value) => translate("demoProfileScreen.skillsSelected", { count: value.length })}
/>
```

10. That's a new txKey so in _srx/i18n/en.ts_ add the following in the `demoProfileScreen` dictionary.

```tsx
skillsSelected: {
      one: "{{count}} skill selected",
      other: "{{count}} skills selected",
    },
```

Looks good now! Try selecting, 0, 1, and many options to see it update accordingly.

11. The list of skills is hard to read through, let's sort it so that our list is alphabetical.

## Exercise 4: Full text search in the dropdown

For our last piece of our form, we're going to update our `SelectField` a little more and make it easier for users to find certain skills with a search bar!

In **src/components/SelectField.tsx**:

### Set Up for Searchability

1. Update the `SelectField` component to have an optional boolean property of `searchable`. We'll set the default value to `false`.

2. Add another optional prop `SearchFieldProps` of type `TextFieldProps` to pass to our search `TextField` for customization

3. Add a `useState` hook to store and set search values (we'll use these in the next step)

4. Let's make our SelectField component used on the profile screen searchable by passing in the new property.

### Add Search Header to Bottom Sheet Modal

5. Add a `ListHeaderComponent` prop on the `BottomSheetFlatList` that renders if `searchable` is `true`

```tsx
<BottomSheetFlatList
  //...
  ListHeaderComponent={
    searchable ? (
      <TextField
        value={searchValue}
        onChangeText={(searchInput) => setSearchValue(searchInput)}
        containerStyle={$searchContainer}
        RightAccessory={() => {
          return searchValue ? (
            <TouchableOpacity
              style={$searchClearButton}
              onPress={() => setSearchValue("")}
            >
              <Icon icon="xCircle" color={colors.text} size={20} />
            </TouchableOpacity>
          ) : undefined;
        }}
        {...SearchFieldProps}
      />
    ) : undefined
  }
/>;

//...

const $searchContainer: ViewStyle = {
  paddingHorizontal: spacing.lg,
};

const $searchClearButton: ViewStyle = {
  overflow: "hidden",
  height: 40,
  paddingHorizontal: spacing.xs,
  alignContent: "center",
  justifyContent: "center",
};
```

This looks a little tight with the 50% snapPoint we have set, why don't we make it almost full screen if the list is searchable.

6. Update the snapPoints on the `BottomSheetModal` to be dynamic based on our `searchable` boolean

```diff
<BottomSheetModal
  ref={sheet}
+ snapPoints={searchable ? ["94%"] : ["50%"]}
/>
```

7. We want this header to stay at the top while we scroll, so let's make it sticky

```diff
<BottomSheetFlatList
//...
+stickyHeaderIndices={searchable ? [0] : undefined}
/>
```

### Filter data based on search values

Let's put this search to work and update our `BottomSheetFlatList` options to be filtered by the search input value

8. Filter the options data if `searchable` is `true`

```tsx
// Filter options for partial name if searchable is true
const filteredOptions = searchable
  ? options.filter((o) =>
      o.label.toLowerCase().includes(searchValue.toLowerCase())
    )
  : options;
```

9. Update `BottomSheetFlatList` data property to use the new `filteredOptions`

### Clean Up Search and Fix Keyboard Dismissal

You might have noticed a couple oddities in the behavior when navigating through the form into the `SelectField`, or leaving the modal with a search value. Let's clean some stuff up!

#### Clear the search when exiting/closing the modal

10. Call `setSearchValue("")` in the following places

- `onPress` for the `BottomSheetBackdrop`
- Within the `dismissOptions` function before dismissal

11. Add an onDismiss prop to the `BottomSheetFlatList` that calls the `dismissOptions` function to handle the clear on drag close.

#### Dismiss the keyboard when opening the `BottomSheetModal`

When we are focused on a different text input when we click into the `SelectField` component, the keyboard stays up over the `BottomSheetModal`. Let's fix that!

12. Add `Keyboard.dismiss()` within the `presentOptions` function

#### Skip the `SelectField` input when navigating with arrows

While we technically used a `TextField` component for displaying our selected values when the bottom sheet closes, we don't want users to edit the text there. It's pretty confusing when navigating with the arrows from the `KeyboardToolbar` as well so let's fix it.

13. Update the `TextField` in our `SelectField` component to not be editable

#### Add `KeyboardToolbar` to help with closing keyboard

That toolbar we added to the keyboard to navigate through our form was great so let's add one here as well. We only have one field though so no need for the arrows to jump between inputs, we'll just use the Done button.

14. Add a `KeyboardToolbar` component within the `BottomSheetModal` component
15. Hide the arrows!

#### Fix bottom padding when keyboard is open to search

Notice when we focus on the search `TextInput` and scroll to the bottom of our full skills list the bottom is somewhat cut off.

16. Add a useState hook that stores and sets a value paddingBottom for our `BottomSheetFlatList`
17. In the search `TextField` component, update `onFocus` and `onBlur` and to set and clear the paddingBottom value (275 is a good number to try).
18. Add the style in the `contentContainerStyle` on the `BottomSheetFlatList`

🏃**Try it.** Does your profile form look better and behave as you would expect it to? Is the keyboard dismissing as expected and the search behaving how you imagined?

## Side Quests

- Form validation
- Improve the slider
- Use theming helpers for dark mode

## See the solution

Switch to branch: `02-inputs-solution`
