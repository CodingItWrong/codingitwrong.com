---
title: The Relationship Between OO and FP
---

Are object-oriented programming and functional programming compatible or incompatible? Does it even make sense to talk about the two terms as though there are any differences at all? They seem on their face to be fairly different, but when this conversation comes up it's common to hear the viewpoint that there is a false dichotomy between them. So what *is* the relationship between object-oriented and functional programming?

## Definitions
First, definitions. For the purposes of this post let's consider “object-oriented programming” (OO) to be programming with *objects*: combinations of data and behavior. Objects encapsulate some state and expose a limited public interface to interact with and mutate it. In that sense, the data and behavior are joined together. (Depending on your view of OO, you might have concerns about this definition—if so, stick with me.)

By contrast, let’s use the term “functional programming” (FP) to refer to a kind of programming where data and behavior are *separated*. There is an emphasis on functions in the mathematical sense: “pure” functions that don’t have any side effects, but only produce the same outputs for the same inputs. There tend to be first-class functions, so functions that can be passed to other functions, as well as “higher-order functions” that return functions. There is also the concept of closure, where when a function is defined, any variables that are in scope at the time will stay available to that function no matter where it’s executed from.

# Language Compatibility
First, OO and FP are *at least* compatible in one sense: the same person can write both types of software, and you don't need to "pick a side." What's more, the same programming language can offer both OO and functional features, and the same project can use OO features in some parts and functional features in others. (This will end up being very important later.)

What's more, OO and FP are compatible in that they are *equivalent*: if a programming language supports one style of coding, you can implement the other on top of it. One place this is demonstrated in detail is in Graham Lee's [_OOP the Easy Way_][oopew]. Although this shows that the two styles have equal power and aren't at odds, it's rare that you would see this done on a real project.

What *is* more likely would be to attempt to apply OO design principles in an FP context—and this can be done fairly successfully. Let's look at a few examples, starting with the SOLID principles:

- **Single Responsibility Principle:** this states that a class should have a single responsibility—a single reason to change. The principle is often applied to individual methods as well. So it can be applied to functions in FP programs just as easily: each function should only have a single responsibility.
- **Open/Closed Principle:** this one is less clear how to apply in an FP context. One of the primary ways to keep code open to extension while closed for modification is via polymorphism: you replace repeated switch statements with several different objects, each of which implements one of the cases. This allows you to extend the code by adding a new object that implements an additional case, without having any switch statements you need to modify. But if a functional language doesn't have dynamic dispatch then there's not a good way to handle different cases with polymorphism. In fact, in some functional languages switch statements or pattern matching aren't avoided, but rather are a core part of idiomatic code.
- **Liskov Substitution Principle:** this says that instances of subclasses should be substitutable for instances of parent classes without requiring type checks. In FP you inject functions instead of objects, and it’s rare to do a type check on a function. So this principle is sort of followed by default.
- **Interface Segregation Principle:** this principle says to minimize the number of methods on an interface you’re depending on. Because FP works with individual functions, this is kind of followed by default as well.
- **Dependency Inversion Principle:** this says that you should depend on abstractions instead of concrete implementations. In OO this is often achieved by injecting dependencies. In FP it’s easy to inject, or pass in, functions to be called instead of hard-coding them.

How about the more foundational principle of **encapsulation**? The most straightforward way to work with data in FP is to pass data structures from which the fields are accessed directly, not mediated by an object's methods. This is straightforward, but it results in all parts of your app being coupled to the structure of the data. This means that when the shape of the data changes, those changes will ripple throughout your app. One way to avoid this coupling is to choose to only access the data through functions that retrieve certain fields, so that you can isolate knowledge of the data format to those functions. If the language doesn't have a way to limit access to certain fields, though, using these functions is not enforceable and can only be a convention.

Despite a few minor limitations, the theme here is that there is much that's compatible between OO and FP programming: they can be used together, they can be demonstrated to be equivalent, and many software design principles apply to both.

Is it correct, then, to say that there’s no real difference between OO and FP? I couldn't shake a nagging sense that programming in a functional language is practically very different. Why is that?

# Two Mindsets
I've come to realize that the difference is not so much between OO and FP *languages* but rather between OO and FP *mindsets*. Graham Lee pointed out to me that "mindset" is what the term “paradigm” is supposed to mean—in the past I had taken the word “paradigm” to simply mean “what type of programming language".

But what is a "mindset"? Does it simply mean "different ways of thinking about doing the same thing?" No, I think it's more practical than that: it means "how you apply the tools your programming language gives you."

So let’s learn about two particular mindsets, one associated with OO and the other with FP. One quick qualification. There are lots of different mindsets that those who write code in OO and FP languages take; I'm describing two specific mindsets here. Some would argue that this OO mindset is truer to the original idea of objects, but it's not necessary to agree with that to consider it here.

Let’s call this particular OO mindset the **message-focused mindset**. In this mindset, the idea is that objects are independent, like tiny computers or even like people. Each has responsibilities that it’s in charge of. Objects communicate via messages, requesting for each other to do things. This leads to a kind of decentralized control, where there isn’t a single “controller” or "manager" that knows everything that needs to be done and tells others what to do. Instead, knowledge of how to achieve ends is spread out equally among different objects. This increases the independence of each object and therefore their reusability for the future.

What does this theory actually look like in practice? Sandi Metz gives a great illustration in [_Practical Object Oriented Design_][pood]. Echoing Alan Kay, she recommends focusing more on the messages being sent than on the objects that receive them. This results in “[c]hanging the fundamental design question from 'I know I need this class, what should it do?' to 'I need to send this message, who should respond to it?'” This can lead you to identifying new objects you need to create that weren’t immediately obvious. The result is a decoupled system with more reusable pieces.

Dave Thomas gives his own assessment on the object-oriented mindset in the introduction to [_Programming Elixir_][programming-elixir] before sharing a functional alternative. He describes the object-oriented mindset as follows:

> When we code with objects, we’re thinking about state. Much of our time is spent calling methods in objects and passing them other objects. Based on these calls, objects update their own state, and possibly the state of other objects. In this world, the class is king—it defines what each instance can do, and it implicitly controls the state of the data its instances hold. Our goal is data-hiding.

That all fits within the OO mindset I described above. Here’s his assessment of it:

> But that’s not the real world. In the real world, we don’t want to model abstract hierarchies (because in reality there aren’t that many true hierarchies). We want to get things done, not maintain state.
>
> Right now, for instance, I’m taking empty computer files and transforming them into files containing text. Soon I’ll transform those files into a format you can read. A web server somewhere will transform your request to download the book into an HTTP response containing the content.
>
> I don’t want to hide data. I want to transform it.

Incidentally, I'm not sure the term “hierarchies” is helpful here. Modeling the real world doesn't require inheritance hierarchies, unless you count classes inheriting from a base Object class. And apart from inheritance I'm not sure what "hierarchies" is meant to represent.

But in any case "hierarchies" aren't the core of Thomas's critique: instead, it centers on *modeling*. He asks, what do you gain by trying to model reality when you can just transform the data you need directly? In this approach, data isn’t hidden and encapsulated, it’s exposed and transformed directly. Elixir's pipeline operator helps to encode this mindset into the language; it's so core to the Elixir mindset that it serves as a kind of syntax pun in the subtitles of _Programming Elixir_ and _Programming Phoenix_. We can call this functional mindset the **transformation-focused mindset.**

## What’s the Difference?
David West writes about concerns with this transformation mindset in [_Object Thinking_][object-thinking]. Although he’s contrasting OO with *procedural* programming rather than functional programming, the concerns he raises apply equally to the functional transformation mindset. He cites David Parnas’ classic paper [“On the Criteria To Be Used in Decomposing Systems into Modules”][parnas] (emphasis mine):

> Parnas’s paper examined two conceptual abstractions for decomposing complex systems for the purposes of developing software. One was top-down functional decomposition, the approach that was gaining widespread acceptance under the label “structured design.” Functional decomposition is based on *an attempt to model the performance of the computer and software* and to translate the requirements of the domain problem into those computer-based constructs.

Later West clarifies what it means to “model the performance of the computer and software” as "decomposition…based on ‘artificial,’ or computer-derived, abstractions such as memory structures, operations, or functions (as a package of operations)". Because the decomposition of the system is based on the “memory structures,” they are exposed directly for the “functions” to operate on. This sounds a lot like what Thomas describes in terms of representing data transformations directly.

West continues:

> Parnas offered an alternative approach called “design decision hiding,” in which *the problem or problem domain is modeled* and decomposed without consideration of how the component parts of that domain or problem would be implemented.

Here the focus is on modeling the problem domain, not modeling the processes of the computer. "Design decision hiding" refers to the idea that decisions on data representation are hidden within modules, not exposed to the entire program.

West then assesses the impact of which of these approaches to decomposition is taken:

> If that decomposition is based on a *“natural” partitioning of the domain*, the resultant models and software components will be significantly simpler to implement and will, almost as a side effect, promote other objectives such as operational efficiency and communication elegance. If, instead, decomposition is based on “artificial,” or computer-derived, abstractions such as memory structures, operations, or functions (as a package of operations), the opposite results will accrue.

**OO and FP languages may be equivalent, but the message-based and transformation-based mindsets are not.** You are either hiding data in objects, or exposing it to be operated on directly—you cannot do both at the same time in the same place. Even if you’re working in an FP language, to the degree that you’re applying OO design principles as we described above, you _aren’t_ directly transforming data in a set of steps, but instead you’re encapsulating data.

Although in these excerpts West may sound as though he thinks the OO mindset is categorically better, he qualifies this elsewhere in the book. If your data format is known to be unlikely to change, and *the domain concept you need to model really is a series of data transformations*, encapsulating that data in an object may not be worth the indirection. But by contrast, what I've found working in business and consumer systems is that how they work is subjective, bending to the whims of product owners, business strategies, and market pressure. Little is set in stone, and changeability is important. In this circumstance, isolating changes to one area of the codebase is extremely valuable.

Although these two mindsets are different, they can coexist in one codebase. It can be useful to ask, *in what parts* of my codebase is direct transformation of data useful, and in what parts is data hiding useful? Graham Lee mentions in [_OOP The Easy Way_][oopew] that data within an object can be treated as immutable and transformed statelessly. But these objects still hide their data representation and are combined together in a decentralized way.

## Influence of Language and Community
When folks say that OO and FP are a false dichotomy, one of the things they may mean is that the two mindsets we’ve described aren’t tied to the type of language you’re using. You can write code in an object-oriented language that transforms data directly. And, as described above, you can write code in a functional language that encapsulates data representation with closures, or even your own object implementation.

But you aren’t equally likely to take these different approaches in any programming language; languages make one approach or the other more natural. In a functional language there are more barriers to an object-oriented approach if you have to implement an object model yourself, rather than having one available in the language already. What’s more, language communities have momentum and norms. Sandi Metz is well-known in the Ruby community, and although not everybody uses her approach to OO, many have at least heard of it. And her book and other resources are available to help describe the message-focused mindset.

If such resources aren’t available in a functional language, then sharing those ideas would require convincing people to read a book targeting *another* programming language, then doing the work themselves to figure out how to apply it in their own language. It’s true that _Practical Object-Oriented Design_ is not intended to be specific to Ruby: it could be fairly easily applied to Java or C#. But it would be more challenging to apply it to a language that doesn’t have classes and objects—challenging enough that it’s not likely for many readers to do so. And even if you do, that makes it even harder to onboard new team members, because you’re programming that language in a way that is different from the language norms, with no written resources to describe the approach.

# Conclusion
We should now be able to answer our initial question with more nuance: are OO and FP compatible or incompatible?

OO and FP *languages* are compatible: they’re technically equivalent, and design principles from one apply pretty well in the other as well.

Two particular OO and FP *mindsets*, though, are not so equivalent—the message-focused mindset and the transformation-focused mindset. They're very different, taking opposite approaches to how code is decomposed. Specifically, the message-focused mindset is about encapsulating and hiding data to decentralize control, and the transformation-focused mindset is about exposing data to make a pipeline of transformations more explicit.

But this doesn't make the two mindsets incompatible; in fact, they're complimentary. Many applications would benefit from applying both approaches in different parts of the code. Recognizing this compatibility is not the same as saying that OO and FP are a false dichotomy, though: there is a real distinction that makes a significant difference in the results in the software we write with each mindset. To emphasize *only* the the compatibility of OO and FP means missing the opportunity to capitalize on the different strengths of each mindset.

If you’d like to learn more about the message-focused mindset from a practical standpoint, I would recommend reading [_Practical Object-Oriented Design_][pood]. And if you’d like more of the theory, history, and philosophy behind these two mindsets, [_Object Thinking_][object-thinking] tackles this in great depth.

Thanks to Graham Lee, Jacob Bullock, J. B. Rainsberger, Jeremy Sherman, and Steven R. Baker for input on this topic.

[object-thinking]: http://www.informit.com/store/object-thinking-9780735619654
[oopew]: https://leanpub.com/ooptheeasyway
[parnas]: https://www.win.tue.nl/~wstomv/edu/2ip30/references/criteria_for_modularization.pdf[pood]: https://www.poodr.com/
[pood]: http://www.informit.com/store/practical-object-oriented-design-an-agile-primer-using-9780134456478
[programming-elixir]: https://pragprog.com/book/elixir16/programming-elixir-1-6
