---
title: 'Rails APIs'
---

## Why Use Rails for JSON APIs?

[From the Rails Guides:](https://guides.rubyonrails.org/api_app.html#why-use-rails-for-json-apis-questionmark)

> The first question a lot of people have when thinking about building a JSON API using Rails is: "isn't using Rails to spit out some JSON overkill? Shouldn't I just use something like Sinatra?". [Or Express.js?]
>
> For very simple APIs, this may be true. However, even in very HTML-heavy applications, most of an application's logic lives outside of the view layer.
>
> The reason most people use Rails is that it provides a set of defaults that allows developers to get up and running quickly, without having to make a lot of trivial decisions.

## Tools

- [apiup](https://github.com/codingitwrong/apiup) - Rails project template to create an API pre-configured for JSON:API and OAuth 2 authentication

## Sample APIs

- [How to JSON:API Sandbox](https://jsonapi-sandbox.herokuapp.com/)—a JSON:API compliant web service with OAuth2 authentication. Try the app hosted or download it locally.

## Resources
* [Rails: The Easiest Way to Create a Web Service](https://codingitwrong.com/2018/07/02/rails-the-easiest-way-to-create-a-web-service.html)—two-part blog series
* [Building a JSON:API Backend with Rails and JSONAPI::Resources](https://howtojsonapi.com/rails.html)
* Books
  * [_Build APIs You Won’t Hate_](https://leanpub.com/build-apis-you-wont-hate)—guidelines on API design practices
  * _Programming Ruby_, aka "The Pickaxe Book"—a popular introduction to the Ruby language
    * [Buy ebook or physical book](https://pragprog.com/book/ruby4/programming-ruby-1-9-2-0)
    * [Read free online, older version](http://ruby-doc.com/docs/ProgrammingRuby/)
  * [_Effective Testing with RSpec 3_](https://pragprog.com/book/rspec3/effective-testing-with-rspec-3)—features and practices for testing with RSpec
  * [_Practical Object-Oriented Design_](https://www.poodr.com/) — a guide to writing changeable and maintainable code, written in Ruby
* Blog posts
  * [Authorizing JSON:API Resources, part 1: Visibility](https://www.bignerdranch.com/blog/authorizing-jsonapi-resources-part-1-visibility/)
  * [Authorizing JSON:API Resources, part 2: Policies](https://www.bignerdranch.com/blog/authorizing-jsonapi-resources-part-2-policies/)
  * [Cookie-Based Token Storage with Doorkeeper](https://codingitwrong.com/2018/11/02/cookie-based-token-storage-with-doorkeeper.html)
* Talks
  * [In Relentless Pursuit of REST](https://www.youtube.com/watch?v=HctYHe-YjnE) - RailsConf 2017 talk by Derek Prior showing how the constaints of REST help lead to simpler code and a better domain model

## Rails Guides

The following pages in the Guides are relevant to API development:

* Active Record—all the sections are useful; the first is [Active Record Basics](https://guides.rubyonrails.org/active_record_basics.html)
* [Action Controller](https://guides.rubyonrails.org/action_controller_overview.html) —especially check out the "Rendering XML and JSON Data" section
* [Routing](https://guides.rubyonrails.org/routing.html)—especially check out the "Resource Routing" section
* [Securing Rails Applications](https://guides.rubyonrails.org/security.html)
* [Command Line](https://guides.rubyonrails.org/command_line.html)
* [API-Only Applications](https://guides.rubyonrails.org/api_app.html)—how to disable Rails server-rendered mode to simplify your API app
* Supplemental APIs
  * [Action Mailer](https://guides.rubyonrails.org/action_mailer_basics.html)—sending email
  * [Active Job](https://guides.rubyonrails.org/active_job_basics.html)—background workers
  * [Active Storage](https://guides.rubyonrails.org/active_storage_overview.html)—file uploads

## Gems
* [rspec-rails](https://github.com/rspec/rspec-rails/blob/master/README.md)—a popular Rails testing library. For APIs, especially check out request specs
* [Active Model Serializers](https://github.com/rails-api/active_model_serializers)—centralizes your logic to serialize models to JSON
* [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper)—authenticate users with the OAuth2 standard
* [Pundit](https://github.com/varvet/pundit)—authorize users' access to read and write records
* [JSONAPI::Resources](http://jsonapi-resources.com/)—helps creating an API following the JSON:API spec. Maintained by the creators of JSON:API.
* [jsonapi-rails](http://jsonapi-rb.org/)—alternate JSON:API gem
* [GraphQL](https://graphql-ruby.org/)—gem for creating GraphQL APIs

## Deployment
* [Heroku](https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction)—"platform as a service" with great Rails support
* [Docker](https://www.docker.com/)—deploy apps in containers to isolate them
  * [Quickstart: Docker Compose and Rails](https://docs.docker.com/compose/rails/)
  * [_Docker for Rails Developers_](https://pragprog.com/book/ridocker/docker-for-rails-developers) book
* [Capistrano](https://capistranorb.com/)—automate deploying Rails to servers
* [Digital Ocean](https://www.digitalocean.com/)—easy cloud server platform

## API Documentation
* [Apiary](https://apiary.io/)
* [Swagger](https://swagger.io/)
* [Yard](https://yardoc.org/)
