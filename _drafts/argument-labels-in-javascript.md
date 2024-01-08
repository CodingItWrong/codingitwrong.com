—-
title: Argument Labels in JavaScript
—-

It’s common in modern JavaScript to use object literals to get a kind of named parameters. But object destructuring also provides us with another interesting option for designing the public interface of our functions. To understand it, though, we’re going to need to take a detour into Swift.

In JavaScript we only have positional arguments built in to the syntax:

```js
function bar(baz, qux) {
  // ...
}

bar(1, 2)
```

But it's been a pattern for a long time to use an "options" object to simulate named parameters. Here's how it was done pre-ES6:

```js
function bar(options) {
  var baz = options.baz;
  var qux = options.qux;
  // ...
}

bar({ baz: 1, qux: 2 });
```

Modern ES makes this even simpler with object destructuring:

```js
function bar({ baz, qux }) {
  // ...
}

bar({ baz: 1, qux: 2 });
```

Swift takes the interesting approach that arguments are named by default:

```swift
func bar(baz: Int, qux: Int) {
  // ...
}

bar(baz: 1, qux: 2)
```

There is still the option to make them unnamed, though, by adding a `_` in front of the parameter name:

```swift
func bar(_ baz: Int, _ qux: Int) {
  // ...
}

bar(1, 2)
```

Swift has an additional option, though. You can give the argument a different name externally than internally. This is referred to as an argument label.

```swift
func drawLine(from start: Point, to end: Point) {
  functionToLogSomething(start)
  functionToLogSomething(end)
  // ...
}

let x = Point()
let y = Point()
drawLine(from: x, to: y)
```

As in this example, argument labels are typically used in Swift to make the call site read like a sentence: "Draw line from X to Y."

Object destructuring can give us the option for a different external and internal name as well. Although we almost always see object structuring with property value shorthands, you can also give the variable a different name than the property name:

```js
function drawLine({ from: start, to: end }) {
  console.log(start);
  console.log(end);
}

let x = {};
let y = {};
drawLine({ from: x, to: y });
```

Notice a few things about this approach:

- Like in Swift, the call site reads like a sentence.
- The function declaration reads like a sentence too: "Draw line from start to end."
- You don't refer to the variables as `from` and `to` in the function implementation; you refer to them as `start` and `end`.

I think one of the reasons we don't see this pattern in JavaScript is that we love the conciseness of property value shorthands. Doing this at both the call site and the function declaration leads away from using the argument label pattern.

But this is a good option to keep in mind as we focus on the clarity of our code. 
