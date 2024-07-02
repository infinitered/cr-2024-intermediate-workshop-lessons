# Module 03: Accessibility Timebox

### Goal

How much of our developer profile form can we make accessible before lunch?

### Concepts

- Experience your form with Talkback / Voiceover
- Use accessibility features so the inputs are clearly identified as the user scrolls down the screen and changes focus

<!-- Talkback / Voiceover work in emulators/simulators, right? Is it better to focus on devices here? -->

### Features to build

- Make the developer profile tab as accessible as possible, facilitatng smoothly navigating through the input fields using voice narration.

### Resources

- iOS
  - [Turning on VoiceOver on your iPhone](https://support.apple.com/guide/iphone/turn-on-and-practice-voiceover-iph3e2e415f/ios)
  - [VoiceOver gestures](https://support.apple.com/guide/iphone/use-voiceover-gestures-iph3e2e2281/ios)
  - [Accessibily Inspector](https://developer.apple.com/documentation/accessibility/accessibility-inspector)
- Android
  - ???

# Philosophy

This lesson is a little different from the others. We have some suggested code in here, and suggested areas you should cover when considering the accessibility of the form, but each platform has different capabilities, and there's general guidelines and recommendations and best practices for accessibility, but the operation requires some nuance. You are _describing_ for form; some people may describe it a bit differently. We provide ideas in the form of code snippets, but feel free to customize them as you experience the accessiblity of the form itself. If you find a better option, we would love to talk about it as a group.

We will also be using primarily standard React Native component accessibilty props. There are libraries that can help streamline building accessibility into your UI design process that may be a better fit for your project. But, it's also important to learn what's under the hood, what are the core capabilities that mobile developers lean on.

# Exercises

## Exercise 0: Experiencing accessibilty tools
To get going on addressing accessibility, it's first important to get the tools working that you'll need to experience accessibility. You may be able to get to a second platform during this exercise, but first start by picking one to focus on and getting through that, as each experience will be a little differernt.

If you are able to use a device for development, this is the one situation where a device is going to be more realistic than the simulator. However, you can also get a lot of essential data about your app's accessibility through the iOS simulator, and it's probably the easiest way to get started, so let's go with that. If you need to use Android, note that you will have to use an Android device in order to access and troubleshoot your app's narration-based accessibility.

### iOS simulator
The iOS simulator doesn't support VoiceOver directly, but it can be used with the macOS Accessibilty Inspector.

Search for "Accessibility Inspector" in Spotlight; it should already be there.

Set it to inspect the workshop app on your simulator:

[<img src="./assets/03/accessibility-inspector-1.png" width="250" />](./assets/05/accessibility-inspector-1.png)

Switch to the Profile tab, make sure the speech bubble is turned on, and press play. It will cycle through the inputs. You can also pause it and go back and forth with the arrows.

[<img src="./assets/03/accessibility-inspector-2.png" width="250" />](./assets/05/accessibility-inspector-2.png)

> Real VoiceOver tends to respond a little better when it comes to narration within your control. For instance, VoiceOver will provide updates as the slider value is changed, but Accessibility Inspector will not. However, you can approximate the results by moving to the previous control and back again after changing the value. One thing that is helpful about Accessibility Inspector is that touch interactions on the simulator largely stay the same as with VoiceOver off.

### iOS device (optional)

You can do basically everything in this module with Accessibilty Inspector, but at some point in app development, you need to move over to VoiceOver to understand how accessibility features work in practice.

You can set an accessibility key (e.g., turn on VoiceOver on triple press of the side key) on your iPhone, or you can turn on VoiceOver from the Control Center.

Learn more:
  - [Turning on VoiceOver on your iPhone](https://support.apple.com/guide/iphone/turn-on-and-practice-voiceover-iph3e2e415f/ios)
  - [VoiceOver gestures](https://support.apple.com/guide/iphone/use-voiceover-gestures-iph3e2e2281/ios)

### Android device

// TODO

### Test accessiblity

Whatever tool you use, since we're focusing on making the Profile tab more accessible, you'll want to cycle through narration on that tab, noting what works well and what doesn't. Also consider the tab navigation itself.

🏃**Try it.** Cycle through the controls on the Profile tab. How would it feel to use this if you depended on the accessibility tools?

## Exercise 1: Text and text inputs

You may have noticed that the narration from the screen reader sounds like a bunch of random words: "Profile, Name Name, Years of Experience, Years of Experience", etc. It doesn't tell you what the fields are, or even the purpose of the form, or even if it is a form.

### Describe this thing like a normal person, please?

First things, first - what are we even focused on. Let's use simple `accessiblityLabel` props to describe the "Profile" a bit better. Even if we don't do anything else, at least someone could understand that this is an entry form, that it is a place for you to enter information about you, the developer.

#### Try these, or something like it:
- Add an `accessibilityLabel` to the `Text` component displaying the "Profile" heading, like:
```tsx
<Text preset="heading" tx="demoProfileScreen.title" style={$title} accessibilityLabel="Developer profile, entry form" />
```
- Consider if it's even clear what a "Profile" is to someone who doesn't need voice narration. Considering context and if it needs to be spelled out can be good for everyone, depending on the situation, you could add a subheading describing the form under the heading, like:
```tsx
<Text preset="heading" text="Enter your developer profile to let us know what you'd like to work on and what React Native Radio episodes you'd like to hear" />
```

### Text Inputs that describe themselves as such

What is a "Name Name"? Clearly, this description will not do.

1. Try adding an `accessibilityLabel` to the name `TextField` to see what happens. Does anything change?

`TextField` isn't a stock React Native component. It does pass additional props to the underlying `TextInput`, but that alone will not have the effect you're looking for. So, let's crack open `TextField` and see how we can modify it to describe itself well.

First off, the whole thing is wrapped by a `TouchableOpacity`. This is actually why it already sounds a little better than it would if it was just a separate `Text` and `TextInput`. By default, buttons have `accessible` set to true, [which means they behave as a container that should treat the child components as a single component for the purposes of accessibility](https://reactnative.dev/docs/accessibility#accessible).

But, the `Text` and `TextInput` controls are announcing their contents and their placeholders, respectively. Hence, "Name Name". One simple way to get it to stop repeating itself is to get rid of one of these.

2. In `TextField`, the `Text` component is optional, so let's stop it from announcing itself by giving it a blank label:
```diff
<Text
+  accessibilityLabel=""
  importantForAccessibility="no-hide-descendants"
  preset="formLabel"
  text={label}
  tx={labelTx}
```

🏃**Try it.** Less repetitive, huh?

3. It's still difficult to understand what is "Name", though. Here's an idea. Add an explicit `accessibilityLabel` prop to the type definition for `TextField`. Update the `TextInput` inside to use the `accessiblityLabel` if its available, and fall back to displayed label if it's not:

```diff
<TextInput
+  accessibilityLabel={`${accessibilityLabel || (labelTx ? translate(labelTx!, labelTxOptions) + ", text input" : "") }`}
  ref={input}
```

> You'll also need to import `translate` from `src/i18n`. This will turn those i18n labels into the text that needs to be read by the narrator.

🏃**Try it.** Back in the Profile tab, just leave off any accessiblity labels from the `TextField`'s. See how it sounds.

5. Now, where we think more context is needed, let's add `accessiblityLabel` to selected `TextField`'s. For instance, on years of experience, you could do this:

```diff
<TextField
+ accessibilityLabel="Years of experience, text input, numeric value"
  labelTx="demoProfileScreen.yoe"
  containerStyle={$textField}
  keyboardType="number-pad"
  placeholderTx="demoProfileScreen.yoe"

6. You might have noticed that Bio still doesn't sound very good. Generally screen readers ignore the placeholder text, but it doesn't seem to be in the case of a multi-line text input. [Apparently placeholders aren't great for accessilbity, anyway](https://www.w3.org/WAI/tutorials/forms/instructions/#placeholder-text). Let's just get rid of it:

```diff
<TextField
+  accessibilityLabel="Bio, text input, multi-line"
  labelTx="demoProfileScreen.bio"
  containerStyle={$textField}
  multiline
-  placeholderTx="demoProfileScreen.bio"
  value={bio}
  onChangeText={(text) => setProp("bio", text)}
```

### Extra Android features

// TODO: Adapt `TextField` to implement `accessibilityLabelledBy`.

## Exercise 2: Toggles and sliders

## Exercise 3: That modal picker thing!

## Exercise 4: Buttons and validation


## Side Quests

- Add any extra stuff to do here

## See the solution

Switch to branch: `03-accessibility-solution`