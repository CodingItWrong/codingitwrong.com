---
title: Using ES Class Properties for Function Binding
---

There's an ES proposal for defining class properties, which you can use with the [Babel `transform-class-properties` plugin][transform-class-properties]. Here's how it works:

```javascript
class MyClass {
  myProperty = 27
}

const myInstance = new MyClass()
myInstance.myProperty // -> 27
```

You can also use it to assign arrow functions to properties:

```javascript
class MyClass {
  myFunction = () => 27
}

const myInstance = new MyClass()
myInstance.myFunction() // -> 27
```

Why would you do this instead of defining methods in the typical way? Well, because of the way arrow functions work, they end up bound to the instance they're on, and this can be pretty useful.

To show how this works, let's demonstrate the problem with the traditional way to define methods on classes:

```javascript
class MyClass {
  constructor(value) {
    this.value = value
  }

  myMethod() {
    return this.value
  }
}

const myInstance = new MyClass(27)
myInstance.myMethod() // -> 27
```

Calling this method on the instance works, but it doesn't work if you assign the function to a variable:

```javascript
const myFunction = myInstance.myMethod
myFunction() // -> TypeError: Cannot read property 'value' of undefined
```

This is because in JavaScript the value of `this` is [assigned at the time you call a function][this]. Assigning the function divorces it from the object it was on, as though it had been defined like this:

```javascript
const myFunction = function() {
  return this.value
}
myFunction() // -> TypeError: Cannot read property 'value' of undefined
```

Obviously the above wouldn't work.

You may wonder when this problem would come up in practice. React!

```jsx
class MyComponent extends React.Component {
  showValue() {
    console.log(this.props.value)
  }

  render() {
    return (
      <button onClick={this.showValue}>Do Something</button>
    )
  }
}
```

If you click on the button above, you get an error, because of the same reason as the previous examples: when you reference `this.showValue` it divorces that function from the instance it was defined on, so it doesn't have access to `this`.

One way around this is to use arrow functions to close over the value of `this`:

```jsx
class MyComponent extends React.Component {
  showValue() {
    console.log(this.props.value)
  }

  render() {
    return (
      <button onClick={() => this.showValue()}>Do Something</button>
    )
  }
}
```

Note that we are never assigning the `this.showValue` function anywhere; we are always calling it in the context of the `this` that is this instance.

Another approach is to use [Function.prototype.bind()][bind]:

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props)

    this.showValue = this.showValue.bind(this)
  }

  showValue() {
    console.log(this.props.value)
  }

  render() {
    return (
      <button onClick={this.showValue}>Do Something</button>
    )
  }
}
```

This overwrites `this.showValue` with a version that has the value of `this` bound. I don't like this approach, though, because it makes methods on the object behave in a way that's different from what's expected. If you forget to bind a function, the errors you get won't make it immediately obvious what's going on.

But if you use the new ES class property syntax for arrow functions, it just works:

```jsx
class MyComponent extends React.Component {
  showValue = () => {
    console.log(this.props.value)
  }

  render() {
    return (
      <button onClick={this.showValue}>Do Something</button>
    )
  }
}
```

Why does it work? It's because arrow functions close over the value of `this`; they don't allow it to be changed at runtime. So we can pass the bare function to React, and no matter what context it's called in, `this` will always refer to the component instance.

Incidentally, this doesn't work if you assign a traditional function using ES class property syntax:

```jsx
class MyComponent extends React.Component {
  showValue = function() {
    console.log(this.props.value)
  }

  render() {
    return (
      <button onClick={this.showValue}>Do Something</button>
    )
  }
}
```

It won't work because traditional functions don't close over the value of `this`.

I like using class property arrow functions for React actions better than either of the other approaches. With manually binding functions, the binding is done in at a place different from _both_ where the method is defined and where it's called, so it's easy to miss. Using an arrow function at the point of use is better because at least you're likely to see it, but it does clutter up your React components. Class property arrow functions allow you to set up the binding at the point the function is defined.

When I define a method on an object, I don't typically want that function to be callable in a different context. Because class property arrow functions ensure that that function will _always_ be called in the context of its instance, that gives me more predictable behavior.

If you'd like to see this in action, I've made [a demo repo][demo].

[bind]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
[demo]: https://github.com/CodingItWrong/es-class-properties
[this]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
[transform-class-properties]: https://babeljs.io/docs/plugins/transform-class-properties
