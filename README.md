# Demo of issue with conditional field suggestions

This requires a specific scenario to trigger, hence the repo.

All of this can be reviewed by cloning and logging in to the CP. 

This relates to [PR 9592](https://github.com/statamic/cms/pull/9592) in the Statamic 4.x core.

## Fieldset setup

There is a `page_builder` fieldset which has a Replicator that stores the different blocks.

There is a `linked_fieldset` which includes a single text field.

One set of the Page Builder replicator has some buttons - we'll use this later.

The other set links to the `linked_fieldset` twice - once with a prefix, once without.

## Issue

When you have a replicator, and a linked fieldset which uses a prefix, the "Add Condition" 
button fails to work. Look at your JS console for the error.

If you remove a prefix from a linked fieldset, it works as expected.

The root of the issue is in Statamic's core:
`resources/js/components/blueprints/SuggestsConditionalFields.js`

The `getFieldsFromImportedFieldset` is concatenating the prefix to the start of the field's *object*
configuration meaning it converts it to a string, like:
`prefix_[Object object]`.

## Solution

A PR has provided a solution to the issue - and while it resolves the issue I'm still not sure if it
is actually correct.

In the use case of a page builder (a Replicator fieldtype), it is suggesting conditional fields
from *other* sets within the replicator - which isn't correct. It should only suggest fields
from within the current set itself.

## Expected

When adding a field to the "Conditional Set" set within the Page Builder, I would expect to 
only have the "buttons" shown as a suggested field.

The fix in the PR still suggests any field in the current replicator setup, but does correctly prefix
the field handles - which is good at least.

### Add a Condition to a Field in the "Conditional Set"

1. Go to Fieldsets
2. Edit Page Builder
3. Click the Page Builder Replicator
4. In the "Conditional Set", click "Create New Field"
5. Select "Text"
6. Go to the "Conditions" tab
7. Select "Show when"
8. Review the dropdown options

You will see:
1. buttons (correct)
2. text_field (correct handle, but incorrectly shown - it is in a different set)
3. prefix_text_field (correct handle, but incorrectly shown - it is a different set)

So while the linked fields are now prefixed correctly (`text_field` and `prefix_text_field`), they
should not be shown in this instance because they belong to a different Set within the Replicator.

## Use case

In our page builder we have a "column" fieldset which has some complex configuration around
how a "column" appears - content type, colours, etc.

This means that we have in our "page builder" fieldset, a set that imports "column" twice with
two prefixes - "left_" and "right_".

Let's say our column has a "content" field - this means we can access "left_content" and "right_content"
easily.

A workaround would be to re-build the "column" every time we use it - but we use it quite a bit, and
the ability to globally configure what a "column" does is really useful: replicating the setup would
just be a workaround, not a solution.

Basically it means in our Page Builder, we cannot use conditional fields via the CP: any changes
need to be done via file changes directly. Given the complexity of some Page Builder blocks, this 
is becoming an issue.
