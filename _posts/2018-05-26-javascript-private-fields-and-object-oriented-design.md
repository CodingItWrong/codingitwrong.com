---
title: JavaScript Private Fields and Object-Oriented Design
---

[Babel beta 7.0.0-beta.49](https://github.com/babel/babel/releases/tag/v7.0.0-beta.49) has just been released, and with it [support for private fields](https://github.com/babel/babel/pull/7842). If you aren't familiar with private fields/properties, check out [Axel Rauschmayer's overview of the private fields proposal](http://2ality.com/2017/07/class-fields.html).

Having this feature available in Babel has gotten me thinking about how we can use private fields in our code and how they influence our designs. And I think the possibilities are pretty significant. To see why, let’s look at a scenario when we might want to refactor our code to private fields, and the impact it has. (Or you can just skip to the end for [the theory](#design-influences).)

## Refactoring to Private Fields

If you'd like to follow along with this coding exercise or see how to set up the Babel beta in a project, [see the sample code repo](https://github.com/CodingItWrong/private-fields).

First, make sure you have Babel 7.0.0-beta.49 or above installed:

```json
{
  ...
  "devDependencies": {
    "@babel/core": "7.0.0-beta.49",
    "@babel/plugin-proposal-class-properties": "7.0.0-beta.49",
    "@babel/plugin-proposal-object-rest-spread": "7.0.0-beta.49",
    "@babel/preset-env": "7.0.0-beta.49",
    ...
  }
}
```

Let’s say we have a simple module for loading blog posts from a web service:

```javascript
import { get } from './api';

export function getSummaries() {
  return get('blogPosts');
}

export function getBody(post) {
  return get(`blogPosts/${post.id}`)
    .then(response => {
      post.body = response.body;
    });
}
```

This module exports two functions:

- `getSummaries()` retrieves an array of objects, probably translated from JSON. The objects represent summaries of blog posts; they don’t have the body field.
- `getBody()` retrieves the body content for a given blog post. It takes a post object as an argument, and it retrieves the body and assigns it into the object.

Let’s look at the fields on a blog post object:

```json
{
  "id": 42,
  "uniqueSlug": "hello-world",
  "title": "Hello World",
  "body": "This is the body."
}
```

We have a numeric ID, a unique slug, title, and (after we request it separately) the body.

![photo of colorful slug](/img/posts/private-fields/unique-slug.jpg)

(Above: an extremely unique slug.)

Now let’s say we want to make sure not to expose the numeric ID in the user interface of our app. We don’t want users to bookmark based on that ID; we want them to use the unique slug. We can try to be careful not to use the ID in any URLs, but we might slip up. Is there a way we can set up our code to prevent ourselves and our team from using the ID field? The numeric ID is an implementation detail. Can we hide it?

We can’t just remove the ID field; `getBody()` uses the ID to retrieve the body. We want the ID available to the service but not to the rest of the app.

Private fields aren’t the only way to do this, but they do make it simple and natural. Let’s see how.

First, let’s create a BlogPost class in the same module:

```javascript
class BlogPost {
  constructor({ ...attributes }) {
    Object.assign(this, attributes);
  }
}
```

When we instantiate it, we’ll pass in the plain object returned from the API. We use `Object.assign()` to assign all the properties of that object to our new BlogPost.

Next, we can declare a private ID field and assign the ID to it instead of to a public property. We can do this with destructuring and the object rest operator:

```javascript
class BlogPost {
  #id;

  constructor({ id, ...attributes }) {
    this.#id = id;
    Object.assign(this, attributes);
  }
}
```

We extract the ID from the object and assign the rest of the attributes using `Object.assign()`. That way, all properties are assigned to BlogPost properties except for the ID; that’s assigned to a private field.

Now we won’t accidentally use the ID elsewhere in our app. But our service still needs access to that field to download the body. How can we make that ID available?

The private field is internal state of the BlogPost, so we can’t ask for it from the outside. Attempting to access `post#id` isn't even valid syntax and will result in an `Unexpected character '#'` error during transpilation. Instead, we need to allow the `BlogPost` to be responsible for what it does with its ID and who it shares it with. This means we need to add some behavior to the `BlogPost` class.

To solve this problem we can move the standalone `getBody()` function and change it into a method on `BlogPost` instead:

```javascript
class BlogPost {
  #id;

  constructor({ id, ...attributes }) {
    this.#id = id;
    Object.assign(this, attributes);
  }

  getBody() {
    return get(`blogPosts/${this.#id}`)
      .then(response => {
        this.body = response.body;
      });
  }
}
```

Because this method is defined within the `BlogPost` class, it can access the `#id` private field using `this.#id`.

With this, our app should be working, and our ID field should be protected. We can’t access it from just anywhere; the only code that can see the ID is `BlogPost` and code that `BlogPost` chooses to pass it to.

Note that private fields don’t function as a security feature here: the ID is still totally visible to the end user in requests and responses. Even apart from this, it’s best not to treat *any* data on the frontend as safe from the user. If it needs to be secret, it should stay on the server. Private fields here are purely a code structuring mechanism.

## Design Influences

Let’s think about how this use of private fields differs from typical JavaScript practice pre-private-fields, and why. This isn’t an introduction into this style of coding, or advocating that you should change to it. My goal is simply to describe how private fields provide a design force that can lead us there.

Whenever you need to add some functionality to your app, you can either add a standalone function or a method on an object. In many object-oriented languages, one of the main motivations for using methods is that they have access to private data on the object. But because JavaScript didn’t have private fields until now, this wasn’t an argument in favor of using methods.

There was, however, a strong argument in favor of using standalone functions: JSON. It’s incredibly easy to parse JSON into JavaScript objects that have data but no methods. It’s more effort to copy that data into a new instance of a class that has methods on it. I think this cost with little benefit is one reason JavaScript has moved toward separating data and logic, which puts you in the functional programming camp. This is great! There are tons of advantages to this style. But like almost everything else in programming, it’s a tradeoff.

Private fields balance the tradeoffs between styles by giving methods an advantage over standalone functions: providing private data that only methods on the object can access. This may lead toward more developers adopting an object-oriented style in which data and logic are situated together. We’ve seen this in our example code. Because our ID was a private field, external functions couldn’t access it. If we want to use the field (and if not, we might as well just not store it) the class itself needed a method to access it.

Our private field ended up influencing us toward a [“tell-don’t-ask”](https://pragprog.com/articles/tell-dont-ask) style of development. Instead of the service *asking* the blog post for its ID and then performing logic based on the response, we *tell* the blog post what we want it to do (load the body) and trust it to handle it. When your objects don’t have any behavior then all you can do is ask them for data, but when you add functionality you have the option of telling them to perform behavior as well.

It’s not like keeping data private was impossible before private fields. A few different approaches are described in the [“Private Data for classes” section](http://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes) of Axel Rauschmayer’s _Exploring ES6_ (available free online). These approaches to implementing private data include using constructor function scope, property naming conventions, symbol keys, and WeakMaps. These are all possible, but they took a little work. And since separating data and behavior is a totally valid alternative way to structure your code, there was often little incentive to jump through these hoops.

Now that private fields are a first-class language feature, storing private data is more idiomatic. It’s easy to reach for; when storing data you can simply decide if it goes into a private field instead of a public property. This is much easier than deciding it’s worth the effort to implement a more complex private data mechanism.

So I think this is one of the most significant impact private fields will make: they lower the barrier to coding in a style that encapsulates state, and so encourages joining data with behavior and a tell-don’t-ask style. This increases our options as JavaScript developers so that we can join logic to data when it’s helpful and separate them when that’s preferable.
