---
title: Core Data
---

Persistence framework for [Cocoa Touch](Cocoa-Touch).

## Quick Reference

<dl>
<dt>Create</dt>
<dd>
```swift
let widget = NSEntityDescription.insertNewObjectForEntityForName("Widget", inManagedObjectContext: managedObjectContext) as! Widget
```
</dd>
<dt>Delete</dt>
<dd>
```swift
managedObjectContext.deleteObject(widget)
```
</dd>
</dl>