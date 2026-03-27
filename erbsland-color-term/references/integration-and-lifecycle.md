# Integration and Lifecycle

Use this reference when adding the library to a project, choosing between direct output, shared cursor output, retained
buffers, and full-screen rendering, or setting up the `Terminal` lifecycle.

## Contents

- Build integration
- Choose the output style
- Terminal lifecycle rules
- Shared cursor output pattern
- Direct terminal output pattern
- Full-screen pattern

## Build Integration

Prefer source integration when possible. It keeps the dependency pinned with the application and matches the library's
recommended usage model.

Top-level `CMakeLists.txt`:

```cmake
add_subdirectory(erbsland-cpp-color-term)
add_subdirectory(my-app)
```

Application `CMakeLists.txt`:

```cmake
add_executable(my-app
    src/main.cpp
)

target_compile_features(my-app PRIVATE cxx_std_20)
target_link_libraries(my-app PRIVATE erbsland-color-term)
```

If the library is installed as a package instead, use:

```cmake
find_package(erbsland-color-term REQUIRED)

target_link_libraries(my-app PRIVATE ErbslandDEV::erbsland-color-term)
```

Notes:

- The cloned library currently uses CMake 3.28 and C++20.
- Use `<erbsland/cterm/all.hpp>` when moving quickly or exploring.
- Narrow includes later only if the project prefers explicit header lists.

## Choose the Output Style

Use direct terminal writes for:

- Short-lived tools
- Status lines and diagnostics
- Prompts or line-oriented interactions
- Simple colored output that does not redraw a whole scene

Use `CursorWriter` helpers for:

- Output code that should work on both `Terminal` and `CursorBuffer`
- Shared banners, status lines, paragraphs, or help blocks
- Reusable formatting that should not depend on a concrete backend

Use `CursorBuffer` plus a view for:

- Retained log history
- REPL or console scrollback
- Streaming text that later needs clipping, scrolling, or re-rendering

Use `Buffer` plus `updateScreen()` for:

- Full-screen apps
- Redraw loops
- Resizable layouts
- Multi-panel dashboards
- Animations
- Anything using frames, wrapped text, views, or screen diffing

## Terminal Lifecycle Rules

- Create one `Terminal` for the whole app lifetime.
- Call `initializeScreen()` once when the app starts.
- Call `restoreScreen()` once just before exit.
- Call `isInteractive()` after initialization when the process may run without a real terminal.
- Keep `OutputMode::FullControl` unless the app truly needs plain text only.
- Keep the default safe margin unless the app explicitly needs the full detected size and newline-free updates.
- If the app has multiple return paths or exception-heavy flow, add a small local RAII wrapper so normal unwinding still
  reaches `restoreScreen()`.

The library restores terminal state automatically for interruptions such as signals between initialization and restore,
but agents should still structure code so normal exits always call `restoreScreen()`.

## Shared Cursor Output Pattern

Use this when the same helper should target a terminal and an in-memory retained buffer:

```cpp
#include <erbsland/cterm/all.hpp>

using namespace erbsland::cterm;

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

This is the 1.7 default for reusable output logic.

## Direct Terminal Output Pattern

Use this for simple colored output:

```cpp
#include <erbsland/cterm/Terminal.hpp>

using namespace erbsland::cterm;

auto main() -> int {
    auto terminal = Terminal{Size{80, 25}};
    terminal.initializeScreen();

    if (!terminal.isInteractive()) {
        terminal.setOutputMode(Terminal::OutputMode::Text);
    }

    terminal.printLine(
        bg::Blue,
        fg::BrightWhite,
        " Signal Board ",
        Color::reset(),
        " ",
        fg::BrightBlack,
        "Direct output with readable color arguments.");
    terminal.printLine(fg::BrightGreen, "Status", fg::BrightBlack, ": online");
    terminal.setDefaultColor();
    terminal.flush();

    terminal.restoreScreen();
    return 0;
}
```

Guidance:

- Prefer `printLine()` over manual ANSI sequences.
- Use `fg::...`, `bg::...`, and `Color` objects instead of hard-coded escape codes.
- Use a `CursorWriter &` helper when this output may later be reused in a `CursorBuffer`.
- Use `CharAttributes::reset()` after emphasized fragments when later text should be plain again.
- Call `setDefaultColor()` before leaving the output region if later output should not inherit styling.
- Rely on line buffering for tidy incremental writes, then `flush()` when needed.

## Full-Screen Pattern

Use this for redraw-based apps:

```cpp
auto terminal = Terminal{Size{96, 28}};
auto buffer = Buffer{terminal.size()};
auto settings = UpdateSettings{};

terminal.initializeScreen();
terminal.setRefreshMode(Terminal::RefreshMode::Overwrite);
settings.setMinimumSize(Size{40, 12});

for (;;) {
    terminal.testScreenSize();
    buffer.resize(terminal.size());
    buffer.fill(Char{" ", Color{fg::Default, bg::Black}});
    // ... draw to buffer ...
    terminal.updateScreen(buffer, settings);
}
```

Guidance:

- `Overwrite` is a good default for redraw loops.
- Use `UpdateSettings` for minimum-size handling, crop marks, and alternate-screen behavior.
- Keep a separate `CursorBuffer` only when the app needs retained streamed text in addition to the composed screen
  buffer.
- Disable `settings.setSwitchToAlternateBuffer(false)` only when the rendered screen should remain visible in the
  normal terminal history.
- Keep back-buffer behavior enabled when diff-based updates are useful; the terminal already optimizes redraws.
