---
title: How I Shipped My Side Project in Two Days
---

In the past two days I've managed to start a side project and get it shipped. I want to describe why that isn't nearly as impressive as it sounds, BUT ALSO that "that isn't nearly as impressive as it sounds" is exactly what is helpful to know to be able to ship your own side projects more quickly.

The project is right now just called “[Bible Reading][bible-reading-native]”; it allows you to track your progress reading through books of the Bible. This is useful for me because as a Christian I want to be reading the Bible, but I’ve had trouble getting motivated to do so.

But enough about the app itself; let’s talk about how I shipped it! As I do, I want to give a major shout-out to Adam Wathan and Ben Orenstein for their podcast episode [“How to Build an App in a Week”][fullstack-episode] for a lot of the inspiration to help me have this focus.

So, looking back, here are some things that helped me ship this side project so quickly.

## Have a Sufficiently Small Definition of "Shipping"
I didn't actually release the app to the App Store in two days; I just got it running on my own phone. But it's running in "production mode”: the data is stored in a real backend per user and authenticated.

For me, "shipping" meant "getting to the point where the app is useful for me, so I can figure out if it's actually an interesting idea, and therefore I'll be motivated to keep working on it.”

Narrowing this definition of "shipping" allowed me to focus on the most interesting features: how to store and present the data.

## Embrace Productivity Features
By "productivity features" I mean two things: abstractions and conventions.

An **abstraction** is roughly when you can work with higher-level concepts instead of implementation details. The app is built with [React Native][react-native], which is already an abstraction on top of iOS and Android; you can create an app with components that I would say are simpler than their underlying counterparts. Using a third-party component library like [React Native Elements][rn-elements] allows you to work at an even higher level of abstraction. For example, instead of having to decide how to style `View`s, `Text` elements, and `TouchableHighlight`s to simulate an iOS table view, you can use a `List` with `ListItem`s. You can say _what_ you want to show instead of _how_ you want to show it.

When you need to build a web service for your app, It can be easy for JavaScript developers to reach for Express.js, but Express operates at a low level of abstraction. You have to wire up routes manually, choose a database library and handle querying yourself, etc. For all my side projects (and all the professional projects I can) I use [Ruby on Rails][rails] for building APIs; even though it’s not written in JavaScript, I think the time it takes to get familiar with Ruby is more than made up by the time you save with all the abstractions Rails offers. In particular, I use the [`JSONAPI::Resources`][jsonapi-resources] library. All I have to do is tell Rails what database tables and fields I have, then tell `JSONAPI::Resources` what data I want to expose publicly, and all the details of setting up routes, performing validations, and executing SQL are handled for me. (You can learn more about [how to build web services with Rails][rails-api-screencasts] in a screencast series I created on The Frontier. It’s a paid site, but you can get a 45-day free trial with code CODINGITWRONG.)

Part of how Rails and `JSONAPI::Resources` are able to accomplish this is by **conventions**. A convention is when a programming community decides on a standard way to do something, and it unlocks a surprising amount of productivity. Rails has standard locations to set up routes, controllers, and models, and this means you have to write less code to get them to work together. The [JSON:API][json-api] spec defines a format for web service requests and responses that allows you to use zero-configuration tools on the frontend and backend to create your web service, instead of having to write custom code on both ends. I think a convention-based approach pays dividends for almost any project, because it allows you to focus on your business logic and features instead of the wiring and glue code. But conventions are *especially* important for side projects, where you want to make progress as quickly as possible.

## Forego Testing
As someone who is a big proponent of automated testing, I was surprised that I didn't feel any need to test the app. But at the very start the main concern is *not* whether the system is bug-free; it's whether the system is interesting. To get to the answer to that question as quickly as possible, I didn't feel the need to do extensive automated testing.

I might still reach for tests if they helped me move more quickly in the short term. For example, in another side project I was working on I had a countdown timer with states of the app that changed in response. To make sure the state calculations were right, it was much more efficient for me to write tests for the function than to run the countdown over and over again.

## For Visuals, Fix the Next Ugliest Thing
I make no secret of the fact that I'm terrible at visual design. That's why I always reach for a UI library like Bootstrap, a Material Design component library, or, in this case, React Native Elements. But even using an off-the-shelf component library is often not enough to make my app look good.

It would take me a large amount of energy to sit down and design out what I wanted the screens to look like. Even having one big "design" story would be too intimidating. Instead, I include visual fixes in my backlog of tasks to work on. If the next most glaring need for the app is a feature, I do that. But if, instead, the next most glaring need is a visual problem to fix, I do that instead. This way I'm able to incrementally make the app look better and better. I avoid a large effort on up-front visual design. And it probably works out better for me anyway: my ideas in advance of what would look good would probably be wrong, so I'd probably have more to fix anyway.

## Beware of Dead Ends
This was actually the second time I tried to make this app. The first time, I got what I thought was a really great idea. Instead of just a checklist of chapters to read in the Bible, the app could actually show the Bible text itself. It would be a better user experience!

There's just one problem with that: most Bible translations aren't free. So after I got the core app functionality working, I had to reach out to Bible translation publishers to see if they would make me a deal to use their translation in the app. What do you think are the chances that they responded?

You'd think that, after this dead end, I would realize I could just switch back to my earlier idea of making the app a Bible reading checklist. But that's not what came to mind. I thought the *whole project* was stalled, so I let it sit for over a year.

The moral of the story is to be very careful what scope you add to the project. Is it something you can do entirely yourself, or do you need to rely on other people or companies for it? And if your project does stall, do what I didn't do: try to rethink the premise of the app, to see if there's something you can take out to allow the project to still move forward.

## Build Knowledge and Tools Over Time
To say "I built an app in two days" isn't the whole story. Really, it took me 36 years to build an app in two days. What I mean is that I bring all of my past programming experience into a project. I was fortunate enough to get to use computers from an early age, I got a college degree in computer science, and I've worked professionally as a programmer for 14 years. That's a privilege and I don't want to take that for granted.

Specifically, I came into the project already having used Rails, Postgres, and Docker for professional projects, and having done research and tutorials for React Native for over a year. If any of those were new to me, it would have been a lot of work to learn them at the same time as doing the project. Now, sometimes doing a side project can be the best way to learn a new technology, but in that case you shouldn't expect to be able to ship it quite as quickly.

In addition to learning these technologies over time, I also built some reusable code I was able to use to make this project go faster. This includes:

* Project templates to create new projects preconfigured with options I like: a [Rails API Template][rails-api-template] and a [React Native Template][nativeup]
* A [Docker-based deployment architecture][docker-yall] for my personal server
* [`@codingitwrong/react-login`][react-login], which handles presenting a login form, storing the login state, and even making OAuth2 token requests.
* [`@reststate/mobx`][reststate-mobx], a zero-configuration data layer for accessing data in a JSON API backend

For me, setting up these reusable tools was motivating, because (for example) I didn’t want to write my data access layer over and over again for every project. I would rather spend _more_ effort writing a reusable one once, than build the same thing over and over again. I had already started `@codingitwrong/react-login` and `@reststate/mobx` before this project, but I ran across some bugs and feature needs that I built out within those libraries. But again, building these features in a shared place sets me up for more quick side projects in the future.

## Follow Your Motivation, Don’t Force It
There’s some advice out there that says to persist no matter what, and never quit. Maybe that’s the right advice for some people sometimes. But for me right now, I find that I need to follow my motivation. I tried several other side projects over the last few months, and I wasn’t able to keep up interest in any of them. If I had persisted, they might have turned out great, but it’s likely I wouldn’t have had a great time working on them. And when you’re doing projects on the side, if it’s not fun what’s the point? By following my motivation I’ve been able to get deep experience in _some_ technologies, (Rails, Ember, React Native) and broad experience in others (Vue, React, Docker, Elixir).

[bible-reading-native]: https://github.com/CodingItWrong/bible-reading-native
[docker-yall]: https://youtu.be/u3yfekH1PWo
[fullstack-episode]: http://www.fullstackradio.com/101
[json-api]: https://jsonapi.org/
[jsonapi-resources]: http://jsonapi-resources.com/
[nativeup]: https://github.com/CodingItWrong/nativeup
[rails]: https://rubyonrails.org/
[rails-api-screencasts]: https://thefrontier.bignerdranch.com/skill-packs/easy-backend-api-with-rails
[rails-api-template]: https://github.com/CodingItWrong/rails-api-template
[react-login]: https://github.com/CodingItWrong/react-login
[react-native]: https://facebook.github.io/react-native/
[reststate-mobx]: https://mobx.reststate.org/
[rn-elements]: https://react-native-training.github.io/react-native-elements/
