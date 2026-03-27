# HTML Integration

`erbsland-ansi-convert` generates an HTML fragment, not a full document:

```html
<div class="{prefix}-block"><pre>...</pre></div>
```

Styled text is emitted as `<span>` elements whose classes change only when the active style changes. That keeps the
markup compact and makes it easy to theme in documentation systems.

## Class Naming Model

For a prefix like `docs-terminal`, expect classes such as:

- `docs-terminal-block`
- `docs-terminal-bold`
- `docs-terminal-dim`
- `docs-terminal-italic`
- `docs-terminal-underline`
- `docs-terminal-blink`
- `docs-terminal-reverse`
- `docs-terminal-hidden`
- `docs-terminal-strike`
- `docs-terminal-red`
- `docs-terminal-bright-blue`
- `docs-terminal-background-yellow`

Foreground and background colors are limited to the basic and bright ANSI color sets documented by the library.

## Recommended Embedding Pattern

1. Pick a stable `class_prefix` such as `docs-terminal` or `cli-demo`.
2. Convert the capture with that prefix:

```python
html_fragment = terminal.to_html(class_prefix="docs-terminal")
```

3. Insert the fragment into the documentation page/template.
4. Ship matching CSS with the docs theme.

## Starter CSS

This is a practical baseline for documentation sites:

```css
.docs-terminal-block {
  background: #111827;
  color: #e5e7eb;
  border-radius: 12px;
  overflow-x: auto;
  padding: 1rem 1.25rem;
}

.docs-terminal-block pre {
  margin: 0;
  font: 500 0.9rem/1.45 "SFMono-Regular", Menlo, Consolas, monospace;
  white-space: pre;
}

.docs-terminal-bold { font-weight: 700; }
.docs-terminal-dim { opacity: 0.75; }
.docs-terminal-italic { font-style: italic; }
.docs-terminal-underline { text-decoration: underline; }
.docs-terminal-blink { text-decoration: underline wavy; }
.docs-terminal-reverse { filter: invert(100%); }
.docs-terminal-hidden { visibility: hidden; }
.docs-terminal-strike { text-decoration: line-through; }

.docs-terminal-black { color: #111827; }
.docs-terminal-red { color: #ef4444; }
.docs-terminal-green { color: #22c55e; }
.docs-terminal-yellow { color: #eab308; }
.docs-terminal-blue { color: #3b82f6; }
.docs-terminal-magenta { color: #d946ef; }
.docs-terminal-cyan { color: #06b6d4; }
.docs-terminal-white { color: #e5e7eb; }

.docs-terminal-bright-black { color: #6b7280; }
.docs-terminal-bright-red { color: #f87171; }
.docs-terminal-bright-green { color: #4ade80; }
.docs-terminal-bright-yellow { color: #fde047; }
.docs-terminal-bright-blue { color: #60a5fa; }
.docs-terminal-bright-magenta { color: #e879f9; }
.docs-terminal-bright-cyan { color: #22d3ee; }
.docs-terminal-bright-white { color: #f9fafb; }

.docs-terminal-background-black { background: #111827; }
.docs-terminal-background-red { background: #7f1d1d; }
.docs-terminal-background-green { background: #14532d; }
.docs-terminal-background-yellow { background: #713f12; }
.docs-terminal-background-blue { background: #1e3a8a; }
.docs-terminal-background-magenta { background: #701a75; }
.docs-terminal-background-cyan { background: #164e63; }
.docs-terminal-background-white { background: #e5e7eb; color: #111827; }

.docs-terminal-background-bright-black { background: #4b5563; }
.docs-terminal-background-bright-red { background: #dc2626; }
.docs-terminal-background-bright-green { background: #16a34a; }
.docs-terminal-background-bright-yellow { background: #ca8a04; color: #111827; }
.docs-terminal-background-bright-blue { background: #2563eb; }
.docs-terminal-background-bright-magenta { background: #c026d3; }
.docs-terminal-background-bright-cyan { background: #0891b2; }
.docs-terminal-background-bright-white { background: #f9fafb; color: #111827; }
```

## Notes For Documentation Systems

- If the docs generator sanitizes raw HTML, prefer `text` output or use a shortcode/component that allows trusted HTML.
- Keep the fragment in a horizontally scrollable container. Terminal layouts often depend on fixed width.
- If the output should wrap instead of scroll, change `white-space: pre;` to a wrapping variant deliberately and verify
  the layout still makes sense.
- `reverse` is inherently theme-dependent. The starter CSS uses a generic inversion effect, but a docs site with a fixed
  palette may prefer an explicit foreground/background mapping instead.
- The library escapes text content for HTML output, so the remaining integration work is mostly CSS and layout.
