---
title: TDD Approaches
---

There are at least two main approaches to test-driven development: test/classical/TDD style and spec/mockist/BDD style.

The main differences between the styles are:

- Classical is conceptually focused on testing, whereas mockist is focused on [specifying the system's behavior][bdd]. The fact that these specifications can be run to test the system is secondary.
- Classical focuses on [verifying state][fowler] (i.e. return values and object properties), whereas mockist focuses on verifying behavior (i.e. method calls). As a result, classical more frequently relies on real collaborators or stubs, whereas mockist more frequently relies on mocks and spies.
- Classical can have a tendency to expose internals of objects via getters for the sake of testing, whereas in the mockist approach this isn't necessary. According to Konstantin, this was one of the main things the mockist approach was [designed to avoid][everzet].
- Classical unit tests tend to [prefer real collaborators][fowler], and so (for better or worse) they also function as mini integration tests. Mockist unit tests are more strict unit tests because they mostly use mocks.

The following concepts are associated with the mockist style, although they can be applied in either style:

- Sentences in test names. Dan North [focused on these][bdd] early in his journey toward creating BDD, but this concept likely didn't originate with him.
- The "given/when/then" terminology around tests. Dan North writes that [this originated with BDD][bdd].
- The Tell-Don't-Ask principle and the Law of Demeter. It can be applied in either approach, but it's [more emphasized][fowler] in a mockist style, probably because "train wrecks" are asking, whereas telling requires behavior verification.

[bdd]: http://dannorth.net/introducing-bdd/
[everzet]: http://www.fullstackradio.com/15
[fowler]: http://martinfowler.com/articles/mocksArentStubs.html
