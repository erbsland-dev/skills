# Cursor Output and Styles

Use this reference when writing reusable terminal-style output, sharing helpers between `Terminal` and `CursorBuffer`,
working with `CharAttributes` and `CharStyle`, or building retained scrollback and history buffers.

## Default Pattern

In `1.7+`, both `Terminal` and `CursorBuffer` implement `CursorWriter`.

Use that as the default abstraction when:

- the same helper should write to a live terminal and an in-memory buffer
- you want terminal-style cursor movement without tying helpers to `Terminal`
- paragraph rendering should work the same for immediate output and retained history

Pattern:

```cpp
auto writeStatus(CursorWriter &writer, std::string_view label, std::string_view value) -> void {
    auto emphasis = CharAttributes{};
    emphasis.setBold(true);

    writer.printLine(
        CharStyle{Color{fg::BrightWhite, bg::Inherited}, emphasis},
        label,
        CharAttributes::reset(),
        fg::BrightBlack,
        ": ",
        fg::BrightGreen,
        value);
}
```

## Mixed Print Arguments

`print()` and `printLine()` accept a mixed stream of:

- `Color`, `Foreground`, and `Background`
- `CharAttributes`
- `CharStyle`
- `Char`
- `String`
- `StringView`
- plain text

As arguments are processed, the active style stays on the writer until it is changed again.

Guidance:

- Use `setDefaultColor()` when later output should return to the terminal default colors.
- Use `CharAttributes::reset()` only when later text must explicitly disable bold, underline, and similar flags.
- Use `Inherited` when a helper should preserve the current or underlying color component.
- Use `Default` when a fragment must reset to the terminal default instead of preserving context.

## `CharAttributes` And `CharStyle`

Use `CharAttributes` when you need explicit ANSI attribute state such as bold, italic, underline, reverse, or
strikethrough.

Semantics that matter:

- unspecified flags inherit from the surrounding writer or base style
- `CharAttributes::reset()` means all flags are explicitly disabled
- `resolvedWith(...)` overlays explicitly specified flags onto a base state

Use `CharStyle` when color and attributes belong together as one reusable value:

```cpp
auto emphasis = CharAttributes{};
emphasis.setBold(true);

const auto heading = CharStyle{Color{fg::BrightWhite, bg::Blue}, emphasis};
const auto muted = heading.withOverlay(CharStyle{Color{fg::BrightBlack, bg::Inherited}});
```

## `CursorBuffer` For Retained Output

Use `CursorBuffer` when output should behave like streamed terminal text but remain available for:

- scrollback panes
- log viewers
- REPL history
- buffered diagnostics panels
- document buffers that later need viewport rendering

Typical constructor shape:

```cpp
auto history = CursorBuffer{
    Size{120, 10},
    CursorBuffer::OverflowMode::ExpandThenShift,
    Size{120, 500},
    Char{" ", Color{fg::Default, bg::Black}}};
```

Overflow guidance:

- `Shift` for fixed-height consoles that should scroll upward
- `Wrap` for ring-buffer style behavior
- `ExpandThenShift` for growing history with an upper bound
- `ExpandThenWrap` for growing history that later becomes cyclic

## Paragraphs And Width-Aware Text

`CursorWriter::printParagraph()` is the shared paragraph path for `Terminal` and `CursorBuffer`.

Use it when wrapped text should behave the same in a real terminal and inside retained history.

Helpful support types:

- `ParagraphOptions` for wrapping, spacing, markers, and fallback behavior
- `ParagraphIndents` when indentation settings should be shared or reused
- `FastCharSet` when separator or grouping character sets are performance-sensitive and reused often
- `StringView` and `IndexRange` for cheap read-only slices of shared terminal strings

Background rule:

- paragraph indentation and trailing fill become real cells
- if `ParagraphBackgroundMode` does not replace those cells, they use the writer's current background
- set the writer color first when the paragraph must blend into a panel or log background

## Viewports

To display only part of a retained buffer, render it through `BufferView` or `BufferConstRefView` instead of copying
rows around:

```cpp
const auto top = std::max(0, history.size().height() - 20);
auto view = BufferConstRefView{history, Rectangle{0, top, 120, 20}};
terminal.updateScreen(view);
```

When explicit cursor positioning matters, use `moveCursor(...)` with `MoveMode::Absolute` or `MoveMode::Relative`.
Remember that `CursorBuffer` follows VT100-style wrapping, including the pending-wrap behavior at the last column.
