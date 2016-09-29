---
title: Xcode Project File
---

The main file containing project settings for [Cocoa Touch](Cocoa-Touch).

## Location of Frequently-Used settings

* **Home Screen App Name**
  * Build Settings > Packaging > Product Name
    * Which by default is set to `$(TARGET_NAME)`
  * This can be overridden by adding Info > Custom iOS Target Properties > Bundle display name
* **Home Screen Icon**
  * General > App Icons and Launch Images > App Icons Source
    * Which by default is set to `AppIcon` in `Assets.xcassets`
* **Bundle Identifier**
  * Affects whether debug, test, beta, and app store versions overwrite each other
  * Target > Product Bundle Identifier
