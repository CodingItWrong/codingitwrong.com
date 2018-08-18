---
title: Changing iOS Behavior Between Debug and Release
tags: [ios]
---

Oftentimes you will want your app to behave slightly differently during development and at release. For example, you may want to perform more logging, or use hard-coded or staging environment data. How can you detect whether your app is running in Debug or Release mode?

One quick precaution: it's safest to minimize differences between your app in different environments, otherwise you risk not finding a production bug until the app is in the app store. To minimize the risk of this happening, instead of just making a change in the debug environment, it's better to expose settings that allow you to toggle back and forth. For example, you might have a switch that allows you to toggle between staging and production data.

Here are the steps:

## 1. Create a custom swift compiler flag

Custom compiler flags allow you to change compilation behavior.

First, choose a name for the custom flag you will be setting, related to the feature you'll be enabling or disabling. For example, on a project I used an `ALLOW_STATIC_DATA` flag, to allow using hard-coded data instead of a real backend. Feature-related flags are more flexible than a generic `IS_PRODUCTION` flag, because it more clearly documents what functionality is affected, and individual flags can be toggled rather than changing code.

In your project, select your app target, then under Build Settings > Other Swift Flags, add `-D [your flag name]`. Using the example above, it would be `-D ALLOW STATIC DATA`. You can add the flag in only Debug, only Release, or both.

## 2. Check the flag in conditional compilation blocks

Swift provides an `#if` macro that can check compiler flags and conditionally include code:

```swift
#if ALLOW_STATIC_DATA
  // ...
#else
  // ...
#endif
```
