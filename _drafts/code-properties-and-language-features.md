I feel like I only just started learning good object-oriented design, and the world has moved on to functional programming! As a result, the OO principles that are fresh on my mind are exactly what people are moving away from. For example, as I've been checking out Elixir and Redux, I've had questions about whether a lack of certain OO features causes problems. For example, encapsulation allows you to isolate implementation knowledge to one part of your app, so that other parts of your app can use that functionality without being coupled to its implementation. Is it a problem that FP doesn't offer the same kind of encapsulation?

It's difficult for me to find answers to these questions. As Yehuda Katz [tweeted][yehuda-1] [recently][yehuda-2], there's a tendency to describe programming advancements as radically new paradigms, and past approaches as terrible. There's no grappling with the arguments older approaches made.

This leaves me with a conundrum: I bought into arguments about encapsulation increasing maintainability. Were those arguments true, or were they just something we made up to tell ourselves so we could build software in a way we liked?

Even though these things aren't discussed much publicly, asking questions of people who know Elixir and Redux helped me with this.

# Questioning Elixir and Redux

First, I asked about the way data is separate from functionality in Elixir, and whether that caused problems. But I learned that when you define a struct inside a module, you also usually define functions that operate on that struct inside the same module. Because the name of the module and the name of the struct are the same, they go together naturally. So data and functionality aren't entirely separated.

Then I asked about encapsulation: the data in a struct is available for everyone else in the app to access directly. But I learned that often you might have a convention of only accessing data in the struct via a function, and that that is a way to isolate knowledge of the data structure.

Then I asked about [the `Plug.Conn` struct][conn] in the Phoenix framework, which is passed through much of the app. I asked about whether this made the entire application fragile, because all of it required intimate knowledge of the Connection. I learned that there was a [`private`][conn-private] field on the struct, and although you aren't prevented from accessing it directly, the convention is that you shouldn't because the implementation in it can change without warning. This reminded me of some Rubyists' point that, because you can get around the private keyword, it's effectively just a convention as well.

After all this, I still had a concern with [Elixir's pattern matching][pattern-matching]. It's a lot like a switch statement, and [switch statements are a code smell][switch] in OO design. It makes it difficult to change your code because switch statements proliferate: every time you have to add a new case, all the switch statements need to be updated. Instead it's best to use polymorphism to handle variations, so that you can always add a new type of object and implement the new case's behavior on it. Each place you previously had a switch statement, you now send a message instead. But Elixir has a potential answer to this as well: [protocols][protocols] allow you to define a function signature, and then you can implement it for specific data types.

I had a similar question about Redux. Having a single reducer you call with actions seemed to combine all the logic in one place. But it's conventional to [compose your reducer][reducer-composition] out of smaller reducers that produce the values for individual keys in your state. So each smaller reducer only has responsibility for one piece of state, and it can just return the data unchanged for any actions that don't pertain to it. Yes, there is just one big list of action names that are strings, but if you like you can introduce namespacing into your action names to group them by data type.

At this point I was really confused. Were there *any* differences in capabilities between OO and FP? Had OO's grouping of data together with functionality really accomplished anything?

# Code Properties

**I think it all comes down to what properties you want your code to have, and whether a given language and paradigm make it easier or more difficult.**

For example, concurrency can be a big benefit for systems that need to handle a high volume of data throughput or that are making multiple asynchronous I/O requests. You can get a little concurrency in a traditional OO language like Ruby or Java, but it's incredibly hard to use it extensively in a reliable way. As my former coworker John Gallager said about concurrency on iOS in particular, "we can safely say that all nontrivial iOS applications have at least one concurrency bug." By contrast, Elixir, Rust, and JavaScript each have a different concurrency model that makes safe concurrency much more achievable.

Or consider immutability, which can simplify testing and minimize bugs from unpredictable data changes. It's possible to write immutable code in just about any language: just create new structures instead of modifying old ones. But mutable languages like Ruby and JavaScript make it incredibly difficult to *ensure* your data structures aren't modified by your code—or by third parties'. Immutability libraries like [`ice_nine`][ice_nine] and [`immutable.js`][immutable-js] can help, but they are extra hoops to jump through. By contrast, Elixir, Clojure, and Haskell are immutable by default, and it's *mutability* that requires you to jump through hoops.

On the flip side, think about encapsulation, as I described above. In large systems maintained by multiple individuals or teams, encapsulation allows parts of the system to change independently of one another, minimizing "[shotgun surgery][shotgun-surgery]," where to make one change you have to modify code all over the system. I listed examples in Elixir and Redux of how you can treat data structures as encapsulated, and by convention only access or modify them via structure-specific logic. Ruby makes this easier, although the fact that the private keyword can be worked around limits how rigid it is. Other OO languages like Swift are more rigid about how they protect encapsulated data.

# The Right Questions

So to ask the question "should I use an OO or FP language (or style)" is to skip several steps. I think a better series of questions are: First, what properties would be beneficial for your application to have: concurrency? Immutability? Encapsulation? Once that's decided, the next question is, **in a given language, are those properties guaranteed, easy, difficult, or not realistically achievable?**

Because of this, I don't think it even makes sense to say "Language X is better because it has Y properties." It's all about [your context][context]. It's more precise to say "Systems of type X benefit from property Y, and language Z makes it easier for your code to have that property."

[conn]: http://www.phoenixframework.org/docs/understanding-plug
[conn-private]: https://hexdocs.pm/plug/Plug.Conn.html#module-private-fields
[context]: /2017/07/16/how-to-be-context-sensitive.html
[ice_nine]: https://github.com/dkubb/ice_nine
[immutable-js]: https://facebook.github.io/immutable-js/docs/
[pattern-matching]: https://elixir-lang.org/getting-started/pattern-matching.html
[protocols]: https://elixir-lang.org/getting-started/protocols.html
[reducer-composition]: https://egghead.io/lessons/javascript-redux-reducer-composition-with-arrays
[shotgun-surgery]: https://sourcemaking.com/refactoring/smells/shotgun-surgery
[switch]: https://sourcemaking.com/refactoring/smells/switch-statements
[yehuda-1]: https://twitter.com/wycats/status/887422270808891392
[yehuda-2]: https://twitter.com/wycats/status/887423765293875200
