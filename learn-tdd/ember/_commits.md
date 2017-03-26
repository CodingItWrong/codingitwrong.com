### Add acceptance test scaffold [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/c05369146b5ff3e2e696396f46a72562dc400a68)

#### tests/acceptance/creating-a-blog-post-test.js

```diff
{% raw %}+/* jshint expr:true */
+import {
+  describe,
+  it,
+  beforeEach,
+  afterEach
+} from 'mocha';
+import { expect } from 'chai';
+import startApp from '../helpers/start-app';
+import destroyApp from '../helpers/destroy-app';
+
+describe('Acceptance: CreatingABlogPost', function() {
+  let application;
+
+  beforeEach(function() {
+    application = startApp();
+  });
+
+  afterEach(function() {
+    destroyApp(application);
+  });
+
+  it('can visit /creating-a-blog-post', function() {
+    visit('/creating-a-blog-post');
+
+    andThen(function() {
+      expect(currentPath()).to.equal('creating-a-blog-post');
+    });
+  });
+});{% endraw %}
```

`ember g acceptance-test creating-a-blog-post`


### Specify the feature for creating a blog post [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/3283febc02b3870e333bd17e3ca5860c02a8ebdf)

#### tests/acceptance/creating-a-blog-post-test.js

```diff
{% raw %}     destroyApp(application);
   });
 
-  it('can visit /creating-a-blog-post', function() {
-    visit('/creating-a-blog-post');
+  it('can create a blog post', function() {
+    visit('/posts/new');
+
+    fillIn('.js-post-form-title', 'Test Post');
+    fillIn('.js-post-form-body', 'This post is a test!');
+    click('.js-post-form-save');
 
     andThen(function() {
-      expect(currentPath()).to.equal('creating-a-blog-post');
+      expect(currentPath()).to.equal('posts.show');
+
+      expect(find('.js-post-detail-title').text()).to.include('Test Post');
+      expect(find('.js-post-detail-body').text()).to.include('This post is a test!');
     });
   });
 });{% endraw %}
```

We set up the entire acceptance test at once. This test will guide us through the rest of the unit testing and implementation of the feature.

Red: The URL '/posts/new' did not match any routes in your application


### Add new blog post route [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/6121cf4ba91796e8509b09f698885eaa588bd1e4)

#### app/router.js

```diff
{% raw %} });
 
 Router.map(function() {
+  this.route('posts', function() {
+    this.route('new');
+  });
 });
 
 export default Router;{% endraw %}
```


#### app/routes/posts/new.js

```diff
{% raw %}+import Ember from 'ember';
+
+export default Ember.Route.extend({
+});{% endraw %}
```


#### app/templates/posts/new.hbs

```diff
{% raw %}+{{outlet}}{% endraw %}
```

`ember g route posts/new`

We only change enough code to get to the next error message. Getting past the "no route" error only requires creating the route in the routes file.

Red: Element .js-post-form-title not found.

The next error is simple: no title form field is found to fill text into.


### Add post-form component scaffold [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/684d9ac0f060718ec5e254916ddfadf437dc61fd)

#### app/components/post-form.js

```diff
{% raw %}+import Ember from 'ember';
+
+export default Ember.Component.extend({
+});{% endraw %}
```


#### app/templates/components/post-form.hbs

```diff
{% raw %}+{{yield}}{% endraw %}
```


#### app/templates/posts/new.hbs

```diff
{% raw %}-{{outlet}}
+<h1>New Post</h1>
+
+{{post-form}}{% endraw %}
```


#### tests/integration/components/post-form-test.js

```diff
{% raw %}+/* jshint expr:true */
+import { expect } from 'chai';
+import {
+  describeComponent,
+  it
+} from 'ember-mocha';
+import hbs from 'htmlbars-inline-precompile';
+
+describeComponent(
+  'post-form',
+  'Integration: PostFormComponent',
+  {
+    integration: true
+  },
+  function() {
+    it('renders', function() {
+      // Set any properties with this.set('myProperty', 'value');
+      // Handle any actions with this.on('myAction', function(val) { ... });
+      // Template block usage:
+      // this.render(hbs`
+      //   {{#post-form}}
+      //     template content
+      //   {{/post-form}}
+      // `);
+
+      this.render(hbs`{{post-form}}`);
+      expect(this.$()).to.have.length(1);
+    });
+  }
+);{% endraw %}
```

`ember g component post-form`

Rather than just getting the test to pass by putting a form input on the route's template, we "write the code we wish we had." In this case, we wish we had a `post-form` component to use that would provide the form inputs for us. We generate it and go ahead and render it in our route template.


### Specify form component should render the form [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/c47a8102f1dfd54639c707973579f62cb9562c46)

#### tests/integration/components/post-form-test.js

```diff
{% raw %}     integration: true
   },
   function() {
-    it('renders', function() {
-      // Set any properties with this.set('myProperty', 'value');
-      // Handle any actions with this.on('myAction', function(val) { ... });
-      // Template block usage:
-      // this.render(hbs`
-      //   {{#post-form}}
-      //     template content
-      //   {{/post-form}}
-      // `);
-
+    it('renders the form', function() {
       this.render(hbs`{{post-form}}`);
-      expect(this.$()).to.have.length(1);
+
+      var title = this.$('.js-post-form-title');
+      expect(title.length,
+             'Element .js-post-form-title not found'
+             ).to.equal(1);
+
+      var body = this.$('.js-post-form-body');
+      expect(body.length,
+             'Element .js-post-form-body not found'
+             ).to.equal(1);
     });
   }
 );{% endraw %}
```

We create a unit test for the component that reproduces the acceptance test error.

We also go ahead and specify the body field as well, even though that's not strictly necessary to reproduce the current acceptance error. We're pretty sure it'll error out on that field missing too, so this is a case where it's safe to go ahead and specify it at the unit level.

Inner red: Element .js-post-form-title not found: expected 0 to equal 1

The default error message isn't the same because the Ember component tests use a different framework than acceptance tests, so we write a custom error message to match more closely.


### Add form component markup [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/a6eb4d90edf67a723622e9e7c2d46022ccb75ce6)

#### app/templates/components/post-form.hbs

```diff
{% raw %}-{{yield}}
+<form>
+  <div>
+    <label for="js-post-form-title">Title</label>
+    <input type="text" class="js-post-form-title" />
+  </div>
+  <div>
+    <label for="js-post-form-body">Body</label>
+    <textarea class="js-post-form-body" />
+  </div>
+
+  <button type="submit" class="js-post-form-save">Save</button>
+</form>{% endraw %}
```

Inner green; outer test hangs after submitting form

Now that we're rendering markup for the component, its unit test is able to find the title field and fill it in. The acceptance test also gets past the point of filling in the title, and now it hangs after clicking the save button. I'm not sure why, but I think it has to do with the fact that, since that form isn't wired up to any Ember behavior, the form is actually submitted in the browser, which closes down the Ember app. In any case, we need to get Ember handling the form submission.


### Specify the component should call the save action [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/a25459d29fb64677d6b61a3a6fed1e8801134ca8)

#### tests/acceptance/creating-a-blog-post-test.js

```diff
{% raw %} /* jshint expr:true */
 import {
   describe,
-  it,
+  // it,
   beforeEach,
   afterEach
 } from 'mocha';
-import { expect } from 'chai';
+// import { expect } from 'chai';
 import startApp from '../helpers/start-app';
 import destroyApp from '../helpers/destroy-app';
 
...
     destroyApp(application);
   });
 
+  /*
   it('can create a blog post', function() {
     visit('/posts/new');
 
...
       expect(find('.js-post-detail-body').text()).to.include('This post is a test!');
     });
   });
+  */
 });{% endraw %}
```


#### tests/integration/components/post-form-test.js

```diff
{% raw %}              'Element .js-post-form-body not found'
              ).to.equal(1);
     });
+
+    it('calls the submit handler', function() {
+      let submitHandlerCalled = false;
+      this.set('testSubmitHandler', () => {
+        submitHandlerCalled = true;
+      });
+
+      this.render(hbs`{{post-form submitHandler=(action testSubmitHandler)}}`);
+
+      this.$('.js-post-form-save').click();
+
+      expect(submitHandlerCalled,
+             "Expected submit handler to be called").to.be.true;
+    });
   }
 );{% endraw %}
```

Inner test hangs after submitting form

We reproduce the acceptance test error at the component level, which is unfortunately that the test hangs.

In order to get clear test output from the component test, we temporarily disable the acceptance test so it's only the component test causing the suite to hang.


### Add form submit action [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/714b3ede96e6018ffcd1842c4a0a30dbd3fe57ee)

#### app/templates/components/post-form.hbs

```diff
{% raw %}-<form>
+<form {{action 'submitForm' on='submit'}}>
   <div>
     <label for="js-post-form-title">Title</label>
     <input type="text" class="js-post-form-title" />{% endraw %}
```

To prevent the form from hanging, we add a submit action to the form. Now the test doesn't hang, and instead we get:

Inner red: Expected submit handler to be called: expected false to be true

Ember doesn't complain about the fact that we don't actually have a "submitForm" action on the component; it just proceeds on its way. So our test's submit handler is never called.


### Add submitForm action [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/7e495c0f92f29680cacff3c6cda75d723358d254)

#### app/components/post-form.js

```diff
{% raw %} import Ember from 'ember';
 
 export default Ember.Component.extend({
+  actions: {
+    submitForm() {  
+      this.get('submitHandler')();
+    }
+  }
 });{% endraw %}
```


#### tests/acceptance/creating-a-blog-post-test.js

```diff
{% raw %} /* jshint expr:true */
 import {
   describe,
-  // it,
+  it,
   beforeEach,
   afterEach
 } from 'mocha';
-// import { expect } from 'chai';
+import { expect } from 'chai';
 import startApp from '../helpers/start-app';
 import destroyApp from '../helpers/destroy-app';
 
...
     destroyApp(application);
   });
 
-  /*
   it('can create a blog post', function() {
     visit('/posts/new');
 
...
       expect(find('.js-post-detail-body').text()).to.include('This post is a test!');
     });
   });
-  */
 });{% endraw %}
```

To get the submitHandler called, we add a submitForm action on the component, so that it will be called when the form is submitted. Then we retrieve the "submitHandler" property of the component, and call it as a function.

Inner green; outer red: TypeError: undefined is not a constructor (evaluating 'this.get('submitHandler')()')

Now our component test is able to verify that the submit handler is called. We go ahead and re-enable our acceptance test, and now it's no longer hanging either--but it errors out because submitting the form expects a submitHandler to be passed in, and there isn't one.


### Add new post controller for save action [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/ced8133d448412dfabeef63cf943acee03af5b26)

#### app/controllers/posts/new.js

```diff
{% raw %}+import Ember from 'ember';
+
+export default Ember.Controller.extend({
+  actions: {
+    savePost() {
+      this.transitionToRoute("posts.show");
+    }
+  }
+});{% endraw %}
```


#### app/templates/posts/new.hbs

```diff
{% raw %} <h1>New Post</h1>
 
-{{post-form}}
+{{post-form submitHandler=(action "savePost")}}{% endraw %}
```

`ember g controller posts/new`

We implement a save handler by adding a new post controller to put it in, adding the handler, then passing it into the form component.

Outer red: The route posts.show was not found

Now the acceptance test successfully attempts to transition to the `posts.show` route, but it doesn't yet exist.


### Add posts.show route [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/5e3bfd172a38ad2cd5d995bde81edf6eb05dd64c)

#### app/router.js

```diff
{% raw %} Router.map(function() {
   this.route('posts', function() {
     this.route('new');
+    this.route('show');
   });
 });
 {% endraw %}
```


#### app/routes/posts/show.js

```diff
{% raw %}+import Ember from 'ember';
+
+export default Ember.Route.extend({
+});{% endraw %}
```


#### app/templates/posts/show.hbs

```diff
{% raw %}+{{outlet}}{% endraw %}
```

`ember g route posts/show`

Outer red: expected '' to equal 'Test Post'

Now the acceptance test is able to display the `posts.show` route, but it can't find the post's title on the page, because we aren't rendering anything to the screen yet.


### Add detail component scaffold [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/54bab2bfa450c6f0d8dd23e922129a4a2b495794)

#### app/components/post-detail.js

```diff
{% raw %}+import Ember from 'ember';
+
+export default Ember.Component.extend({
+});{% endraw %}
```


#### app/templates/components/post-detail.hbs

```diff
{% raw %}+{{yield}}{% endraw %}
```


#### app/templates/posts/show.hbs

```diff
{% raw %}-{{outlet}}
+<h1>Post</h1>
+
+{{post-detail post=model}}{% endraw %}
```


#### tests/integration/components/post-detail-test.js

```diff
{% raw %}+/* jshint expr:true */
+import { expect } from 'chai';
+import {
+  describeComponent,
+  it
+} from 'ember-mocha';
+import hbs from 'htmlbars-inline-precompile';
+
+describeComponent(
+  'post-detail',
+  'Integration: PostDetailComponent',
+  {
+    integration: true
+  },
+  function() {
+    it('renders', function() {
+      // Set any properties with this.set('myProperty', 'value');
+      // Handle any actions with this.on('myAction', function(val) { ... });
+      // Template block usage:
+      // this.render(hbs`
+      //   {{#post-detail}}
+      //     template content
+      //   {{/post-detail}}
+      // `);
+
+      this.render(hbs`{{post-detail}}`);
+      expect(this.$()).to.have.length(1);
+    });
+  }
+);{% endraw %}
```

`ember g component post-detail`

Again, instead of making the acceptance test pass as quickly as possible, we "write the code we wish we had": a post display component.


### Specify detail component should display model fields [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/2a08f9e70704af1cdd53b700751d7663a528aeb4)

#### tests/integration/components/post-detail-test.js

```diff
{% raw %}     integration: true
   },
   function() {
-    it('renders', function() {
-      // Set any properties with this.set('myProperty', 'value');
-      // Handle any actions with this.on('myAction', function(val) { ... });
-      // Template block usage:
-      // this.render(hbs`
-      //   {{#post-detail}}
-      //     template content
-      //   {{/post-detail}}
-      // `);
+    it('displays details for the passed-in post', function() {
+      this.set('testModel', {title: 'Test Title', body: 'This is a test post!'});
 
-      this.render(hbs`{{post-detail}}`);
-      expect(this.$()).to.have.length(1);
+      this.render(hbs`{{post-detail post=testModel}}`);
+      expect(this.$('.js-post-detail-title').text()).to.include('Test Title');
+      expect(this.$('.js-post-detail-body').text()).to.include('This is a test post!');
     });
   }
 );{% endraw %}
```

We add a component test that reproduces the acceptance test error: we specify that the component displays the post's title. And we go ahead and specify that it displays the body, too, because that seems safe in this case.

Inner red: expected '' to equal 'Test Title'


### Add post detail display markup [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/7f036fd2218001bbf1347a71023dbffc22b55e7c)

#### app/templates/components/post-detail.hbs

```diff
{% raw %}-{{yield}}
+<h2 class="js-post-detail-title">{{post.title}}</h2>
+
+<div class="js-post-detail-body">
+  {{post.body}}
+</div>{% endraw %}
```

We make the component test pass by adding markup to display the post.

Inner green; outer red: expected '' to equal 'Test Post'

The acceptance test still has the same error, because we aren't passing the model into the component.


### Hook routes into post model [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/30dc61f27995c04bc5585b279a121459d78a9e6f)

#### app/controllers/posts/new.js

```diff
{% raw %} 
 export default Ember.Controller.extend({
   actions: {
-    savePost() {
-      this.transitionToRoute("posts.show");
+    savePost(postData) {
+      let post = this.store.createRecord('post', postData);
+      post.save().then((post) => {
+        this.transitionToRoute("posts.show", post.id);
+      });
     }
   }
 });{% endraw %}
```


#### app/router.js

```diff
{% raw %} Router.map(function() {
   this.route('posts', function() {
     this.route('new');
-    this.route('show');
+    this.route('show', { path: ':postId' });
   });
 });
 {% endraw %}
```


#### app/routes/posts/show.js

```diff
{% raw %} import Ember from 'ember';
 
 export default Ember.Route.extend({
+  model(params) {
+    return this.store.findRecord('post', params.postId);
+  }
 });{% endraw %}
```

This acceptance test error drives a lot of logic: to display the post's title on the show page, we need to save the post on the new page, include the ID in the transition to the show route, then load the post on the show page's model hook.

Outer red: No model was found for 'post'

With this logic added, the acceptance test errors out quickly: there _is_ no `post` model.


### Add post model [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/c72828ff16ef5c913d28c763ff08ecc5fc8819be)

#### app/models/post.js

```diff
{% raw %}+import DS from 'ember-data';
+
+export default DS.Model.extend({
+
+});{% endraw %}
```

`ember g model post`

Outer red: Your Ember app tried to POST '/posts', but there was no route defined to handle this request. Define a route that matches this path in your mirage/config.js file.

Next we get an error from Mirage, our fake server. It needs a corresponding post creation endpoint created.


### Add mirage create post route [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/e2bb7e4ea133bed54a9e930ffce125d2202b597d)

#### mirage/config.js

```diff
{% raw %} export default function() {
+  this.post('/posts');
 }{% endraw %}
```

Outer red: Pretender intercepted POST /posts but encountered an error: Mirage: The route handler for /posts is trying to access the post model, but that model doesn't exist. Create it using 'ember g mirage-model post'.

Now that Mirage has an endpoint, it returns another error: a Mirage model for `post` needs to be added too.


### Add mirage model [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/a8775df1abf8f6070a618ae45fb4edda6c1c3d5f)

#### mirage/models/post.js

```diff
{% raw %}+import { Model } from 'ember-cli-mirage';
+
+export default Model.extend({
+});{% endraw %}
```

`ember g mirage-model post`

Outer red: Pretender intercepted POST /posts but encountered an error: Mirage: You're using a shorthand or #normalizedRequestAttrs, but your serializer's normalize function did not return a valid JSON:API document. http://www.ember-cli-mirage.com/docs/v0.2.x/serializers/#normalizejson

The next error is pretty obscure. What's happening is that no attributes are being sent in the create request, and Mirage is erroring out right away. We do want the `title` attribute to be sent, so this is a valid error situation.


### Specify form should send the form data [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/a5e305f4fe1268bde0fd59344e3eef91c1967fe8)

#### tests/integration/components/post-form-test.js

```diff
{% raw %}              ).to.equal(1);
     });
 
-    it('calls the submit handler', function() {
+    it('calls the submit handler with the form data', function() {
       let submitHandlerCalled = false;
-      this.set('testSubmitHandler', () => {
+      this.set('testSubmitHandler', (postData) => {
         submitHandlerCalled = true;
+        expect(postData.title).to.equal('New Title');
+        expect(postData.body).to.equal('New Body');
       });
 
       this.render(hbs`{{post-form submitHandler=(action testSubmitHandler)}}`);
 
+      this.$('.js-post-form-title').val('New Title').blur();
+      this.$('.js-post-form-body').val('New Body').blur();
       this.$('.js-post-form-save').click();
 
       expect(submitHandlerCalled,{% endraw %}
```

We add a component test case specifying that the post form should send the data from the form fields into the submit handler.

Inner red: undefined is not an object (evaluating 'postData.title')

This means that no postData object is being sent into the submit handler at all.


### Send post form data to save action [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/cba7cec04bda940b27c914856d8677bfbb204b25)

#### app/components/post-form.js

```diff
{% raw %} 
 export default Ember.Component.extend({
   actions: {
-    submitForm() {  
-      this.get('submitHandler')();
+    submitForm() {
+      let postData = {
+        title: this.get('title'),
+        body: this.get('body'),
+      };
+      this.get('submitHandler')(postData);
     }
   }
 });{% endraw %}
```


#### app/templates/components/post-form.hbs

```diff
{% raw %} <form {{action 'submitForm' on='submit'}}>
   <div>
     <label for="js-post-form-title">Title</label>
-    <input type="text" class="js-post-form-title" />
+    {{input type="text" class="js-post-form-title" value=title}}
   </div>
   <div>
     <label for="js-post-form-body">Body</label>
-    <textarea class="js-post-form-body" />
+    {{textarea class="js-post-form-body" value=body}}
   </div>
 
   <button type="submit" class="js-post-form-save">Save</button>{% endraw %}
```

We get the component test to pass by making the input an Ember input helper, retrieving the saved value in it, and sending that in an object to the `save` action closure.

Inner green; outer red: Pretender intercepted POST /posts but encountered an error: Mirage: You're using a shorthand or #normalizedRequestAttrs, but your serializer's normalize function did not return a valid JSON:API document. http://www.ember-cli-mirage.com/docs/v0.2.0-beta.9/serializers/#normalizejson

Now the acceptance test still gives the same error, and in this case the tests aren't that helpful to guide us to why. What's going on is that the Ember Data `post` model doesn't have any fields defined on it, so it isn't saved and retrieved by the show page.


### Add fields to post model [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/c5564deb41fe3e3cf1b675de5542085b61e6bf95)

#### app/models/post.js

```diff
{% raw %} import DS from 'ember-data';
 
 export default DS.Model.extend({
-
+  title: DS.attr(),
+  body: DS.attr(),
 });{% endraw %}
```

We add the title and body fields to the post model, so now it's saved by the new page.

Outer red: Your Ember app tried to GET '/posts/1', but there was no route defined to handle this request. Define a route that matches this path in your mirage/config.js file.

The next error is in Mirage again: we now get to the post show page, but Mirage isn't configured to retrieve a post by ID.


### Add mirage get post route [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/ember/commit/68e8d05be5186696c6a83c0d9e1e31d9ae5bea0d)

#### mirage/config.js

```diff
{% raw %} export default function() {
+  this.get('/posts/:id');
   this.post('/posts');
 }{% endraw %}
```

Outer green

We configure Mirage to return a post retrieved by ID, and the acceptance test passes. We've successfully let our acceptance test drive out the behavior of this feature!

