---
title: "The Cost of Change"
---

Early in my career, the biggest problem on most of my projects was that the cost of change increased over time. Martin Fowler summarizes it well:

> When I talk to software developers who have been working on a system for a while, I often hear that they were able to make progress rapidly at first, but now it takes much longer to add new features. Every new feature requires more and more time to understand how to fit it into the existing code base, and once it’s added, bugs often crop up that take even longer to fix. The code base starts looking like a series of patches covering patches, and it takes an exercise in archaeology to figure out how things work. This burden slows down adding new features—to the point that developers wish they could start again from a blank slate.

— Martin Fowler, *Refactoring*

What helped me solve this problem were agile development practices. Specifically:

- **Evolutionary Design:** building the simplest possible implementation of today’s feature set, then adjusting the code to incorporate new features as they are added.
- **Refactoring:** making small changes to the arrangement of the code to ensure it’s maximally changeable and the best fit for today’s feature set.
- **Test-Driven Development:** writing a test before I write a line of production code, which allows me to make changes with confidence, and drives me to loosely-coupled code that's easier to change.

It's no coincidence that agile development practices help me manage the cost of change. This is one of the key goals of Extreme Programming (XP), one of the early agile methodologies:

> XP teams work hard to create conditions under which the cost of changing the software doesn’t rise catastrophically.

— Kent Beck, *Extreme Programming Explained*

This theme is echoed by Freeman and Pryce, Fowler, Metz, and probably many others in the agile movement in different ways.

I’m realizing that I’ve internalized “controlling the cost of change” as one of my personal key goals as well. The contrast is intense between the pain of the cost of change on my early projects, and the relief of being able to control it on my later projects. This contrast is so stark that minimizing the cost of change has continued to be one of my highest priorities. In fact, researching how to minimize the cost of change is more motivating to me than other technical research. I’m more excited to better understand how to control the cost of change, and how to apply such principles in new programming contexts, than I am to solve other technical problems.

Not every software system is controlled by the cost of change. Some systems are controlled by the cost of hardware, or networking bandwidth. Others might be controlled by the cost of licensing fees. In such systems, you would get more bang for your buck focusing on minimizing those costs—for example, by increasing performance in the former case, or switching to a lower-license-cost system in the latter.

But I think many more systems are controlled by the cost of change than you’d think looking at most discussions in the software world in the last few years. Most discussions I see are around topics like performance and mathematically-provable correctness, and few about minimizing the cost of change. But your system might be controlled by the cost of change if:

* Development costs are your highest software-related expense.
* You are slowed down from delivering features due to bugs or lengthy time to test.
* You aren't even able to do new feature development due to the quantity of bugs to work through.
* You don't upgrade dependencies (maybe even to apply security patches!) because the cost of retesting is too high.
* Feature development needs to stop for lengthy periods to handle updates for new operating system versions.

If your system isn't controlled by the cost of change, there is little reason to focus on lowering that cost. But if your system *is* controlled by the cost of change, it's extremely important to focus on lowering it.

One way to think about the cost of change is, when I'm asked the question "why is this feature taking so long to build?", I want to have a good answer. Good answers include "because our system needs to handle 1 million transactions per second" or "because our system is a pacemaker and it needs to never crash." Those are examples of situations where there is a different controlling cost other than the cost of change, and they may mean you need to incur a higher cost of change. Poor answers to "why is this feature taking so long to build?" include "because we didn't write any tests" or "because we don't actually know what the code is doing." Those are answers I would like to avoid having to give. If you're being asked that question a lot, it's an indication that the asker thinks the cost of change is the controlling cost for your system.

One reason I haven't been as excited about some newer technologies in the last five years is that they haven't been described in terms of lowering the cost of change. There could be a few reasons for that. They may actually increase the cost of change, or they may be neutral. Maybe they *do* lower the cost of change, but they just haven't been described in those terms. Maybe they have been described in terms *related* to lowering the cost of change, I just haven't seen the connection. Going forward, when I run across new technologies, I'll ask myself "does this lower the cost of change? How?"

Some would argue this agile approach is useful for any type of software system. But *at least* we can say there is a type of system that it’s especially applicable for. In the past I've struggled with defining that class of system. Is it business applications? Information systems? CRUD systems? But I think the clearest classification is simply saying, the class of systems agile development practices are especially helpful for are *systems where the controlling cost is the cost of change.* (These practices may work great in other systems too, and since I'm used to them I might always reach for them and get some small, incremental benefit.)

If it's true that so many systems are controlled by the cost of change, this suggests that I evaluate new technologies and practices by asking the following questions:

1. What problem is this attempting to solve? Is it the high cost of change? If so, does it lower the cost of change more than alternative approaches?
2. If this is *not* attempting to solve the problem of the high cost of change, what problem *is* it trying to solve? Do I have that problem? If not, there’s no reason for me to consider this technology or practice.
3. If I *do* have the problem this is solving, while solving that problem, does it raise or lower the cost of change? If it raises the cost of change, is that a price I am willing to pay to get the benefit it offers?

It also suggests that I evaluate my own projects and ask, is the cost of change really the controlling cost of this system? Or am I just carrying habits over from past projects? If there is a different controlling cost, then I need to be prioritizing that, no matter what my habits or preferences are.

I suspect that some would argue that there are more important concepts to focus on than controlling the cost of change. What about performance? What about provable correctness? Isn’t it better to be faster and more provably correct? Yes…in the absence of tradeoffs—and there are almost always tradeoffs. As I mentioned earlier, there are certainly classes of systems for which these factors are the most important—that is, they are the controlling cost. But there are many systems where shaving 100 milliseconds off an HTTP response time is so unnoticeable that it’s not worth delaying a release by so much as a week to achieve it. For a system like that, arguing that performance is the most important factor is a philosophical statement that doesn't match business realities. Someone who believes that would be best served by switching to work on a system where performance *is* the greatest business need as well, aligning their beliefs with the requirements. In the same way, if I believed minimizing the cost of change is always the most important thing, it would be best for me to work on systems where that is the controlling cost, so my preferences are aligned with what the business needs. (I don’t *think* I believe it’s always the most important, so much as just *preferring* to focus on it, but the outcome is the same: I should work on systems where that preference lines up with business needs.)

Focusing on performance or correctness independent of real-world priorities is computer science, and computer science is extremely important—and as a society we probably need to find ways to fund it better. But building software for businesses isn't computer science, it's software engineering (among other imperfect metaphors), and engineering is done in view of real-world constraints. Putting a Ferrari engine in a Honda Civic would probably result in higher performance, but it doesn't make economic sense either for the business or for the customer.

*(There’s a good chance anything correct in this post is taken from another agile writer without me remembering—if so, I apologize for not citing.)*
