---
title: How to Seed and Migrate Rails Data
tags: [rails]
---

There are a few things about setting up initial data and migrating existing data that aren't obvious in Rails. Here are some best practices I've picked up from a few different coworkers and articles.

## 1. Separate Reference and Sample Data

Rails provides a `rails db:seed` command that makes it easy to [set up initial data in a database](http://guides.rubyonrails.org/active_record_migrations.html#migrations-and-seed-data). But what kind of data is that supposed to be? There are at least two kinds of initial data:

* **Reference data:** data needed in all environments, from dev to production. This could include things like lists of countries and states. Usually hard-coded data.
* **Sample data:** data needed only for the dev environment, so that there are already fake records in every table to make the app easy to interact with. Usually generated with Factory Girl.

The seeds file is commonly used for both of these, but that makes it difficult to set up initial data in production. The Rails guides don't make it clear which of these the seeds file is for.

It can be useful to separate the two out. [Thoughtbot recommends](https://robots.thoughtbot.com/data-migrations-in-rails) using the seeds file for reference data, and setting up a separate Rake task for sample data. A plain bin script could also work in place of a Rake task.

## 2. Transform Data Outside of Migrations When Possible

When data in your database needs to be transformed, it can be common to do this in a Rails migration. After all, Rails migrations are for making changes to your database, and Rails will ensure they're run once and only once in each environment.

The problem is that migrations are meant for schema transformations, not data transformations. [Thoughtbot points out](https://robots.thoughtbot.com/data-migrations-in-rails) a few problems with using migrations for data transformations.

When possible, it's better to write a temporary Rake task or bin script to transform the data, run it when you deploy, and then remove the task/script afterward.

This isn't always possible, however. The main time it's not possible is when data needs to be transformed in order to do a schema migration. For example, if you're changing what table a column is stored in, you will want to add the new column, transfer the data over, and then drop the old column. This can't be done easily outside of schema migrations. And in this case, the data transformation is closely tied to the schema change, so keeping it in a migration makes sense.

## 3. Data Transformations Should be Rerunnable

Whether you transform data in a migration or a separate script, data transformations can fail mid-migration for various reasons. Because of this, it's important to make them rerunnable. This allows you to easily start them back up in case of failure, without having to worry that data will be duplicated. Codecrate provides [an example of how to do this](http://codecrate.com/2014/09/using-rails-migrations-to-manipulate-data.html).

## 4. Use Inner Model Classes

As Rails developers we're used to doing all our database access through Active Record models. As a result, when we need to run data transformations, it's tempting to reach for our models to do so. For a temporary Rake task or bin script, this is probably fine.

However, if you need to do data transformation within a migration, using a model can cause problems. Migrations stick around forever, but models will change over time, and can even be removed. This will cause the migrations to fail if they're run in an environment in the future. Writing direct SQL queries is an alternative, but these can quickly become difficult to write and understand, and that means missing out of the benefit of writing transformations in the Active Record syntax we're used to as Rails developers.

Instead, [Makandra explains](http://blog.makandra.com/2010/03/how-to-use-models-in-your-migrations-without-killing-kittens/) how to create "mirror" Active Record models as inner classes within your migration. You only have to implement enough of the Active Record model to support the query you need to run: often just the table name and relationships are enough. These inner model classes will still be available after your core models change, and you're safe!

## That's what I've got.

Do you have other tips for seeding and migrating data in Rails? Let me know via [Twitter](https://twitter.com/CodingItWrong) or [email](me@codingitwrong.com)!
