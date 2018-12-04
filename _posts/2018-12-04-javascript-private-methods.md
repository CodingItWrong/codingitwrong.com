---
title: JavaScript Private Methods
---

Earlier this year, when Babel support for private fields was released, I wrote a blog post showing [how to simulate private methods using private fields and arrow functions](/2018/06/04/private-arrow-functions.html). Now [in Babel 7.2.0 private method support proper has been released](https://babeljs.io/blog/2018/12/03/7.2.0), so I wanted to publish an updated version of that post demonstrating how to use private methods. (Note that there is still a place for private field arrow functions, for the same reason there is for class property arrow functions: `this` binding.) I also rename the refactoring "Replace Method with Method Object" to the name it's known by in Martin Fowler's newly-released [_Refactoring, 2nd Edition_](http://www.informit.com/store/refactoring-improving-the-design-of-existing-code-9780134757599?ranMID=24808): "Replace Function with Command".

***

What are private methods useful for? Let's take a look at how they allow us to break up a complex method into more manageable chunks.

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

This method isn't too bad to take in, but it's procedural. All the details of what to do are included. It's hard to quickly see what it's doing at a glace.

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

The problem is that we have a fair number of parameters to pass around. There's a classic refactoring that addresses this: [Replace Function with Command](https://refactoring.com/catalog/replaceFunctionWithCommand.html). Let's see how we can do that.

In this context, a "command" is an object that represents a single operation you're performing. In that sense it's a lot like a function. The difference is that its implementation can be broken up into a number of private methods, and all of them have access to the private data on the object--without needing to do extensive parameter passing.

As a first step, let's take the contents of `postComment()` and extract them to a new class:

```javascript
class BlogPost {
  //...
  postComment(commentData) {
    return new CommentSender({
      blogPost: this,
      id: this.#id,
      commentData,
    }).execute();
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

  execute() {
    return post(url(this.#id), makeRequestPayload(this.#commentData))
      .then(response => (
        handleResponse(response, this.#commentData, this.#blogPost)
      ))
      .catch(error => handleError(error, this.#blogPost));
  }
}
```

Note that we pass the parameters the method needs in the constructor, then we call a zero-argument `execute()` method.

After this first step, this command object still calls our standalone functions. But now we can change these functions into private methods on the command object, and so they will no longer need any parameters passed to them; they can access that data from the private fields. Here's what our `execute()` method looks like after this:

```javascript
  execute() {
    return post(this.#url(), this.#requestPayload())
      .then(response => this.#handleResponse(response))
      .catch(error => this.#handleError(error));
  }
```

Note how we don't have to write any parameters for `#url()` and `#requestPayload()`, because they only rely on private fields. For the `then` and `catch` callbacks we _do_ need to write out an arrow function, because private methods (like normal methods) don't have their `this` context bound. (But we do have an alternative. Just like you can change methods to class property arrow functions to bind the `this` context, you can change private methods to private field arrow functions. You can see this approach in my older [blog post on private field arrow functions](/2018/06/04/private-arrow-functions.html).)

Here's what the private methods look like:

```javascript
  #url() {
    return `blogPosts/${this.#id}/comments`;
  };

  #requestPayload() {
    return {
      data: {
        type: 'comment',
        attributes: this.#commentData,
      },
    };
  };

  #handleResponse(response) {
    const { id } = response.data;
    return new Comment({
      id,
      blogPost: this.#blogPost,
      ...this.#commentData,
    });
  };

  #handleError(error) {
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

- At a high level, the `BlogPost.postComment()` method simply delegates to the `CommentSender` command object; there's a nontrivial operation here that is nonessential for understanding the `BlogPost` class.
- At a medium level, `CommentSender.execute()` shows us the steps at a glance: we compute a URL and a request payload, we post it, and we handle either the response or the error.
- At a low level, the private methods for each of these operations shows the details of how each step works.

## More Options

I'm not attempting to argue that you should always use command objects. In fact, in _Refactoring, 2nd Edition_ Martin Fowler states that he will use a standalone function rather than a command 95% of the time. But command objects are one good illustration of the usefulness of private methods.

Before we had a private method option, I was hesitant to make small methods in JavaScript because they were all exposed on the public interface of the object, potentially allowing other parts of the code to use them in ways I didn't intend. And I was hesitant to extract small functions because of the work required to pass multiple parameters to them. As a result, I ended up writing methods that were much longer and more procedural than I'd prefer. I'm excited to see the impact that private methods have on my design.
