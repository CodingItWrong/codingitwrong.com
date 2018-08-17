At my software crafting meetup we read through _Growing Object-Oriented Software, Guided by Tests_. I’ve also presented its outside-in style of testing in several different contexts.

One piece of common feedback I get is that the approach of “write the code you wish you had” seems to contradict the typical “red/green/refactor” approach of writing the simplest thing you can to get the tests to pass, then refactoring to a better design under green. “Writing the code you wish you had” seems to be choosing the ultimate design from the start. How should we think about this difference?

I think this gets at a core difference between classical inside-out TDD and mockist outside-in TDD.

In inside-out TDD you are building out one domain object at a time. Typically you start with the lowest-level object. This doesn’t mean that you will definitely keep it as one object: in the refactoring step you may find that extracting a new object cleans up the code. But keeping it as one object is at least a possibility. If you regularly know for sure that you will be decomposing the object into others, that’s starting to operate in more of an outside-in style.

In outside-in development you start with an acceptance test using the system from a user’s perspective. Because the whole system is the scope, you are already pretty sure that you are going to be decomposing things into multiple objects. Otherwise your whole system would be one object!

Still, it could be argued, why decide on collaborators up-front instead of letting the tests drive you there?

There is still room for refactoring, just less extract-class refactoring. It’s not the extreme of “the simplest thing that could possibly work,” but rather 
