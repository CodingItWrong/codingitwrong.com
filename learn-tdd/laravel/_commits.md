### Specify the feature for creating a blog post [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/e3049aee40d7533c8ee29e1c37838273dc3892cc)

#### tests/features/CreatingABlogPostTest.php

```diff
{% raw %}+<?php
+
+use Illuminate\Foundation\Testing\DatabaseMigrations;
+use App\BlogPost;
+
+class CreatingABlogPostTest extends BrowserKitTestCase
+{
+  use DatabaseMigrations;
+
+  /**
+   * A basic functional test example.
+   *
+   * @return void
+   */
+  public function testCreatingABlogPost()
+  {
+    $this->visit('/blog-posts/create')
+         ->type('Hello, World!', 'title')
+         ->type('Hello, I say!', 'body')
+         ->press('Create Blog Post')
+         ->see('Hello, World!')
+         ->see('Hello, I say!');
+
+    $post = BlogPost::first();
+    $this->assertEquals($post->title, 'Hello, World!');
+    $this->assertEquals($post->body, 'Hello, I say!');
+  }
+}{% endraw %}
```

We start by writing out an acceptance test for the full feature we want to implement. In this case, we want to visit a blog post creation page, enter a title and body, save it, and then see on the results page that title and body, as well as confirming it's in the database.

Red: A request to [http://localhost/blog-posts/create] failed. Received status code [404].

The first error we get is that there is no `blog-posts/create` route.


### Add blog posts resource route [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/dec37c0b09e2d76d921f894ab1387e5a6b17e38e)

#### routes/web.php

```diff
{% raw %} Route::get('/', function () {
     return view('welcome');
 });
+
+Route::resource('blog-posts', 'BlogPostsController');{% endraw %}
```

We add the route, but we don't just write the simplest code possible to get the test to pass; we "write the code we wish we had." In this case, we wish we had a resourceful blog posts controller, so we create a resource route and point to that controller by name.

Red: Class App\Http\Controllers\BlogPostsController does not exist

The next error we get is that that controller doesn't exist.


### Add empty blog posts controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/d81eebf7103e7422fea91047872835d0d110a743)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %}+<?php
+
+namespace App\Http\Controllers;
+
+class BlogPostsController extends Controller {
+  
+}{% endraw %}
```

We add an empty controller that inherits from our app's base controller class. We could have gotten past this error message by creating a class that didn't inherit from anything, but in this case we're so sure we'll inherit from the base controller class that we can go ahead and do it.

Red: Method App\Http\Controllers\BlogPostsController::create() does not exist

The acceptance test can now find the controller, but not a "create" action on it.


### Add `create` action to blog posts controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/c140525fdc113a48940b5b645f894569747be816)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %} namespace App\Http\Controllers;
 
 class BlogPostsController extends Controller {
-  
+
+  function create() {
+    
+  }
+
 }{% endraw %}
```

Red: Nothing matched the filter [Title] CSS query provided

Laravel is now able to render the create page, but when the test looks for a "Title" field to fill in, it can't find one, because we haven't rendered any output yet.


### Add form template with fields and submit button [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/3a9577e159f7c70b6ae805fa64877b4897204a21)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %} class BlogPostsController extends Controller {
 
   function create() {
-    
+    return view('blog-posts.create');
   }
 
 }{% endraw %}
```


#### composer.json

```diff
{% raw %}     "type": "project",
     "require": {
         "php": ">=5.6.4",
-        "laravel/framework": "5.4.*"
+        "laravel/framework": "5.4.*",
+        "laravelcollective/html": "5.4.*"
     },
     "require-dev": {
         "fzaninotto/faker": "~1.4",{% endraw %}
```


#### config/app.php

```diff
{% raw %}         App\Providers\EventServiceProvider::class,
         App\Providers\RouteServiceProvider::class,
 
+       /*
+        * Third party service providers
+        */
+       Collective\Html\HtmlServiceProvider::class,
     ],
 
     /*
...
         'Validator' => Illuminate\Support\Facades\Validator::class,
         'View' => Illuminate\Support\Facades\View::class,
 
+        /*
+         * Third party aliases
+         */
+        'Form' => Collective\Html\FormFacade::class,
+        'Html' => Collective\Html\HtmlFacade::class,
     ],
 
 ];{% endraw %}
```


#### resources/views/blog-posts/create.blade.php

```diff
{% raw %}+{!! Form::model($post, ['route' => 'blog-posts.store']) !!}
+    <div>
+      {!! Form::label('title') !!}
+      {!! Form::text('title') !!}
+    </div>
+    <div>
+      {!! Form::label('body') !!}
+      {!! Form::textarea('body') !!}
+    </div>
+    {!! Form::submit('Create Blog Post') !!}
+{!! Form::close() !!}{% endraw %}
```

Instead of just adding the title field, we go ahead and add the entire form, including the body field and the submit button. This is more than is strictly necessary to get the current error to pass, but it seems reasonable. You'll never have acceptance tests drive out every detail of your template markup anyway. We also go ahead and use Laravel Collective's form helpers instead of creating the markup by hand.

Red: Undefined variable: post

The next error we get is that the `$post` variable we attempt to pass into the form doesn't exist.


### Add blog post variable assignment in controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/61185ce6f37227941dd142eccc677808b5ab8cc4)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %} <?php
 
 namespace App\Http\Controllers;
+use App\BlogPost;
 
 class BlogPostsController extends Controller {
 
   function create() {
-    return view('blog-posts.create');
+    return view('blog-posts.create', ['post' => new BlogPost]);
   }
 
 }{% endraw %}
```

We pass a `$post` variable into the view from the controller, sending it a `BlogPost` instance.

Red: Class 'App\BlogPost' not found

Next we get an error that the `BlogPost` class doesn't exist yet.


### Add blog post model [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/263f0147a388c5ba50987f4b072617ea2ff5909a)

#### app/BlogPost.php

```diff
{% raw %}+<?php
+
+namespace App;
+
+use Illuminate\Database\Eloquent\Model;
+
+class BlogPost extends Model
+{
+    //
+}{% endraw %}
```

Now that the BlogPost model exists, the controller is able to render the view, and the test is able to submit the form to the `store` route.

Red: Method App\Http\Controllers\BlogPostsController::store() does not exist

The next error is that there is no `store` action configured on the controller.


### Add store method to blog posts controller [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/a29d73cbb504c9cb6ece632feaca20bfafd23521)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %}     return view('blog-posts.create', ['post' => new BlogPost]);
   }
 
+  function store() {
+    
+  }
+
 }{% endraw %}
```

Red: The current node list is empty.

There is now a `store` action on the controller, but when the test attempts to look for content on the page, there is no content rendered.


### Render store page [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/4478d9cfcbbe8e24adf7ed8d903aa6da40dc0ffd)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %}   }
 
   function store() {
-    
+    return view('blog-posts.store');
   }
 
 }{% endraw %}
```


#### resources/views/blog-posts/store.blade.php

```diff
{% raw %}+hi{% endraw %}
```

We do just enough to get past the current error: we add a store view with some dummy content, and render it from the `store` action. Normally you would redirect from a `store` action to another route, but for the purposes of this test we'll just render content directly.

Red: Failed asserting that the page contains the HTML [Hello, World!]

Now that content is showing up, the test fails when it can't find the post title on the page.


### Render blog post on store page [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/1db5aa67ae472532957d965c728039ce7c9dbcdb)

#### resources/views/blog-posts/store.blade.php

```diff
{% raw %}-hi
+<h1>{{ $blogPost->title }}</h1>
+
+<div>
+  {{ $blogPost->body }}
+</div>{% endraw %}
```

We add markup to output the blog post's title and body.

Red: Undefined variable: blogPost

Now we get an error that there is no `$blogPost` variable sent to the view template.


### Create blog post for the view [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/5cfdb3eddd571864730a01d16f533802993ce0bb)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %} <?php
 
 namespace App\Http\Controllers;
+use Illuminate\Http\Request;
 use App\BlogPost;
 
 class BlogPostsController extends Controller {
...
     return view('blog-posts.create', ['post' => new BlogPost]);
   }
 
-  function store() {
-    return view('blog-posts.store');
+  function store(Request $request) {
+    $blogPost = new BlogPost($request->only(['title', 'body']));
+    return view('blog-posts.store', ['blogPost' => $blogPost]);
   }
 
 }{% endraw %}
```

We create a BlogPost instance with the submitted form data and pass it to the view. We do mass assignment since that's the most convenient approach.

Red: Illuminate\Database\Eloquent\MassAssignmentException: title

There's a problem with the mass assignment, though: the model throws an exception because by default it disallows mass assignment for all fields.


### Specify the model should allow mass assignment [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/e3d4ba179396ab5ffbce7a106711cd17889bce3a)

#### tests/models/BlogPostTest.php

```diff
{% raw %}+<?php
+
+use App\BlogPost;
+
+class BlogPostTest extends TestCase
+{
+
+  /** @test */
+  public function itAllowsAssigningAllPublicFields()
+  {
+    $fields = [ 'title' => 'My Title', 'body' => 'This is the body.' ];
+
+    $post = new BlogPost($fields);
+
+    $this->assertEquals('My Title', $post->title);
+    $this->assertEquals('This is the body.', $post->body);
+  }
+
+}{% endraw %}
```

Since enabling fields for mass assignment is a logic change to the BlogPost class, we create a unit test for it to specify this behavior. We reproduce the error happening at the acceptance level.

Inner Red: Illuminate\Database\Eloquent\MassAssignmentException: title


### Allow title and body to be mass-assigned [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/b2500a4dd3088fb9d19cbcd38c44008e37b26ccd)

#### app/BlogPost.php

```diff
{% raw %} 
 class BlogPost extends Model
 {
-    //
+    protected $fillable = ['title', 'body'];
 }{% endraw %}
```


#### tests/models/BlogPostTest.php

```diff
{% raw %} <?php
 
+use Illuminate\Foundation\Testing\DatabaseMigrations;
 use App\BlogPost;
 
 class BlogPostTest extends TestCase
 {
+  use DatabaseMigrations;
 
   /** @test */
   public function itAllowsAssigningAllPublicFields(){% endraw %}
```

We add mass assignment support for the title and body fields.

Inner green; outer red: PDOException: SQLSTATE[42S02]: Base table or view not found: 1146 Table 'laravel-tdd-testing.blog_posts' doesn't exist

Now the unit test is satisfied, and the acceptance test throws an exception that the table for the model is not found. We're finally hitting the database!


### Create blog posts table [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/4f1ac682c687d5b986ab2bd134248c39bb0f0028)

#### database/migrations/2016_06_10_205256_create_blog_posts_table.php

```diff
{% raw %}+<?php
+
+use Illuminate\Database\Schema\Blueprint;
+use Illuminate\Database\Migrations\Migration;
+
+class CreateBlogPostsTable extends Migration
+{
+    /**
+     * Run the migrations.
+     *
+     * @return void
+     */
+    public function up()
+    {
+        Schema::create('blog_posts', function (Blueprint $table) {
+            $table->increments('id');
+
+            $table->string('title');
+            $table->text('body');
+
+            $table->timestamps();
+        });
+    }
+
+    /**
+     * Reverse the migrations.
+     *
+     * @return void
+     */
+    public function down()
+    {
+        Schema::drop('blog_posts');
+    }
+}{% endraw %}
```

Outer red: Trying to get property of non-object

Now that the acceptance test is able to access a `blog_posts` table, it gives this unhelpful message. What's going on is that the `BlogPost::first()` call returns null, because no post is actually saved to the database.


### Save blog post to database [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/laravel/commit/f3312e6427d25c2e9d4561a9c1c1925bdc4a1c75)

#### app/Http/Controllers/BlogPostsController.php

```diff
{% raw %}   }
 
   function store(Request $request) {
-    $blogPost = new BlogPost($request->only(['title', 'body']));
+    $blogPost = BlogPost::create($request->only(['title', 'body']));
     return view('blog-posts.store', ['blogPost' => $blogPost]);
   }
 {% endraw %}
```

We change the `store` action to not only instantiate a `BlogPost`, but also save it to the database.

Outer green

With that, all our tests are passing and our feature is done. We've successfully used an acceptance test to drive our design of this feature!

