---
title: Simulating Private Methods with Arrow Functions
---

Private fields are now available in Babel 7.0.0-beta.49, but private methods are not yet available. But we can effectively simulate private methods by combining private fields and arrow functions. Let's look at how, why it works, and how we might take advantage of them.

## How to Simulate Private Methods

The [private methods proposal](https://github.com/tc39/proposal-private-methods) proposes a syntax for private methods that looks like this:

```javascript
class MyClass {
  #myPrivateMethod() {
    return 42;
  }

  myPublicMethod() {
    return this.#myPrivateMethod();
  }
}
```

`#myPrivateMethod()` can't be called from anywhere in your code except from within the lexical scope of `MyClass`. So you can't call `myClassInstance.#myPrivateMethod()`, but from within `myPublicMethod()` you can call `this.#myPrivateMethod()`.

So, how do arrow functions allow us to simulate private methods? You may have used class property arrow functions before:

```javascript
class MyClass {
  myValue = 42;

  myArrowFunction = () => {
    return this.myValue;
  }
}
```

This isn't a special JS syntax; it's just assigning an arrow function to a class property. Like always, the arrow function preserves the value of `this`, so that when you call it, `this` points to the object it was created within. When you realize this, you may wonder, can I assign an arrow function to a private field? And can it still access other private fields on the class? The answer is yes! And the other answer is also yes!!

```javascript
class MyClass {
  #myPrivateValue = 42;

  #myPrivateMethod = () => {
    return this.#myPrivateValue;
  }

  myPublicMethod() {
    return this.#myPrivateMethod();
  }
}
```

How do these arrow functions have access to private fields? The private fields are accessible anywhere within the lexical scope of the class they are defined on. This means you can't assign a function to a class instance later to get access to its private fields, but both methods and arrow functions declared within the body of the class have access.

Now, private field arrow functions come with all the same caveats as public (class property) arrow functions. The most significant one is that a new instance of each arrow function is created for each instance of the class, and this will increase memory usage if you're creating a lot of instances. You can read more about this and other tradeoffs in ["Arrow Functions in Class Properties Might Not Be As Great As We Think"](https://medium.com/@charpeni/arrow-functions-in-class-properties-might-not-be-as-great-as-we-think-3b3551c440b1).

## Using Private Methods

What are private field arrow functions useful for? Let's take a look at how they allow us to break up a complex method into more manageable chunks.

In [my blog post on private fields](https://codingitwrong.com/2018/05/26/javascript-private-fields-and-object-oriented-design.html) we created a `getSummary()` function that returned `BlogPost`s with summary data populated. Each `BlogPost` instance had a `getBody()` method for retrieving its body text.

Now let's add a method to `BlogPost` to add a comment to it.

```javascript
class BlogPost {
  #id;

  //...

  postComment(commentData) {
    const url = `blogPosts/${this.#id}/comments`;

    const requestPayload = {
      data: {
        type: 'comment',
        attributes: commentData,
      },
    };

    return post(url, requestPayload)
      .then(response => {
        const { id } = response.data;
        return new Comment({ id, blogPost: this, ...commentData });
      })
      .catch(error => {
        switch (error.status) {
        case 500:
          throw new Error(
            `A server error occurred when trying to comment on "${this.title}"`,
          );
        case 403:
          throw new Error(
            `Sorry, you do not have access to comment on "${this.title}".`,
          );
        }
      });
  }
}
```

This is a pretty typical method to retrieve data from an API. It goes through a few steps:

1. It creates the necessary URL and request payload.
2. It sends the request.
3. If the request succeeds, it converts the response data to a format appropriate for the rest of the app; in this case, instantiating a `Comment`.
4. If the request fails, it checks the response status and throws an appropriate application-specific Error.

This method isn't too bad to take in, but it's procedural. All the details of what to do are included. It's hard to quickly see what it's doing at a glace. MORE DETAIL ON THIS

Let's try going through a few different refactorings and see if they improve things.

First, we can simply extract a methods for each high-level step I described above:

```javascript
  postComment(commentData) {
    return post(this.url(), this.requestPayload(commentData))
      .then(response => this.handleResponse(response, commentData))
      .catch(error => this.handleError(error));
  }
```

This a lot more succinct. What's more, the method now describes what is happening at a higher level of abstraction. You can see setting up the data, making the post, and responding to two possible outcomes. (If you preferred to store the results of url() and requestPayload() to local variables, you could do that instead--the level of abstraction would be the same.)

Here are the methods we extracted that `postComment()` calls:

```javascript
  url() {
    return `blogPosts/${this.#id}/comments`;
  }

  requestPayload(commentData) {
    return {
      data: {
        type: 'comment',
        attributes: commentData,
      },
    };
  }

  handleResponse(response, commentData) {
    const { id } = response.data;
    return new Comment({ id, blogPost: this, ...commentData });
  }

  handleError(error) {
    switch (error.status) {
    case 500:
      throw new Error(
        `A server error occurred when trying to comment on "${this.title}"`,
      );
    case 403:
      throw new Error(
        `Sorry, you do not have access to comment on "${this.title}".`,
      );
    }
  }
```

What do we think about having so many methods? Well, it's a lot of logic, and it's all still in our `BlogPost` class. This is just to support one operation, and one that has only secondary relationship with `BlogPost` at best--adding a comment! This is really cluttering up our class. Where else can we move that logic?

One option is to extract these methods into standalone functions that aren't exported from the module they're in. Then `postComment()` would look like this:

```javascript
  postComment(commentData) {
    return post(url(this.#id), requestPayload(commentData))
      .then(response => handleResponse(response, commentData, this))
      .catch(error => handleError(error, this));
  }
```

In one sense this has made our `postComment()` method a bit simpler, because we no longer have to prefix these function calls with `this.`. But we also have to pass more parameters to the functions now:

- `url()` previously had no parameters, and now it has one: the ID of the blog post for which to create the URL.
- `requestPayload()` has no change, still just one parameter.
- `handleResponse()` and `handleError()` have each increased by one parameter: they both need access to `this`, the `BlogPost`. (You could also implement `handleError()` by just passing the `title` to it; the effect on complexity is about the same.)

So this approach has decreased the number of methods that are made available to the rest of the app, but the `postComment()` method has about the same complexity. Can we make `postComment()` even simpler?

The problem is that we have a fair number of parameters to pass around. There's a classic refactoring that addresses this: [Replace Method with Method Object](https://refactoring.com/catalog/replaceMethodWithMethodObject.html). Let's see how we can do that.

A method object is an object that represents a single operation you're performing. In that sense it's a lot like a function. The difference is that its implementation can be broken up into a number of private methods, and all of them have access to the private data on the object--without needing to do extensive parameter passing.

As a first step, let's take the contents of `postComment()` and extract them to a new class:

```javascript
class BlogPost {
  //...
  postComment(commentData) {
    return new CommentSender({
      blogPost: this,
      id: this.#id,
      commentData,
    }).send();
  }
}

class CommentSender {
  #blogPost;
  #id;
  #commentData;

  constructor({ blogPost, id, commentData }) {
    this.#blogPost = blogPost;
    this.#id = id;
    this.#commentData = commentData;
  }

  send() {
    return post(url(this.#id), makeRequestPayload(this.#commentData))
      .then(response => (
        handleResponse(response, this.#commentData, this.#blogPost)
      ))
      .catch(error => handleError(error, this.#blogPost));
  }
}
```

Note that we pass the parameters the method needs in the constructor, then we call a zero-argument `send()` method.

After this first step, this method object still calls our standalone functions. But now we can change these functions into private methods on the method object, and so they will no longer need any parameters passed to them; they can access that data from the private fields. Here's what our send method looks like after this:

```javascript
  send() {
    return post(this.#url(), this.#requestPayload())
      .then(this.#handleResponse)
      .catch(this.#handleError);
  }
```

Note how we don't have to write any parameters. `#url()` and `#requestPayload()` only rely on private fields, and `#handleResponse()` and `#handleError()` are passed the response and error, respectively, by `then()` and `catch()`. This makes for a much more readable method.

Here's what the private field arrow functions look like:

```javascript
  #url = () => {
    return `blogPosts/${this.#id}/comments`;
  };

  #requestPayload = () => {
    return {
      data: {
        type: 'comment',
        attributes: this.#commentData,
      },
    };
  };

  #handleResponse = (response) => {
    const { id } = response.data;
    return new Comment({
      id,
      blogPost: this.#blogPost,
      ...this.#commentData,
    });
  };

  #handleError = (error) => {
    switch (error.status) {
    case 500:
      throw new Error(
        `A server error occurred when trying to comment on "${this.#blogPost.title}"`,
      );
    case 403:
      throw new Error(
        `Sorry, you do not have access to comment on "${this.#blogPost.title}".`,
      );
    }
  };
```

Now, let's evaluate this solution. It's more lines of code, and for an example this simple it might have been easy enough for us to follow what was happening in the `postComment()` method. Think of it as a stand-in for the truly monster functions and methods you've seen.

One of the biggest benefits of our final solution is that it splits up the code into several levels of abstraction:

- At a high level, the `BlogPost.postComment()` method simply delegates to the `SendPostComment` method object; there's a nontrivial operation here that is nonessential for understanding the `BlogPost` class.
- At a medium level, `CommentSender.send()` shows us the steps at a glance: we compute a URL and a request payload, we post it, and we handle either the response or the error.
- At a low level, the private field arrow function for each of these operations shows the details of how each step works.

## More Options

I'm not attempting to argue that you should always use method objects. But method objects are one good use of private field arrow functions (and native private methods, when we have them). Before we had a private method option, I was hesitant to make small methods in JavaScript because they were all exposed on the public interface of the object, potentially allowing other parts of the code to use them in ways I didn't intend. And I was hesitant to extract small functions because of the work required to pass multiple parameters to them. As a result, I ended up writing methods that were much longer and more procedural than I'd prefer. I'm excited to see the impact that private methods have on my design.
