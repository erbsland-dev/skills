# Cursor Output and Styles

Use this reference when writing reusable terminal-style output, sharing helpers between `Terminal` and `CursorBuffer`,
working with `CharAttributes` and `CharStyle`, or building retained scrollback/history buffers.

## Contents

- Shared `CursorWriter` workflow
- Mixed print arguments and style state
- `CharAttributes` and `CharStyle`
- `CursorBuffer` for retained output
- Paragraphs and scrollback

## Shared `CursorWriter` Workflow

In version 1.7, `Terminal` and `CursorBuffer` both implement `CursorWriter`.

Use that as the default abstraction when:

- The same output helper should write to a live terminal and an in-memory buffer
- You want terminal-style cursor movement without tying helpers to `Terminal`
- Paragraph printing should work the same for immediate output and retained history

Pattern:

```cpp
auto writeBanner(CursorWriter &writer, std::string_view title) -> void {
    auto emphasis = CharAttributes{};
    emphasis.setBold(true);
    emphasis.setUnderline(true);

    writer.printLine(
        CharStyle{Color{fg::BrightWhite, bg::Inherited}, emphasis},
        title,
        CharAttributes::reset());
}
```

Then call it from either target:

```cpp
writeBanner(terminal, "Service Console");
writeBanner(logBuffer, "Captured Output");
```

## Mixed Print Arguments and Style State

`print()` and `printLine()` accept a mixed stream of:

- `Color`, `Foreground`, and `Background`
- `CharAttributes`
- `CharStyle`
- `Char`
- `String`
- Plain text

As arguments are processed, the active style stays on the writer until it is changed again.

Guidance:

- Use `setDefaultColor()` to reset colors to the terminal defaults.
- Use `CharAttributes::reset()` to explicitly disable all attributes.
- Use `style()` and `setStyle()` when reading or updating the combined color-plus-attributes state is clearer than
  separate calls.
- Use `Inherited` color components when a helper should preserve the active or underlying color.

## `CharAttributes` and `CharStyle`

Use `CharAttributes` when you need explicit ANSI attribute state:

- `Bold`
- `Dim`
- `Italic`
- `Underline`
- `Blink`
- `Reverse`
- `Hidden`
- `Strikethrough`

Important semantics:

- Unspecified flags inherit from the surrounding writer or base style.
- `CharAttributes::reset()` means all flags are explicitly disabled.
- `resolvedWith(...)` overlays explicitly specified flags onto a base state.

Use `CharStyle` when color and attributes belong together as one reusable value:

```cpp
auto emphasis = CharAttributes{};
emphasis.setBold(true);

const auto headingStyle = CharStyle{Color{fg::BrightWhite, bg::Blue}, emphasis};
const auto mutedOverlay = CharStyle{Color{fg::BrightBlack, bg::Inherited}};
const auto mutedHeading = headingStyle.withOverlay(mutedOverlay);
```

Prefer `CharStyle` for theme fragments, headings, reusable labels, and helper APIs that would otherwise pass color and
attributes separately.

## `CursorBuffer` for Retained Output

Use `CursorBuffer` when output should behave like terminal text, but remain available for:

- Scrollback panes
- Log viewers
- REPL history
- Text consoles
- Buffered help or diagnostics panels

Recommended constructor shape for log-style history:

```cpp
auto history = CursorBuffer{
    Size{120, 10},
    CursorBuffer::OverflowMode::ExpandThenShift,
    Size{120, 500},
    Char{" ", Color{fg::Default, bg::Black}}};
```

Overflow mode guidance:

- `Shift` for fixed-height visible consoles that should scroll upward.
- `Wrap` for ring-buffer style behavior.
- `ExpandThenShift` for growing history with a cap.
- `ExpandThenWrap` for growing history that later becomes cyclic.

Keep the `fillChar` background aligned with the intended panel background so newly exposed rows remain visually stable.

## Paragraphs and Scrollback

`printParagraph()` is now part of the shared cursor-output path. Use it when the same wrapped text should work in a
real terminal and inside a retained buffer.

Pattern:

```cpp
auto options = ParagraphOptions::defaultOptions();
options.setMaximumLineWraps(2);

history.setColor(Color{fg::BrightYellow, bg::Black});
history.printParagraph("Cache refresh is still pending", options);
```

To display only part of the retained history, render it through a view:

```cpp
const auto top = std::max(0, history.size().height() - 20);
auto view = BufferConstRefView{history, Rectangle{0, top, 120, 20}};
terminal.updateScreen(view);
```

When explicit cursor positioning matters, use `moveCursor(...)` with `MoveMode::Absolute` or `MoveMode::Relative`.
Remember that `CursorBuffer` follows VT100-style wrapping behavior, including the pending-wrap behavior at the last
column.
