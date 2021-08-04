---
title: Two Approaches to Changing Software
---

I’m a breadth-first programmer; I have my hands in a lot of languages. My client work focus is currently on frontend JavaScript. My background is in Ruby on the server. And in my current role as web platform lead at Big Nerd Ranch, I work with web developers in a variety of languages from TypeScript to Java to Go, and I interface with Swift and Kotlin developers. In this melting pot, it’s hard to understand how to apply the software design principles I value that emerged from an object-oriented context.

Are there software design principles that apply across all of these programming languages and paradigms? Or are there fundamental differences between the way you approach software design in statically typed and dynamically typed languages, object-oriented and functional languages?

The conclusion I’ve come to is that there *is* a fundamental difference that leads to different approaches to software design, but it’s not a difference between languages. Instead, it’s a difference between two approaches to changing software.

## Changeability
When I distill down the software design principles I use to things I would use across all language paradigms, I end up with the following:

The goal is **changeability**, because most software changes and so we want to make it easy to change.

Some key practices that support changeability are:

* Communicative code: so you can understand it to change it.
* Fully-specifying tests: so you can change it with confidence.
* Minimal code: so there is less to change.
* Refactoring: the changing process.

(Readers familiar with the Four Rules of Simple Design may notice this list is fairly close to those rules.)

This list meets the goal of having nothing specific to object-oriented programming, functional programming, dynamic typing, or static typing. So is it something that most developers would agree on?

It depends on your view of changeability. When I think of changeability I think of *evolutionary design*: making the system a good fit for the requirements of today, then setting it up to be easy to change, so I can reorganize it to be a good fit for the requirements of tomorrow.

But that’s not the only way to change a system. People who don’t follow evolutionary design still change their systems. If a requirement comes in and the system isn’t set up to be easily extended to handle it, they might rewrite portions of their app to fit the new requirements.

In that approach to changeability, are the practices above still important?

- Communicative code: it’s less important for the code to be easy to understand if you’re usually either going to leave it unchanged or replace it.
- Fully-specifying tests: same thing. If a unit of code usually doesn’t change, you usually don’t need unit tests to support that change, so it may not be worth the work to create them. Manual testing may be enough.
- Minimal code: including some unused functionality or a more-verbose way to implement something isn’t as problematic if you aren’t touching that code.
- Refactoring: in this context the term is usually used to refer to broad sweeping changes across the app, rather than small adjustments.

So whether it’s worth doing these practices depends on whether you’re trying to do evolutionary design. In light of that, it’s futile to try to convince people who aren’t trying to do evolutionary design to do the above practices; they have good reason to resist.

## Evolutionary Design
A more productive conversation would be, should you consider doing evolutionary design? What causes someone to embrace it?

In *Smalltalk Best Practice Patterns*, Kent Beck wrote (emphasis mine):

> Did you know that *your program talks to you?* I don’t mean those “Buy more Jolt Cola” messages. I mean important stuff about how your system ought to be structured.
>
> If you’re programming along, doing nicely, and all of a sudden your program gets balky, makes things hard for you, it’s talking. It’s telling you there is something important missing.
>
> Many of the patterns [in this book] tell you what to do when your program talks to you. Sometimes you need to create a new method, sometimes a new object, sometimes a new variable. Whatever the needed response, *you have to be listening before you can react.*

The concept of your program talking to you could seem like a metaphor dreamed up by the author just to engage the reader, but it’s actually a deeply-held belief in some segments of the evolutionary design world—closer to a quasi-mystical worldview. No surprise, therefore, that it’s not universally accepted by software engineers. For my part, I believe in listening to your program talking to you enough that I hesitate to explain it at the risk of “explaining it away.” So let me just say that I’ll describe *one* aspect of it.

One key aspect of “your program talking to you” is recognizing that there is something outside yourself that you can learn from, and deciding to try to learn from it. It’s the difference between (a) deciding in advance that you know how the software should be structured and (b) figuring out how the software should be structured as you go, as you see what the program actually ends up being.

You could call it top-down design vs. bottom-up design. This kind of top-down design is not precisely the same as Big Design Up Front, because it doesn’t necessarily happen at the start of a project. It can happen when you start a new section of code, or even when you realize a section of code needs to be restructured. When that happens, you decide how the code should be arranged, then you program what you decided, then you’re done. Top-down design is also not the same as the similar-sounding “outside-in test-driven development” as described in _Growing Object-Oriented Software, Guided by Tests_, which is very much a bottom-up approach.

In bottom-up design, you look at the actual code that is written and make decisions about the structure based on that. You don’t design in a vacuum. Architect and coder aren’t separate roles or even phases. As Beck also writes,

> To me, development consists of two processes that feed each other. First, you figure out what you want the computer to do. Then, you instruct the computer to do it. Trying to write those instructions inevitably changes what you want the computer to do, and so it goes.
>
> In this model, coding isn’t the poor handmaiden of design or analysis. Coding is where your fuzzy, comfortable ideas awaken in the harsh dawn of reality. It is where you learn what your computer can do. If you stop coding, you stop learning.

I think *learning* is the key principle: specifically, continuous learning about the optimal design of your code. This is why this approach to development emphasizes feedback, specifically fast feedback loops. You don’t need fast feedback if you already know from the start how the software should be structured.

## Conclusion
Somewhere Beck describes the origin of Extreme Programming in finding practices that work and pushing them as far as you can, to the extreme. Evolutionary design is this principle applied to design. Sometimes, no matter how little you expect to learn, your code *forces* you to listen, because your original design idea just won’t work at all. Evolutionary design says, if it’s good to listen to the code *then*, then why don’t we try listening to the code *all* the time? And why don’t we set up processes that make it easier for us to listen and respond? *That’s* the motivation behind writing communicative, minimal code, comprehensive tests, and doing refactoring.

When you do top-down design (or top-down redesign) you’re making a bet that more often than not you know how requirements are going to change in the future and you know the best design for those requirements. Bottom-up design seems to be more work, but you do that work because you’re making a bet that you *don’t* know how requirements are going to change in the future and you *don’t* know the best design for those requirements. Top-down design is a bet that I’m smart; bottom-up design is a bet that I’m dumb. And I’d rather bet on myself being dumb, because it takes the pressure off.

Beck and some other TDD practitioners talk about the anxiety they feel writing large chunks of code and how TDD alleviates that anxiety by allowing you to make small steps of concrete progress. The rest of evolutionary design has the same anxiety-reducing benefits. When you evaluate a top-down design based on design principles, if you find a design problem, it’s difficult to act on because you aren’t set up to be able to change the design easily. You got it wrong and now you’re stuck. So it’s mostly non-actionable blame. But in bottom-up design, if there is a design problem, you are set up to change it easily. It’s not even really a problem, it’s your code talking to you, and you’re set up to respond easily. It’s actionable and there’s no reason for blame.

Top-down coding is inherently somewhat exclusionary because it privileges those with more experience, those who are wired to hold more information in their heads at once, and lucky guesses. Bottom-up coding is more robust to support a variety of experience levels, brain wirings, and luckynesses. It’s empowering; it says “you can figure it out.”
