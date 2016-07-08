---
title: Rails Model Generation
---

Generates a model class and a migration to create the corresponding table.

<http://guides.rubyonrails.org/command_line.html#rails-generate>

```
rails generate model NAME [field[:type][:index] field[:type][:index]] [options]
```

[Field types](http://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_column):

* :primary_key
* :string
* :text
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
