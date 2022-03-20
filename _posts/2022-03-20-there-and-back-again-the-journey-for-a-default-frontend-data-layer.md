---
title: "There and Back Again: The Journey for a Default Frontend Data Layer"
---

I got into software development of the early days of the web, when server-rendered web pages were the only web pages. Every time you made a request to the server for a URL, the server queried the database, and you got the latest data for that page. You wouldn't get any updates while you were on the page, but if you refreshed or navigated away and back, you'd get the latest. It was good enough for a lot of scenarios.

With the transition to rich client applications, though, the potential for much richer data experiences came about. You could store data locally so that it could be viewed while offline, or even edited while offline. Those changes could be synced back up once you came back online, even if another client or user had made other changes in the meantime. Changes from one client could be automatically pushed to other clients looking at the same data.

Seeing these possibilities of rich client applications, I started asking, what's the right default way to approach data layers? By "default" I mean a way that works for simple use cases with little to no downside. What approach for a data layer should I reach for unless I have a reason to do otherwise?

You can see posts on this blog across 2018 to 2020 as I explored answers to this question. Here's a summary of what I explored and where I've landed for now.

## Firebase

For a lot of folks, [Firebase](https://firebase.google.com) is their default data layer for client applications. Firebase is probably the most publicly-visible exemplar of offline, sync, and push functionality. And at least to me (someone who has only used Firebase at a basic level) it seems like you get all this functionality for no effort, out of the box.

There are a few downsides to Firebase, though. First, all your data lives in a proprietary system--Google's system. Second, the data isn't stored relationally, so if your data is relational in nature (and Sarah Mei argues that [a lot more data is relational in nature than you might think](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/)), then you have to implement relational-like functionality by hand.

Because of these downsides, I didn't want to choose Firebase as my default data layer, but it set a high bar that I tried to look for elsewhere.

## Orbit

As I looked for a richer way to handle relational data, [Orbit.js](https://orbitjs.com) came on my radar screen. Orbit is a JavaScript framework for orchestrating changes to multiple data sources in a declarative way. For example, you can have a web service, local in-memory data, and local persisted data, and when properly configured Orbit will keep them in sync.

Orbit works with JSON:API REST web services by default, and JSON:API is a relationally-modeled format, so we have a natural way to work with relational databases. Orbit and JSON:API server-side libraries are open source and work with a wide variety of open-source server-side frameworks, so we don't have any lock-in.

The first limitation is that Orbit doesn't currently support pushes of data out of the box. But that's okay--maybe that's the price we have to pay for everything else working well.

The next issue that came up is offline writes, which Orbit does attempt to handle. If you have a local and remote data store configured, when you perform a write operation, the local data is updated, then the update is attempted to be sent to the server. If it fails due to being offline, the update operation can be saved and retried later, and in the meantime you can continue to work with the data locally.

So far, so good. The problem is that these saved update operations can cause problems once you do come back online. They can overwrite changes made on the server in a way users don't intend. Alternatively, they might conflict with those changes and error out, in which case there's no built-in facility to handle those errors, so you have to code it yourself. This can be fine for some systems, but not for the kind of default data layer I'm envisioning. I found that for the default case it was simpler to avoid the possibility of these issues by not allowing offline updates, only offline reads.

But once you're only doing offline reads, it's fairly easy in many data layers to persist the data to local storage and restore it when you restore the app. Orbit isn't adding much value in that case, and you're taking on the cost of going through its layers of abstraction and coupling yourself to an open-source library without a lot of adoption. This tradeoff is likely not worth it.

The root reason that Orbit can't handle offline writes easily has nothing to do with Orbit itself; the root reason is that JSON:API isn't a protocol designed to handle conflicting changes. To have those handled easily, you need a protocol designed to do it from the start

## CouchDB and PouchDB

[CouchDB](https://couchdb.apache.org) is a database based on a protocol designed for handling conflicting changes. I think of it a lot like an open-source Firebase (it's possible that's not fully accurate, but it's close enough for my default data layer use case). It handles offline, sync, and push. There are implementations of CouchDB that run in various rich clients, including Android, iOS. The browser implentation of CouchDB Is called [PouchDB](https://pouchdb.com). Because you have a full database running in your client, you access that database directly, and the database handles syncing with remote and other clients.

The first downside of CouchDB is that it's not relational--it's a document database. But, again, maybe that's the price we have to pay for everything else working well.

To work with CouchDB, I had to get it installed on a server and learn enough basic CouchDB administration to set up access and back up the data. As someone with a lot of relational database experience, it wasn't that motivating to have to learn this--do I really want to be investing this effort in learning a new ecosystem when I would prefer to be using a relational database anyway? But I gave it a shot.

The way CouchDB pushes changed data to clients works quite well. Making a change in one client and seeing it show up in another online client right away is pretty exciting.

But one day, the excitement gave me a sinking feeling: some changes I made in one client didn't show up in another one. Why not? I was never able to figure it out. Doing so would have required digging deeper into CouchDB and the way it tracks and transfers changes. This was a possible source of bugs and customer support requests that I didn't want to introduce—not when offline support was just a nice-to-have.

If it's possible to have data fall out of sync, then you need some option to blow away the local database and pull down an exact copy of remote. But if you have changes locally that haven't been synced up, that results in lost data. And if I have to keep track of local changes, then I'm doing exactly what I want my data layer to be doing for me. For my default data layer I don't want the possibility of lost data. So I found that offline access was not as high a priority for me as data integrity for default data layers.

## Ember Data and the Identity Map

I started my frontend journey on the [Ember.js](https://emberjs.com) framework. Ember provides a pretty full-featured data library, [Ember Data](https://guides.emberjs.com/release/models/), that could serve as a paradigm for a default data layer. It's based on the concept of an identity map, which is a data store in the frontend where there is one authoritative copy of each record shared across the application. When you query the server, the data is stored in the identity map, and then individual routes and components use the data they need from the identity map. Ember Data has a straightforward API where the same method call performs two functions: first, it queries the server for data, and second, it indicates to the frontend that those are the records from the identity map that should be used.

Ember Data works with JSON:API out of the box, so we can interface with relational data easily. But it doesn't support push, sync, or offline out of the box. But by this point in my journey, I'd seen that these features come with a lot of possibility for errors, so I was willing to give them up for a default data layer. Maybe the focus on richly representing relational data was what was best for a default data layer.

After building out a substantial app in Ember and experiencing some challenges with Ember Data, I wrote [a blog post about how I addressed those challenges](https://codingitwrong.com/2020/06/23/ember-list.html). Some of the challenges related to implementation details of Ember Data that could hypothetically be changed, but here are the challenges that seem fundamental to the identity map paradigm:

- When you are displaying all records of a given type from the server, new records you create will show up in that list automatically, because the client knows how to get "all records of this type" from the identity map. But if you're retrieving a filtered list of records from the server, new records don't show up in that list automatically, because the logic to do the filtering lives on the server. You either need to duplicate the filtering logic on the client, or else re-request the data from the server. If you do that, you're losing a lot of the benefit of an identity map. And Ember Data actually makes it harder to do this re-requesting than it otherwise would be, because this isn't the default way to use Ember Data.
- Reloading doesn't work perfectly, because it doesn't handle deleted records. Ember Data takes records returned by queries and inserts or updates them in the identity map. But if a record has been deleted, nothing in the returned data indicates that record needs to be removed, so it remains in the identity map and will be shown in the UI. The "soft delete" pattern of keeping deleted records can be some help, but it doesn't help with the following issue:
- You need to handle records "removed" from a query, as well. Consider the case where a record was returned by a certain query, but then its data changes and it is no longer returned by that query. You rerun the query. Because the record is not returned, it still has its old values in the identity map, so the local filtering still returns that record, incorrectly. To get the identity map to remove it, you would need a way to for a query to indicate that a record previously matched this query, but now no longer does. This would require designing a sophisticated state-change tracking system, and we are now far beyond the realm of a basic data layer. Instead, the alternative is to clear out the *entire* identity map locally before running the query. And at that point, why even have an identity map?

It's important to note that I didn't run across these issues while working on a large, complex corporate application. I ran across them while working on--and I'm not making this up--literally a todo list, the canonical trivial example. If even a simple todo list exposes these issues, then it seems that when you're building apps the way I want to build them, an identity map doesn't add a lot of value.

Also, I'm a big fan of JSON:API, but it doesn't have the widest adoption, and so many of the server libraries implementing it are not very well maintained. Even the Rails-based library by the authors of the spec can have some delays before issues with the latest release of Rails are fixed. JSON:API isn't the easiest to implement yourself. It can still be worth it to *choose* to use JSON:API if it helps your scenario, but defaulting to a data layer that *requires* using JSON:API seems risky.

## Dumb Global Store

Since maintaining an identity map has downsides for the default data layer case, the next option we could try is a "dumb" global store. We don't try to implement any identity map logic; there is just a global store of the results of a given query. When you rerun the query, you discard the local results and replace them with the new results. Multiple screens that use that query can share the data, and if you navigate away from a screen and back to it the data is already available.

Because this solution is so simple, it's easy to implement in most frontend JS frameworks. In React, you would probably implement it with a Context.

In this approach, a good option is to take the "Cache Then Network" design as described in [Jake Archibald's "Offline Cookbook"](https://web.dev/offline-cookbook/):

1. Display the cached data right away, if any.
2. Make a request to the server to retrieve updated data.
3. If and when the network request returns successfully, cache that data, and update what is displayed.

The advantage of this approach is that you get to see *some* data immediately, and after a short delay you get to see the latest data barring any errors. The downside is that there can be a jump in the data, which can be visually jarring, or even result in misclicks if records are added or removed.

I thought a dumb global store was the ideal default way to approach data storage, until an insight from a coworker.

## Local State

I was working on a React client project on a team with my coworker Taylor Martin. We had some data we needed to query from the server. We knew we needed it on multiple screens, so I suggested we store it in a React Context.

Taylor pushed back. "I think we should store it in local state."

I said "but we know we need it on multiple screens. There will be a delay when we re-query it."

Taylor said "yes, but think about it like server-rendered web pages. Every time you go to a web page, you get the latest data, right?"

"Right," I agreed.

"So it's reasonable for users to expect that every time they go to a screen they'll get the latest data."

"That makes sense."

"So we need to re-query the data each time the screen loads anyway. Now, this is a very small amount of data, right?"

"Right."

"And because of the nature of this specific app, we've already decided it won't work offline anyway, right?"

"Right."

"So, let's just load the data each time we go to the screen and see how it works. If it's too slow, we can add a Context at that time. Besides, this will be implemented behind a React custom hook anyway, so the callsite won't need to change. Whether or not we use a Context behind the custom hook is just an implementation detail."

This conversation has been simplified to make it easier to follow. In reality I was more argumentative, because I thought "always have a global Context for domain data" was a better default. But Taylor convinced me. He was speaking out of the principle of incremental design: build the simplest thing that can possibly work, and only add complexity when it's warranted by requirements.

Sure enough, storing data in local state within the screen we were on was plenty performant enough, and we had no need to make it more complex.

## Takeaways

So my journey for a default data layer ends where it began: loading data for a screen or page each time you access it, storing that data locally, and discarding and reloading it when needed. A lot like server-rendered web pages. There were some things that the architecture of server-rendered pages gave us quite nicely. Rich clients still add a lot of value, and building more sophisticated data interactions can make sense in some cases. But there is almost always a tradeoff for that complexity, so they aren't a good fit for a default data layer.

Specifically, here's what I would recommend:

- If you're building an app just to experiment or for personal use, especially if you don't have server-side development experience, just go ahead and use Firebase. You probably won't run into any downsides with it.
- If you have a custom backend, prefer local state as the default approach. It's good enough for many cases.
- If you need to share data across the application, or if you need to cache data locally for reading, that is fairly easy to add.
- If you're considering anything more advanced--offline writes, syncs, or pushes--then think hard about the value it adds. Is it important enough that it's worth taking on significantly more complexity? If so, you'll need to weigh the different options to see which has the right tradeoffs for you.

This doesn't mean this will be my answer forever. New paradigms continue to be explored. The streaming HTML approach, pioneered by Phoenix LiveView and adopted by Laravel Livewire and Rails Hotwire, has interesting possibilities. React Server Components do as well. Both of them are different kinds of moves away from having a data transfer layer and client storage, preferring instead to transfer HTML in the former case and React components in the latter. It remains to be seen how these paradigms will shift what default data layer approach is optimal for the default case.
