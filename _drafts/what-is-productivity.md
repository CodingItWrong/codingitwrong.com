We've talked about a number of patterns that can be added to Rails to separate out concerns from your models, views, and controllers. But what does the Rails core team think about this? Do they encourage you to reorganize your app any way you like? Are they working to add decorators and form objects to Rails 5? Well, no. Rails *is* gradually adding new patterns, like Active Job and Action Cable, but they're about more than just code organization. They're about entirely new ways of functioning: queues and persistent client connections, respectively.

On the topic of code organization, the message from DHH has generally been that what Rails gives you should be enough. He objects to some patterns more strenuously than others. But when it comes to code organization, I've heard him generally give two pieces of advice: use Rails classes for what they're intended, and use concerns.

Concerns are effectively just mixins with a little extra Rails-specific sugar. They allow you to take related functionality in a class (specifically, a controller or model) and put it in its own module, that is then included back into 

- Common thread: concerns vs collaborators
- Each says the other is a junk drawer
    - Jason DHH reply: sweeping under the rug
    - Bryan post/talk: junk drawer
- Why the different views of concerns and collaborators? Different views of what achieves productivity
    - DHH is focused on readability, and modules/concerns are good for that: smaller files, logical groupings, simple calling syntax
    - Classical OO folks are focused on modularity, and modules/concerns don't help with that, because it's still one huge model, with no protections for how things access each other
        - Sandi "nothing" video on modules not being good enough
- Do you have to pick one or the other? It's a tension to manage. That's what it means to be a senior developer: to know when to apply different principles and when not to.
- My recommendations:
    - Start with the Rails standards
    - Evolve your architecture in an agile way
    - Know these patterns so you can identify a need to refactor when it arises
    - Make exemplary code so others on the team will follow
   
- API video on different views of simplicity: same pattern vs. single responsibility