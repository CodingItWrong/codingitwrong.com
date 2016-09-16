---
title: Core Data
---

Persistence framework for [Cocoa Touch](Cocoa-Touch).

## Quick Reference

### Create

```swift
let widget = NSEntityDescription.insertNewObjectForEntityForName(
                "Widget", inManagedObjectContext: managedObjectContext)
                as! Widget
// then at some point:
try managedObjectContext.save()
```

### Read All

```swift
let fetchRequest = NSFetchRequest(entityName: "Widget")
let widgets = try managedObjectContext.executeFetchRequest(fetchRequest)
                    as? [Widget]
```

### Read Some

```swift
let fetchRequest = NSFetchRequest(entityName: "Widget")
fetchRequest.predicate = NSPredicate(format: "widgetId == %@", idToFind)
let widgets = try managedObjectContext.executeFetchRequest(fetchRequest)
                    as? [Widget]
```

### Update

Individual objects don't need to be saved; you just need to save the entire managed object context at some point:

```swift
try managedObjectContext.save()
```

### Delete

```swift
managedObjectContext.deleteObject(widget)
```
