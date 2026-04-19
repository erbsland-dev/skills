# Interactive and Advanced Features

Use this reference when you are staying on the manual `Terminal` and `Buffer` workflow and need key input, resize
handling, scrollable views, host integration, or custom backends.

If the task wants a structured event-driven surface tree, read `ui-framework.md` instead.

## Input Loop Pattern

Use key mode plus timed reads for responsive redraw loops:

```cpp
using namespace std::chrono_literals;

terminal.input().setMode(Input::Mode::Key);

while (!quitRequested) {
    terminal.testScreenSize();
    buffer.resize(terminal.size());
    buffer.fill(Char{" ", Color{fg::Default, bg::Black}});

    if (const auto key = terminal.input().read(90ms); key.valid()) {
        if (key == Key{Key::Character, U'q'}) {
            quitRequested = true;
        }
    }

    terminal.updateScreen(buffer, settings);
    terminal.flush();
}
```

Guidance:

- use a timeout to avoid busy waiting
- handle resize every frame with `testScreenSize()`
- keep the buffer persistent and only resize it
- render the full frame before calling `updateScreen()`

## Mixing Key And Line Input

Use `Input::Mode::Key` for interactive screens and `Input::Mode::ReadLine` for free-form prompts.

Switch modes on the same terminal when the app mixes both styles:

```cpp
terminal.input().setMode(Input::Mode::ReadLine);
terminal.print("Name: ");
const auto name = terminal.input().readLine();

terminal.input().setMode(Input::Mode::Key);
```

Use `InputDefinition` when the app also needs user-facing shortcut descriptions or configurable key bindings.

## Minimum Size And Crop Feedback

Use `UpdateSettings` instead of ad-hoc checks when the terminal may be too small:

```cpp
auto settings = UpdateSettings{};
settings.setMinimumSize(Size{60, 18});
settings.setMinimumSizeBackground(Char{" ", Color{fg::Inherited, bg::Red}});
settings.setMinimumSizeMessage(String{"Please enlarge the terminal"});
settings.setShowCropMarks(true);
```

## Scrollable Content

Use `CursorBuffer` for retained streaming text, then show only the interesting window through a view:

```cpp
auto view = BufferConstRefView{history, Rectangle{0, topRow, width, height}};
terminal.updateScreen(view, settings);
```

Move the view rectangle instead of copying data into a new buffer whenever possible.

## Host Integration And Custom Backends

Stay on the built-in backend unless the task truly needs:

- integration with another console abstraction
- deterministic output capture in a test harness
- a specialized runtime or embedded host environment

When the host already owns restoration or signal handling, inspect `TerminalFlag` and `TerminalFlags` before adding
custom cleanup layers.

Use a custom `Backend` only when higher-level APIs cannot meet the integration requirement.

## Practical Escalation Path

Default progression:

1. Direct `printLine()` output
2. Shared helpers that take `CursorWriter &`
3. `CursorBuffer` for retained terminal-style output
4. `Buffer` plus `updateScreen()`
5. `BufferView` for scrolling or clipping
6. `Text`, frames, fonts, and bitmaps
7. Custom backend only if integration requirements demand it
