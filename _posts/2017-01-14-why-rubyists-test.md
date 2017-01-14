---
title: Why Rubyists Test
---

Ruby is known as a language where tests are expected. Why is that the case? This is a question I decided to ask old-time Rubyists and influential Rubyists, including DHH, Dave Astels, and Avdi Grimm. Here are the answers I got:

## Influence of Agilistas

In a personal email, Dave Astels wrote: "Ruby started getting popular outside of Japan as XP/Agile/TDD was gaining awareness...Many in the Agile community started playing around with Ruby early on and so tools were built and courses were taught." Astels' RSpec, in particular, was an early BDD tool and the one that gained the most traction. Dave Thomas and Andy Hunt were early influencers as well.

## Rails' Built-In Test Support

Astels recollects that when DHH was working on Rails, he had recently learned about TDD from Astels at a workshop. Undoubtedly as an influence from this, Rails had testing well-integrated from the start.

Not only this, but when you used the Rails command line tool to auto-generate a model or a controller, the corresponding test was generated as well. In other frameworks the choice was whether to create tests or not, but in Rails the choice was between *deleting, ignoring, or using* the tests generated for you. This changed developers' perception of testing from the exception to the default.

## _Agile Web Development with Rails_

This book Dave Thomas and DHH was the first book on Rails, and it included a chapter on testing. Although testing was separated out for the purposes of instruction, the chapter stated that ideally the tests would be written as you go. After introducing testing, it also introduced Test-Driven Development. The effect was that developers getting excited about Rails came to consider testing part of the normal Rails development process. The Rails 5 edition of the book is available [here](https://pragprog.com/book/rails5/agile-web-development-with-rails-5).

## A Personal Plug

I'd also like to throw in a plug for Michael Hartl's [_Rails Tutorial_](https://www.railstutorial.org/book). Amazon is suggesting it was first published in 2010, so that's a bit later than the above. But the amazing thing about the _Rails Tutorial_ is that testing is intermixed all throughout the book: when you learn how to do something, you learn how to test it at the same time. It's extremely hard to write this way, but Hartl pulls it off. This is the book that first gave me a vision that testing is something I could actually do. So for me, at least, it was influential!