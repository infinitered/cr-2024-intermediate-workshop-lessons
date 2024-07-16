## Side Quest: Bzzzt! Adding haptics

Often you'll build an experience and it functions great. Sometimes though, it can feel a little flat. Haptics is a great way to add an extra dimension pretty easily, giving the user some physical feedback as they experience your application.

First, let's provide some feedback if the user submits incorrect credentials.

🏃**Try it.** For the ease of testing this, from `log-in` route, backspace until you have an invalid email address format (e.g., leave off the domain) and submit the form.

You'll notice the only feedback is the helper text in red. It's something, but feels lacking. Let's iterate on that feedback to enhance it with haptic feedback.

`src/app/log-in.tsx`

```diff
+ import * as Haptics from 'expo-haptics'

// ...

// inside function login() {
- if (validationError) return
+ if (validationError) {
+   Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium)
+   return
+ }
```

Pretty simple stuff, but now you'll get a nice buzz when the form bounces back with an issue.

🏃**Try it.** Submit an invalid email again and actually feel the feedback this time. Play with the intensity as well via `ImpactFeedbackStyle`.

Another example of where we could add this feedback is on certain controls, such as the Slider found at the `/profile` route. Add some feedback when the user drags the slider to each value:

`src/app/(app)/(tabs)/profile.tsx`

```diff
+ import * as Haptics from 'expo-haptics'

// ...

// find <Slider />
- onValueChange={(value) => setProp("rnFamiliarity", value)}
+ onValueChange={(value) => {
+   Haptics.selectionAsync()
+   setProp("rnFamiliarity", value)
+ }}
```

🏃**Try it.** Change the slider values around. Feel the difference?

Simple, but effective.
