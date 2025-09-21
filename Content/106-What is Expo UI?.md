# What is Expo UI?

Expo UI (package @expo/ui) is a cutting-edge library in the Expo ecosystem that provides a set of native input components enabling you to build fully native user interfaces leveraging SwiftUI for iOS/tvOS and Jetpack Compose for Android. ￼

It is currently in alpha—meaning things are actively developing, APIs may change, and it’s not yet stable. Notably, Expo Go doesn’t support Expo UI, so to try it out you’ll need to use development builds. ￼

---

## Why You Should Care

Here are some of the benefits and reasons developers are excited about Expo UI:

- **Native performance & look & feel**: By using SwiftUI and Jetpack Compose under the hood, Expo UI ensures you’re getting components that behave like first-class native widgets.
- **Less bridging / wrapper overhead**: Instead of manually writing React Native wrappers or bridging into native view toolkits, Expo UI aims to streamline that process.
- **Modern UI paradigms**: These libraries (SwiftUI / Jetpack Compose) are declarative, responsive, and designed for modern app UX; Expo UI lets React Native apps tap into that paradigm.
- **Unified development experience**: For teams already using Expo / React Native, Expo UI gives access to native UI componentry without jumping fully out of the React Native / Expo world.

---

## Key Features of Expo UI

Here are some of the components and features currently available, plus example usage patterns. These are good to know if you’re evaluating or beginning to use the library. ￼

### Installation

To add Expo UI to your project:

```bash
npx expo install @expo/ui
```

If adding to an existing React Native app, you also need to have expo installed. ￼

---

## SwiftUI Examples (iOS / tvOS)

Here are some of the components you can use, with example code:

- **BottomSheet**: Slide-up panels, modals etc. ￼
- **Button**: Standard clickable buttons; has variants. ￼
- **CircularProgress**: Circular progress or spinner component. ￼
- **ColorPicker**: Let users pick colors with UI that fits the native platform. ￼
- **ContextMenu (a.k.a DropdownMenu)**: For menus/triggers inside native UI. ￼
- **DateTimePicker**: Date or time pickers, with variants (wheel, etc.). ￼
- **Gauge**: For visualizing values within a range; circular or capacity‐style. ￼
- **Host**: A component that allows embedding these native UI components within React Native views. It acts somewhat like **`<Canvas>`** or **`<View>`** and uses UIKit’s UIHostingController under the hood. ￼

---

## Jetpack Compose Examples (Android)

On Android, via Jetpack Compose, you also have:

- Button
- CircularProgress
- ContextMenu
- Chip (variants such as assist, filter, input, suggestion) ￼
- DateTimePicker (date / time)
- LinearProgress
- Picker (segmented, radio, etc.)
- Slider
- Switch / Toggle / Checkbox
- TextInput ￼

---

## Limitations & Things to Watch Out For

Because Expo UI is still in alpha, there are several important caveats:

1. **Breaking changes are likely**: APIs may change often. Keep up with the changelog. ￼
2. **Not in Expo Go**: You’ll need to use development builds to test. ￼
3. **Not every component is available on every platform**: Some variants or components are missing or not supported on tvOS (for example). ￼
4. **Documentation is still growing**: The API docs are still limited; you may need to look at TypeScript types or example code to fill gaps. ￼

---

## How to Integrate Expo UI in Your App

Here’s a suggested workflow when adopting Expo UI in an app:

1. Set up a development build: Because Expo Go doesn’t support Expo UI, configure a dev build.
2. Install the package via npx expo install @expo/ui.
3. Experiment with small components— start using simple components like Button, TextInput, etc., inside a Host.
4. Test on each platform (iOS, Android, tvOS if relevant) to see how variants behave.
5. Monitor updates: stay current with Expo’s changelog; alpha releases might introduce breaking API changes.
6. Fallback strategy: Have a fallback UI (React Native / existing components) ready for parts not yet supported or unstable.

---

## Sample Code Snippet

Here’s a concise example showing a React Native + Expo UI workflow combining components:

```javascript
import React, { useState } from "react";
import { View, Text } from "react-native";
import {
  Host,
  Button,
  DateTimePicker,
  Switch,
  TextField,
} from "@expo/ui/swift-ui"; // or '/jetpack-compose' on Android

function SettingsScreen() {
  const [date, setDate] = useState(new Date());
  const [enabled, setEnabled] = useState(false);
  const [text, setText] = useState("");

  return (
    <View style={{ flex: 1, padding: 20 }}>
      <Host>
        <TextField
          defaultValue={text}
          onChangeText={setText}
          placeholder="Type something..."
        />
        <Switch
          checked={enabled}
          onValueChange={setEnabled}
          label="Enable feature"
        />
        <DateTimePicker
          displayedComponents="date"
          initialDate={date.toISOString()}
          onDateSelected={(d) => setDate(d)}
        />
        <Button
          variant="default"
          onPress={() => {
            console.log("Settings saved!");
          }}
        >
          Save Settings
        </Button>
      </Host>
    </View>
  );
}

export default SettingsScreen;
```

---

## Future Outlook & Why It’s Exciting

- As Expo UI matures past alpha, it may become a go-to library for developers wanting native look & feel while staying within React Native/Expo.
- Could reduce the fragmentation between React Native UI component libraries and truly native UI paradigms.
- Better performance, more consistent behavior, and easier maintenance vs. custom native bridging.
- Strong relevance for developers building apps with high UX fidelity (animations, complex input forms, native gestures etc.).

---

## Conclusion

Expo UI is an ambitious library that bridges modern declarative native UI frameworks (SwiftUI, Jetpack Compose) with the flexibility and wide community adoption of React Native / Expo. For developers who care about native performance and want advanced UI components without stepping fully outside React/Expo, it’s definitely worth exploring.

Because it’s in alpha, though, careful planning and fallback strategies are important. But for the right projects, especially ones willing to experiment and keep up with updates, Expo UI offers a glimpse of what the future of hybrid-native app UI might look like.

---

### 🤝 Need a Custom RSVP System or Dashboard?

I help businesses build tools that _actually work_ , even on tight deadlines.

Whether you're planning an event, need internal tools, or want a custom dashboard for your team , I can help.

### Reach out

📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
