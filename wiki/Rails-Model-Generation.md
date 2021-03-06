---
title: Rails Model Generation and Migrations
---

## Model Generation

Generates a model class and a corresponding `create_table` migration.

<http://guides.rubyonrails.org/active_record_migrations.html#model-generators>
<http://guides.rubyonrails.org/command_line.html#rails-generate>

```
rails generate model NAME [field[:type][:index] field[:type][:index]] [options]
```

[Field types](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_column):

* :primary_key - note that an `id:integer` primary key column is added by default if this isn't specified
* :string - i.e. of limited short length
* :text - i.e. of indeterminate length
* :integer
* :bigint
* :float
* :decimal
* :numeric
* :datetime
* :time
* :date
* :binary
* :boolean
* :references or :belongs_to - creates an ID column referencing another model table by name, along with a foreign key constraint

## Create Table Migrations

Whether a `create_table` migration is generated or written by hand, [column modifiers](http://guides.rubyonrails.org/active_record_migrations.html#column-modifiers) can be added to the `t.column` lines to further configure the column creation. The most common are:

* default: the default value
* index: whether to create an index for the column
* limit: max characters
* null: whether the column is nullable

##  Modifying Columns

Outside of a `create_table` migration, columns on existing tables can be added, removed, or [changed](http://guides.rubyonrails.org/active_record_migrations.html#changing-columns).

* Change type: `change_column`
* Change nullability: `change_column_null`
* Change default value: `change_column_default`
* `add_foreign_key`
* `remove_foreign_key`
* `add_index`
* `remove_index`
