# Names and Name Conflicts

Use this file for naming rules and conflict analysis while editing configs.

## Regular Names

- Allowed chars: letters, digits, spaces, underscore.
- Must start with a letter.
- No consecutive separators (`__`, double spaces, mixed runs like ` _`).
- No trailing underscore.
- Max length: 100 characters.
- Names are case-insensitive.

Normalization for comparison:

1. Replace spaces with `_`.
2. Lowercase ASCII letters.

So `This Name`, `this_name`, and `THIS_NAME` are equal.

## Name Uniqueness

- A section name path and value name path can be used only once.
- Section names and value names share one namespace, so they can conflict with each other.
- Text names and regular names are always distinct types; they never compare equal.

## Intermediate Sections

- Missing parent sections are created implicitly when defining deep paths.
- Those implicit section names can later be explicitly defined as regular sections.
- But an existing implicit section cannot later be redefined as:
  - a section list
  - a scalar/value assignment

## Section List Exception

- Repeating a section path is allowed only with section-list syntax (`*[...]`).
- Each `*[...]` creates a new list entry, not a duplicate section definition.

