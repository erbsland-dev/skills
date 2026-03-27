---
name: erbsland-ansi-for-sphinx
description: Use the `erbsland-sphinx-ansi` Sphinx extension and its `.. erbsland-ansi::` directive to embed ANSI-formatted terminal output in Sphinx documentation. Use when enabling `erbsland.sphinx.ansi` in `conf.py`, authoring ANSI blocks with `:escape-char:` or `:theme:`, preparing documentation-friendly ANSI snippets, or normalizing dynamic terminal captures with `erbsland-ansi-convert` before embedding them.
---

# Erbsland ANSI For Sphinx

Use this skill when Sphinx documentation should show ANSI-colored terminal output directly in the docs. The key rule is:
the directive is for already-static ANSI text, not for replaying a live terminal session. If the source contains cursor
movement, line rewrites, alternate-screen updates, or other dynamic terminal behavior, normalize it first with
`$erbsland-ansi-convert`.

## Quick Workflow

1. Enable `"erbsland.sphinx.ansi"` in `conf.py`.
2. If the source came from a real terminal capture, first flatten it with `$erbsland-ansi-convert` into static ANSI.
3. Replace the real escape character with the visible placeholder `␛`.
4. Embed the result with `.. erbsland-ansi::` and `:escape-char: ␛`.
5. Use the default theme unless the docs need a custom class prefix and matching CSS.

## Default Authoring Pattern

Prefer this form in reStructuredText:

```rst
.. erbsland-ansi::
    :escape-char: ␛

    ␛[32mSuccess:␛[0m build completed
    ␛[36mHint:␛[0m run ␛[1mpython -m sphinx -b html doc _build/html␛[0m
```

This is the recommended default because `␛` is visible, searchable, and much easier for editors and reviewers to
handle than the real ESC byte.

## Core Rules

- Prefer `:escape-char: ␛` unless there is a strong reason to keep literal ESC bytes in the source.
- Treat `.. erbsland-ansi::` as a rendering directive for static ANSI text, not as a terminal emulator.
- For captured terminal sessions, normalize first with `$erbsland-ansi-convert`, then convert ESC to `␛`, then paste
  the final block into the docs.
- Expect HTML output to keep formatting and non-HTML builders to strip ANSI and leave plain text.
- Use `:theme:` only when you are also providing matching CSS classes in the Sphinx HTML build.

## Minimal Setup

```python
extensions = [
    # ...
    "erbsland.sphinx.ansi",
]
```

The extension ships its default CSS automatically for the default theme.

## When To Use `erbsland-ansi-convert` First

Read `references/preparing-snippets.md` when the source is not already static ANSI. Use `$erbsland-ansi-convert` first
for:

- progress bars using `\r` or `ESC[2K`
- cursor movement and in-place rewrites
- full-screen or alternate-screen terminal output
- copied transcripts that need to be flattened before embedding

After normalization, replace real ESC characters with `␛` before inserting the snippet into `.rst` sources.

## The `:theme:` Option

Use `:theme: my-theme` to switch the CSS class prefix:

```rst
.. erbsland-ansi::
    :escape-char: ␛
    :theme: my-theme

    ␛[32mhello␛[0m
```

This changes classes from `erbsland-ansi-*` to `my-theme-*`. Read `references/themes-and-output.md` before using a
custom theme, because the extension only auto-includes its default stylesheet.

## Output Model

- HTML builds convert ANSI SGR sequences into themed inline spans inside a literal block.
- Non-HTML builds strip ANSI control sequences and keep only plain text.
- Non-SGR CSI sequences are not rendered as styling. If the source depends on terminal emulation instead of static ANSI
  styling, preprocess it first.

## Decision Guide

- Need the workflow for captured output, normalization, and converting ESC to `␛`:
  Read `references/preparing-snippets.md`.
- Need to understand default theme vs custom theme and CSS loading:
  Read `references/themes-and-output.md`.
