---
title: Letting Static Typing Coexist
---

As someone who emphasizes the value of dynamically-typed programming languages, I'm discouraged to see the industry shift seemingly so universally toward static typing. But recently I had a realization that increased my understanding of why this shift happens and an openness to why static types are a good fit for some (but not all) scenarios.

## The Lay of the Land
If you aren't sure what I mean by the industry shift to static typing, here's what I mean. The largest mobile platforms have each created a statically-typed language as their officially-supported programming language: Kotlin for Andorid and Swift for iOS. For frontend browser applications, although the one supported language is dynamically-typed, there's tremendous interest in a proprietary statically-typed language (TypeScript) that compiles down to it. And in server-side development the previous shift toward dynamic languages like Ruby and JavaScript has been replaced by a counter-shift toward statically-typed languages like Golang and TypeScript.

I would like to build applications for native mobile, frontend web, and backend—and I would like to build them in a dynamically-typed language, but barriers to doing so are increasing. It’s not officially supported on mobile, and for frontend development there’s a trend toward TypeScript tooling being enabled when I don’t want it.

Now, React Native is a way to build native mobile applications with a dynamically-typed language, JavaScript. I write React Native professionally and I recommend it. But only having one dynamically-typed language isn't ideal. I want to be able to develop in an object-oriented approach, but JavaScript isn't consistently object-oriented and React is structured more functionally. So the point remains: I want *more* dynamically-typed options for native mobile development.

There are a few other attempts to build for native mobile development with dynamically-typed languages. RubyMotion allows building native mobile applications in Ruby. Turbo Native allows integrating a server-rendered Rails application with a native app. These are appealing for some use cases, but not for mine or for all. Build issues and lack of a native feel can cause problems, and because in the best-case scenario they are as good as (not better than) apps built with native technologies, there is little market adoption and limited maintenance resources.

## Why Dynamic?
Why do I prefer to build in dynamically-typed languages? Most applications need to change over time. To support that change, they need to be well-factored so that their code is a straightforward expression of their behavior. To be able to change the code to keep it expressing the behavior well, you need thorough test coverage. And the most reliable way I know to get thorough test coverage is to do test-driven development (TDD).

So far, all these things can be done in both dynamically-typed and statically-typed languages. The difference is this: if you've test-driven your application, the benefits of static typing are much lower, and the costs are much higher. Most of the errors that static typing can catch for you, your tests will have already caught, along with logic errors that almost no static type systems even attempt to catch. But many static type systems make it harder to develop using this approach: making test double objects harder to create, encouraging doing up-front design, increasing the cost of making broad changes to your application so you're discouraged from doing so. In light of this, I think that for most applications, if you are going to do TDD, refactoring, and evolutionary design, you will get more value for less cost with a dynamically-typed language than a statically-typed language.

This line of thinking led me to see static typing as usually a poor tradeoff. But that thinking has started to shift, to become more nuanced.

## Why Static?
The key to the shift is recognizing an important conditional: *if* you thoroughly TDD then dynamic typing is better, but *will* you thoroughly TDD? In [“Entropy Essays 8: Why Entropy?”](https://noelrappin.com/blog/2021/05/entropy-essays-8-why-entropy/), Noel Rappin writes about some reasons you might not thoroughly apply agile practices like TDD:

> - The practice is hard and success doing the practice on one project doesn’t mean you’ll be successful the next time.
> - The practice adds an in-the-moment cost to all code, with the promise of a cost savings later. (So, there’s a cost to writing tests up front, but the hope is that later change is less expensive)
> - The practice is not *resilient* – once you slide away from doing the practice 100% of the time, it’s very hard to make up lost ground.

The response to these challenges is that:

> …for all of these hard, not very resilient practices, a simplified version has emerged. The simplified version seems more resilient, but I strongly suspect it also has less long-term value.
>
> This is why I started using the term “entropy”, where a low-entropy state is hard to maintain, but a high-entropy state is more stable.

One example he gives of a high-entropy solution is not doing TDD, but instead writing mostly end-to-end tests. He goes on to describe what it means that these solutions are easier to maintain:

> These high entropy solutions are attractive, by which I mean not just that people find them pretty, but they are attractors. As a team…[for example] can’t quite drive design from tests, the higher entropy solutions are there, and they are more resilient. They are so resilient, that it’s nearly impossible to break out of the gravitational pull of one of these solutions once you’ve started along them.

So, there is this strong pull away from the agile practices I would argue are ideal. And if you do end up sliding away from the ideals of TDD, refactoring, and simple design, as I said before, the benefits of static typing increase. It can catch nil/null errors and incorrect assumptions about the data you have. The compiler can guide you through changes to your codebase.

## Different Strokes
If we agree with (1) the benefits of agile practices, (2) the challenge of sticking with them and (3) the pull of simplified practices, how should that affect our view of development? I think the answer is different for individuals, companies, and platforms.

Individually, there's no question as to whether I'll follow agile practices: I’ll continue to shoot for them, despite the fact that I’ll slip up at times. The question is this: for the times I *do* slip up, is it worth the cost of static typing to get the extra safety? For my personal projects, I’ll stick with dynamic languages when I can, because I enjoy them more, and if I make a mistake it’s not a big deal because either I'm the only user or I have a few users who aren't paying me anything.

For companies, the choice is different. Your engineering leadership may decide they're committed to agile practices and the effort it takes to train in them and keep developers accountable to follow them. In that case, the productivity benefits and cultural appeal of dynamic languages can make the most sense. GitHub, Shopify, and Basecamp all demonstrate that you can build a world-class tech company on Ruby. But at some point the costs can mount up. Stripe found the benefits of static typing so great that it was worth the cost to [build a static type checking tool from scratch](https://sorbet.org/) for a language that is not set up for it very well. If even a company using Ruby can find the benefits of static types, it makes all the more sense that a company might choose from the start to opt in to the safety of static types.

For proprietary platforms, the choice of statically-typed languages seems inevitable. Even if Apple or Google engineering leadership strongly believed in the benefits of dynamic languages, it's not fathomable that they would effectively say "to build for our platform you have to use a language that is only safe if you follow a certain development methodology." There is no technical barrier to doing mobile development with a dynamically-typed language, but there is a practical and economic barrier. The platforms need to adopt a language that provides some level of safety no matter which development approach you follow. I may wish that they had adopted dynamically-typed languages, but I realize that they couldn’t have done so.

## Acceptance
What this realization does is reduce my frustration level with platforms and companies that adopt statically-typed languages. I no longer see them as making a bad decision that categorically makes development harder and apps worse. I see them as choosing a tradeoff that makes sense for them in their best assessment—understandable for companies, and inevitable for platforms until if and when the paradigm of commercial software development changes.

The result of this is that, as someone who prefers writing dynamically-typed Ruby code, I accept that native mobile development simply isn't the place where it’s ever going to be well-supported. And frontend browser applications will be a place where the dynamically-typed option is not ideal, and for the foreseeable future the proprietary TypeScript will exert a draw. The place for me to do the kind of dynamic programming I'm most interested in is on the server, in the command line, and potentially in new interfaces like chatbots. That's because these platforms don't have constraints on the language you use, so there aren't market forces that provide friction to the use of dynamic languages.
