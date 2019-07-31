---
title: Refactoring is a Watershed
---

There are two definitions of the word “refactoring,” and which one you use is a tell as to what your approach to architectural evolution is, as well as deeper beliefs about certainty. Your definition of “refactoring” also has a significant influence on the programming paradigms, type systems, testing approaches, and emphasis on code quality that you take. The differences are striking enough that when someone who takes one view of refactoring describes the benefits they get from their tooling, a listener who takes the other view of refactoring may not even be able to understand what they mean—and they won’t necessarily get the same benefits under the different refactoring approach.

That's a lot of claims, so let's dive into what I mean by all this.

## Definitions of “Refactoring”
The term "refactoring" was popularized by the book of the same name. Its author, Martin Fowler, recently published a second edition. In it he points out two contemporary usages of the term "refactoring":

> Over the years, many people in the industry have taken to use “refactoring” to mean any kind of code cleanup—but the definitions above point to a particular approach to cleaning up code. Refactoring is all about applying small behavior-preserving steps and making a big change by stringing together a sequence of these behavior-preserving steps.

That is to say, "refactoring" can be used to mean either (1) making small changes, or (2) making *any* changes, large or small. Now, this quote could be read to mean that small-step refactoring is *always* part of a bigger change, but that’s not the case, as Fowler clarifies elsewhere in the book. He describes many kinds of small change that are made without a view to a big change end goal.

But let's specifically consider the case where small refactorings *are* building toward a big change. Why would you want to break such a big change into small steps? There is an inherent overhead to doing so, but Fowler gives several reasons this overhead may be worth it.

> …the tiny steps allow me to go faster because they compose so well—and, crucially, because I don’t spend any time debugging.

If doing refactoring in small steps helps avoid bugs then that’s certainly appealing. But maybe there are other ways to achieve that goal. For example, lots of folks would argue that modern static type systems make bugs during large refactorings far less likely. If that's the case then you might not need to incur the overhead of taking small steps. (I’m not making a claim here about whether modern static type systems *do* or *don’t* provide this assurance, only saying that *if* you determine they do, that would reduce the value of small steps for avoiding bugs.)

> Each individual refactoring is either pretty small itself or a combination of small steps. As a result, when I’m refactoring, my code doesn’t spend much time in a broken state, allowing me to stop at any moment even if I haven’t finished.

If a large refactoring will take a while, then it’s extremely useful to be able to stop partway through to integrate with master, or to have the flexibility to do other feature work. But again, if modern static type systems mean that large refactoring proceed much faster, then stopping points may be less of a reason to take small steps. (Again, not making a claim here about whether or not modern static type systems do provide this speed.)

## The Best Reason
This brings us to what I think is Fowler's best reason to do small-step refactoring (emphasis mine):

> Early on, I do comprehension refactoring on little details. I rename a couple variables now that I understand what they are, or I chop a long function into smaller parts. *Then, as the code gets clearer, I find I can see things about the design that I could not see before.* Had I not changed the code, I probably never would have seen these things, because I’m just not clever enough to visualize all these changes in my head. Ralph Johnson describes these early refactorings as wiping the dirt off a window so you can see beyond. *When I’m studying code, refactoring leads me to higher levels of understanding that I would otherwise miss.*

Fowler makes a similar point in discussing refactoring during code review:

> Before I started using refactoring, I could read the code, understand it to some degree, and make suggestions. Now, when I come up with ideas, I consider whether they can be easily implemented then and there with refactoring. If so, I refactor. When I do it a few times, I can see more clearly what the code looks like with the suggestions in place. I don’t have to imagine what it would be like—I can see it. *As a result, I can come up with a second level of ideas that I would never have realized had I not refactored.*

A small refactoring can result in code that brings clarity and suggests additional refactorings that weren’t clear before. In this approach, you aren’t just breaking up a predetermined end-point into smaller steps; you’re adjusting your direction as you go.

This concept is explored in depth in _99 Bottles of OOP_ by Katrina Owen and Sandi Metz. Here's how they express the general principle:

> However, it just so happens that solving easy problems, through a magical alchemy of code, sometimes transmutes hard problems into easy ones.

And here's how they discuss it in the context of a specific large goal where the way to achieve it is not yet clear:

> Before undertaking this refactoring, it must be admitted that *there is no direct connection* between removing the duplication, and succeeding in making the code open to the six-pack requirement. That, however, is the beauty of this technique. *You don’t have to know how to solve the whole problem in advance. The plan is to nibble away, one code smell at a time, in faith that the path to openness will be revealed.*

One more author who describes this dynamic succinctly is Kent Beck:

> I’m constantly amazed at how even a little help cleaning up small problems reveals the source and solution of much bigger problems.

— Kent Beck, _Smalltalk Best Practice Patterns_

This kind of feedback isn’t available to you when you do a large refactoring in one step. You’re proceeding with the plan you have, and it’s not until it’s done that you can evaluate how that plan worked out. It’s a smaller form of Big Design Up Front: yes, you’re adjusting your system as you go, but when you see the need for a change, you do a Big Design for what you need that change to be, then put it into place. A more agile approach would be what we’re describing here: to do a series of small refactorings and get feedback from each.

## Uncertainty
I'm sure many developers haven't run across this idea of looking for feedback from small refactorings. But for developers who have, what leads some to seek this feedback and others not to? I think a major factor is their beliefs about change and uncertainty. A frequent theme among agile authors is that change and uncertainty are unavoidable, and should be embraced instead of resisted:

> The biggest issue with finishing architecture before coding is that such an approach assumes the requirements for the software can be understood early on. But experience shows that this is often, even usually, an unachievable goal.

— Martin Fowler, _Refactoring (Second Edition)_

> Unfortunately, something will change. It always does. The customers didn’t know what they wanted, they didn’t say what they meant. You didn’t understand their needs, you’ve learned how to do something better. Even applications that are perfect in every way are not stable. The application was a huge success, now everyone wants more. Change is unavoidable. It is ubiquitous, omnipresent, and inevitable.

— Sandi Metz, _Practical Object-Oriented Design_

> Everything in software changes. The requirements change. The design changes. The business changes. The technology changes. The team changes. The team members change. The problem isn’t change, because change is going to happen; the problem, rather, is our inability to cope with change.

— Kent Beck, _Extreme Programaming Explained_

> Everyone involved in a software project has to learn as it progresses. For the project to succeed, the people involved have to work together just to understand what they’re supposed to achieve, and to identify and resolve misunderstandings along the way. They all know there will be changes, they just don’t know what changes. They need a process that will help them cope with uncertainty as their experience grows—to anticipate unanticipated changes.

— Steve Freeman and Nat Pryce, _Growing Object-Oriented Software, Guided by Tests_

> It is a myth that we can get systems “right the first time.” Instead, we should implement only today’s stories, then refactor and expand the system to implement new stories tomorrow.

— Uncle Bob Martin, _Clean Code_

If things are always changing and you can never be certain about the way things are, then it makes sense you would (as these authors argue) take an approach to your code that would maximize changeability and learning, such as small refactorings.

In my experience, those who take a different view wouldn’t argue that things *never* change. Instead, they say that for their company/projects/industry, the problems are well-understood, scope is set in advance, and requirements change much more rarely. And I’d agree that this stability is more common in some industries. Then, whether they would say it or not, there is the implication that since they know the needs of the system in advance, the design they create for it is more-or-less correct. If this is the case, major changes will be few and far between, so they could be batched up into large refactorings.

To the degree that you believe that your domain and business are fixed and understandable, you will tend to design up-front and batch up changes to avoid the overhead of small changes. But to the degree that you believe that your domain and business are ever-changing and can’t be perfectly understood, you will tend to defer design decisions and optimize for many frequent changes and learning.

My experience in software development is on the side of change and uncertainty, but I don’t want to argue that that is true for every developer in every organization in every industry. However, I think it’s true for a lot more developers than think it is.

If you would say that small refactorings aren’t necessary because you got the design right from the start, my question would be, does your application have any code that slows you down? Code that’s difficult to work with? Code like this comes from situations where the author’s knowledge of what’s needed isn’t perfect. “Change” doesn’t just mean “I thought our business was selling shoes but now we trade cryptocurrency”—it can also mean something as simple as “I thought these two functions had different branching logic, but now it seems like they are branching on the same conditions.” You don’t have to embrace a worldview of “you can never know anything for certain” to benefit from small refactorings. Check out one of the books I’ve mentioned above, give small refactorings a try, and see if they help you code faster.

## Downstream Effects
I said at the start of this post that your view on refactoring has a significant influence on the programming paradigms, type systems, testing approaches, and emphasis on code quality that you take. Let’s talk about why.

### Paradigm
When object-oriented programming is described at a high level, one of the most common benefits given is that it makes your software more maintainable by allowing you to control the dependencies in your code to isolate changes. Surprisingly, in the literature I’ve read about functional programming, very little is said about how it enables change or allows you to control dependencies. And when I read about the downsides of procedural programming when it comes to coupling, it sounds very similar to what I see in functional programming:

> Back in the bad old days, you wanted to get the representation right as quickly as possible because every change to the representation bred changes in many different computations.
>
> Objects (done right) change all that. No longer is your system a slave of its representation. Because objects can hide their representation behind a wall of messages, you are free to change representation and only affect one object.

— Kent Beck, _Smalltalk Best Practice Patterns_

> Back in the days when data was separated from computation, and seldom the twain should meet, encoding decisions were critical. Any encoding decision you made was propagated to many different parts of the computation. If you got the encoding wrong, the cost of change was enormous. The longer it took to find the mistake, the more ridiculous the bill.

— Kent Beck, _Smalltalk Best Practice Patterns_

If most functional programmers don’t write about how their language allows them to cope with change, then I suspect that frequent changes are not part of their regular experience. If you’re taking a large-scale refactoring approach this is fine, but if you’re taking a small-scale refactoring approach it’s problematic. As functional programming becomes more mainstream I’m looking forward to hearing more research about if and how dependencies and changes can be isolated in a functional paradigm.

### Type System
The proponents of dynamic languages I read generally argue for them on the basis of a small-refactoring approach. And on the flip side, when I’ve heard statically-typed advocates describe the benefits for refactoring, it’s always been in the sense of changing your entire codebase at once. The flexibility dynamically-typed languages provide make it easy to make changes a little at a time. By contrast, statically-typed languages will often by default force you to propagate a change throughout the whole codebase before you can even run a test against the changed code, unless you intentionally work against this.

This isn’t to say you can’t do small refactorings in a statically-typed language; the first edition of _Refactoring_ has all of its examples written in Java. But I think it’s fair to say that dynamic types align more with a small refactoring mindset and static types align more with a large refactoring mindset. A quote I hear from time to time captures this well: an advocate of a statically typed language will say “the type system is great because it forces me to think about my types before I start writing my app.” This is a big benefit in a large-refactoring mindset but, I would argue, precisely the kind of thing an agile small-refactoring mindset tries to avoid.

### Testing Approach
In my time learning and speaking about testing in various circles, I’ve been surprised by how different the views on testing are. Some folks love Test-Driven Development, and others prefer to write their tests after their production code. Some find value in the testing pyramid, where relatively few acceptance tests are supported by a large base of unit tests. But others prefer mostly higher-level tests like integration or acceptance tests, and only unit test when absolutely necessary.

I think one of the major reasons for these differences in views on testing is your view on refactoring. As Uncle Bob argues in [“Architecture: The Lost Years”](https://youtu.be/HhNIttd87xs?t=3017), if you want to do small refactorings frequently, you need to have absolute confidence that you haven’t broken anything. But it’s hard to be disciplined enough to write thorough test coverage after your production code is written, and it’s difficult for code coverage tools to correctly measure that every case is covered. If instead you Test-Drive your code, you can know that it’s fully covered by tests because you didn’t write any production code until you had a test driving you to do so. This means you can make many small changes with confidence.

How about the issue of which types of test to write? If your refactoring mostly involves major changes, then unit tests won’t help because you’ll mostly be rewriting or creating entirely new modules. In that approach, it’s better to have end-to-end tests that cover the functionality of your app as you do those major rewrites under the hood. But if you’re doing lots of small changes within modules you want quick and thorough feedback that they still work, and unit tests are great for that.

### Code Quality
On dev twitter recently there’s been a trend of dunking on the idea of “clean code.” Why put all that effort into code if I’m either going to keep it untouched, or else replace it? Developers as prestigious as Katrina Owen and Sandi Metz are cited (I think incorrectly, as we'll see) in support of this view:

> “Writing Shameless Green is fast, and the resulting code might be "good enough." Most programmers find it embarrassingly duplicative, and the code is certainly not very object-oriented. However, if nothing ever changes, the most cost-effective strategy is to deploy this code and walk away.”

— Sandi Metz, Katrina Owen, _99 Bottles of OOP_

However, there’s a minimum level of quality that they are still advocating in any code. They aren’t advocating using misleading variable names, or using mutable global state for no reason. If there’s pre-existing code that has these things you can leave it, but that doesn’t mean it’s a good idea to write it that way. In _99 Bottles_ they evaluate an “Incomprehensibly Concise” alternative and say that “Shameless Green” is better, an alternative that maximizes understandability.

If you aren’t planning on changing code much, or if any refactoring you do involves throwing away and rewriting entire large modules, then code quality may not matter very much. But if you’re planning on incrementally changing modules to account for new needs, then you need to be able to quickly understand the code that’s in them—and that requires code quality. That doesn’t mean applying every design pattern or overabstracting (just the opposite, actually)—but it does mean thinking about the arrangement of your code and making it the best representation of the requirements as you have them today.

## Takeaways
If you're only doing large-scale refactoring, you'll probably benefit from taking a look at statically-typed languages, functional programming languages, more end-to-end testing, and maybe worrying a bit less about code quality. All of these things have a track record of helping in that situation. The prevalence of the large-scale refactoring mindset is a major factor in the rising popularity of these approaches today.

If you're doing small-scale refactoring, the benefits for it from dynamic typing, object-oriented languages, test-driven development, and high code quality are well-documented. The other approaches *might* be beneficial as well, but there are few people evaluating them from a small-scale refactoring mindset. The body of knowledge we have on small-scale refactoring seems to indicate that they will hinder it, but we can always learn new things. In particular, advances in statically-typed languages may bend the curve so that we get more of the benefits of them with fewer of the downsides for small-scale refactoring. My caution, though, is that when you hear a statement like "statically-typed languages aid refactoring," don't assume that applies in the same way to the small-scale refactoring you do.

And whatever your approach to refactoring is, don't assume that large-scale refactoring is just small-scale refactoring sped up. *Small-scale refactoring is attempting to do something that large-scale refactoring is not:* to get feedback from the code that influences your design. If you haven't tried small-scale refactoring, I'd encourage you to do so!

It’s true that tooling advances can change the paradigm for how we program in fundamental ways. At the same time, I spent the first ten years of my career putting my hope for things to get better in languages, frameworks, and tools, and I was regularly disappointed. It wasn’t until I started to learn about the *process* of programming—testing, design, refactoring—that I started to see real improvements in the reliability of my applications and the quality of my life as a developer. This has given me a wariness towards claims of tools that they solve all our problems. You *might* have run across the next big thing that will fundamentally change programming. But whether or not you have, I think your best bet is to invest in the process of programming. Those skills will go with you however much the tools do or don’t change.
