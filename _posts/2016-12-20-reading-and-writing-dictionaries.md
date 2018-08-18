---
title: Reading and Writing Dictionaries
tags: [ios]
---

When you need to store a dictionary, there are a few different options for doing so.

## Info.plist

First, a quick note about `Info.plist`. You don't need to manually read in a file to access the contents of this file as a dictionary: you can just call `Bundle.main.infoDictionary`.

## URLs to the docs directory

First, let's talk about how to refer to file locations. The `URL` is the main way locations are referred to, even though they're local URLs on the device.

To write files, you use the documents directory. The base documents directory is a bit more complex to calculate, because it uses the same API as on macOS, where there are more options:

```swift
let docsBaseURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
```

From there, you can get the path to a specific file by adding a path component again:

```swift
let customPlistURL = docsBaseURL.appendingPathComponent("custom.plist")
```

## Reading and Writing Plists

To read a property list file, you can use a constructor built in to `NSDictionary`:

```swift
let customDict = NSDictionary(contentsOf: customPlistURL) as? [String : AnyObject]
```

`NSDictionary` is the old Objective-C dictionary class; if you're working in swift, you probably want a Swift-style dictionary, which is why it's cast to `[String : AnyObject]`.

To write a property list, there is also a method built in to `NSDictionary`:

```swift
NSDictionary(dictionary: customDict).write(to: customPlistURL, atomically: true)
```

We need to use a `NSDictionary` constructor again to turn the Swift dictionary back into an `NSDictionary`. If you used the `NSDictionary` as-is, you wouldn't need to translate it.

Note that the value type of the Swift array  is `AnyObject`, not `Any`. this means that the values of the dictionary have to be objects, so `Int`s and `Double`s can't be used. Instead, you need to wrap them in a `NSNumber`. If you would rather be able to use `Int`s and `Double`s directly, there's another option: archives.

## Reading and Writing Archives

Archives can be used to store a wide variety of types, including Swift dictionaries. To read an archive, you use the `NSKeyedUnarchiver` class:

```swift
let customDict = NSKeyedUnarchiver.unarchiveObject(withFile: customArchiveURL.path) as? [String: Any]
```

Note that `unarchiveObject()` takes a path String instead of a URL, but all you need to do differently is access the `path` attribute of the `URL` object.

Note, as well, that the type of the object we unarchived is a Swift dictionary with a value of type `Any`, so `Int`s and `Double`s can be used directly.

To save an archive, you use `NSKeyedArchiver`:

```swift
NSKeyedArchiver.archiveRootObject(dictionary, toFile: url.path)
```
