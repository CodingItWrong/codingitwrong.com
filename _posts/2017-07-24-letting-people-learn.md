---
title: Letting People Learn
---

Back when my understanding of agile development was very rudimentary, I wanted nothing more to be in charge of **"Standards"** for a dev team. **"Standards"** is what would solve all our problems. I knew what kind of code we needed to write and what methodology to use. If everyone would just do as I said, we'd all be fine.

!["Standards" must be read in a Sam the Eagle voice.](/img/posts/letting-people-learn/standards.jpg)

One problem with this is obvious in retrospect: I didn't always know the right way. But there's a deeper problem too: even if you assume there were *some* things where I knew the right way to do it, there was still a problem with the way I wanted to influence others.

When I arrived at Big Nerd Ranch, I needed to learn a lot at once: things like Rails, a pull-request workflow, working in small stories, testing, and object-oriented design. Eventually, the more concrete things like Rails and a pull-request workflow became second nature. But that's not the case with stories, testing, and object-oriented design. You can't just follow some simple steps for these things and get results. There are usually challenges with doing them. They require intense thought. They certainly haven't become second-nature for me yet, and everyone I've learned them from says that they *never* become second-nature.

If small stories, testing, and object-oriented design always require careful thought, then there's a problem with simply telling people to do them without teaching them *how* or motivating them to *want to*. If it's difficult to test a certain behavior of the code even for someone who *wants* to test it, how much more so for someone who thinks testing is a waste of time?

This challenge can lead to a kind of lowest-common-denominator management where you choose processes that *do* simply consist of a series of steps to follow without thinking too hard. This does allow you to get started quickly, but it also leads to all the problems of traditional waterfall software development. This tendency is related to Taylorism, a view of industrial engineering centered around treating workers as interchangeable laborers. You can read more about the conflicts between Taylorism and agile development in chapter 18 of [_Extreme Programming Explained_][xp].

But what's the alternative to lowest-common-denominator management? How can we make people practice development methodologies that are difficult to do?

The answer becomes clearer if we make the question more specific: how can we make people care about testing and OO design? The answer, of course, is that we can't *make* people care about anything. If people need to have an intrinsic desire in order to do these things, then the best we can hope to do is to motivate and inspire people. Do a pair programming session. Walk them through doing it together. Show them the benefits. When they're stuck with a problem, show them how you would go about addressing it.

In [_Object Thinking_][ot], David West writes that Extreme Programming is "a method for producing better developers rather than a method for producing better software." You can't ultimately get better software by giving people a set of steps to follow; you need them to work on their skills as developers. That's a challenging thing to do. And this may be the ultimate reason that true agile has such limited adoption.

This all sounds rather bleak, but I also find it pretty liberating. I'm not responsible for what development processes others use, so I don't need to put pressure on myself to make them change. What I can control is to do my best and to try to offer others practices that I think will help them.

[xp]: http://www.informit.com/store/extreme-programming-explained-embrace-change-9780321278654
[ot]: http://www.informit.com/store/object-thinking-9780735619654
