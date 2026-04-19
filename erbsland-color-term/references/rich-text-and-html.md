# Rich Text and HTML

Use this reference when the task is document-like rather than cell-by-cell: rendered help pages, terminal articles,
formatted release notes, or HTML fragments that should become styled terminal output.

## Start With `text::HtmlRenderer`

Use `text::HtmlRenderer` when the input is already structured HTML or when generating document-like terminal output is
easier than hand-building `printParagraph()` calls.

The renderer can:

- parse a focused HTML subset
- render to a `String` with `renderString()`
- render directly to any `CursorWriter` with `renderTo(...)`

Typical subset coverage:

- headings and paragraphs
- ordered and unordered lists
- definition lists
- links
- blockquotes
- code blocks
- common inline formatting such as emphasis and strong text

## Pick The Output Target Deliberately

- `renderString()`:
  minimal styling only, use when you want a self-contained result to store, combine, or print later
- `renderTo(writer)`:
  use when block spacing, margins, and cursor positioning should flow directly into a `CursorWriter`

Pattern:

```cpp
namespace rich = erbsland::cterm::text;

const auto html = std::string_view{
    "<h2>Status</h2>"
    "<p>Hello <strong>world</strong>.</p>"
    "<ul><li>One</li><li>Two</li></ul>"};

const auto summary = rich::HtmlRenderer{html}.renderString();
terminal.print(summary);
```

## Styling

Use `text::Style` when the default HTML rendering is not enough.

Important pieces:

- `StyleSelector` targets semantic roles such as headings, list items, code, strong text, or links
- `StyleRule` stores the resolved formatting for one selector target
- `ParagraphIndents` controls reusable block indentation and margin behavior

Pattern:

```cpp
namespace rich = erbsland::cterm::text;

auto style = std::make_shared<rich::Style>();
style->setBaseTextStyle(CharStyle{Color{fg::BrightWhite, bg::Inherited}});
style->edit(rich::StyleSelector::strong())
    .setTextStyle(CharStyle{Color{fg::BrightYellow, bg::Inherited}});
style->edit(rich::StyleSelector::heading(1))
    .setTextStyle(CharStyle{Color{fg::BrightWhite, bg::Blue}})
    .setLineFill(U'=');
```

## `TextNode` When You Need More Control

Use `text::TextNode` when you need to:

- inspect parsed HTML
- transform content before rendering
- build structured terminal documents programmatically
- detect unsupported content explicitly

Unsupported block-level HTML is represented as `TextNode::Type::Unsupported` rather than being silently dropped.

## Good Fits

- render help documents into a `CursorBuffer`, then show them through a viewport
- convert HTML snippets to styled `String` values for terminal output
- build themed document output with custom heading, list, and code block styles

If the result needs scrolling, pair this reference with `interactive-and-advanced.md`.
