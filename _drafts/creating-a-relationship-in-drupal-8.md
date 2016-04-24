---
title: Creating a Relationship in Drupal 8
---

I'm playing around with using Drupal 8 as a simple relational data store, rather than implementing CRUD by hand in a web framework. The first challenging thing I ran into was creating one-to-many relationships between content items. It wasn't hard once I figured it out--there are just a lot of options that aren't totally intuitive. So here's how to do it.

* Log in as an administrator, then go to Manage > Structure > Content Types
* Make sure both content types you want to relate exist--create them if not.
* For the content type you want to be the "many" side of the relationship (i.e. it will have a field with _one_ of the other side), click "Manage fields"
* Click "Add field"
* From the "- Select a field type -" dropdown, choose "Reference > Content"
* For "Label", choose the name of the field.
* Click "Save and continue", then click "Save field settings"
* Under "Reference type" > "Content types", choose the content type you want to relate to in that field.
* Under "Sort by", choose the order you want to sort that field if it's displayed as a list. Your name/title field is a good choice.
* Click "Save settings"
* Click the "Manage form display" tab
* Next to the field you created, choose the widget you want to use to input it:
    - Autocomplete: type in a text field, and the names of possible content items will show up in a dropdown.
    - Autocomplete (Tags style): not sure the difference
    - Select list: a dropdown
    - Check boxes/radio buttons

Now, to query based on this relationship in a view on a block on a page:

* Go to Manage > Structure > Views
* Click "Add new view"
* Enter a name for the view
* Under "View settings", leave "Show: Content", and next to "of type:" choose the content type you want to show
* Choose to create a block
* Click "Save and edit"
* Under Advanced > Relationships, click "Add"
* Find "Content referenced from [name of field you created]" and select it
* Click "Add and configure relationships", then click "Apply"
* Under Advanced > Contextual Filters, click "Add"
* Find the relationship field you created and select it
* Click "Add and configure contextual filters"
* Under "Relationship", choose the field you created
* Under "When the filter value is not available > Default actions", choose "Provide default value", type "Content ID from URL"
* Under "When the filter value is available or a default is provided", choose "Specify validation criteria", validator "Content", content type (the content type you relate to)
* Click "Apply"
* Click "Save"

<http://www.webomelette.com/related-content-block-views-drupal-7>