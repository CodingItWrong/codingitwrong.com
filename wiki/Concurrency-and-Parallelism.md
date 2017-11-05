---
title: Concurrency and Parallelism
---

First, although people insist on certain meanings for the terms concurrency and parallelism, there is [not a universally agreed-upon difference](https://www.quora.com/What-is-the-difference-between-concurrency-and-parallelism).

Approaches to concurrency:

* JavaScript code [runs on a single thread in an event loop](https://www.bignerdranch.com/blog/cross-stitching-elegant-concurrency-patterns-for-javascript/). Asynchronous operations such as setTimeout, network requests, and disk I/O in Node use callbacks to allow the current block of code to run to completion, and the results of an asynchronous operation will be picked up and start a new block execution. Run-to-completion means that global state will not change within the execution of a single block of code, avoiding many multithreading errors. But JavaScript code doesn’t take advantage of multiple processors out of the box.
* Ruby code (other than JRuby) has a Global Interpreter Lock such that only one thread can execute Ruby code at a time. [I/O operations such as network requests and database access do yield to other threads](http://yehudakatz.com/2010/08/14/threads-in-ruby-enough-already/), so these requests can be made concurrently. But Ruby code can only take advantage of multiple cores if you run multiple processes. And this doesn’t even protect you from threading errors.
* Java code has the classic multithreading model: Java code can be executed on multiple threads and processors, and it shares global state, along with all the concomitant risks.
* Elixir uses the actor model—I need to reread about how this works to add more detail. It’s more OO than OO, apparently!
* Clojure’s functional nature has built-in parallelization constructs to allow work to safely be done on multiple threads, because they do not access any shared state. But this requires using Clojure’s unfamiliar Lispy syntax.

In short:

* Java is unsafe for nontrivial threading.
* Ruby’s GIL allows concurrent I/O, and it prevents global state from changing while you access it, but still allows global state to change while switched to another thread.
* JavaScript handles async I/O, and its run-to-completion prevents threading issues, but you can’t run your code on multiple cores.
