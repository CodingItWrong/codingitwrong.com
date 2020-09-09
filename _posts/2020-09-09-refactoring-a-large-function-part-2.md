---
title: "Refactoring a Large Function, part 2: The Closure Approach"
---

In [my last post](/2020/09/08/refactoring-a-large-function.html) we refactored a large function using the Replace Function with Command refactoring to create a class. This allowed us to create small methods that called one another without needing to pass a lot of repeated arguments. Some JavaScript developers prefer to stay away from ECMAScript classes, though.

Luckily, the Replace Function with Command refactoring can be applied in a different way, using function scopes instead of classes. Let’s go through the refactoring again, this time using function scope. At the end we’ll compare and contrast the two approaches.

## Replace Function With Command (Scope)
If you need a refresher on the function we want to refactor, check out [the part 1 blog post](/2020/09/08/refactoring-a-large-function.html).

The easiest way to split up a large function is to use the Extract Function refactoring. But there is a lot of data flying around, so if I extracted functions I would need to pass the same repeated arguments to them. Instead, I’ll use Replace Function with Command—with a twist. Instead of using an ECMAScript class, I’m going to use a function to create a scope for variables that inner functions will use.

I already have the `day` available in the scope of this function, so I’m going to take advantage of this by creating and calling child functions that will also have access to it.

To start, we’ll wrap the entire contents of `schedulingsGroupedByStudent()` in an inner function and call it. This will allow us to decompose this inner function into other smaller inner functions.

```js
function schedulingsGroupedByStudent(day) {
  return groups();

  function groups() {
    const students = sortBy(
      uniq([
        ...day.studentDays.map(property('student')),
        ...day.schedulings.map(property('student')),
      ]),
      'name',
    );

    const groups = students.map(student => {
      const studentDay = day.studentDays.find(
        matchesProperty('student.id', student.id),
      );

      const studentContents = studentDay?.contentDay?.contents ?? [];

      const contentSchedulingPairs = uniqBy(
        [
          // schedulings must come first so uniqBy prefers them
          ...day.schedulings
            .filter(matchesProperty('student.id', student.id))
            .map(scheduling => ({
              content: scheduling.content,
              scheduling,
            })),
          ...studentContents.map(content => ({
            content,
            scheduling: null,
          })),
        ],
        property('content.id'),
      );

      return {
        student,
        studentDay,
        contentSchedulingPairs,
      };
    });

    return groups;
  }
}
```

Note that we define `groups()` using the `function` keyword instead of the arrow function syntax (`=>`). We do this so we can call `groups()` as the first line of the function and define it down below. Otherwise, the call to it would need to come at the end, and it would be much harder to find at a skim. A common way to describe functions defined with the `function` keyword is that they are “hoisted” to the top of the scope they’re in, so they are available to be called before the line where they're declared. But arrow functions aren't hoisted. (If this aspect of JavaScript is interesting to you, I’d highly recommend the video course [Deep JavaScript Foundations](https://frontendmasters.com/courses/deep-javascript-v3/) by Kyle Simpson; I learned a ton from it.)

I have the tests running in the background, so when I save the changes I see that the tests still pass.

## Extract Function
Now we’re ready to look through the `groups()` function to see other functions we could extract from it. The first chunk of code we see is this:

```js
const students = sortBy(
  uniq([
    ...day.studentDays.map(property('student')),
    ...day.schedulings.map(property('student')),
  ]),
  'name',
);
```

This is retrieving a list of unique students from two different parts of the data structure. This could be pulled out into another function using the Extract Function refactoring. To do this, first we create the new function in the same scope as `groups()`:

```js
function getStudents() {
}
```

Then we copy the statements over and return the result:

```js
function getStudents() {
  const students = sortBy(
    uniq([
      ...day.studentDays.map(property('student')),
      ...day.schedulings.map(property('student')),
    ]),
    'name',
  );
  return students;
}
```

We replace the executing code with a call to this function:

```diff
   function groups() {
-    const students = sortBy(
-      uniq([
-        ...day.studentDays.map(property('student')),
-        ...day.schedulings.map(property('student')),
-      ]),
-      'name',
-    );
+    const students = getStudents();
```

We save and the tests pass. Now we can simplify the `getStudents()`  function a bit. We can use the Inline Variable refactoring to return the result of `sortBy()` directly instead of assigning to a temp variable:

```diff
   function getStudents() {
-    const students = sortBy(
+    return sortBy(
       uniq([
         ...day.studentDays.map(property('student')),
         ...day.schedulings.map(property('student')),
       ]),
       'name',
     );
-    return students;
  }
```

We save and the tests pass.

Next, looking at the `groups()` function, there is only one place the `students` variable is used. So we can the Inline Variable refactoring again here, to replace the usage of that variable with the function call:

```diff
   function groups() {
-    const students = getStudents();
-
-    const groups = students.map(student => {
+    const groups = getStudents().map(student => {
```

Now that there is not a `students` variable, we can rename the `getStudents()` function to `students()` for simplicity:

```diff
   function groups() {
-    const groups = getStudents().map(student => {
+    const groups = students().map(student => {
      const studentDay = day.studentDays.find(
...
-  function getStudents() {
+  function students() {
    return sortBy(
```

Now we can see that the body of `groups()` is just one statement to create the `groups` variable, and then returning it. So we can use Inline Variable yet again here:

```diff
   function groups() {
-    const groups = students().map(student => {
+    return students().map(student => {
...
     });
-
-    return groups;
  }

```

This last change is an example of the cumulative power of refactoring. After you do one round of refactorings, more options for refactoring become visible to you.

## Extract Student Functions
`groups()` is still pretty long, so let’s see what else we can do to split it up. Often it’s good to have separate functions to define (1) what iteration occurs and (2) what is performed during the iteration. In this case, for each student, what are we mapping to? We are mapping to the group for the student. So let’s extract a `groupForStudent()` function in the same scope as `groups()`:

```js
function groups() {
  return students().map(groupForStudent);
}

function groupForStudent(student) {
  const studentDay = day.studentDays.find(
    matchesProperty('student.id', student.id),
  );

  const studentContents = studentDay?.contentDay?.contents ?? [];

  const contentSchedulingPairs = uniqBy(
    [
      // schedulings must come first so uniqBy prefers them
      ...day.schedulings
        .filter(matchesProperty('student.id', student.id))
        .map(scheduling => ({
          content: scheduling.content,
          scheduling,
        })),
      ...studentContents.map(content => ({
        content,
        scheduling: null,
      })),
    ],
    property('content.id'),
  );

  return {
    student,
    studentDay,
    contentSchedulingPairs,
  };
}
```

Looking at `groupForStudent()`, the temporary variables we’re using show that we compute three different values, then use those to return a result. Let’s split each of those computations out into its own function. First, `getStudentDay()`:

```js
function groupForStudent(student) {
  const studentDay = getStudentDay(student);
//...
}

function getStudentDay(student) {
  return day.studentDays.find(matchesProperty('student.id', student.id));
}
```

Note that the `studentDay` variable is used for two things: it’s used directly in `groupForStudent()` as part of the return value, and it’s used to compute `studentContents`. To get the code well-factored, we’ll call `getStudentDay()` in both places; after some more refactoring, we may consider a performance improvement.

First we find the two places the `studentDay` variable is used and we inline the call to `studentDay()` at each: this will make future refactoring easier.

```diff
   function groupForStudent(student) {
-    const studentDay = getStudentDay(student);
-
-    const studentContents = studentDay?.contentDay?.contents ?? [];
+    const studentContents = getStudentDay(student)?.contentDay?.contents ?? [];
...
     return {
       student,
-      studentDay,
+      studentDay: getStudentDay(student),
       contentSchedulingPairs,
     };
   }
```

Now that the `studentDay` variable is gone we can rename `getStudentDay()` to `studentDay()`:

```diff
   function groupForStudent(student) {
-    const studentContents = getStudentDay(student)?.contentDay?.contents ?? [];
+    const studentContents = studentDay(student)?.contentDay?.contents ?? [];
...
     return {
       student,
-      studentDay: getStudentDay(student),
+      studentDay: studentDay(student),
       contentSchedulingPairs,
     };
   }
...
-  function getStudentDay(student) {
+  function studentDay(student) {
     return day.studentDays.find(matchesProperty('student.id', student.id));
   }
```

Next, we extract a `studentContents()` function:

```js
function studentContents(student) {
  return studentDay(student)?.contentDay?.contents ?? [];
}
```

Then we remove the `studentContents` variable and replace its usages with direct calls to the `studentContents()` function:

```diff
   function groupForStudent(student) {
-    const studentContents =
-      studentDay(student)?.contentDay?.contents ?? [];
-
     const contentSchedulingPairs = uniqBy(
...
-        ...studentContents.map(content => ({
+        ...studentContents(student).map(content => ({
```

Finally, we extract the `contentSchedulingPairs` variable to a function as well. We can go ahead and inline it right away:

```js
function groupForStudent(student) {
  return {
    student,
    studentDay: studentDay(student),
    contentSchedulingPairs: contentSchedulingPairsForStudent(student),
  };
}

function contentSchedulingPairsForStudent(student) {
  return uniqBy(
    [
      // schedulings must come first so uniqBy prefers them
      ...day.schedulings
        .filter(matchesProperty('student.id', student.id))
        .map(scheduling => ({
          content: scheduling.content,
          scheduling,
        })),
      ...studentContents(student).map(content => ({
        content,
        scheduling: null,
      })),
    ],
    property('content.id'),
  );
}
```

Now `groupForStudent()` is a nice single level of abstraction. We can see at a glance that the group for a student is an object with three keys: the student, the student day, and the content scheduling pairs. If we want to dig into how the student day or content scheduling pairs are retrieved, we can look at the corresponding function.

`contentSchedulingPairsForStudent` still has some complexity, so it could be good to decompose further. But before we do, I’m starting to feel some hesitation.

## Combine Functions into Class
We now have four inner function that take a `student` as an argument:

- `groupForStudent(student)`
- `studentDay(student)`
- `studentContents(student)`
- `contentSchedulingPairsForStudent(student)`

Avoiding duplicate argument passing is why we applied the Replace Function With Command refactoring in the first place. But now we’re running into duplicate arguments again. This is because these functions all operate on a single student, rather than on all the students in the day. This is an evidence of lack of cohesion among the inner functions: one group operates on a `student`, and another group does not.

Combine Functions Into Class is a refactoring that allows us to address this situation. It describes taking the functions that all take the same arguments, move them into a class, and then allow them to access those arguments as instance properties instead. In our case, instead of using ECMAScript classes, we’ll use a second nested function scope instead.

First we create an empty `group(student)` function in the same scope as `groups()`:

```js
function group(student) {
}
```

Next, we copy the functions that take a `student` argument into it:

```js
function group(student) {
  function groupForStudent(student) {
    return {
      student,
      studentDay: studentDay(student),
      contentSchedulingPairs: contentSchedulingPairsForStudent(student),
    };
  }

  function studentDay(student) {
    return day.studentDays.find(matchesProperty('student.id', student.id));
  }

  function studentContents(student) {
    return studentDay(student)?.contentDay?.contents ?? [];
  }

  function contentSchedulingPairsForStudent(student) {
    return uniqBy(
      [
        // schedulings must come first so uniqBy prefers them
        ...day.schedulings
          .filter(matchesProperty('student.id', student.id))
          .map(scheduling => ({
            content: scheduling.content,
            scheduling,
          })),
        ...studentContents(student).map(content => ({
          content,
          scheduling: null,
        })),
      ],
      property('content.id'),
    );
  }
}
```

Next, in the first line of `group()`, we call `groupForStudent()`, passing in the student:

```diff
   function group(student) {
+    return groupForStudent(student);
+
     function groupForStudent(student) {
```

Next, we replace the call to `groupForStudent()` in the original function to `group()`:

```diff
   function groups() {
-    return students().map(groupForStudent);
+    return students().map(group);
   }
```

We save and the tests pass.

Now we can remove the functions from the scope of `schedulingsGroupedByStudent` that we’ve copied into the scope of `group`, as they’re no longer called in the original scope. We remove `groupForStudent`, `studentDay`, `studentContents`, and `contentSchedulingPairsForStudent` from the scope created by `schedulingsGroupedByStudent`. We save and the tests pass.

Next, since the `student` argument to `groups()` is in scope to the child functions, we don’t need to pass it to them, fixing the argument-passing problem. One function at a time, we update them to remove their `student` argument. Since `groupForStudent()` takes the argument and forwards it along to children, we shouldn’t change `groupForStudent()` first as it will break the children; we should change the others first.

First, `studentDay()`:

```diff
-  function studentDay(student) {
+  function studentDay() {
     return day.studentDays.find(matchesProperty('student.id', student.id));
  }
```

We save and confirm the tests pass, then move on to `studentContents()`. In this case it was just forwarding the `student` argument along to `studentDay()`, so we can just remove the incoming and outgoing argument:

```diff
-  function studentContents(student) {
+  function studentContents() {
-    return studentDay(student)?.contentDay?.contents ?? [];
+    return studentDay()?.contentDay?.contents ?? [];
  }
```

Next, `contentScheduingPairsForStudent`:

```diff
-  function contentSchedulingPairsForStudent(student) {
+  function contentSchedulingPairsForStudent() {
     return uniqBy(
       [
         // schedulings must come first so uniqBy prefers them
         ...day.schedulings
           .filter(matchesProperty('student.id', student.id))
           .map(scheduling => ({
             content: scheduling.content,
             scheduling,
           })),
-        ...studentContents(student).map(content => ({
+        ...studentContents().map(content => ({
           content,
           scheduling: null,
         })),
       ],
       property('content.id'),
     );
   }
```

Now we can go back to `groupForStudent()` and remove the argument:

```diff
-  function groupForStudent(student) {
+  function groupForStudent() {
     return {
       student,
-      studentDay: studentDay(student),
+      studentDay: studentDay(),
-      contentSchedulingPairs: contentSchedulingPairsForStudent(student),
+      contentSchedulingPairs: contentSchedulingPairsForStudent(),
    };
  }
```

We save and the tests pass. Now we can remove the `student` argument where `groupForStudent()` is called:

```diff
   function group(student) {
-    return groupForStudent(student);
+    return groupForStudent();

     function groupForStudent() {
```

Notice how simple `groupForStudent()` is at this point:

```js
function groupForStudent() {
  return {
    student,
    studentDay: studentDay(),
    contentSchedulingPairs: contentSchedulingPairsForStudent(),
  };
}
```

We return a record with `student`, `studentDay`, and `contentSchedulingPairs`. `student` is an instance property, and the other two have functions to compute them.

Now we can proceed with extracting more functions from `contentSchedulingPairsForStudent()`. Let’s look at it again:

```js
function contentSchedulingPairsForStudent() {
  return uniqBy(
    [
      // schedulings must come first so uniqBy prefers them
      ...day.schedulings
        .filter(matchesProperty('student.id', student.id))
        .map(scheduling => ({
          content: scheduling.content,
          scheduling,
        })),
      ...studentContents().map(content => ({
        content,
        scheduling: null,
      })),
    ],
    property('content.id'),
  );
}
```

There are two expressions we combine into one array, then remove the duplicates. What do those two expressions represent? Both are creating “content scheduling pair” records, which have a `content` property and a `scheduling` property. One is creating pairs from the `schedulings`, and the other is creating them from the `contents`. So let’s extract a function for each of these in the same scope as `contentSchedulingPairsForStudent`.

The first we can call `contentSchedulingPairsFromSchedulings`:

```js
function contentSchedulingPairsFromSchedulings() {
  return day.schedulings
    .filter(matchesProperty('student.id', student.id))
    .map(scheduling => ({
      content: scheduling.content,
      scheduling,
    }));
}
```

We can call this in `contentSchedulingPairs()`:

```diff
   function contentSchedulingPairs() {
     return uniqBy(
       [
         // schedulings must come first so uniqBy prefers them
-        ...day.schedulings
-          .filter(matchesProperty('student.id', student.id))
-          .map(scheduling => ({
-            content: scheduling.content,
-            scheduling,
-          })),
+       ...contentSchedulingPairsFromSchedulings(),
        ...studentContents().map(content => ({
          content,
          scheduling: null,
        })),
      ],
      property('content.id'),
    );
  }
```

Now we can do the same for `contentSchedulingPairsFromContents`:

```js
function contentSchedulingPairsFromContents() {
  return studentContents().map(content => ({
    content,
    scheduling: null,
  }));
}
```

```diff
   function contentSchedulingPairs() {
     return uniqBy(
       [
         // schedulings must come first so uniqBy prefers them
        ...contentSchedulingPairsFromSchedulings(),
-       ...studentContents().map(content => ({
-         content,
-         scheduling: null,
-       })),
+       ...contentSchedulingPairsFromContents(),
      ],
      property('content.id'),
    );
  }
```

After this, `contentSchedulingPairsForStudent` is a nice single level of abstraction as well:

```js
function contentSchedulingPairsForStudent() {
  return uniqBy(
    [
      // schedulings must come first so uniqBy prefers them
      ...contentSchedulingPairsFromSchedulings(),
      ...contentSchedulingPairsFromContents(),
    ],
    property('content.id'),
  );
}
```

## Review
Let’s review the final state of the code:

```js
function schedulingsGroupedByStudent(day) {
  return groups();

  function groups() {
    return students().map(group);
  }

  function students() {
    return sortBy(
      uniq([
        ...day.studentDays.map(property('student')),
        ...day.schedulings.map(property('student')),
      ]),
      'name',
    );
  }

  function group(student) {
    return groupForStudent();

    function groupForStudent() {
      return {
        student,
        studentDay: studentDay(),
        contentSchedulingPairs: contentSchedulingPairsForStudent(),
      };
    }

    function studentDay() {
      return day.studentDays.find(matchesProperty('student.id', student.id));
    }

    function studentContents() {
      return studentDay()?.contentDay?.contents ?? [];
    }

    function contentSchedulingPairsForStudent() {
      return uniqBy(
        [
          // schedulings must come first so uniqBy prefers them
          ...contentSchedulingPairsFromSchedulings(),
          ...contentSchedulingPairsFromContents(),
        ],
        property('content.id'),
      );
    }

    function contentSchedulingPairsFromSchedulings() {
      return day.schedulings
        .filter(matchesProperty('student.id', student.id))
        .map(scheduling => ({
          content: scheduling.content,
          scheduling,
        }));
    }

    function contentSchedulingPairsFromContents() {
      return studentContents().map(content => ({
        content,
        scheduling: null,
      }));
    }
  }
}
```

Now each of our functions is pretty simple; the longest one is eight lines, including a comment. Our tests are still passing, so we know it works. Let’s see if this helps us understand our code:

- Our externally-facing API is still the `schedulingsGroupedByStudent()` function.
- That function now delegates to an inner `groups()` function.
- The `groups()` function gets an array of `students()` and, for each one, it uses the `group()` function to get the group for that student.
- The `group()` function delegates to an inner `groupForStudent()` function, which returns an object with three fields: `student`, `studentDay`, and `contentSchedulingPairs`. Of the three, `contentSchedulingPairs` is the most interesting. Its data comes from `contentSchedulingPairsForStudent()`.
- `contentSchedulingPairsForStudent()` gets pairs from two sources: from schedulings and from contents. Then it removes any duplicates.
- The way pairs are derived from schedulings and contents is defined in two different functions.

This a nice stepwise way we can understand the functionality here. We don’t need to read a long procedural function and try to understand the intent of each step. We have created functions that give names to each concept. And because we’re using nested function scope, we don’t need to pass a lot of arguments to the functions; they have access to variables in the outer scope and to call other functions to get the information they need.

## Meta-Review
How does this scope-based refactoring compare to [the class-based refactoring in part 1](/2020/09/08/refactoring-a-large-function.html)?

The benefits to the scope-based approach I see are:

- There are much fewer steps to split a function into inner functions, than to turn it into a class. No defining constructors or instance variables; you just add a function.
- You don’t need to change references to a variable to reference an instance property on `this`, because it’s always plain variables you’re referencing.
- Objects created by classes typically have multiple public-facing methods, but we don't intend to expose multiple methods. We just want to run one operation. So a function is a more natural fit. ECMAScript and Babel have added support for private methods, but [there are tooling issues](https://github.com/babel/babel-eslint/issues/728#issuecomment-568548685) getting private methods working that have not yet been addressed. Even once they are, a single function is a more natural fit than an object that exposes one public method. We are mitigating this by wrapping the class in an outer function to not expose this outside the module, but this is just hiding the mismatch inside the module.
- The inner helper functions aren't available to be called publicly. With the class, since we don't have private methods, the helper methods that aren't intended to be called publicly are still available to do so, which could cause confusion. Again, wrapping the class in a module hides but does not fully fix this problem.
- If you want to avoid ECMAScript classes and `this`, this approach allows you to do so. (I don’t feel that need.)

But there are benefits to the class-based approach as well:

- Defining a class gives you a convenient way to communicate the purpose of it. Talking about the purpose of the nested scopes is a bit trickier.
- It’s tricky to name the “main” inner function that the outer function calls; I kind of want to name it the same as the outer function.
- It’s tempting to put functionality in the outer function, which would clutter it up and make it harder to see the nested structure.
- Just syntactically, when you see a `function`, you don’t know at a glance if it’s intended to create a scope or to execute functionality; you have to read it to see.
- Two separate parallel classes make it easy to see which class a method is in. With the scope-based approach, because the second level of scope has to be nested, it’s difficult to tell at a glance what level of scope a given function is nested within.
- In the inner functions of the `group(student)` function, I felt like I needed to keep the word “student” in their name, because it was not obvious that they are part of a context that exposes a student. By contrast, with the `StudentSchedulingGrouper` class, I felt like I could remove the word “student”, because it was easy to see the class name.
- There are fewer issues with name collisions between variables and extracted methods, because methods are called on the `this` context, so they can be named the same as instance variables.
- Classes use prototypal inheritance to reuse the same method code for each instance. By contrast, with the scope approach, the inner functions are redefined each time the outer function executes. This isn’t a dealbreaker for all applications (in particular, modern React applications are architected to recreate inner functions each time a component renders), but may be expensive for large data sets.

Which approach would I use? On a team, I would choose based on what is common in the framework ecosystem we’re working in, what the team is familiar with, and where we want to develop our skills as a team. For a personal project, it would probably come down to which of the two approaches I wanted to explore more. At the moment, the code-communication benefits of the class-based approach are appealing to me, so I would probably lean that direction.

Whichever of the two refactoring approaches you take, you get the same benefits of the Replace Function With Command refactoring: breaking up a complex function into smaller ones at separate levels of abstraction without a lot of arguments passed around. I would encourage you to look at the long functions in your app and think about if any of them are costing you in terms of effort to understand and modify. If so, either form of the Replace Function With Command refactoring could help.

If you’d like to learn more about this kind of small-step refactoring, there are two great JavaScript resources:

- [*Refactoring*](https://martinfowler.com/books/refactoring.html) is the original book that described this process. It’s a reference work that describes many refactorings in detail. Its second edition was rewritten in JavaScript, making it extra accessible.
- [*99 Bottles of OOP*](https://sandimetz.com/99bottles) is an extended walkthrough of a refactoring process of one bit of code, and the benefits that can come from it. Its second edition added a JavaScript version of the book, so it’s also easily accessible.

(I don’t receive any referral fees for these links, they’re just great books I want folks to benefit from.)
