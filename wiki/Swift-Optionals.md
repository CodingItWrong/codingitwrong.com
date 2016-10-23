---
title: Swift Optionals
---

Optionals are a feature in [Swift](Swift) that take the place of nillable
variables.

## Declaring an Optional

`var possibleInt: Int?`

## Forced Unwrapping

`possibleInt!` - gives an `Int`; blows up if there is no value

## Checking for a value

```
if possibleInt != nil {
    print("The int is \(possibleInt!)")
}
```

## Optional Binding

```
if let theInt = possibleInt {
    print("The int is \(theInt)")
}
```

## Nil Coalescing Operator

Provides a default value when the optional is nil.

`let name = possibleName ?? "hi"`

## Implicitly Unwrapped Optionals

For when an optional starts out without a value, but it's clear from a program's structure that an optional will always have a value after that value is first set. Useful especially in class initialization.

```
var assumedInt: Int!
assumedInt = 42
print(assumedInt)
```

## Optional Chaining

```
var possibleString: String!
print(possibleString?.lowercased)
```

Different from forced unwrapping in that optional chaining fails gracefully and returns a `nil`, whereas forced unwrapping triggers a runtime error.

```
print(optionalDate.map(dateFormatter.stringFromDate))
```

If `optionalDate` is defined, calls `dateFormatter.stringFromDate(optionalDate)` and returns the result; otherwise returns `nil`.

## Downcasting

### Conditional type cast

`control as? UIButtonView`

Returns a `UIButtonView?` that has no value if the type cast couldn't occur.

### Forced type cast

`control as! UIButtonView`

Force unwraps and triggers a runtime error if the type cast couldn't occur.
