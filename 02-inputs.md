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

- Add any helpful links here

# Exercises

## Exercise 1: Basic keyboard avoidance with `react-native-keyboard-controller`

- Update screen with scrolling to have KeyboardAvoidingScrollview instead of basic ScrollView
- Add KeyboardProvider wrapped around Slot in App
- Add some padding to the keyboard offset to have full view of the text field
- Weird extra padding at the bottom because of both KeyboardAvoidingView and KeyboardAwareScrollView https://github.com/kirillzyusko/react-native-keyboard-controller/issues/451 so only add it on scroll screens
- Weird behavior on android forcing screen up because of the "height" setting so set to undefined and rebuild

## Exercise 2: Smooth transitions between fields from the keyboard

## Exercise 3: New dropdown component

## Exercise 4: Full text search in the dropdown

## Side Quests

- Form validation?
- Improve the slider
- ???

## See the solution

Switch to branch: `02-inputs-solution`
