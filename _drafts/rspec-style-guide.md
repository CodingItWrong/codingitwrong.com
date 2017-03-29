---
title: RSpec Style Guide
---

Here are the style principles I try to follow personally in my RSpec specs. In my experience, they tend to result in specs that are easier to understand and less fragile. Most of these principles would likely apply to other spec/test frameworks as well.

## Organization

- **Prioritize readability above DRY.** In my opinion readability is even *more* important in specs than in production code. You can afford to take a little effort to understand production code, but specs need to be *stupid* simple to understand. Here are a few cases where this comes up:
  - **Try not to nest `context` blocks: only use them one level deep.** More than one level of context can make it hard to find all the `let` statements and `before` blocks that apply. It's okay for you to have to duplicate some `let` statements, and it's okay for there to be an "and" in your context description!

    Bad:

    ```
    context "not logged in" do
      # ...
    end
    context "logged in" do
      context "as a non-admin user" do
        context "unconfirmed" do
          # ...
        end
        context "confirmed" do
          context "active" do
            # When will it end!?!?...
          end
          context "deactivated" do
            # ...
          end
        end
      end
      context "as an admin user" do
        # ...
      end
    end
    ```

    Good:

    ```
    context "not logged in" do
      # ...
    end
    context "as an unconfirmed user" do
      # ...
    end
    context "as an active user" do
      # Don't need to say "confirmed" and "not-admin"; those are assumed.
      # ...
    end
    context "as a deactivated user" do
      # ...
    end
    context "as an admin user" do
      # ...
    end
    ```

  - **Don't use `let()` statements and `before` blocks to separate code that needs to be understood together.** In that case, some duplication for the sake of readability is probably better.

    Bad:

    ```
    let(:path) { "/api/posts" }

    let(:params) {
      {
        title: title,
        author: author,
        body: body,
      }
    }

    # 50+ lines of specs

    context "valid post" do
      let(:title) { "Title" }
      let(:author) { "Test Person" }
      let(:body) { "Body" }

      it "should create a post" do
        post path, params # what even ARE path and params???
        # ...
      end
    end
    ```

    Good:

    ```
    # 50+ lines of specs

    context "valid post" do
      it "should create a post" do
        post "/api/posts", {
          title: "Title",
          author: "Test Person",
          body: "body",
        } # ah, all here in one place
        # ...
      end
    end
    ```


- **In unit specs, specify one behavior per example.** This might just be one expectation, or it might be a few--but probably not a lot. The key is that you aren't specifying unrelated things. Stick with the Single Responsibility Principle when it comes to examples.

  Bad:

  ```
  describe "#push" do
    it "pushes the object onto the stack" do
      stack.push(object_to_push)

      expect(stack.count).to eq(1)
      expect(stack.empty).to be_falsy
      expect(stack.peek).to eq(object_to_push)
    end
  end
  ```

  Good:

  ```
  describe "#push" do
    before do
      stack.push(object_to_push)
    end

    it "increments the count" do
      expect(stack.count).to eq(1)
    end

    it "makes the stack no longer empty" do
      expect(stack.empty).to be_falsy
    end

    it "makes that object the top of the stack" do
      expect(stack.peek).to eq(object_to_push)
    end
  end
  ```

- **In acceptance specs, many behaviors per example.** For any spec that's hitting the whole application stack (a feature spec, or maybe a request spec), having only one behavior per example may make your test suite too slow. In this case, it's best to specify all the behaviors for a given request/call in one example, so that request/call is only made once.

  Bad:

  ```
  before(:each) do
    post '/api/confirmation', {
      confirmation: {
        password: "Foobarbaz123",
        password_confirmation: "Foobarbaz123",
        confirmation_token: confirmation_token
      }
    }
  end

  it 'returns a created status' do
    expect(response.status).to eq(201)
  end

  it 'returns the user email' do
    expect(JSON.parse(response.body)["confirmation"]["email"]).to eq(user_login.email)
  end

  it 'sets the user to confirmed' do
    expect(user_login.reload).to be_confirmed
  end
  ```

  Good:

  ```
  it 'sets the user to confirmed and returns the email' do
    post '/api/confirmation', {
      confirmation: {
        password: "Foobarbaz123",
        password_confirmation: "Foobarbaz123",
        confirmation_token: confirmation_token
      }
    }

    expect(response.status).to eq(201)
    expect(JSON.parse(response.body)["confirmation"]["email"]).to eq(user_login.email)

    expect(user_login.reload).to be_confirmed
  end
  ```  

## Test Data

- **Only use `let!` instead of `let` when absolutely necessary.** If you get into the habit of always using `let!`, you'll end up instantiating variables that most of your examples don't need. Write `let` first, and change it to `let!` only if you have to do so to get a spec to pass.

  Bad:

  ```
  let!(:user) { User.create }
  let!(:blog) { Blog.create }
  let!(:post) { blog.posts.create }

  it 'returns blog posts' do
    get "/blogs/#{blog.id}/posts", user.auth_header
    expect(JSON.parse(response.body).count).to eq(1)
  end
  ```

  Good:

  ```
  let(:user) { User.create }
  let(:blog) { Blog.create }
  let!(:post) { blog.posts.create } # will fail if not `let!`, because never referenced directly

  it 'returns blog posts' do
    get "/blogs/#{blog.id}/posts", user.auth_header
    expect(JSON.parse(response.body).count).to eq(1)
  end
  ```

- **Minimize data setup per expectation.** Whether it's in `let` statements or in method calls in a `before` block or the example itself, lots of data setup for each expectation makes it hard to grok. Ideally, you should only need to see the data setup that's necessary to understand the example. Here are a few approaches that can help:
    - Define default data using a factory system like FactoryGirl. That way, when instantiating classes, you only need to override the fields you care about for that example. This also prevents you from having to make changes in lots of different specs when you add a new required field to a class.

      Bad:

      ```
      let(:titleless_post) {
        Post.create( title: nil,
                     body: "Foo",
                     author: "Josh",
                     publish_status: Post::PUBLISHED,
                     publish_date: DateTime.now ) # which of these do I care about?
      }

      let(:bodyless_post) {
        Post.create( title: "Foo",
                     body: nil,
                     author: "Josh",
                     publish_status: Post::PUBLISHED,
                     publish_date: DateTime.now ) # which of these do I care about?
      }
      ```

      Good:

      ```
      # factories/post.php

      factory :post do
        title "Test Post"
        body "This is a test post."
        author "Josh"
        publish_status Post::PUBLISHED
        publish_date DateTime.now
      end

      # spec

      let(:titleless_post) { FactoryGirl.create(:post, title: nil) }
      let(:bodyless_post) { FactoryGirl.create(:post, body: nil) }
      ```

    - Use shared contexts for things more complex than an individual factory. - good example hospital "with a hospital staff access token"

      Bad:

      ```
      describe "#show" do
        context "as a staff member" do
          let(:staff) { FactoryGirl.create(:staff) }
          let(:user_login) { FactoryGirl.create(:user_login, role: staff) }
          let(:access_token) { Doorkeeper::AccessToken.create!(resource_owner_id: user_login.id).token }
          let(:auth_header) { { "Authorization" => "Bearer #{access_token}" } }

          #...
        end
      end

      describe "#create" do
        context "as a staff member" do
          let(:staff) { FactoryGirl.create(:staff) }
          let(:user_login) { FactoryGirl.create(:user_login, role: staff) }
          let(:access_token) { Doorkeeper::AccessToken.create!(resource_owner_id: user_login.id).token }
          let(:auth_header) { { "Authorization" => "Bearer #{access_token}" } }

          #...
        end
      end
      ```

      Good:

      ```
      # support/with_a_staff_member_access_token.rb

      RSpec.shared_context "with a staff member access token" do
        let(:staff) { FactoryGirl.create(:staff) }
        let(:user_login) { FactoryGirl.create(:user_login, role: staff) }
        let(:access_token) { Doorkeeper::AccessToken.create!(resource_owner_id: user_login.id).token }
        let(:auth_header) { { "Authorization" => "Bearer #{access_token}" } }
      end

      # spec
      describe "#show" do
        context "as a staff member" do
          include "with a staff member access token"

          #...
        end
      end

      describe "#create" do
        context "as a staff member" do
          include "with a staff member access token"

          #...
        end
      end
      ```

    - Define objects in the broader context using `let` statements, then override only the fields you need to for the given test.

      Bad:

      ```
      define "#process_email_bounce_webhook" do
        context "soft bounce" do
          let(:webhook_data) {
            {
              bounce_type: "soft",
              recipient: "test@example.com",
              subject: "Test Message",
              date_sent: DateTime.now,
            }
          }
          # ...
        end

        context "hard bounce" do
          let(:webhook_data) {
            {
              bounce_type: "hard",
              recipient: "test@example.com",
              subject: "Test Message",
              date_sent: DateTime.now,
            }
          }
          # ...
        end
      end
      ```

      Good:

      ```
      define "#process_email_bounce_webhook" do
        let(:webhook_data) {
          {
            bounce_type: bounce_type,
            recipient: "test@example.com",
            subject: "Test Message",
            date_sent: DateTime.now,
          }
        }

        context "soft bounce" do
          let(:bounce_type) { "soft" }
          # ...
        end

        context "hard bounce" do
          let(:bounce_type) { "hard" }
          # ...
        end
      end
      ```

    - Create custom matchers that match for something specific you're looking for, so that you don't have to construct complex objects to use for matching.
- **Always set the pertinent fields on objects in the expectation, even if you know they'll already be set to an appropriate value.** For example, you are specifying what happens when a certain field on an object is set to true. You happen to know that the factory for that object sets that field to true by default. One problem with this is that it leads to test fragility: if you change the default value of that field in the factory, lots of specs could start failing. It also hinders readability: it's not as clear to the reader that that field will be set to true. Go ahead and set it.

  Bad:

  ```
  # factories/post.rb

  factory :post do
    published true
    # ...
  end


  # spec

  context "when the post is published" do
    let(:post) { FactoryGirl.create(:post) } # are we *sure* this is published?
    # ...
  end
  ```

  Good:

  ```
  # factories/post.rb

  factory :post do
    published true
    # ...
  end


  # spec

  context "when the post is published" do
    let(:post) { FactoryGirl.create(:post, published: true) }
    # ...
  end
  ```

## Test Doubles

- **Stub in a before block, mock in the example.** If you're specifying only one behavior per example, there will often be a lot of outgoing messages you *don't* care about for that example, but that still need to work. Stubbing (`allow()`) instead of mocking (`expect()`) is a way to make it clear that you aren't specifying that behavior, just supplying something to get the method call to work. But all those stubs will clutter up your examples, and there will be a lot of duplication. Instead, put all your `allow()` statements in a single `before` block; that sets up the mock collaborators for all the behavior that all the examples will need. Then your examples will only have the `expect()` statements related to what they're specifying. This makes for much simpler examples.

  Not so good:

  ```
  let(:model) { instance_double(Model) }
  let(:mailer) { instance_double(Mailer) }
  let(:job) { instance_double(NewModelJob) }

  it "saves the model" do
    expect(model).to receive(:save).with({validate: false})
    allow(mailer).to receive(:send)
    allow(job).to receive(:enqueue)

    command.execute
  end

  it "sends a notification email" do
    allow(model).to receive(:save)
    expect(mailer).to receive(:send)
    allow(job).to receive(:enqueue)

    command.execute
  end

  it "enqueues a job" do
    allow(model).to receive(:save)
    allow(mailer).to receive(:send)
    expect(job).to receive(:enqueue).with(model)

    command.execute
  end

  ```

  Better:

  ```
  let(:model) { instance_double(Model) }
  let(:mailer) { instance_double(Mailer) }
  let(:job) { instance_double(NewModelJob) }

  before {
    allow(model).to receive(:save)
    allow(mailer).to receive(:send)
    allow(job).to receive(:enqueue)
  }

  it "saves the model" do
    expect(model).to receive(:save).with({validate: false})
    command.execute
  end

  it "sends a notification email" do
    expect(mailer).to receive(:send)
    command.execute
  end

  it "enqueues a job" do
    expect(job).to receive(:enqueue).with(model)
    command.execute
  end
  ```

- **But actually spy instead of mock.** When your method calls are stubbed, you don't need to set expectations before calling your production code; the stubs will allow them to pass. Then you're free to use spying instead of mocking. Spies tend to be more readable than mocks, because they follow the "arrange, act, assert" order more clearly.

  Okay: (see above)

  Even Better:

  ```
  let(:model) { spy(Model) }
  let(:mailer) { spy(Mailer) }
  let(:job) { instance_double(NewModelJob) }

  before {
    allow(model).to receive(:save)
    allow(mailer).to receive(:send)
    allow(job).to receive(:enqueue)

    command.execute
  }

  it "saves the model" do
    expect(model).to have_received(:save).with({validate: false})
  end

  it "sends a notification email" do
    expect(mailer).to have_received(:send)
  end

  it "enqueues a job" do
    expect(job).to have_receive(:enqueue).with(model)
  end
  ```

## Subjects

- **Use the implicit subject when and only when the expectation reads like English.** If `it { is_expected.to... }` makes it harder to understand what you're specifying, and you have to explain it in a surrounding `context` block's name, just use a regular `it` statement instead.

  Bad:

  ```
  subject(:product_ids) { JSON.parse(response.body)["product_ids"] }
  it { is_expected.to include matching_category.products.map { |p| p["id"] }) }
  ```

  Good:

  ```
  subject(:product_ids) { JSON.parse(response.body)["product_ids"] }

  it "includes all expected products" do
    expect(product_ids).to include matching_category.products.map { |p| p["id"] }
  end
  ```

  Also good:

  ```
  subject(:product_ids) { JSON.parse(response.body)["product_ids"] }
  
  let(:matching_product_ids) { matching_category.products.map { |p| p["id"] } }
  it { is_expected.to include matching_product_ids }
  ```

- **The `subject` should be whatever you're specifying against**. This isn't always the class that the unit spec relates to: in some `describe`s, it might be the result of a method call.

  Bad:

  ```
  describe DateFormatter do
    subject(:stack) { described_class.new(format_string) }

    describe "#format" do
      let(:formatted_date) { stack.push(object_to_push) }

      # tests that all refer to formatted_date, not to the "subject" stack
    end
  end
  ```

  Good:

  ```
  describe DateFormatter do
    let(:stack) { described_class.new(format_string) }

    describe "#format" do
      subject(:formatted_date) { stack.push(object_to_push) }

      # tests that all refer to the subject formatted_date
    end
  end
  ```

- **Always name your `subject`, even if you don't need to refer to it by that name.** Leaving the name of the subject `subject` makes for less readable specs; name it what it is. Even if all of your examples use an implicit subject (i.e. `it { is_expected.to… }`), naming the subject will still make the `subject` line clearer. And then you're ready for if you eventually have to add examples that don't use an implicit subject.

  Bad:

  ```
  subject { JSON.parse(response.body)["products"].map { |p| p.id } }
  it { is_expected.to include matching_product.id }
  ```

  Good:

  ```
  subject(:product_ids) { JSON.parse(response.body)["products"].map { |p| p.id } }
  it { is_expected.to include matching_product.id }
  ```

- **Implement your "act" step in the way that works best for its needs.** There are a few scenarios I frequently see:
    - **When you will always be checking the return value of your "act" step, make it your subject and use one-liner syntax.** This makes it flexible for you to refer to it wherever you need it in your examples.

      Bad:

      ```
      it "includes a matching product" do
        expect(catalog.product_ids).to include matching_product.id
      end

      it "does not include a non-matching product" do
        expect(catalog.product_ids).not_to include matching_product.id
      end

      it "does not include a deactivated product" do
        expect(catalog.product_ids).not_to include deactivated_product.id
      end

      it "does not include a product from another catalog" do
        expect(catalog.product_ids).not_to include other_catalog_product.id
      end
      ```


      Good:

      ```
      subject(:product_ids) { catalog.product_ids }

      it { is_expected.to include matching_product.id }
      it { is_expected.not_to include non_matching_product.id }
      it { is_expected.not_to include deactivated_product.id }
      it { is_expected.not_to include other_catalog_product.id }
      ```

    - **When you will not always be checking the return value of your "act" step, but it needs to run before each example, put it in a `before` block.** For example, if you are specifying the behavior of the `push` method on an object that acts as a stack, your "act" step might be `stack.push(object_to_push)`. That step would likely be repeated in all of your examples for the push behavior, but you wouldn't necessarily always check its return value. If that's the case, remove duplication by moving the call to `push` into a `before` block. That way, all your examples might consist entirely of `expect` statements.

      Bad:

      ```
      subject(:stack) { Stack.new }

      it "should put the object on top of the stack" do
        stack.push(object_to_push)
        expect(stack.peek).to eq(object_to_push)
      end

      it "should result in the count being 1" do
        stack.push(object_to_push)
        expect(stack.count).to eq(1)
      end
      ```

      Good:

      ```
      subject(:stack) { Stack.new }

      before do
        stack.push(object_to_push)
      end

      it "should put the object on top of the stack" do
        expect(stack.peek).to eq(object_to_push)
      end

      it "should result in the count being 1" do
        expect(stack.count).to eq(1)
      end
      ```

    - **When you won't always be checking the return value of your "act" step, and you can't put your "act" step in a `before` block, name it as a "bang variable".** One situation I run into a lot when I can't put my "act" step in a `before` block is when I need to use `let` statements to define different variables for each example that are needed for the "act" step. If I put each `let` statement and example in its own `context`, then put a `before` block outside of all the `context`s to run the "act" step, the `let` statements are run *after* the `before` block, so they aren't ready in time. In a case like this, the only option may seem to be to duplicate the "act" step in each example. But there's a way to remove that duplication. For example, say our "act" step is `list.remove_random(how_many)`. We can define `let(:remove_random!) { list.remove_random(how_many) }`, so that in each example we only have to write `remove_random!`. The bang makes it clear that it's a method call with side effects.

      Bad:

      ```
      before do
        list.remove_random(how_many)
      end

      context "when one is removed" do
        let(:how_many) { 1 } # the `before` block has already been run though...
        it "..." do
          expect...
        end
      end

      context "when many are removed" do
        let(:how_many) { 3 }
        it "..." do
          expect...
        end
      end
      ```

      Good:

      ```
      let(:remove_random!) { list.remove_random(how_many) }

      context "when one is removed" do
        let(:how_many) { 1 }
        it "..." do
          remove_random! # the above `how_many` is used
          expect...
        end
      end

      context "when many are removed" do
        let(:how_many) { 3 }
        it "..." do
          remove_random!
          expect...
        end
      end
      ```
