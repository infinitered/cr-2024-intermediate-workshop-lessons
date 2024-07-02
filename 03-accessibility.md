# Module 03: Accessibility Timebox

### Goal

How much of our developer profile form can we make accessible before lunch?

### Concepts

- Experience your form with Talkback / Voiceover
- Use accessibility features so the inputs are clearly identified as the user scrolls down the screen and changes focus

<!-- Talkback / Voiceover work in emulators/simulators, right? Is it better to focus on devices here? -->

### Features to build

- Make the developer profile tab accessible, facilitatng smoothly navigating through the input fields using voice feedback.

### Resources

- iOS
  - [Turning on VoiceOver on your iPhone](https://support.apple.com/guide/iphone/turn-on-and-practice-voiceover-iph3e2e415f/ios)
  - [VoiceOver gestures](https://support.apple.com/guide/iphone/use-voiceover-gestures-iph3e2e2281/ios)
  - [Accessibily Inspector](https://developer.apple.com/documentation/accessibility/accessibility-inspector)
- Android
  - ???

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

## Exercise 1: Text inputs

### Extra Android features

## Exercise 2: Toggles and sliders

## Exercise 3: That modal picker thing!

## Exercise 4: Buttons and validation


## Side Quests

- Add any extra stuff to do here

## See the solution

Switch to branch: `03-accessibility-solution`