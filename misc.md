# Miscellaneous stuff
We'll be put interesting stuff up here as it comes up right before or during the workshop

## Module 01

## Module 02

## Module 03

## Module 04

## Module 05

- `Bound renderChildren: Support for defaultProps ...` warning when navigating to Podcast detail screen. Turns out this is just an open issue with `react-native-render-html` (https://github.com/meliorence/react-native-render-html/issues/661). It's more annoying than an actual issue at the moment. So add `"Support for defaultProps"` to **ignoreWarnings.ts**.
  - Ah, but wait? Why isn't it working? In the conversion for Expo Router, we might have missed an import of **ignoreWarnings.ts**. Add this in the root layout: `import "src/utils/ignoreWarnings"`.
 
## Other
- How to add a little bit of Expo to an RNC CLI app: https://github.com/keith-kurak/AddSomeExpo2023
