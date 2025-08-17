# React Native 0.80 Is Here: What’s New, What Changed, and Why It Matters

React Native just dropped version **0.80**, and it’s a big one. Not flashy. But meaningful. This release is all about **stability**, **cleaner architecture**, and getting rid of long-standing quirks that annoyed devs for years.

If you’ve been holding off upgrading, 0.80 might be the version that finally earns your trust. Let’s break it down.

---

## **TL;DR — What’s New in React Native 0.80?**

- **No more deep imports**
- **Hermes upgraded to 121**
- **PHC (Project Health Checklist)** fully implemented
- **TypeScript updates**
- **TurboModules & Fabric** closer to mainstream
- **Improved Expo support**

---

## **Goodbye Deep Imports**

You know the drill. You want to use something like TextInput, and next thing you know, you’re importing from some internal, undocumented path like this:

```js
import TextInputState from "react-native/Libraries/Components/TextInput/TextInputState";
```

In 0.80, this stops.

React Native now hides all internal modules by default. Deep imports are officially **blocked** unless you explicitly mark them in your Metro config. This makes the codebase safer, updates less painful, and gives the core team more flexibility to refactor without breaking apps.

If you still need deep imports (hopefully not), you’ll need to **manually allow them** using:

```
unstable_allowRequireDeepImports: true
```

But honestly? Time to refactor those imports.

---

## **Hermes 121 Is the New Default**

Hermes — Meta’s JavaScript engine — just got bumped to version **121**. It’s faster, lighter, and more compatible with modern JS syntax.

The update brings:

- Better **garbage collection**
- Smaller **bundle sizes**
- Faster **cold start**
- Improved **error messages**

If you’re not using Hermes yet, now’s a great time to switch. It’s officially the default and well-supported in the ecosystem.

---

## **PHC Complete: What That Means**

React Native’s **Project Health Checklist (PHC)** is done.

It was a long effort aimed at:

- Cleaning up unstable APIs
- Improving cross-platform consistency
- Making the dev experience smoother

Think of PHC as a long overdue spring cleaning. The result? Fewer surprises. More predictability. And a healthier foundation for future features.

---

## **TypeScript Upgrades**

React Native now includes **TypeScript 5.4** support. That means better typings, smarter IntelliSense, and improved DX across modern editors like VS Code.

Also, the core types are more polished. If you’ve struggled with confusing or missing type hints in the past, especially with FlatList, StyleSheet, etc., you’ll feel the difference.

---

## **TurboModules & Fabric… Almost There**

While still not 100% default, **TurboModules** and **Fabric renderer** are more stable than ever. A bunch of internal modules now use them, and Expo already has partial support.

These bring:

- Better performance
- Seamless async bridging
- UI thread optimizations

If you’re building native modules, start looking into TurboModule migration. You’ll be future-proofing your code.

---

## **What About Expo?**

React Native 0.80 is now **fully supported in Expo SDK 51**.

So if you’re an Expo dev, you get:

- All the new stability perks
- Faster builds with Hermes 121
- No need to worry about deep import issues
- Smoother native module integration

And yes, expo-modules-core is getting better TypeScript typings too.

---

## **Should You Upgrade?**

If you’re on React Native 0.72 or earlier, **yes**. Absolutely.
This release isn’t packed with UI features or new APIs, but it **simplifies**, **stabilizes**, and **future-proofs** your code.
Great for teams. Great for long-term maintenance.

Just watch out for these breaking changes:

- Deep imports are blocked by default
- Hermes is now the standard (you might need to tweak config)
- Some legacy modules are now deprecated or moved

---

## **Final Thoughts**

React Native 0.80 feels like a turning point. Not in a dramatic way, but in a quiet, professional, “we’re finally cleaning up the house” kind of way. If you’ve been craving a more stable, less quirky experience with React Native… this is your moment.

---

Want help upgrading your app to 0.80? Or need a hand with TurboModules and Fabric migration? I’ve been down that road, reach out anytime.

---

**🚀 Let’s build something amazing! If you have a project in mind or need help with your next design system, feel free to reach out.**  
📧 Email: [safi.abdulkader@gmail.com](mailto:safi.abdulkader@gmail.com) | 💻 LinkedIn: [@abdulkader-safi](https://www.linkedin.com/in/abdulkader-safi/) | 📱 Instagram: [@abdulkader.safi](https://www.instagram.com/abdulkader.safi/) | 🏢 [DSRPT](https://www.dsrpt.com.au/kw/contact)

_Drop me a line, I’m always happy to collaborate!_ 🚀
