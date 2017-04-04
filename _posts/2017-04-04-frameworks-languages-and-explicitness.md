---
title: Frameworks, Languages, and Explicitness
---

We often evaluate different web frameworks in terms of how explicit or implicit they are. People who prefer implicitness call explicitness "boilerplate", and people who prefer explicitness call implicitness "magic". Either way, the focus is on evaluating the framework. But I think the programming language has a lot more influence on the explicitness of a framework than we give it credit. Certain kinds of implicitness aren't even possible in some languages. And the trends of a language towards explicitness or implicitness will tend to rub off on the framework.

With that in mind, let's compare some simple controllers from three web frameworks. We'll look at:

- Rails, possibly the ultimate in implicit frameworks thanks in part to Ruby's metaprogramming features.
- Phoenix, a functional web framework that "favors explicitness in most of its stack", [according to its creator](https://dockyard.com/blog/2015/11/18/phoenix-is-not-rails).
- Laravel, a PHP framework inspired by both Rails and C#, with an appropriate mix of explicitness and implicitness.

# Ruby and Rails

In each of these examples we'll look at a tiny controller that is set up to display a "new post" form and save it to the database. Here's the Rails implementation:

```ruby
class PostsController < ApplicationController
  def new
    @post = Post.new
  end

  def create
    @post = Post.new(post_params)

    if @post.save
      redirect_to @post
    else
      render :new
    end
  end

  private

  def post_params
    params.require(:post).permit(:title, :body)
  end
end
```

Rails is certainly at the extreme of implicitness, relying on some of Ruby's features to reach that extreme:

- There are no `import` or `require` or `use` statements: the controller can reference the `Post` model directly. In Ruby, any constant that is loaded can be referenced directly. And Rails' autoloader handles loading constants as they're requested.
- The `create` method has a `render` call to render a view, but the `new` method has no such call. If no `render` or redirect is done, Rails implicitly renders a view corresponding to the controller and method; in this case, `views/posts/new.html.erb`.
- In order to have implicit view rendering, Rails also needs to pass variables to the view implicitly. Any instance variables on the controller are passed to the view, so just assigning `Post.new` to `@post` is enough to make the post available in the view. Even when `render` is called explicitly, as in the `create` method, no data is passed to it explicitly.
- The decision about what to do if saving a new post succeeds or fails is actually explicit: the code states that if it succeeds the user should be redirected to the new post, and if it fails the `new` view should be rendered again.
- Redirecting to a model's page is implicit. You don't need to pass the full URL to the `redirect_to` method, or even call a path helper. If a model is passed to `redirect_to`, Rails implicitly looks up the `show` page for that resource.
- Method calls to `self` don't need `self` to explicitly be provided: note that `post_params` is called instead of `self.post_params`. Because parens aren't required on method calls, this means that method calls on `self` without parameters are syntactically identical to references to local variables. This is great for refactoring, but the implicitness can make the code a bit harder to follow upon first read.

# PHP and Laravel

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Post;

class PostsController extends Controller
{
    function create() {
        return view('posts.create', ['post' => new Post()]);
    }

    function store(Request $request) {
        $this->validate($request, [
            'title' => 'required',
            'body' => 'required',
        ]);

        $post = Post::create($request->only(['title', 'body']));
        return redirect()->route('posts.show', $post->id);
    }
}
```

PHP is a more typical OO language, with a good amount of explicitness required by the language.

- The class being declared is explicitly defined to belong to a namespace (`App\Http\Controllers`), and necessary classes from other namespaces are explicitly `use`d (e.g. the framework `Illuminate\Http\Request` class, and even the app's own `App\Post` model).
- Method calls on `$this` must be explicitly prefixed with `$this->`, otherwise the call is to a global function instead. But note that Laravel defines `view()` and `redirect()` global functions for convenience, because they're called so often.
- Rendering a view is explicit: the `view()` function is called. The name of the view is explicitly passed, and data is explicitly passed to it.
- Validation can be handled a few different ways, but the simplest facility is to call the `$this->validate()` controller method. In surprising implicitness, the behavior of the app when validation fails is implicit: it rerenders the form you came from, passing it the submitted data and the validation errors.
- Redirecting takes a route ID string (`'posts.show'`) rather than a fully-written-out path string.

## Elixir and Phoenix

```elixir
defmodule CrudTestPhoenix.PostController do
  use CrudTestPhoenix.Web, :controller

  alias CrudTestPhoenix.Post

  def new(conn, _params) do
    changeset = Post.changeset(%Post{})
    render(conn, "new.html", changeset: changeset)
  end

  def create(conn, %{"post" => post_params}) do
    changeset = Post.changeset(%Post{}, post_params)

    case Repo.insert(changeset) do
      {:ok, post} ->
        conn
        |> redirect(to: post_path(conn, :show, post.id))
      {:error, changeset} ->
        render(conn, "new.html", changeset: changeset)
    end
  end
end
```

As a functional language, Elixir requires some things to be more explicit than an OO language would, and Phoenix specifically takes explicitness as a design goal.

- Accessing the `Post` model from within the controller simply requires using the `alias` method, similar to PHP's `use`.
- In Elixir, a struct for data storage is closely related to its module, which can contain behavior, but the two aren't synonymous. Therefore, when we call the `changeset` function in the `Post` module, we also need to pass in a new `%Post` struct to it explicitly.
- Data is explicitly passed when rendering the view.
- The `Repo` for handling storage of models is separate from the model structs and functions. This makes the distinction between in-memory representation, manipulation, and persistence of the model explicit.
- Handling of the success and error cases for inserting a model into the database are explicitly spelled out, using the common Elixir pattern of returning a tuple with either `:ok` or `:error`.
- The `redirect()` function must be passed the path of the page to redirect to, not just the model itself. But a `post_path` helper is available so that you don't have to explicitly write out the path to the model.
- The `conn` struct is explicitly passed around a lot. This is a core part of Phoenix's design for explicitness: methods gradually transform the `conn` from one that holds the request to one that holds the response. Because Elixir is functional, there is no `self` or `$this` to reference for data; the `conn` is what core framework functions are called with instead.

# So what?

When you're picking a framework to learn or use for a project, evaluate it in terms of explicitness vs. implicitness, and see where you land on the spectrum. Think about the language's features for explicitness or implicitness too. The longer you spend in web application development, the more your focus will tend to migrate from framework specifics to usage of the language. It's worth considering if the facilities of that language are what you want to be working in.
