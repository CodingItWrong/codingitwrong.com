---
title: 'Software Development Values'
---

For a few different reasons recently, I've been thinking through what adds the most value to the software development process.

First, nine years of consulting put me in many different organizations and teams, forcing me to think through what things I was used to would _really_ make a difference and which were just preferences. Second, when I decided to move out of consulting into a "product shop," I had to decide what I valued so I could look for it and communicate what I was bringing. Third, when I accepted a role at Chick-fil-A helping stand up a new development team working with external partner consultants, I had to figure out the key things I wanted to align with them and what made sense to leave them to their own devices.

Here's what I landed on:

1. Incremental delivery
2. Layered architecture
3. Simple design
4. Comprehensive testing

## 1. Incremental delivery

In this context by "delivery" I mean shipping functionality to customers that provides value to them.

The key goal I've found in delivery is to ship value as early and often as possible. Functionality that's been built but not shipped is not only not providing value, it's actually harming you because it slows down your other development.

To achieve the goal of shipping value early and often, what I've found works best is _incremental_ delivery: organizing epics and stoires into the smallest possible user-visible slices, and bringing the highest-priority one to completion before starting the next. This contrasts with approaches where, for example, you might spend months building a "data layer" without starting any user-visible screens.

I've written about incremental delivery in more depth in the post ["Four Essentials for an Effective Software Development Process"](/2024/04/18/four-essentials-for-effective-software-development-process). An excellent book that hits on this as one of its main themes is [_The Nature of Software Development_](https://pragprog.com/titles/rjnsd/the-nature-of-software-development/) by Ron Jeffries.

## 2. Layered architecture

"Architecture" is a tricky concept to define, but in this context I mean "the big-picture organization of your software."

The key goal I've found in architecture is to enable adapting to change. If you give no thought to architecture, you can end up with code that is harder and harder to change over time, as it becomes a mess of spaghetti.

To achieve the goal of adapting to change, what I've found works best is _layered_ architecture: organizing the software into layers that change for different reasons. These layers can extend across system boundaries (web and native clients, web service APIs, middlewares), or within individual systems (routing organization, individual screens, business logic, data management, and web service client). This doesn't mean that each layer needs to be monolithic (there is room for multiple services or microservices), but only that the layer organizational principle is the one that I've found is consistently beneficial.

## 3. Simple design

"Design" here doesn't refer to visual design or user experience design, but rather to the design of your code. This is a lower-level concept than architecture: it refers to how you write individual lines of code, functions, and modules/classes.

The key goal I've found in code design is keeping the speed of development high. The first factor in speed of development is ease of understanding the code. The second factor is ease of making the changes you need to make: to add new code, or to adjust the existing code.

To achieve the goal of a consistently high speed of development, what I've found works best is applying software design principles. And some of the principles I've found the most consistently helpful are the Four Rules of Simple Design. These were articulared by Kent Beck as part of Extreme Programming, and [Martin Fowler summarizes them](https://www.martinfowler.com/bliki/BeckDesignRules.html):

- Passes the tests
- Reveals intention
- No duplication
- Fewest elements

I've written [blog posts about each of the Four Rules of Simple Design](https://codingitwrong.com/2024/03/06/simple-design-reveals-intention) exploring them in more detail.

Testing was so important to Kent Beck that he included it in the list of Simple Design rules. It's so important to me that I've pulled it out of that list into its own item:

## 4. Comprehensive testing

"Testing" here refers here generally to confirming that software works as expected.

I've read a lot of authors about software testing, and there are a lot of views on what is most important in testing. For me, the most important value of testing is to support ongoing maintenance. If you know a system is going away in two weeks or that it will never need to change again, then the importance of testing is minimal: it's pretty easy to make sure it works. But if you need to add more features to your app, or change it to new market needs, or upgrade its dependencies for security fixes or changes in underlying platforms, then you need a way to ensure your app continues working through those changes.

The best way I've found to support ongoing maintenance is comprehensive automated regression tests:

- _Regression_ because their essential purpose is to make sure things don't break during maintenance. You may use the same tests or other tests for different purposes, but _at least_ regression safety is a key role.
- _Automated_ because as your software grows the amount of time it takes to thoroughly regression test it increases exponentially. Computers can handle that testing load much more reliably and cheaply.
- _Comprehensive_ because if you only test portions of the app, or only the happy path, or do so inconsistently, then you don't actually get confidence that your app is still working.

There's nuance as to what exactly constitutes "comprehensive" in a given context, depending on factors like whether user interfaces are involved, what kinds of test tooling are available, and the consequences of errors to users and the business.

As to how to learn about comprehensive testing… that's a whole industry. It's probably a good idea to start with a book or course on test tooling specific to your programming language and platform.

## A solid foundation

Based on my professional experience, I'm confident that because we're building a system with incremental delivery, layered architecture, simple design, and comprehensive testing, the system will be a great experience for customers and the business. We'll undoubtedly find other important factors too, some of which will be specific to our context.
