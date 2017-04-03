---
title: Picking a Guard Clause Form
---

Every time I have to write a [guard clause](http://wiki.c2.com/?GuardClause) I find myself thinking a lot about what form to write it in. This might just be obsession with detail, but it's _more_ obsession with detail than I have with other code constructs, so I think there's something to it. The whole point of a guard clause is readability: taking a special case and handling it quickly so you can get to the meat of the method. If we can find ways for guard clauses to more effectively disappear, our code will be better off. Ruby has a wide variety of ways you can write a guard clause, and Swift has one particular unique take. Let's look at some examples.

# Block If
Ruby:

```ruby
if unexpected_condition
  return_or_raise
end
```

Swift:

```swift
if unexpectedCondition {
    returnOrThrow
}
```

This is the most basic form of guard clause, and works in just about every modern programming language. You immediately know you're guarding if you see an `if` block at the top of a method and inside it a statement that exits the current scope (`return`, `raise`/`throw`, or `fatalError`/`preconditionFailure`). Being able to see the exiting keyword is important, and even if the condition is long, it's easy to see because it's on a line of its own.

This form is actually not the most common in either Ruby or Swift, because they each have more intention-revealing alternatives.

# Block Unless

Ruby:

```ruby
unless expected_condition
  return_or_raise
end
```

In keeping with Ruby's pattern of having more than one way to do things, there is an `unless` keyword that functions like `if !`: if the condition is false, the block is executed, and you can use this for a guard clause.

Unless blocks never seemed clear to me, for guarding or any other purpose. Reading `unless` at the start of a sentence was never very clear. Plus, it sounds like the body of the block is the expected thing to be done, *unless* some special condition holds, and that would be the opposite of a guard clause.

But Rubyists don't tend to do `if !condition` either: they will often make a small helper method to represent the opposite of the condition. For example, if there is a `complete?` predicate method, they might write the following:

```ruby
def incomplete?
  !complete?
end
```

so that you could do `if incomplete?`

# Block Guard

```swift
guard expectedCondition else {
  returnOrThrow
}
```

Swift has a special syntax for guard clauses. It functions like an `unless` structure in Ruby: a condition is checked, and the block is executed if the condition is false. I find it much clearer to understand: "guard that this is true; if not, do this block." The difference with the `guard` keyword is that the compiler ensures you don't allow the flow of control to continue out of its block; you need to either return a value, throw an error, or halt execution. The upshot of this is that when you see the `guard` keyword you don't need to look for the `return` or `throw` keyword; you already know it's functioning as a guard clause.

Like many Swift developers, I reach for the `guard` keyword first when I need a guard clause, but if the conditional needs to be negated (`guard !condition`), I might prefer an `if` statement (`if condition`). This might be a bit surprising for other devs who might expect the `guard` keyword, but if it aids readability enough, I'll go for it.

# One-line If, Unless, Guard

Ruby:

```ruby
if unexpected_condition then return_or_raise end
unless expected_condition then return_or_raise end
```

Swift:

```swift
if unexpectedCondition { returnOrThrow }
guard expectedCondition else { returnOrRaise }
```

`if`, `unless`, and `guard` constructs can all be written with their block on the same line as the conditional. Ruby blocks can always be written with either `do`/`

You could technically do this even if there were multiple statements in the block, but I've only ever seen it done with a single statement. It's harder to find the `return` or throw statement because it's far to the right, so I have hesitations about using the if or unless variants; but because the guard keyword ensures you exit scope, I'm much more amenable to that one.

I definitely wouldn't use single-line format for anything long, like complex conditionals or errors with init parameters. You have to be able to see the return or throw at a glance, which means it needs to be very short.

Stylistically I'd be less likely to use a single-line guard clause in languages that require a semicolon inside the block. And I wouldn't use these in Ruby because it has a number of other one-line constructs that are more common. This is likely because the `then`/`end` keywords make it more verbose than the others.

# Statement Modifiers

Ruby:

```ruby
return_or_raise if unexpected_condition
return_or_raise unless expected_condition
```

Ruby inherits a syntax from Perl where you can add an `if` or `unless` to the end of a statement to execute it conditionally. It effectively switches the order of a conditional, putting the check at the end. Because of this, the condition is de-emphasized, and the exceptional action is emphasized. You would think this would make it harder to tell that it was a guard clause, but because they end up starting with `return` or `raise`, it's not so bad. If you see a `return` or `throw` keyword at the top of a method and there is code below it, you know it must be conditional, because otherwise the code below would never be executed.

Their readability is increased if you keep the value returned or thrown as simple as possible, so it's easy to see the if or unless:

```ruby
return nil if unexpected_condition
raise ArgumentError unless expected_condition
```

Or, even better, if you have no argument to return:

```ruby
return if unexpected_condition
```

In this case I'm in favor of the `unless` just as much as the `if`: it reads naturally to me.

# Low-Precedence Boolean Operators

Ruby:

```ruby
expected_condition or return_or_raise
unexpected_condition and return_or_raise
```

This syntax is also inherited in Ruby from Perl: simple Perl scripts often tacked `or die` on to the end of lines of code.

These constructs emphasize the condition rather than the response, but it's harder to tell at a glance that a check is being performed because there is no keyword. The one thing that can help is that Ruby is so concise: often the condition is just one predicate method on `self`, and the response is an empty or default value `return`, or raising an error with no params. A four-word line isn't too bad:

```ruby
valid? or raise ValidationError
```

I would use the `or` variant more likely than the `and`. The `expected_condition` reads like you're stating reality, but with the unexpected condition it's strange. Even then I would prefer conditional modifiers because the `return`/`raise` at the start makes them clearer.

# So In Conclusion

In Ruby I prefer to use, in priority order:

```ruby
return_or_raise if unexpected_condition
return_or_raise unless expected_condition
expected_condition or return_or_raise
```

In Swift I prefer to use, in priority order:

```swift
guard expected_condition else {
  return_or_throw
}

guard expected_condition else { return_or_throw }

if unexpected_condition {
  return_or_throw
}
```

_Thanks to [@rossta](https://twitter.com/rossta) for some Ruby syntax corrections!_
