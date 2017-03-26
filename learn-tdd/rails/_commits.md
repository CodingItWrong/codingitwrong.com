### Specify the feature for creating a blog post [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/576c89e3ed6ad2d45f2ca78797b42518c5231a4e)

#### spec/features/creating_a_blog_post_spec.rb

```diff
{% raw %}+require 'rails_helper'
+
+describe 'Creating a blog post' do
+
+  it 'saves and displays the resulting blog post' do
+    visit '/blog_posts/new'
+
+    fill_in 'Title', with: 'Hello, World!'
+    fill_in 'Body', with: 'Hello, I say!'
+
+    click_on 'Create Blog Post'
+
+    expect(page).to have_content('Hello, World!')
+    expect(page).to have_content('Hello, I say!')
+
+    blog_post = BlogPost.order("id").last
+    expect(blog_post.title).to eq('Hello, World!')
+    expect(blog_post.body).to eq('Hello, I say!')
+  end
+
+end{% endraw %}
```

We start by writing out an acceptance test for the full feature we want to implement. In this case, we want to visit a blog post creation page, enter a title and body, save it, and then see on the results page that title and body, as well as confirming it’s in the database.

Red: No route matches [GET] "/blog_posts/new"

The first error we get is that there is no blog-posts/new route.


### Add blog posts resource route [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/c1e5ccaad0ceb97ee01c9902f854e7e7d7f1dd6e)

#### config/routes.rb

```diff
{% raw %} Rails.application.routes.draw do
+  resources :blog_posts
 end{% endraw %}
```

We add the route, but we don’t just write the simplest code possible to get the test to pass; we “write the code we wish we had.” In this case, we wish we had a resourceful blog posts controller, so we create a resource route, which corresponds to a resourceful controller of the same name..

Rails allows you to "unit test" routes, but for trivial configuration like this, it's fine to let the acceptance test cover it without stepping down to the unit level.

Red: uninitialized constant BlogPostsController

The next error we get is that that controller doesn’t exist.


### Add empty blog posts controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/d6d87dc7a72495c975cd5fa7e225e181d93f2c78)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %}+class BlogPostsController < ApplicationController
+end{% endraw %}
```

We add an empty controller that inherits from our app’s base controller class. We could have gotten past this error message by creating a class that didn’t inherit from anything, but in this case we’re so sure we’ll inherit from the base controller class that we can go ahead and do it.

Red: The action 'new' could not be found for BlogPostsController

The acceptance test can now find the controller, but not a "new" action on it.


### Add `new` action to blog posts controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/b75596026d5428b42449dcf14496a9c98dc2f67f)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %} class BlogPostsController < ApplicationController
+  def new
+  end
 end{% endraw %}
```

Again, we only add enough code to get the test to pass.

Red: BlogPostsController#new is missing a template for this request format and variant.

       request.formats: ["text/html"]
       request.variant: []

Even though we didn't ask to render a template, Rails' default behavior for a controller action is to render a corresponding template, so that's the error it's running across next.


### Add new blog post template [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/5e25a7a4ac23aba095135e131d6dd689f203e562)

#### app/views/blog_posts/new.html.erb

```diff
{% raw %}+<%# empty view %>{% endraw %}
```

Red: Unable to find field "Title"

The acceptance test is finally able to successfully `visit '/blog_posts/new'` and move on to attempt the next step, which is `fill_in 'Title', with: 'Hello, World!'`.


### Add fields and submit button to form template [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/b0cd909133a46b0943a316256b8e263e042a431a)

#### app/views/blog_posts/new.html.erb

```diff
{% raw %}-<%# empty view %>
+<%= form_for @blog_post do |f| %>
+  <div>
+    <%= f.label :title %>
+    <%= f.text_field :title %>
+  </div>
+  <div>
+    <%= f.label :body %>
+    <%= f.text_area :body %>
+  </div>
+  <%= f.submit 'Create Blog Post' %>
+<% end %>{% endraw %}
```

This is another case where all we needed to add to get the test to pass was the title field. But this is a reasonable case where you know what the form will consist of, so you can go ahead and create it all. Plus, your tests will never drive out the full markup of your templates, so you'll need to do that by hand anyway.

Red: First argument in form cannot contain nil or be empty

<%= form_for @blog_post do |f| %>

In other words, @blog_post is nil but needs to be a model.


### Assign blog post in controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/c835432f499421d3cb67a58adbbac7ca3839789b)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %} class BlogPostsController < ApplicationController
   def new
+    @blog_post = BlogPost.new
   end
 end{% endraw %}
```

We attempt to make the acceptance test pass by assigning the @blog_post instance variable in the controller. Note that we don't create a unit test for the controller. Many developers discourage controller tests because controller behavior is so closely related to acceptance test cases. Also, Rails 5 removes some controller test features that result in limiting how effective they can be.

Red: uninitialized constant BlogPostsController::BlogPost

The line of code we added isn't actually able to succeed because there is no class named BlogPost defined yet.


### Add blog post model [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/722db6c7d436b79158b4b2a3217d322e82474967)

#### app/models/blog_post.rb

```diff
{% raw %}+class BlogPost < ApplicationRecord
+end{% endraw %}
```

We add the BlogPost model class. Although we don't strictly need to subclass `ApplicationRecord` to get the acceptance test error to pass, we're sure enough that this will be an Active Record model that it's safe for us to go ahead and subclass it.

Note that we will soon step down from our acceptance test to create a model test, but we don't do it just yet. We write model tests and other lower-level tests to specify behavior, but we haven't reached a behavioral error yet. This is just a structural error: there is no `BlogPost` class. That's simple enough that we can just implement it directly.

Red: PG::UndefinedTable: ERROR:  relation "blog_posts" does not exist

Because we subclass `ApplicationRecord`, our call to `BlogPost.new` checks for the existence of a `blog_posts` table (a "relation"), and doesn't find one.


### Specify model should be instantiable [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/e4c4299fac4ee55e8346f6ea42dfd0b9de121a7d)

#### spec/models/blog_post_spec.rb

```diff
{% raw %}+require 'rails_helper'
+
+describe BlogPost do
+
+  it "is instantiable" do
+    expect{ blog_post = BlogPost.new }.not_to raise_error
+  end
+
+end{% endraw %}
```

The previous error wasn't enough for us to want to create a `BlogPost` model test, because it was just a structural error: there was no `BlogPost` class. But now we have an error that a `BlogPost` can't be instantiated, which is a logic error: and that means we should create a model test. It's not strictly a unit test because it's not isolated from the database, but it _is_ following the outside-in-testing principle of stepping down from the acceptance test to the test of an individual class.

We reproduce the error that occurred in the acceptance test in the model test: the error of being unable to find the `blog_posts` table when creating a new instance.

Inner red: PG::UndefinedTable: ERROR:  relation "blog_posts" does not exist


### Create blog posts table [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/f8a8071136fca67af681e691d76c1b252bd89a9a)

#### db/migrate/20160614162712_create_blog_posts.rb

```diff
{% raw %}+class CreateBlogPosts < ActiveRecord::Migration
+  def change
+    create_table :blog_posts do |t|
+    end
+  end
+end{% endraw %}
```

We fix the model test error by creating the `blog_posts` table.

Inner green; outer red: undefined method `title' for #<BlogPost id: nil>

Now the `BlogPost` can be instantiated, so the model test passes. We step back up to the acceptance test level and rerun it, and it's gotten past that error.

The next acceptance test error occurs when Rails attempts to render the form. The form helper tries to get the current value of the `title` field on the model, but there is no `title` method to retrieve it. Since we want `title` to be a database-persisted field on our model, the problem is that there is no `title` column in our database table.


### Specify blog post accessors [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/2b9d59d639068f973e9b84bc41df8025e40f6aab)

#### spec/models/blog_post_spec.rb

```diff
{% raw %}     expect{ blog_post = BlogPost.new }.not_to raise_error
   end
 
+  it "defaults fields to nil" do
+    blog_post = BlogPost.new
+
+    expect(blog_post.title).to be_nil
+    expect(blog_post.body).to be_nil
+  end
+
 end{% endraw %}
```

Once again, rather than fixing the acceptance error directly, we drop down to the model test level to reproduce the error. We specify that we want a `title` field--as well as a `body` field, since we know we'll need that and it feels safe to go ahead and add it. The behavior we want is that we want those fields to be `nil` for a new model instance.

Inner red: undefined method `title' for #<BlogPost id: nil>

With this specification, we've reproduced the error from the acceptance test.


### Add columns to blog posts [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/45bc97643fb879161c4c43bbe7e3d26905a4e51f)

#### db/migrate/20160614163119_add_title_and_body_to_blog_posts.rb

```diff
{% raw %}+class AddTitleAndBodyToBlogPosts < ActiveRecord::Migration
+  def change
+    add_column :blog_posts, :title, :string, null: false
+    add_column :blog_posts, :body, :text, null: false
+  end
+end{% endraw %}
```

We create a new migration to add the title and body column to the blog posts table.

Inner green; outer red: The action 'create' could not be found for BlogPostsController

With this, the model test passes, so we step back up to the acceptance test level and rerun it. The acceptance test moves on to the next error: the form is rendered, the fields are populated, it submits the form, but Rails doesn't find a `create` action on the controller to submit it to.


### Add create method to blog post controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/a540e7ad1b4068e5e39918ef914cc1eacc652a84)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %}   def new
     @blog_post = BlogPost.new
   end
+
+  def create
+  end
 end{% endraw %}
```

We add the `create` method to the blog post controller directly, then rerun the acceptance test.

Outer red: Unable to find xpath "/html"

The error we get is a bit obscure, but effectively it means the acceptance test can't even find an HTML document returned to search within. Unlike with the `new` action, with the `create` action, if no explicit render or redirect happens, Rails 5 does not return an error that a template is missing; it just returns empty content.


### Add blog post create template [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/17d38da84c53f18227cb07c8f54eac5454d71158)

#### app/views/blog_posts/create.html.erb

```diff
{% raw %}+<%# empty view %>{% endraw %}
```

Usually a `create` action would redirect to another route instead of rendering a template directly. But for the sake of keeping this tutorial simple, we'll just go ahead and render it--so we add a `create` template file. We leave it empty since that's all we need to do to get past the current error.

Outer red: expected to find text "Hello, World!" in ""

The next error is that, after submitting the blog post, the acceptance test expects to see the title of the post somewhere on the page, but it doesn't see it--because we haven't actually rendered any content at all.


### Render blog post on create page [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/4371bf4cd41e1cbf8780932991ee0f12c549c311)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %}   end
 
   def create
+    @blog_post = BlogPost.new(params[:blog_post])
   end
 end{% endraw %}
```


#### app/views/blog_posts/create.html.erb

```diff
{% raw %}-<%# empty view %>
+<h1><%= @blog_post.title %></h1>
+
+<div>
+  <%= @blog_post.body %>
+</div>{% endraw %}
```

To get the blog post content rendered in the view, we attempt to create a new BlogPost instance with the passed-in form params.

We don't yet attempt to save it to the database because the acceptance test hasn't led us there yet. All it's said is that we need an object on the `create` page that responds to a `title` method (and a `body` method). That doesn't require us to save anything to the database: all that requires is us instantiating an object.

Outer red: ActiveModel::ForbiddenAttributesError

Attempting to instantiate a BlogPost leads to a new error: Rails' "strong parameters" security feature means that we can't just pass user-submitted params directly into a model; that could result in users hacking our system by setting fields they shouldn't be able to, like the user a post belongs to.


### Switch to strong params for blog post creation [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/262a8023a0db9629e7d98708b54d3c6a55f584ae)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %}   end
 
   def create
-    @blog_post = BlogPost.new(params[:blog_post])
+    @blog_post = BlogPost.new(blog_post_params)
+  end
+
+  private
+
+  def blog_post_params
+    params.require(:blog_post).permit(:title, :body)
   end
 end{% endraw %}
```

We fix the error by using the conventional Rails strong parameters approach of creating a private controller method that specifies which parameters are permitted. Because this is conventionally done in the controller, we don't need to step down to any kind of unit test--letting the acceptance test drive this code is fine.

Outer red: undefined method `title' for nil:NilClass

The next error isn't quite as readable as most we've gotten. What's going on is that the acceptance test attempts to load the last-saved `BlogPost` from the database, but none are saved, so we get `nil` back instead. When we try to check the `title` field, we get an error that there is no `title` method on `nil`. So the actual error is that no blog post is found in the database.


### Save blog post to database [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/rails/commit/f68a15ace5b603027d897778f13a59b81e8c47f3)

#### app/controllers/blog_posts_controller.rb

```diff
{% raw %}   end
 
   def create
-    @blog_post = BlogPost.new(blog_post_params)
+    @blog_post = BlogPost.create(blog_post_params)
   end
 
   private{% endraw %}
```

We fix this error by `create`ing a `BlogPost` in the database instead of just instantiating one in memory.

Outer green

With thise, our acceptance test is passing. We've successfully allowed the acceptance test to drive us through implementing a complete feature!

