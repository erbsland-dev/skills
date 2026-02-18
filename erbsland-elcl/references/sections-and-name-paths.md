# Sections and Name Paths

Use this file when creating/restructuring section hierarchies.

## Section Forms

- Regular section: `[name.path]`
- Section list entry: `*[name.path]` (optional trailing `*` is allowed)
- Visual separators with hyphens are allowed: `---[ ... ]---`, `---*[ ... ]*---`

## Absolute and Relative Paths

- Absolute section starts with a name: `[main.server]`
- Relative section starts with `.`: `[.connection]`
- Relative sections resolve from the last absolute section.
- First section in a document must be absolute.
- After `@include`, there is no last absolute section context until a new absolute section appears.

## Path Constraints

- Path elements are separated by dots; spacing around dots is allowed.
- No consecutive dots (`..`).
- No trailing dot.
- Max path length: 10 elements.

## Section-List Path Behavior

- If a name path continues through a section list without explicit index syntax in the document,
  the continuation applies to the last list element currently present.
- This is how nested list structures are built incrementally.

## Text Names in Sections

- Text names are allowed in section paths (feature `text-names`).
- Section lists must not use text names.

