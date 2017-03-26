---
title: Changing iOS Behavior Between Debug and Release
---

Oftentimes you will want your app to behave slightly differently during development and at release. For example, you may want to perform more logging, or use hard-coded or staging environment data. How can you detect whether your app is running in Debug or Release mode?

One quick precaution: it's safest to minimize differences between your app in different environments, otherwise you risk not finding a production bug until the app is in the app store. To minimize the risk of this happening, instead of just making a change in the debug environment, it's better to expose settings that allow you to toggle back and forth. For example, you might have a switch that allows you to toggle between staging and production data.

Here are the steps:

## 1. Create a user-defined build setting

User-defined build settings allow you to capture environment differences at build time.

Rather than creating a user-defined build setting that simply records whether it is Debug or Release, it's better to create settings that determine the specific behavior you want to change. For example, you could have a `ALLOW_STATIC_DATA` setting, set to `NO` for Debug and `YES` for release.

## 2. Add the build setting to the Info.plist

Build settings aren't directly accessible at runtime. Instead, the common pattern is to write them into your `Info.plist` at build time, where they can then be read at runtime.

In your `Info.plist`, create a custom property. Name it the name of the build setting, formatted `UpperCamelCase` to match `Info.plist` conventions. For example, `AllowStaticData`. Set the value of the setting to the name of your build setting within a `$()`. For example, `$(ALLOW_STATIC_DATA)`.

## 3. Read the value at runtime

You can read the value of this `Info.plist` setting at runtime in the standard way. It's stored as a String, so you'll need to cast it to a string, and from there to any other type you might need

```swift
if let allowStaticDataString = Bundle.main.infoDictionary?["AllowStaticData"] as? String {
  let allowStaticData = (allowStaticDataString == "YES")

  //...
}
```
