---
title: Rails Application Architecture
---

The following is a list of some of the kinds of component that can be included in a three-tier architecture [Rails](Rails) application.

Rails includes more than just models, views, and controllers. The main difference here is that Rails doesn't have an application tier in between the presentation and data tiers: controllers call models directly, and business logic lives in one or the other of those. This is fine for smaller applications, but for larger ones an additional layer can be helpful. Not every application needs all of these types of components; each should be added only when a specific application's needs warrant it.

This list adds components in all three tiers following different [design patterns](Patterns). Components built-in to Rails are listed in **bold**.

# Presentation Tier

## Web Application

* **HTML View**: [ERB](http://guides.rubyonrails.org/layouts_and_rendering.html)
* **Controller**: [ActionController](http://guides.rubyonrails.org/action_controller_overview.html)
* *Presenter*: gets some display logic out of the view and the controller - PORO [(video)](http://youtu.be/bHpVdOzrvkE)
* *Decorator*: takes the place of helpers, providing display logic in an object-oriented way - [Draper](http://youtu.be/bHpVdOzrvkE)
* *Form Object*: for when a form maps to multiple models; can have form-specific validations - [ActiveModel::Validations](http://culttt.com/2015/11/04/using-form-objects-in-ruby-on-rails)

## Web API

* **Controller** - [ActionController](http://guides.rubyonrails.org/action_controller_overview.html)
* *Serializer*: gets JSON mapping out of the model - [ActiveModel::Serializers](https://github.com/rails-api/active_model_serializers)
* *JSON-API*: exposes models as a JSON-API without having to custom-code routes and controllers - [JSONAPI::Resources](http://www.cerebris.com/blog/2014/08/22/introducing-jsonapi-resources/)

## Command Line

* **Command Line Task**: [Rake](http://guides.rubyonrails.org/command_line.html#custom-rake-tasks)

# Application Tier

* **Queued Job**: [ActiveJob](http://guides.rubyonrails.org/active_job_basics.html)
* **Event**: [ActiveSupport::Notifications](http://www.rubydoc.info/gems/activesupport/4.2.4/ActiveSupport/Notifications) [(video intro)](https://youtu.be/dgUhP606F9w)
* *Service Object*: business logic accessible from multiple presentations - [PORO](http://stevelorek.com/service-objects.html); one speaker recommends [using ActiveJob](https://youtu.be/60LH3em78V8) for these
* *Domain Model*: PORO

# Data Tier

* **Model**: [ActiveRecord](http://guides.rubyonrails.org/active_record_basics.html)
* **Concern**: groups related or reusable model functionality into a separate module
* *Repository*: PORO using [ActiveRecord](http://guides.rubyonrails.org/active_record_basics.html)
