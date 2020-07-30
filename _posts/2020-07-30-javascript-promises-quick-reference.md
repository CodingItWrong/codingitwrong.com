---
title: JavaScript Promises Quick Reference
---

Here's a quick reference about how to use Promises in JavaScript.

Say we have a set of functions that make calls to a web service and return the data via Promises.

```js
import { getRestaurant, getSpecialDish } from './api';
```

When we call one of the functions, the return value is a Promise object.

```js
const promise = getRestaurant();
console.log(promise); // Promise { <pending> }
```

We can't get access to the web service response data right away, because that happens synchronously, and the web service responds asynchronously.

## Handling Results

To get access to its value when it *is* available, we call the Promise's `.then()` method and pass a callback function to it. The callback function receives the value as its argument. This is called the Promise "resolving", and the value is called the "resolved value."

```js
getRestaurant().then(restaurant => {
  console.log(restaurant); // Sushi Place
});
```

The `.then()` method returns another Promise.

```js
const promise = getRestaurant().then(() => {
  /*...*/
});
console.log(promise); // Promise { <pending> }
```

If you return a value from a `.then()` callback, its returned Promise resolves to the returned value. This allows you to chain multiple `.then()` callbacks so you can transform values and pass them along.

```js
getRestaurant()
  .then(restaurant => {
    return `The Best ${restaurant}`;
  })
  .then(editedRestaurant => {
    console.log(editedRestaurant); // The Best Sushi Place
  });
```

If within a `.then()` callback you return another Promise that resolves, the outer Promise resolves to the inner Promise's resolved value.

```js
getRestaurant()
  .then(() => {
    return Promise.resolve('Pizza Place');
  })
  .then(resolvedValue => {
    console.log(resolvedValue); // Pizza Place
  });
```

Returning a Promise you make yourself isn't that useful, because, as we saw previously, you can just return the value directly. What *is* useful is calling another function that returns a Promise, and passing that Promise along.

```js
getRestaurant()
  .then(restaurant => {
    return getSpecialDish(restaurant);
  })
  .then(dish => {
    console.log(dish); // Sushi Place Special Roll
  });
```

You have to make sure to return that Promise as the return value of the function, though. Just calling the function doesn't work, because its return value is lost.

```js
getRestaurant()
  .then(restaurant => {
    getSpecialDish(restaurant);
  })
  .then(dish => {
    console.log(dish); // undefined
  });
```

If you need to get access to the resolved values of two Promises you call in sequence, you can chain a `.then()` on the second promise inside the first Promise's `.then()`. That way both resolved values are in scope.

```js
getRestaurant()
  .then(restaurant => {
    return getSpecialDish(restaurant).then(dish => {
      return { restaurant, dish };
    });
  })
  .then(resolvedValue => {
    console.log(resolvedValue); // { restaurant: 'Sushi Place',
                                // dish: 'Sushi Place Special Roll' }
  });
```

## Handling Errors

When an error occurs, instead of resolving, a Promise is said to "reject". You can handle that error using a `.catch()` callback function. When an error occurs, any `.then()` callbacks are skipped.

```js
getRestaurant()
  .then(restaurant => {
    console.log(restaurant); // (not called)
  })
  .catch(error => {
    console.error(error); // Something went wrong
  });
```

Like `.then()`, `.catch()` returns a new Promise. If an error is caught, the Promise returned by `.catch()` resolves; it doesn't reject. This may be surprising.

```js
getRestaurant()
  .catch(error => {
    /* ... */
  })
  .then(resolvedValue => {
    console.log(resolvedValue); // undefined
    // Wait, why did this "succeed"?
  });
```

One reason Promises work this way is so that you can handle an error and pass along a value to "get things working again". You can just return a value you want the Promise to resolve to. As we saw before, returning Promise.resolve() is unnecessary; you can return the value directly.

```js
getRestaurant()
  .catch(error => {
    // an error occurred; let's use a default value
    return 'a default value';
  })
  .then(resolvedValue => {
    console.log(resolvedValue); // a default value
  });
```

If you want to do something in response to an error but then pass it along, you can return a rejected Promise. This will make the outer Promise reject, so you can chain another `.catch()` call. This is useful if another part of your app also needs to know about the error.

```js
getRestaurant()
  .catch(error => {
    console.error(error); // Something went wrong
    return Promise.reject(error);
  })
  .catch(error => {
    console.error(error); // Something went wrong
  });
```

Alternatively, you can use the `throw` keyword to throw the error.

```js
getRestaurant()
  .catch(error => {
    console.error(error); // Something went wrong
    throw error;
  })
  .catch(error => {
    console.error(error); // Something went wrong
  });
```

Throwing an error also works inside a `.then()` callback:

```js
getRestaurant()
  .then(() => {
    throw 'actually an error';
  })
  .catch(error => {
    console.error(error); // actually an error
  });
```

So the Promise returned by both `.then()` and `.catch()` works the same: they both return a Promise that resolves by default, and they only reject if you `throw` or return a rejected Promise.

## Learn More

To dive in deeper into how Promises work, check out [the Promises chapter of Dr. Axel Rauschmayer's _JavaScript for Impatient Programmers_](https://exploringjs.com/impatient-js/ch_promises.html).
