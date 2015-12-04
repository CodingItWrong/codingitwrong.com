We've talked about a number of patterns that can be added to Rails to separate out concerns from your models, views, and controllers. But what does the Rails core team think about this? Do they encourage you to reorganize your app any way you like? Are they working to add decorators and form objects to Rails 5? Well, no. Rails *is* gradually adding new patterns, like Active Job and Action Cable, but they're about more than just code organization. They're about making apps work in entirely new ways: queues and persistent client connections, respectively.

On the topic of code organization, the message from DHH has generally been that what Rails gives you should be enough. He objects to some patterns more strenuously than others. But when it comes to code organization, I've heard him generally give two pieces of advice: use Rails classes the way they're intended, and use concerns.

Concerns are effectively just mixins with a little extra Rails-specific sugar. They allow you to take related functionality in a class (specifically, a controller or model) and put it in its own module, that is then included back into the original class. Specifically, DHH argues for domain-related concerns. So instead of putting all validations in one concern, then all callbacks in another, etc., you would but everything email-related in one concern: validation, downcasing, actually emailing, etc. One major advantage of concerns is that they are easy to refactor to: the model is still used in the same way, and tests should still pass.

Interestingly, both the patterns camp and the concerns camp have very similar criticisms of each other's solution. In his conference talk about patterns for refactoring models, Bryan Helmcamp called concerns a "junk drawer." But in a code review of a service object refactoring cited in another conference talk, DHH said that service objects usually involve sweeping code problems under the rug.

The similarity of these comments should give us pause. How is it that both sides of the argument find the *same* problem with each other's solution? I think the answer lies in the fact that both sides are actually solving different problems. Of course, at one level the goal is the same: writing code that is reliable and easy to work with in the shortest time possible. But the philosophies of how to get there are somewhat different.

The "patterns" group is applying classical object-oriented design principles. In classical OO, the main way to achieve the above goals is modular code. If classes are small, have a single responsibility, and are loosely coupled to other classes, then they're easy to reason about, unit test, and reuse. None of those goals are achieved by concerns: even though the source files are smaller, the model object is just as large. It has multiple responsibilities, is difficult to unit test, and there is nothing stopping the the modules from all being tightly coupled to one another.

DHH clearly lays out his contrasting philosophy in his 2014 RailsConf keynote. In his view, the way to achieve quick, reliable code is readability. As someone once said, "code is written for people to read and only incidentally for computers to execute"--meaning that it's essential for people to be able to easily understand it. In this view, breaking code up into too many different types of class means difficulty drilling through them all to figure out what a method call ultimately does. And in this view, unit tests are less important than integration tests, so the ability to test individual classes is less important.

So what are we as developers supposed to do with these two different views? Pick one side? Not entirely. One way to summarize the difference between junior and senior developers is that junior developers write code without intentional structure, 


- Do you have to pick one or the other? It's a tension to manage. That's what it means to be a senior developer: to know when to apply different principles and when not to.
- My recommendations:
    - Start with the Rails standards
    - Evolve your architecture in an agile way
    - Know these patterns so you can identify a need to refactor when it arises
    - Make exemplary code so others on the team will follow
   