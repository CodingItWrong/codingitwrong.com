---
title: 'Four Essentials for an Effective Software Development Process'
---

Working in software consulting for nine years, I’ve gotten to work on teams with all kinds of different processes. Some have very heavyweight waterfall processes. Some have no processes at all. Some have tried scrum and have moved to kanban, or tried kanban and have moved to scrum. When I join a team I try not to bring too many preconceptions. There isn’t just one right process; any of these approaches can work for a team. (Except waterfall, obviously!)

Instead of recommending just one approach, over time I’ve seen a pattern of what are the most essential elements of a project process to work effectively together. These are the elements that, when absent, I feel the pain the most. And when they are introduced, they are the elements that result in the biggest improvement in delivering reliable software quickly.

All of these patterns address two different perceived problems: slow delivery and buggy delivery. We all want more reliable features quickly, and we’re frustrated when it doesn’t happen. When I ask clients what they’re looking for help with, often they ask “can we get features delivered more quickly” or “can we have fewer bugs?” It’s tempting to try to address these issues by trying to go faster or trying to be more careful. But this reminds me of an illustration Kent Beck uses: trying to make a car go faster by pushing the speed dial higher. It’s an indicator, not a cause.

What does help with speed and reliability is this: work needs to be split into pieces that are **visible, small, prioritized, and completed:**

- **Visible:** visible to all team members in some kind of tracking system
- **Small:** split into the smallest user-facing chunks possible
- **Prioritized:** assigned to people as their main priority for the majority of their work week
- **Completed:** work should be brought all the way to completion before moving on

Let’s talk about what each of these means.

## Visible

Work should be visible in some kind of system for tracking projects, stories, or tasks.

I’ve always used electronic systems for this. I know early Agile movement folks are often in favor of physical systems instead: cards or sticky notes on a board. I’ve had good experiences with electronic systems, and as I’ve often worked remotely an electronic system is a necessity. One of the advantages of physical systems, though, is that they prevent these tracking systems from getting overly complex.

The reason for tracking work in a system is for the visibility, so that everyone can be see what is being worked on. This means keeping the _status_ of the work visible as well: is it in progress? Blocked? Finished development but waiting to be tested? Tested but waiting to be deployed? A given team might have any number of different stages for the work; if a stage is important to the team, make sure that stage is visible in the system.

Some developers might bristle at this kind of tracking because of the fear that it will be used for micromanagement. That’s definitely a possibility, and that kind of micromanagement does cause problems! Adding a lot of overhead to individual tasks can slow a developer down. But there are _good_ reasons for visibility of work. What are they?

- Avoiding duplicate work: if everyone is working on things that are not visible, multiple developers might be working on the same thing, or overlapping things.
- Avoiding delays: if work is not visible, a unit of work might be done, but not everyone on the team might know that. For example, if something is finished being coded, it’s important for the person who will test it to know it’s in their court. Otherwise there can be a long delay between when the work is done and when the testing begins. This can cause delays finding and addressing problems, as well as making the fixes take longer because the original developer has lost context on what they were working on.
- Avoiding getting stuck: any developer can get stuck. I like to think I can avoid but, but really I can get mired in a tricky story and not realize that I’ve been stuck for a while and could use help. When work is visible, others on the team can see what I’m working on and how long it’s taken. Ideally we trust each other and give each other time to work on things. But if someone seems stuck, we can offer to help, make suggestions, brainstorm if things can be split into smaller pieces, or discuss if we should try a different approach.
- Get a sense of progress: Now, it’s important to emphasize that tracking work doesn’t mean you can perfectly predict exactly how long a project will take. Estimates are never exact. I’m not even suggesting _any_ estimation or velocity tracking here. All I’m saying is, when your work is visible, the team can review it and get _some sense_ of progress. Without any estimation, it’s easy to tell the difference between when there are three bugs to be fixed vs. when there are three complex screens to be built. If you aren’t aware of what work is remaining to be done before a given release, that’s a lack of visibility.

Some teams that prefer low process will say, instead of tracking that information in a system, we just ask the team what they’re working on, how it’s coming along, and how much is remaining. There are downsides with this approach, however. Asking me these questions means that each time the pressure is on me to make sure I remember everything that’s remaining without any omissions. If I do forget part of it, then I’ve given a misleadingly optimistic response. I worry about doing that, causing stress. If it’s valuable to have this information, track it in a system. Systems are better at keeping tracking of information than humans, and anyone can see the state of work at any time without having to take up others’ time. If there are obstacles to entering that data and keeping it up-to-date, the right answer isn’t to give up on using systems, but to work through those obstacles.

Now, a system doesn’t take the place of a daily standup; I recommend a daily standup _alongside_ a system for tracking work. Specifically, I recommend that the focus of the standup is precisely to _review the system_. I recommend looking at the system together (on a screen share virtually, or on a large monitor if in person). Each person points out what they are working on, and if it’s out-of-date, they update it right then so it _becomes_ up-to-date. Even though I value tracking systems, I often forget to update the status of an item in it; a standup is good accountability.

Visible work helps with “bugs”, because if what you think of as a bug was not actually recorded as functionality that needed to be built, that’s an issue of visibility. Track the work and then you should see fewer such “bugs.”

Visible work helps with speed of delivery, because if not everything being worked on is visible, then effort is going towards things you don’t know about. You’re not able to readjust team priorities.

## Small

The units of work that are tracked should be small. _Very_ small. Probably smaller than you think.

To start, try to split work into the smallest chunks of user-visible work that you can. User-visible means that you don’t work on building out the whole database access layer in a way that isn’t visible to users. Work in vertical slices. If you’re adding a new type of data to the system, maybe the first story you work on is a list screen. Don’t build the detail screen at the same time, or the ability to add, edit, or delete records. Just build the list and get it displaying the data. Viewing details, adding, editing, and deleting can all be separate additional stories.

A lot of the time, we can resist splitting up work into smaller chunks because it’s not clear how to split it up, and splitting it up adds overhead.

But there are a few different reasons to split work up into small pieces. First, there are a number of reasons it helps with speed of delivery:

- You get _something_ working sooner, rather than nothing until the very end.
- A small unit of work keeps you focused, so that you bring it to completion. Otherwise you might be making progress on multiple parts of a large unit of work, without getting any of them to the point of being useful.
- It’s easier to keep a small unit of work in your head. That way you can understand it more deeply, so you’ll be more effective at completing it.
- A small unit of work gets feedback sooner, because it can be delivered to someone to test it. When we think of “speed” we probably don’t mean we want non-working code faster. If we want something _working_ faster, then getting feedback sooner is important.
- Splitting up a large chunk of work into smaller pieces can often lead to parallelization. Even if just two developers can work within that large area, that means that the overall large chunk will be finished twice as fast. And you can often parallelize more than two. (The practice of “swarming” is related to this.)
- It’s motivating to be able to bring a small chunk of work to completion, and that motivation helps with speed.
- Once you commit work to the main branch (rather than just locally or in a feature branch), you no longer have to repeatedly handle merge conflicts to it. When you work in smaller units, they can be committed to the main branch sooner. This avoids this merge conflict cost, further speeding you up.

Small units of work help with “bugs” as well. Thinking through splitting work into small chunks can help you think through all the different edge cases that need to be handled, so that they’re handled from the start instead of being reported as bugs.

One more benefit of small units of work is flexibility. When you are working on a large unit of work, you may have something higher-priority come up. You’d like to pause what you’re doing to work on that higher priority, but you know there will be overhead at pausing that in-progress work. So you ask how soon the developer will be done, and they feel like they should say it will be soon, even if it won’t be. You’re now in a lose-lose situation. But if you’re working in small chunks, a developer can switch priorities as soon as one small chunk is complete. Even if that small chunk doesn’t add up to a fully-complete large feature, the progress that has been made is complete and tested.

One of the best resources I’ve run across on the value of smaller steps is this talk by “GeePaw” Hill with an ironically large title: [“Want More Value Faster? Take Many More Much Smaller Steps”](https://youtu.be/1mOs1_pvS9A)

## Prioritized

Work assigned to people should be their main priority for the majority of their work week.

Ideally, developers should be fully allocated to working on one development task. They shouldn’t have other responsibilities outside of development, such as support for other systems. They shouldn’t have development responsibilities across two different projects. Either of these take time away from the forward development of the system.

For similar reasons, it’s also best for developers not to be actively working on two different stories within the _same_ project either. (Kanban follows this via the concept of work-in-progress (WIP) limits.) If you’re using a pull-request-based workflow like many developers today, it can seem impossible to work on just one story at a time: what do you do when your pull request is up and waiting for review? First, work on all the other things you can do that _aren’t_ starting another story (the things that it’s tempting to delay when you’re heads-down in the code):

- Review _others’_ pull requests to help _them_ get unblocked for their next story.
- Look over your own PR to ensure you’ve covered all the edge cases.
- Research your next potential story to understand it, ask any clarification questions, and begin thinking through an approach _without_ starting coding.
- Catch up on emails and messages, other responsibilities you have that aren’t coding.

If you’ve done all of this and _still_ don’t have pull request feedback, as a team you may decide that it’s okay to pick up a second story—but if you do, you should consider it fully interruptible. Responding to pull request feedback takes priority over working on the second story, no matter how much overhead it causes to stop and start it. (See Completed below.)

What if the team _needs_ you to work on something other than the story you have assigned? Sometimes it happens. This is another good reason to work on small chunks: this way you may be able to get it finished quickly before you work on this other responsibility. If you haven’t started coding, I’d recommend unassigning the story from yourself and letting the team know it’s available to be picked up by someone else; that way it’s not blocked on you. If you’re in the middle of coding it, things are trickier. It may be tempting to keep it assigned to you, but this means it will be blocked on you, as well as any other work that it blocks. It can be better to put your work-in-progress code somewhere that anyone can get back to in the future (for example, a git branch on the origin server, or a draft pull request), then make a note of the state of the story in the story itself, including a link to the in-progress code. (This is one good reason for using an electronic system that allows comments).

How do we fit in work to handle urgent bugs and other production issues? If you have several team members, a common practice is to have a rotation where, at any given time, one team member is assigned support. A rotation means it’s not stuck with just one person, or left for the person who is most stressed by them to pick them up. If you’re using a sprint-based approach you can rotate which developer is on support each sprint. When a person is on support rotation, _do not_ assign them any feature work. Even if you say “support is your main priority and this feature comes second,” it is very hard to break the temptation to focus first on the feature stories and let support slip.

What can the support developer if there are no bugs or issues? They can look through error logs for anything that may indicate hidden issues, look into them, write up findings, or research possible fixes. They can also take on optional non-urgent “tech debt” cleanup tasks, but it’s important for them to treat any support needed as the higher priority.

For teams and organizations that are very small, you may not have the luxury of having a developer fully allocated to one system. If not, splitting the developer’s attention across multiple things may be unavoidable. Just be aware of the impact it will have on speed.

## Completed

A unit of work should be brought all the way to completion before moving on to another unit of work.

When a developer has a story assigned, they’re responsible for moving it all the way to done: getting it reviewed, merged, and recorded in the tracker in a state that lets the next person know it’s ready for testing.

In a pull-request-based workflow, it can be easy to lose attention once you’ve opened a pull request and asked for reviews, especially if there’s a significant delay on your team between when a pull request is opened and when it gets reviews. It’s the responsibility of the author of the pull request to ask for reviews, follow up if they haven’t received them, actively check for feedback and respond to it with discussion or code changes, and ask the reviewers for re-review after updates—whatever is needed to get the pull request to the point that it’s approved and merged.

If this flow isn’t going efficiently, think about what you can do to make it more efficient. Can GitHub notifications be set up to automatically inform people via email and the GitHub notifications button that their review is needed? Would a Slack integration help to automatically post new PRs? Should team members just manually ask for reviews in Slack?

If you have moved on to something else, including potentially coding another story, remember that finishing your first story is the highest priority. When pull request feedback comes in, stop what you’re doing and act on it. This will likely feel like an interruption, because it is. But ultimately it doesn’t help the team if you have multiple units of in-progress stories. Your goal is to get stories finished, so it’s best to prioritize whatever is needed to get the _first_ story finished. Any amount of inefficiency is worth it to that end.

Once a pull request is merged and the code is ready for testing, the developer should update the state of the story in the tracking tool to let the person who needs to test it know that it’s ready for them.

Another thing that can get in the way of completing stories is not coding everything that’s needed. Maybe a developer hesitates to open up a pull request because the last 10% of the functionality is tricky. It can be tempting to respond to this in a few different ways.

- You might want to move on to another story, because it feels more productive. It may feel that way, but we need to finish the work eventually, so it’s best to stay on one story and bring it to completion.
- You might want to mark the story as complete even though 10% of the coding is remaining. This won’t help because that work is no longer visible, and it might not be discovered until it’s an emergency, resulting in nights-and-weekends work, a delayed release, or issues in production.

Instead, if part of the functionality will take a lot more effort, consider splitting that functionality out into a separate story in the tracker. Be sure to update the first story to make it clear what is _not_ included in it. This way, you can land the earlier work in a way that’s visible, and you have a clear indication of the work that’s remaining. It’s a way of discovering that a story could be smaller than it initially was, and gets you all the benefits of small stories we discussed above.

## Conclusion

Think through how your team is operating now. Does it feel like things are going slower than you like, or that the code is less reliable than you’d like? If so, think about the four elements we’ve discussed:

- Is all of your work visible to the team in a system, and kept up-to-date there?
- Is it split into the smallest units of user-visible work it can be?
- Are developers empowered to put the majority of their effort into a single unit of work?
- Are developers bringing the work all the way to completion?

If the answer isn’t a clear “yes” for all of them, consider spending some time on it. I think you’ll find it has major payoffs in speed and reliability.

If you’d like to learn more about these dynamics, I recommend checking out [_The Nature of Software Development_](https://pragprog.com/titles/rjnsd/the-nature-of-software-development/) by Ron Jeffries. It’s a methodology-agnostic look at the dynamics of software development and how we can tailor our approach to be more effective. (This isn’t an affiliate link, so feel free to get the book wherever you like; I’ve linked to it at The Pragmatic Bookshelf as they offer an option for DRM-free ebooks.)
