---
title: "Small Commits and the Power of Git Bisect"
---

*This is a blog post that would benefit from an example. Unfortunately, I don't have one right now: demonstrating this process requires a complicated code problem that isn't easy to come up with outside of NDA client work. So this post will be less useful for learning from scratch. It will be more useful to people who already know some of the pieces, or have a more experienced developer they can pair with to see it in action, or for developers who know these things but aren't sure how to explain them to others. So for now I write this post, and maybe someday I'll have the example to add.*

Many developers newer to git treat it as a save function: whatever changes they’ve made, no matter how in-progress they are or how unrelated they are, they save them by making a commit. By contrast, I and many other developers advocate for small, focused, well-explained commits. But it can be hard to convince someone of the benefit of doing so. If your code reviewer doesn’t review commit-by-commit, what is the point of crafting a clean commit history?

Some of the benefits of small commits include:

- They help you organize your thoughts as you work on your feature.
- They help you confirm you are only committing intentional changes, not unintentional ones.
- They help keep you accountable that you understand the changes and why they work.
- They help confirm your code is decoupled, because you’re able to make small changes that don’t impact the rest of the app.
- They allow you to rearrange commits later, if you find they need to be done in a separate order or in separate PRs--especially when you find a bug fix needs to get merged before your overall PR.

But maybe the most tangible benefit of small commits is that they make it easier to track down and fix errors. Let's talk about how.

Say you’ve reached the end of a long feature branch, but now something isn’t working, and you don’t know why. Maybe you haven’t been running the automated tests all along, and now that you’ve run them at the end, you get a mysterious error (or 500 of them). Maybe you’re doing manual testing of edge cases that weren’t feasible to test all the way along, and things aren’t working right. The error is so mysterious and the amount of code changes is so big that it’s hard to know where to start the investigation. You feel stuck. What do you do?

There’s an approach you can take that will help prepare you for this situation that will eventually arise: write your code in small, focused git commits that move the functionality forward while keeping everything working. If you have commits like this, then if you run into a problem at the end, you can narrow it down to which commit introduced the error. There is a git command, `git bisect`, that helps automate this process. Once you’ve narrowed the problem down to one commit, that is a lot less you need to focus on for troubleshooting. You could even throw that commit away entirely and start over, safe in the knowledge that your previous commits didn’t cause a problem.

Before we look at `git bisect` itself, though, we need to talk about some prerequisites for being able to use it effectively.

## Working in Small Steps
It may not always be obvious how you can split your work into small commits that keep everything working. It may seem like every change depends on every other change. Splitting it into small pieces may seem like a lot of overhead, or even impossible.

Small commits get easier with practice. In Kent Beck’s book _Extreme Programming Explained_ in the section “Baby Steps”, he proposes a question that can help with this: "what's the least you could do that is recognizably in the right direction?" Each piece of our code should have a single responsibility; each commit should have a single responsibility too. As a result, a clear commit name functions as a test of whether the commit is focused: if you can't summarize it in 50 characters, you might be doing too many things at once.

Here are some specific suggestions for ways to split work into smaller commits:

* Put cleanup of white space, comments, or names in one or more commits, and your feature change in a different commit.
* Put rearranging existing code (refactoring) in separate commits from adding in new code.
* Implement a new function in isolation in one commit, then call it in another commit.
* Make visual changes in separate commits from functionality changes.

## The Mechanics of Small Commits
When you get ready to commit changes, review all the diffs to see if they make up one focused change or not. Git has the concept of staging: not every change in the working directory needs to be included in the commit. GUI git clients allow you to see the diffs and choose which lines get staged. On the command line, `git add --patch` allows you to review one change at a time and choose which lines are staged. You may be able to take one set of changes in the working directory and make several separate commits from it, one at a time.

One thing *not* to keep in separate commits is fixes for problems in previous commits. If you leave broken commits in your history, it will be hard to use `git bisect` to find where the error was introduced, because other errors will crop up along the way, and may even mask you from seeing the error you’re focused on.

What if you make a small commit and then realize it wasn't correct as intended, and you have a fix to make to it? You can amend the previous commit in git by adding additional changes to it instead of making them a separate commit; on the command line this is done with `git commit --amend`. If you find a fix that needs to be applied to a commit earlier than the last one, you can use `git rebase --interactive` (“interactive rebase”) in two different ways. First, you can go back in time to edit a previous commit to amend changes to it. Second, you can make the changes in a new commit, then use interactive rebase to move that commit back to after the previous one and squash it down.

How about if you make a commit and then realize that it really should have been two? You can use interactive rebase to go back to that commit, then `git reset [previous hash]` to remove that commit and put its changes back into the working directory. From there, you can re-commit those changes in separate commits using staging.

## Getting a Clean Commit History
When it's time to bisect, you need to choose a “good” and “bad” commit—“bad” demonstrating the problem, and “good” as one commit where it’s known the problem hadn’t been introduced yet. When you have the branch with the problem checked out, a good choice for your “bad” commit reference is `HEAD`, which points to the latest commit on your current branch. For your “good” commit, often it's simplest to refer to the long-running branch your feature branch branched off of (maybe `main` or `develop`).

But this works the most simply if your branch is a simple linear series of commits after branching off of the parent. If you've done a `git merge` to pull in changes from the parent along the way, the history will be more complex. This is why I recommend using `git rebase` instead of `git merge`. For example, if you’re on a branch off of `main`, and additional changes have been added to `main` since then, running `git rebase main` while on your feature branch will rebase your branch off of the latest `main` commit. Rebasing reapplies your commits on top of the new starting commit. Something tedious about rebasing is that you might have file conflicts at many or all of the commits as they’re applied. This is another good reason to keep your commits small and focused: it decreases the likelihood that any given commit will have conflicts. (One situation where I avoid `git rebase` is when I’ve already opened a PR and reviewers have commented; rebasing changes the history of your commit and review comments will likely be harder to find. Consider the downsides, but if you’re stuck on an error the rebase may be worth it.)

If you have a lot of conflicts, another alternative is to create a new branch off of your base branch, and copy over one commit at a time with `git cherry-pick`. You will still get the same number of conflicts, but because you aren't in the middle of an interactive rebase, there is less pressure to finish it all at once. You can stop in the middle, pick it up again later, switch branches to compare, etc.

If the idea of small commits is newer to you, you could end up in a situation that seems hard to recover from. You have a lot of commits that each include a lot of intermixed concerns. Future commits correct things in previous commits, so they aren't moving things forward. What can you do? You can treat your current branch as a destination end state, and recreate it from scratch, this time via a series of focused commits. Look at the overall diff of what changed, split it up into small steps, and apply them one at a time on a new branch. There’s a chance that the error will inadvertently be fixed as part of this process as you think more precisely about what you’re changing. But likely the end state will still have the mysterious error, but you shouldn’t be discouraged: your new git history of small commits will allow you to troubleshoot that error using `git bisect` (using steps that we'll eventually get to below). You don’t need to make a full plan at the start for how you’ll split the work into small commits: find one thing that you can put in a focused commit and do it, then look for the next change.

## Visualizing Your Commit History
In order to be able to even determine if you have a clean git history or not, you may need to visualize your git history in ways you aren’t used to. The goal is to be able to see the sequence of commits, how they relate to other branches, and the contents of each. GUI clients usually provide easy ways to do this. On the command line, here are some helpful commands:

- `git show` - Shows the contents of the last commit.
- `git show [revision]` - Shows the contents of the commit with the given revision: for example, a commit hash or a branch name.
- `git log` - The most basic git history command. Shows the history of commit messages, but doesn’t show the contents of the commit.
- `git log --oneline --patch` - Shows the contents of each commit, and only the minimal one-line commit message. Helpful for seeing how your code changed over time.
- `git log --oneline --graph` - Shows commit history with one line per commit, including visualizing intersecting branches as a graph. This makes it easy to see when branches fork or merge.

To make these commands more memorable and encourage yourself to use them more, it’s a good idea to have command-line aliases for them. If you’re using `zsh` (including users of macOS Catalina and above by default), you can install [Oh My Zsh](https://ohmyz.sh/) and then [its git plugin](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git) to get a lot of useful git shortcuts, including `glo` for a one-line log and `glog` for the graph. Alternatively, you can add them as aliases yourself: in `zsh` you can add `alias` commands to `~/.zshrc`, and any other shell should provide an analogous mechanism.

## The Mechanics of Git Bisect
With all the above steps in place, here’s how you can use `git bisect` to find the problem:

1. Check your git history to confirm it’s a simple linear sequence of commits going back to your parent branch.
2. Check out the parent branch and confirm that the error is not present there.
3. Check out your feature branch again.
4. Run `git bisect start HEAD [parent-branch-name]`
5. Git bisect will check out a commit in the middle of your branch.
6. Test to see if the functionality is broken. If broken, run `git bisect bad`. If not broken, run `git bisect good`.
7. Git bisect will check out another commit, depending on whether you said the previous one was `good` or `bad`. If `good`, it assumes the error was introduced later, so will check out a later commit. If `bad`, it will check if the error was introduced earlier by checking out an earlier commit.
8. Test again and run `git bisect good` or `bad` again. Repeat.
9. Eventually, `git bisect` will narrow it down to the commit that introduced the problem and show that commit to you. Save that commit hash.
10. Run `git bisect reset` to end the bisect process.
11. If you want to be cautious, check out the commit you saved and make sure the problem does occur there. Then, check out the previous commit and make sure the problem does _not_ occur there. If so, you’ve confirmed the problem commit.

After examining that commit to figure out the problem, you _could_ add a fix as a new commit at the end of the branch. But why not take the opportunity to keep a clean commit history? You can use interactive rebase to go back and edit the problem commit. One benefit of this is that if _another_ tricky error comes up, your git history is ready for another bisect.

## Conclusion
One difference between small focused commits and large unfocused commits is a difference of who is in control: you, or the code you’ve already written. When you have large unfocused commits, you are kind of stuck with your code. If something is wrong, you have a ton of work to do to figure it out. If you need to undo something, you have to do it by hand, which is a big pressure against changing what you’ve already worked so hard to do. By contrast, when you have small focused commits, you are in control. You can go back and see why you made a change. You can modify those changes, reorder them, combine or split them, put them on separate branches, or redo them later after other branches are merged. Your code is fluid, malleable, adjustable. This kind of setup encourages you to improve your code, to make it the best you can.

What's the takeaway from all this? There are a number of different skills you can learn and practice:

* Learn git features, either in a GUI if it supports them, or in the command line:
	* Visualizing git history
	* Examining individual commits
	* Reviewing diffs when committing
	* Staging just some of the changes in the working copy, not all of them
	* Amending commits
	* Rebasing off of the parent branch rather than merging
	* Interactive rebase to edit, reorder, and squash commits
	* Bisect to identify commits that introduce errors
* Practice general strategies for development:
	* Small, focused commits
	* Working on one thing at a time
	* Refactoring in preparation for functionality changes
	* Decoupled code

There's a lot to this process; if it's new to you it's not something you can learn in an afternoon. But if you think about the long-term benefit of when all the pieces come together, it can be motivating to learn and practice each individual piece, even if the benefits don’t materialize right away.
