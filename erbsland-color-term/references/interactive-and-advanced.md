# Interactive and Advanced Features

Use this reference when building redraw loops, handling key input, reacting to resize, showing scrollable content, or
integrating the library with `CursorBuffer`, custom backends, and testing tools.

## Contents

- Input loop pattern
- Switching between key and line input
- Resize, minimum size, and crop feedback
- Scrollable, retained, and viewport-based UIs
- Animated colors and reusable styles
- Custom backends
- Good escalation path

## Input Loop Pattern

Use key mode plus timed reads for responsive redraw loops:

```cpp
using namespace std::chrono_literals;

terminal.initializeScreen();
terminal.setRefreshMode(Terminal::RefreshMode::Overwrite);
terminal.input().setMode(Input::Mode::Key);

while (!quitRequested) {
    terminal.testScreenSize();
    buffer.resize(terminal.size());
    buffer.fill(Char{" ", Color{fg::Default, bg::Black}});

    if (const auto key = terminal.input().read(90ms); key.valid()) {
        if (key == Key{Key::Character, 'q'}) {
            quitRequested = true;
        }
    }

    terminal.updateScreen(buffer, settings);
    terminal.flush();
}
```

Guidance:

- Use a timeout to avoid busy waiting.
- Handle resize every frame with `testScreenSize()`.
- Keep the buffer persistent and only resize it.
- Put all drawing for the frame into the buffer before `updateScreen()`.
- Use `isInteractive()` after initialization if the same app must fall back to non-interactive output.

## Switching Between Key and Line Input

Use `Input::Mode::Key` for interactive screens and `Input::Mode::ReadLine` for free-form prompts.

Switch modes on the same terminal when the app mixes both interaction styles:

```cpp
terminal.input().setMode(Input::Mode::ReadLine);
terminal.print("Name: ");
const auto name = terminal.input().readLine();

terminal.input().setMode(Input::Mode::Key);
```

Use `InputDefinition` when the app needs user-facing key binding text such as help screens or configurable shortcuts.

## Resize, Minimum Size, and Crop Feedback

Use `UpdateSettings` instead of ad-hoc checks when the terminal may be too small:

```cpp
auto settings = UpdateSettings{};
settings.setMinimumSize(Size{60, 18});
settings.setMinimumSizeBackground(Char{" ", Color{fg::Inherited, bg::Red}});
settings.setMinimumSizeMessage(String{"Please enlarge the terminal"});
settings.setShowCropMarks(true);
```

Use this for:

- Enforcing a minimum usable layout size
- Displaying a centered fallback message
- Showing that content is cropped on the right or bottom

## Scrollable, Retained, and Viewport-Based UIs

Use `CursorBuffer` when text arrives over time and should behave like streamed terminal output. Then show only the
interesting window through `BufferView` or `BufferConstRefView`.

Typical uses:

- Scrollable log panes
- Editors
- Minimap or preview windows
- Large world buffers in games or simulations
- REPL or console history
- Dashboard panes with retained event history

Pattern:

```cpp
auto history = CursorBuffer{
    Size{120, 10},
    CursorBuffer::OverflowMode::ExpandThenShift,
    Size{120, 500},
    Char{" ", Color{fg::Default, bg::Black}}};

history.printParagraph("2026-03-27 09:14:22 INF Service started");

auto view = BufferConstRefView{history, Rectangle{0, 0, 40, 12}};
terminal.updateScreen(view, settings);
```

Move the view rectangle instead of copying the whole buffer. Enable crop characters when the UI should explicitly show
that more content exists off-screen.

## Animated Colors and Reusable Styles

Reach for these higher-level features when a plain static panel is not enough:

- `ColorSequence` for rotating or striped palettes
- `TextAnimation` for animated titles
- `FrameDrawOptions` for animated or reusable border styles
- `BitmapDrawOptions` for animated pixel graphics
- `CharStyle` for reusable headings, labels, and emphasis that combine color with ANSI attributes

Keep animation state in your app model, usually as an incrementing cycle counter. Pass that counter into `drawText(...)`,
`drawFrame(...)`, or `drawBitmap(...)` instead of storing timing logic inside rendering helpers.

## Custom Backends

Use a custom `Backend` only when the built-in terminal integration is not enough, such as:

- Integration with another console abstraction
- Test harnesses that record output
- Specialized runtime environments

Implement backend hooks for:

- Platform initialization and restoration
- Color and cursor capability reporting
- Supported character attributes
- Screen size detection
- Cursor movement
- Text emission and flushing
- Input mode changes plus key/line reads

Most applications should stay on the built-in backend.

## Good Escalation Path

Default to this progression:

1. Direct `printLine()` output
2. Shared helpers that take `CursorWriter &`
3. `CursorBuffer` for retained terminal-style output
4. `Buffer` plus `updateScreen()`
5. `Text`, frames, rectangles, and colors
6. Key input and resize-aware redraw loop
7. `BufferView` for scrolling
8. `Bitmap` and advanced drawing styles
9. Custom backend only if integration requirements demand it
