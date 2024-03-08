---
title: "Simple Design: Fewest Elements"
---

_This is the third post in a [series about the Four Rules of Simple Design](/2024/03/06/simple-design-reveals-intention.html)._

"Fewest elements" means that if there is a way to accomplish the same functionality with fewer programming elements, prefer the option with fewer.

## Many examples of fewest elements

Examples of unnecessary elements you can remove include:

**Unreachable code.** Most modern programming languages have code linters that can alert you to unreachable code. Leaving in unreachable code is confusing to the reader, because it suggests that it’s needed, but it’s not. You could put anything in there that compiles and it wouldn’t affect the behavior of the program.

**Commented-out code.** Sometimes developers will leave in commented-out code just because they didn’t think to delete it when they finished their unit of work. Delete it now. If you leave it, a future developer will have more uncertainty than you about whether it is needed for something, and it will stay in the codebase forever. Since you’re using version control, you can always get deleted code back. (If you aren’t sure how to use your version control tool to get deleted code back, delete this code and then take this opportunity to learn; it’s a good skill to have!) Sometimes developers will comment out a unit test that they know they need, but is currently failing, and they can’t fix it yet but plan to soon. That may be a reason to leave in a commented-out test with a `TODO` comment. But it’s even better if the test tool has a way to skip a test or to mark that it’s expected to fail—for example, [Jest’s `.skip()`](https://jestjs.io/docs/api#testskipname-fn) or [RSpec’s pending and skipped features](https://rspec.info/features/3-13/rspec-core/pending-and-skipped-examples/). These are better than commenting out the code because the test runner will report them as a reminder that they exist.

**Code that is not currently used.** If there are functions or branches in your codebase that aren’t currently being called by the running app, delete them. If you need them later, remember that you can get deleted code back from version control. Keeping them in the codebase incurs an ongoing cost: the cost of reading over them, the risk that they don’t actually work right because they aren’t being called to verify, and the cost of changing them when you do other refactorings in the code.

**Comments that don’t add value.** As mentioned in [Reveals Intention](/2024-03-06-simple-design-reveals-intention.html), if the code can be updated to communicate the same thing a comment does, update the code and then remove the comment. And if you have comments that just duplicate what the code already says, just delete them—unless they’re used to generate API documentation that’s *actually* used by somebody.

**Unnecessary low-level code.** If a higher-level abstraction exists and works for your purposes, use it instead of reimplementing it. If you’re writing a loop by hand that is the equivalent of a `map` or `filter` function, that’s unnecessary complexity in your code; just use `map` or `filter` function directly.

**Unused abstractions and indirection.** Some design approaches involve thinking in advance about the ways a system will need to change, and adding in an inheritance hierarchy or configuration points to support all of them. The problem is that programmers are notoriously bad fortune tellers; we can’t know the future. When a codebase contains abstractions and indirection that aren’t used, they make the codebase harder to understand and harder to change. They only provide benefit once they begin being used—if in fact they ever are. It's better to remove abstractions and indirection that aren't currently being used. Instead, to prepare for the future, set up your code to be easy to change so that you can add in just the bits of flexibility that are needed *when* they're needed.

**Unneeded syntax elements.** This one may not be very clear, so let me give a variety of examples: a variable you assign a value to but then don't use the value. A return value that isn't used. Calling a function that returns a value you ignore, instead of calling an equivalent function that doesn't return a value (`map` instead of `forEach`). A catch block that simply re-throws the error. In JavaScript, `await`ing the result of a promise and then returning the result unchanged (since an `async` function returns a promise, this is the same as if you just returned the promise directly). In all these cases the program functions exactly the same without the element I mentioned. If this happens, you should remove that unnecessary element.

# Fewer elements, fewer problems

If you apply the Fewest Elements rule in this way, the result is that every syntax element in your program has to justify its existence. If it is removed, something in the program should break, and a test should report that.

Why does Fewest Elements matter? Is it just an aesthetic preference? No. Keeping unnecessary elements in the codebase incurs an ongoing cost: the cost of reading over them, the cost of intent being less clearly revealed, the cost of having more code to change when you do other refactorings.

For this rule, there is an extreme you can take it to: [code golf](https://en.wikipedia.org/wiki/Code_golf), where you attempt to shorten some code to the fewest possible characters. Like real golf, code golf can be fun recreationally, but it's not a good idea to do it in an office. Fewest Elements is not quite the same as code golf because the latter focuses on fewest *characters*, not elements. The reason not to go so far as focusing on minimizing characters is that it can hinder Revealing Intent. Code that has been golfed works, but it is often much harder to understand (see examples from the [International Obfuscated C Code Contest](https://www.ioccc.org/)). The goal of Fewest Elements is to remove elements that don't add value, not to remove elements that add clarity. For example, you might have a complex calculation used in only one place, but instead of using it directly, you assign it to an explaining variable to give it a descriptive name. That variable is not functionally necessary, but it is valuable to reveal intent.

While I've contrasted a beneficial focus on Fewest Elements and detrimental code golfing, that is not to suggest that the difference will always be clear. There are tensions around the rules of Reveals Intention, No Duplication, and Fewest Elements, and you'll need to make judgment calls for yourself or among your team about what is best in a given situation. In cases like this, these rules give you a way to see some of the tradeoffs so you can make an informed decision.
