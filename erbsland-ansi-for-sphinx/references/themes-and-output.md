# Themes And Output

`erbsland-sphinx-ansi` has a useful default path and a more manual custom-theme path.

## Default Theme

If you do not set `:theme:`, the directive uses the default prefix `erbsland-ansi`.

That default path is the simplest one:

- enable `"erbsland.sphinx.ansi"` in `conf.py`
- use `.. erbsland-ansi::`
- keep the default theme classes
- let the extension auto-add `erbsland-ansi/ansi.css`

This is the recommended default unless the docs site needs custom colors.

## Custom Theme

If you set:

```rst
.. erbsland-ansi::
    :escape-char: ␛
    :theme: my-theme

    ␛[32mhello␛[0m
```

the generated classes become `my-theme-block`, `my-theme-green`, `my-theme-bold`, and so on.

Important consequence:

- The extension still auto-loads only its own default CSS file.
- Your custom theme CSS must be provided by the Sphinx project.

Typical Sphinx setup:

```python
html_static_path = ["_static"]
html_css_files = ["custom-theme.css"]
```

Then place your stylesheet under `_static/custom-theme.css`.

## HTML And Non-HTML Builders

Builder behavior differs intentionally:

- HTML builders keep styling and emit themed inline spans inside a literal block.
- Non-HTML builders strip ANSI sequences and keep plain text only.

That makes the directive safe for mixed-output documentation, but it also means the text should still make sense
without color.

## What Styling The Directive Understands

The directive parses ANSI SGR color/style sequences and maps them to CSS classes.

Examples of class shapes:

- `erbsland-ansi-red`
- `erbsland-ansi-background-blue`
- `erbsland-ansi-bold`
- `erbsland-ansi-underline`
- `erbsland-ansi-bright-green`

The extension is not a full terminal emulator. Non-SGR CSI sequences are stripped for non-HTML output and do not create
styling in HTML output. If a snippet depends on cursor movement or screen updates, preprocess it with
`$erbsland-ansi-convert` first.

## Practical Advice

- Default theme first, custom theme second.
- Keep the ANSI snippet semantically understandable even after styles are removed.
- When using a custom theme, inspect both foreground and background colors for contrast in the docs theme.
- If backgrounds are important to the example, verify the block container and inline background classes work together.
