---
title: Swift and Shared Mutable State
---

Shared mutable state is a big source of bugs. To get around this, functional languages with immutability promise simpler code. But this also involves separating data from the behavior that operates on it.

But that separation isn't a necessary part of immutability: OO folks are experimenting with immutability as well. But this can lead to awkward interfaces where entire new objects are instantiated just to change one property.

Swift's structures provide an alternative way to manage shared mutable state. Structures address the problem from two angles: the mutability, and the sharing. 

- Structures have value semantics, which means that they're never *shared*: they're passed by copy.
- When a structure is stored in a `let` constant, its properties are immutable—and if any of its properties are structures, *their* properties are immutable as well.
- Structures can't modify their own properties by default: you have to add the `mutating` keyword before a method on a structure that modifies its properties.
- Any of a structure's properties that are `let` constants can't be modified.
