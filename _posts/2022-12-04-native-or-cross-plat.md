---
title: 'To Native or not to Native. If not Native, what to non-Native?'
---
Circa Dec 2022.

# Dilemma:
I want to create an app that is available on all platforms - Web, iOS, Android; and on all form-factors - Desktop, Laptop, Tablet, Phone, Watch.

# My Criteria:

1. Easyness - Less ongoing effort.
2. Freshness - Easy to keep clients fresh.
3. Insightness - Unified monitoring.

# Options considered:
I see the choices on a Sprectrum, from the safest to the easiest:

- Create Native Apps on each Platform
    
  - Web, iOS, Android, WearOS, WatchOS.

- Flutter App.

  - Has a "Skia" Rendering Engine, similar to Unity.
  - Flutter for Web is a joke, according to t3dotgg.
  - Code push has to go through Apple/Google store review.

- .NET MAUI App.

  - Haven't looked deeper into this.
  - Smells like WPF which I hate.

- React Native App.

  - Has something called [Expo](https://expo.dev/) for pushing out updates.
  - Does this work for Web? Apparently yes, but too high of an abstraction according to t3dotgg.
  - React Native has an impression of being slow.
  - [Theo likes React Native](https://www.youtube.com/watch?v=3_FcxGCCnUs&ab_channel=Theo-ping%E2%80%A4gg)
  - Some guy on Twitter mentioned an [alternate stack](https://twitter.com/_andypeacock/status/1597159611735281664).
 
- Progressive Web App (PWA).

  - There is only "partial" support for iOS, so this is not a great option.

- Web App only.

  - Won't give free marketing through App Stores.
  - Won't feel native - tab in a browser that would be too clunky to get back to. Can't pin to lock screen.
  - May not get proper access to native features (eg. gps).

# Conclusion:

Going with [Expo](https://expo.dev/) + React Native for now.

# References:

https://youtu.be/0zY-5z_8D4o

