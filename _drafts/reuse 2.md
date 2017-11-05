
* MMM (1975) talks about the idea of code reuse as a major productivity boost
* SBPP (1996) said it failed
* OT (2004) continued to argue for it
* OSS achieved it
* Rails took it further. Why?
* OO concerns with the approach, especially AM
* Hearing about FP opened me up to tradeoffs of approach
* Conventional reuse good

Part 2
Rails gives you great reuse
But I was concerned it wasn't good OO
First realization: FP showed me there are different approaches.
Second realization: Rails' conventions help easy integration. (I don't know for sure that they're required; maybe Phoenix can disprove it.)
Conclusion: it's not inconsistent for me to use OOD in my core domain code, and use Rails for my infrastructure code.

***

“The best way to attack the essence of building software is not to build it at all. Package software is only one of the ways of doing this. Program reuse is another. Indeed, the promise of *easy reuse of classes, with easy customization via inheritance*, is one of the strongest attractions of object-oriented techniques.” - MMM

“Interest in software reuse reveals a recognition that software engineering is just as repetitious as other engineering disciplines. If leveraging the commonality in software development is the problem, though, *large scale code reuse has not proven to be the answer*.” - SBPP


# On domain-specific reuse

* OT suggested reuse in terms of problem space: “Reuse efforts (as opposed to composability efforts, which have been almost nonexistent) have been characterized by a focus on the solution space—computer code, algorithms, and data structures. True composability will require an understanding of the problem space—of the natural world as advocated by object thinking”
* Rails is much higher level than that, but still solution space
* Reuse not in terms of domain but in terms of technical implementation (sometimes called solution space in OT). Because the parts of the domain relevant for a given business differ (DDD)

“Effective domain modelers are knowledge crunchers. They take a torrent of information and probe for the relevant trickle. They try one organizing idea after another, searching for the simple view that makes sense of the mass. Many models are tried and rejected or transformed. Success comes in an emerging set of abstract concepts that makes sense of all the detail. This distillation is a rigorous expression of the particular knowledge that has been found most relevant”

“For similar reasons, it is unlikely that the CORE DOMAIN can be purchased. Efforts have been made to build industry-specific model frameworks, conspicuous examples being the semiconductor industry consortium SEMATECH’s CIM framework for semiconductor manufacturing automation, and IBM’s “San Francisco” frameworks for a wide range of businesses. Although this is a very enticing idea, so far the results have not been compelling, except perhaps as PUBLISHED LANGUAGES facilitating data interchange”

“The greatest value of custom software comes from the total control of the CORE DOMAIN. A well-designed framework may be able to provide high-level abstractions that you can specialize for your use. It may save you from developing the more generic parts and leave you free to concentrate on the CORE”

OT described domain-specific reuse



# Rails factors

Vs. iOS (credit Robert)

* Open ecosystem
* Package manager
* Limited domain

Vs. JS (credit Tom Dale)

* Server-side so large code so high cohesion

Vs. Java, many PHP

* Conventions. "Not a snowflake"

Vs. PHP

* Monkey patching

Laravel and Phoenix should be limited by less strong conventions
Ember feels like more lock-in

Make simple things easier
I want reusable solution domain solutions
I want to focus on the problem domain
I want to focus on problems for which the solution is well-defined

Rails Examples

How does being able to assume you have AR help?

Devise

- Generate a model and a migration and a controller with views
- Or just add mixins to an existing model
- Just customize the views, or customize the controllers, or have your own controllers call the right things
- Test helpers

Pundit

- Inferring name of policy from the class passed to it, name of method from the controller action it's called in
- Infers user retrieved by current_user, as in Devise
- Infers the way scopes work from AR

Draper
- Monkey patches Active Record to have a decorate method, that instantiates a decorator inferred by class name