---
title: Getting Started With Glimmer-Native
---

*Updated 2019-8-1: updated for Glimmer-Native 0.0.5*

[NativeScript](https://www.nativescript.org/) is a platform for building truly native mobile apps using a variety of different web frameworks. Up until now Angular and Vue.js have been your options--but now the [Glimmer-Native](https://github.com/bakerac4/glimmer-native) project gives you the option to build native apps with [Glimmer.js](https://glimmerjs.com/) as well! Glimmer-Native is currently in alpha, and input from the community is welcomed to help it get built out.

Let's give Glimmer-Native a try building the traditional todo list app!

NativeScript works on both iOS and Android. If you'd like to preview your app on iOS, you'll need to have a Mac with [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12) installed. If you'd like to preview your app on Android, you'll need to [install Android Studio](https://developer.android.com/studio) and [set up an Android virtual device](https://developer.android.com/studio/run/managing-avds.html).

To begin, install Ember CLI and the NativeScript CLI if you don't already have both installed:

```bash
$ npm install -g ember
$ npm install -g nativescript
```

Next, create a new Ember project with the `glimmer-native-blueprint` -- this will set up your project to be a NativeScript app instead of a web app:

```bash
$ ember new todo-glimmer-native -b glimmer-native-blueprint
$ cd todo-glimmer-native
```

Let's go ahead and run the app to confirm it works. Run one of the following two commands, depending on if you want to run on iOS or Android:

```bash
$ tns run ios --bundle
```

or

```bash
$ tns run android --bundle
```

After a minute or two, your app should appear on the simulated device saying "Welcome to Glimmer Native". Sweet!

Your app's components live under `src/ui/components/`. Currently there's just one component, `TodoGlimmerNative`. Open the `TodoGlimmerNative/` directory, and the `component.ts` file inside it. You'll see:

```javascript
import Component from '@glimmer/component';

export default class TodoGlimmerNative extends Component {
  title = "Welcome to Glimmer Native"
}
```

So far this looks just like a normal Glimmer.js component class. Let's check out the `template.hbs`:

```handlebars
{% raw %}<Page>
  <ActionBar>
    <Label text={{this.title}} class="label" />
  </ActionBar>
</Page>{% endraw %}
```

The template is where the main difference is between Glimmer.js web apps and Glimmer-Native apps. Instead of rendering HTML elements, we "render" NativeScript components that, in turn, control your native app. You can read a [list of supported Glimmer-Native components](https://github.com/bakerac4/glimmer-native#how-to-use-glimmer-native) in Glimmer-Native's readme.

We can see that the title shown in our action bar comes from the `title` property of the component. As our first change, let's change that title. Make this change in `TodoGlimmerNative/component.ts`:

```diff
 export default class TodoGlimmerNative extends Component {
-  title = "Welcome to Glimmer Native"
+  title = 'Todos'
 }
```

When you save the file, your native app should reload, and the title should now be "Todos".

Next, let's start to add the core data for our app: the list of todos. We'll define some starting data in `TodoGlimmerNative/component.ts`:

```diff
 export default class TodoGlimmerNative extends Component {
   title = "Welcome to Glimmer Native"

+  todos = [
+    { id: 1, label: 'Buy bread' },
+    { id: 2, label: 'Buy milk' },
+    { id: 3, label: 'Buy eggs' },
+  ]
 }
```

Let's render that out on the screen, in `TodoGlimmerNative/template.hbs`:

```diff
{% raw %} <Page>
   <ActionBar>
     <Label text={{this.title}} class="label" />
   </ActionBar>
+  <StackLayout>
+    {{#each this.todos key="id" as |todo|}}
+      <Label
+        text={{todo.label}}
+      />
+    {{/each}}
+  </StackLayout>
 </Page>{% endraw %}
```

`StackLayout` is one of the [NativeScript layout containers](https://docs.nativescript.org/ui/layouts/layout-containers) available to our app. It simply stacks elements one after the other. We use a normal Glimmer/Handlebars `#each` helper to loop through our labels and output them. Note that we pass a `key` argument to the `#each` helper to help Glimmer track which item in the list is which.

Save the file and your todo items should appear in the app.

Next, let's add the ability to add new todos to the list. Add a few form elements to `TodoGlimmerNative/template.hbs`:

```diff
{% raw %} <StackLayout>
+  <TextField
+    hint="New Todo"
+    text={{this.newTodoLabel}}
+    {{on "textChange" (action handleChangeText)}}
+  />
+  <Button
+    text="Add Todo"
+    {{on "tap" (action addTodo)}}
+  />
+
   {{#each this.todos key="id" as |todo|}}
     <Label
       text={{todo.label}}
     />
   {{/each}}
 </StackLayout>{% endraw %}
```

The `TextField` will allow us to input a label for the todo. We read its text value from the `newTodoLabel` property, then we call a `handleChangeText` action when it's changed, to store its value.

The `Button` will indicate that we're ready to add the todo, and it calls an `addTodo` action.

Now, let's wire up these properties and actions in `TodoGlimmerNative/component.ts`:

```diff
 import Component from '@glimmer/component';
+import { tracked } from '@glimmer/tracking';

 export default class TodoGlimmerNative extends Component {
   title = "Welcome to Glimmer Native"

+  newTodoId = 4
+
+  @tracked
+  newTodoLabel = ''
+
+  @tracked
   todos = [
     { id: 1, label: 'Buy bread' },
     { id: 2, label: 'Buy milk' },
     { id: 3, label: 'Buy eggs' },
   ]

+  handleChangeText(event) {
+    this.newTodoLabel = event.value
+  }

+  addTodo() {
+    const newTodo = { id: this.newTodoId, label: this.newTodoLabel }
+    this.todos = [...this.todos, newTodo]
+    this.newTodoId++
+    this.newTodoLabel = ''
+  }
 }
```

Here's what's going on here:

- The `tracked` function we import is a decorator that tells Glimmer to rerender when the decorated property changes.
- `newTodoId` allows us to keep track of the ID to assign to the next todo we create. We start with three todo items so our first new ID is 4.
- `newTodoLabel` will store the value of the todo label the user inputs. It starts out as an empty string. When we manually change it in our actions, we want the component to rerender, so we mark it as `@tracked`.
- We mark the `todos` array as `@tracked` as well, because when we add a new todo, we want it to be shown in the component.
- `handleChangeText` takes the text the user has entered and stores it on the `newTodoLabel` property so it's available for us to use in `addTodo`
- `addTodo` creates a new todo in four steps:
  1. It creates a `newTodo` object with the next ID and the entered label.
  2. The `todos` property is overwritten with a new array, containing all the existing todos, plus the new one added onto the end. By overwriting the array, Glimmer is able to track that it changed. Changes inside of tracked arrays and objects aren't detected, only reassignments of the tracked property.
  3. We increment `newTodoId` so the next todo we add will have a unique ID.
  4. We clear out the `newTodoLabel` so the user can enter an additional todo.

After your app reloads, tap on the text field, enter a todo name, and tap "Add Todo". Note that the todo is added to the list. In particular, note that when we changed the values of `newTodoLabel` and `todos`, the component rerendered to display them on the screen.

Next, let's add the ability to complete a todo. For the purposes of this app, we'll just remove the todo when it's completed.

Edit `TodoGlimmerNative/template.hbs`:

```diff
{% raw %} {{#each this.todos key="id" as |todo|}}
-  <Label
-    text={{todo.label}}
-  />
+  <FlexboxLayout
+    flexDirection="row"
+    justifyContent="space-between"
+    alignItems="center"
+  >
+    <Label
+      text={{todo.label}}
+    />
+    <Button
+      text="Complete"
+      {{on "tap" (action completeTodo todo)}}
+    />
+  </FlexboxLayout>
 {{/each}}{% endraw %}
```

Instead of each todo being a single label, we now need to display a label on the left and a button on the right. To accomplish this, we use a `FlexboxLayout`. It's based on the CSS flexbox layout algorithm. In particular, here, we set the `flexDirection` to `row` so the items are side-by-side, and we set `justifyContent` to `space-between` so that the two items will be on the left and right, respectively. `alignItems` set to `center` means that they'll be centered top-to-bottom with regard to one another.

The second item we add is a `Button` hooked up to the `completeTodo` action. We pass the `todo` for the iteration of the loop into it.

The only change we need to make to `TodoGlimmerNative/component.ts` is to add this `completeTodo action`:

```diff
     this.newTodoId++
     this.newTodoLabel = ''
   }
+
+  completeTodo(todo) {
+    this.todos = this.todos.filter(testTodo => testTodo.id !== todo.id)
+  }
 }
```

We just filter out of `todos` the todo matching the ID that's passed in. This will remove the todo from the list.

When the app reloads, try adding and completing todos.

## Splitting Out Components

Our app is functional, but as our app grows, we definitely won't want everything in one big component. Instead, let's split our app into a few child components and see how we can pass data and actions around.

First, let's make a `NewTodoForm` component for the text field and Add button. Create a `components/NewTodoForm/` folder.

Add `NewTodoForm/template.hbs` and copy over the relevant components:

```handlebars
{% raw %}<TextField
  hint="New Todo"
  text={{this.newTodoLabel}}
  {{on "textChange" (action handleChangeText)}}
/>
<Button
  text="Add Todo"
  {{on "tap" (action addTodo)}}
/>{% endraw %}
```

Add `NewTodoForm/component.ts` with the related property and actions:

```javascript
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';

export default class NewTodoForm extends Component {
  @tracked
  newTodoLabel = ''

  handleChangeText(event) {
    this.newTodoLabel = event.value
  }

  addTodo() {
    const { onAdd } = this.args
    onAdd(this.newTodoLabel)
    this.newTodoLabel = ''
  }
}
```

The `addTodo` is a bit different than before, though. This component doesn't have access to the `todos` list or the `newTodoId`, so we can't create the todo itself here. Instead, we take a passed-in action argument and pass the label to it instead. In Glimmer component arguments are accessible on `this.args`.

Let's update `TodoGlimmerNative/template.hbs` to use `NewTodoForm`, passing the action to it:

```diff
{% raw %} <StackLayout>
+  <NewTodoForm @onAdd={{action addTodo}} />
-  <TextField
-    hint="New Todo"
-    text={{this.newTodoLabel}}
-    {{on "textChange" (action handleChangeText)}}
-  />
-  <Button
-    text="Add Todo"
-    {{on "tap" (action addTodo)}}
-  />

  {{#each this.todos key="id" as |todo|}}{% endraw %}
```

Now update `TodoGlimmerNative/component.ts` to remove the property and action that were moved to `NewTodoForm`, and update the `addTodo` action:

```diff
 export default class TodoGlimmerNative extends Component {
   title = "Welcome to Glimmer Native"

   newTodoId = 4
-
-  @tracked
-  newTodoLabel = ''

   @tracked
   todos = [
     { id: 1, label: 'Buy bread' },
     { id: 2, label: 'Buy milk' },
     { id: 3, label: 'Buy eggs' },
   ]
-
-  handleChangeText(event) {
-    this.newTodoLabel = event.value
-  }

-  addTodo() {
-    const newTodo = { id: this.newTodoId, label: this.newTodoLabel }
+  addTodo(label) {
+    const newTodo = { id: this.newTodoId, label }
     this.todos = [...this.todos, newTodo]
     this.newTodoId++
-    this.newTodoLabel = ''
   }

   completeTodo(todo) {
     this.todos = this.todos.filter(testTodo => testTodo.id !== todo.id)
   }
 }
```

Now `addTodo` receives the `label` as a parameter. We still construct the new todo object and add it to the array. The `newTodoLabel` property is no longer on the `TodoGlimmerNative` component so it doesn't need to be cleared out here anymore. Now we have a nice separation between the temporary state of the form and the core application state of the list of todos.

After the app reloads, confirm you can still add todos.

Next, let's extract the `TodoList` to a component as well. Create a `components/TodoList/` folder.

Create a `TodoList/template.hbs` file and copy over the loop:

```handlebars
{% raw %}{{#each @todos key="id" as |todo|}}
  <FlexboxLayout
    flexDirection="row"
    justifyContent="space-between"
    alignItems="center"
  >
    <Label
      text={{todo.label}}
    />
    <Button
      text="Complete"
      {{on "tap" (action completeTodo todo)}}
    />
  </FlexboxLayout>
{{/each}}{% endraw %}
```

Note that instead of iterating over `this.todos` we iterate over `@todos`--this is the Glimmer template syntax for accessing an argument passed to the component. Everything else is the same.

Next, create the corresponding `TodoList/component.ts` file and add the following:

```javascript
import Component from '@glimmer/component';

export default class TodoList extends Component {
  completeTodo(todo) {
    const { onComplete } = this.__owner__.args
    onComplete(todo)
  }
}
```

When the `completeTodo` action is called, we just call the passed-in `onComplete` action argument.

Finally, update `TodoGlimmerNative/template.hbs` to call `TodoList` and pass that action:

```diff
{% raw %} <StackLayout>
   <NewTodoForm @onAdd={{action addTodo}} />
+  <TodoList @todos={{this.todos}} @onComplete={{action completeTodo}} />
-
-  {{#each this.todos key="id" as |todo|}}
-    <FlexboxLayout
-      flexDirection="row"
-      justifyContent="space-between"
-      alignItems="center"
-    >
-      <Label
-        text={{todo.label}}
-      />
-      <Button
-        text="Complete"
-        {{on "tap" (action completeTodo todo)}}
-      />
-    </FlexboxLayout>
-  {{/each}}
 </StackLayout>{% endraw %}
```

The `completeTodo` action in `TodoGlimmerNative` doesn't need to be changed; it continues to work as-is.

When the app reloads, confirm you can still complete todos.

## Styling

Now that our app is functioning, let's apply a little bit of styling. We add classes to the components we want to style.

In `NewTodoForm/template.hbs`:

```diff
{% raw %} <TextField
   hint="New Todo"
+  class="new-todo-text"
   text={{this.newTodoLabel}}
   {{on "textChange" (action handleChangeText)}}
 />
 <Button
   text="Add Todo"
+  class="add-todo-button"
   {{on "tap" (action addTodo)}}
 />{% endraw %}
```

Then, in `TodoList/template.hbs`:

```diff
{% raw %}   <FlexboxLayout
     flexDirection="row"
     justifyContent="space-between"
     alignItems="center"
+    class="todo-row"
   >
     <Label
       text={{todo.label}}
+      class="todo-label"
     />
     <Button
       text="Complete"
+      class="todo-button"
       {{on "tap" (action completeTodo todo)}}
     />
   </FlexboxLayout>{% endraw %}
```

Now, in `app/app.css`, we add these styles:

```css
.new-todo-text {
  font-size: 16pt;
  margin: 5pt;
}

.add-todo-button {
  height: 44pt;
}

.todo-row {
  border-color: gray;
  border-bottom-width: 1pt;
  height: 44pt;
}

.todo-label {
  margin-left: 10pt;
  font-size: 16pt;
}

.todo-button {
  margin-right: 10pt;
}
```

Note that these rules are all normal CSS rules from the web. There are some slight differences, though: for example, the `border-bottom` shorthand doesn't work; you have to separately declare `border-color` and `border-bottom-width`. Also note that we use points instead of pixels as the unit. Unlike web browsers, NativeScript maps `px` to device pixels, so if we want to work with "normal" virtual pixels we need to use `pt`.

When the app reloads, there's a lot more breathing room and it looks nicer. With that, our app is done!

## Conclusion

Notice a few things about Glimmer-Native:

- We got a declarative UI: our UI was automatically updated each time we changed our data.
- We didn't have to follow a convention like calling a `useState` hook or setting up a `data` property; we just accessed properties on normal JavaScript objects. All we needed was a decorator to indicate which properties to track.
- We didn't need to explicitly import components; they were all automatically available from within templates.
- Portions of templates could be extracted directly into child components; we didn't need to wrap them with a containing element.

These are all really exciting possibilities for building apps productively. Give Glimmer-Native a try and [please provide feedback via GitHub Issues](https://github.com/bakerac4/glimmer-native/issues)!
