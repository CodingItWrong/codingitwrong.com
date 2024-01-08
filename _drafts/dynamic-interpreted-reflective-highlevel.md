---
title: The Case for Dynamic, Interpreted, Reflective, High-Level Languages
---

Recently someone asked me why people choose high-level programming languages. They pointed out that you can do categorically more in a machine-level language — for example, operating systems and embedded drivers. The implication was, why limit yourself to only the things that high-level languages can do? I can add a few more arguments for languages that tend to group together:

- Compiled languages are significantly faster at runtime than interpreted languages. Why would you hinder your performance like that?
- In statically-typed languages, the compiler gives you more information about errors you're making. Why would you not want to avail yourself of that information?
- In reflective languages, you can do all kinds of things to rewrite code at runtime to make it more difficult to reason about. Why would you not want the comprehensibility benefits that come from avoiding metaprogramming?

I was pretty sure I had an answer, but I couldn't think of it at the time. But as I've reflected, it came to me. In all these cases, the reason you wouldn't want these things is Extreme Programming. The benefits you get from incremental design, test-driven development, and refactoring are so significant that, for many application domains, they far outweigh the above costs.

Let's look at how dynamic, interpreted, reflective, high-level languages aid extreme programming.

- **Dynamically typed**: A dynamic type system supports refactoring. In a statically-typed language (even one with type inference), when you want to introduce a polymorphic variant for a type you already have, you need to change type annotations throughout your app to refer to the new interface instead of the concrete type. Even if you change the concrete type's name and use its former name as the name of the new interface, you still need to change the name at all the places that instantiate the concrete type. Also, unless your interfaces are extremely small, you have to implement a number of dummy methods on your new concrete class before you can even begin test-driving the first one. With dynamically-typed languages, you don't have any of this cost: all you have to do is introduce a new object that responds to the same relevant methods as the existing object, and you can test-drive one method at a time. This lower barrier to polymorphism means you'll be much more likely to reach for it, getting you the design flexibility advantages that polymorphism provides.

    - ["Types and Tests"](http://blog.cleancoder.com/uncle-bob/2017/01/13/TypesAndTests.html), Bob Martin

- **Interpreted**: Interpreted languages support extremely fast test cycles for test-driven development and refactoring. Instead of a compilation step, the interpreter only needs to run the relevant files, which is typically much faster. If you're doing true isolation testing that doesn't require booting a framework or connecting to an external resource like a database, individual tests can easily run in under 300 milliseconds. This enables you to move in much smaller steps. You can adhere more closely to the ideal of test-driven development, adding only enough production code to get past the current test failure. You can also refactor in much smaller, safer steps, because the overhead of running the tests after each step is so small. This makes it much more likely that you'll do test-driven development and refactoring, realizing their safety and design benefits.

- **Reflective**: While writing metaprogramming features in your application code is risky, it can be helpful for _frameworks_ to use to allow you to reduce duplication in your application code. For example, take [this database model class](https://vapor.github.io/documentation/fluent/model.html#full-model) from Vapor, a Swift web framework:

```
final class User: Model {
    var id: Node?
    var name: String

    init(name: String) {
        self.name = name
    }

    init(node: Node, in context: Context) throws {
        id = try node.extract("id")
        name = try node.extract("name")
    }

    func makeNode(context: Context) throws -> Node {
        return try Node(node: [
            "id": id,
            "name": name
        ])
    }
}
```

Notice the amount of duplication in the fields defined: they're declared as properties, set in the initializer, copied from a `Node` in one method and to a `Node` in another. In web frameworks in reflective languages, fields on model classes are usually set once, and initialization and database conversion is handled automatically. In some frameworks, the fields are dynamically read in from the database, and aren't defined in code at all! This reduction of duplication keeps knowledge of which fields exist in one place. If you add a new field, the compiler will catch if you don't initialize it as long as it's not nillable, but it won't catch it if you forget to set it in the method that converts the model to a `Node`. In addition to this safety benefit, DRYing out your code allows you to more quickly make changes, making it easier for you to change the design of your application.

    - _Refactoring_, Martin Fowler

- **High-level**: High-level language code is more intention-revealing because it describes high-level operations, not low-level ones. Details about memory storage and iteration are hidden, and what you see is business logic. It's true that in lower-level languages you can create small helper methods to hide some of these operations — but in practice this isn't done as often as in high-level languages. This makes it much more likely that you'll have longer methods, which make the code even less intention-revealing: you have to read through longer methods to understand what exactly they're doing. This limitation slows down new developers' ability to learn the code, as well as your own ability when you haven't worked on a particular piece of code for a while. Longer methods also limit your ability to refactor your code, because you're less likely to make changes when you don't understand its operation as well, and larger methods make it more difficult to see potential refactorings.

    - The entirety of _Smalltalk Best Practice Patterns_, Kent Beck
