# Integration and Lifecycle

Use this reference when adding the library to a project, choosing includes, deciding between direct output and
higher-level APIs, or setting up the `Terminal` lifecycle correctly.

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

Current repository guidance for the `1.8.0` tree is `CMake 3.28+` and `C++20`. Prefer the checked-out repository's
current top-level requirement over older snippets copied from stale docs.

## Include Strategy

- Use `<erbsland/cterm/all.hpp>` for the core terminal, buffer, text, and drawing APIs.
- Add `<erbsland/cterm/text/all.hpp>` when using the rich-text and HTML module.
- Add `<erbsland/cterm/ui/all.hpp>` when using the beta UI framework.
- Generated top-level wrappers such as `<erbsland/all_cterm_text.hpp>` and `<erbsland/all_cterm_ui.hpp>` also exist.
  Use them only if the surrounding project already prefers that include style.

## Pick The Workflow First

- Direct `Terminal` output:
  short-lived tools, prompts, diagnostics, colored CLI output
- `CursorWriter` helpers:
  shared text output that should work on `Terminal` and `CursorBuffer`
- `CursorBuffer`:
  retained logs, consoles, scrollback panes, REPL history
- `Buffer` plus `updateScreen()`:
  manual redraw UIs, dashboards, composed layouts, animation
- `text::HtmlRenderer`:
  document-like output, rendered help pages, HTML fragments, structured prose
- `ui::Application`:
  event-driven TUIs with pages, surfaces, focus, key bindings, timers, workers

## Terminal Lifecycle

Prefer `TerminalSession` for normal apps and tools:

```cpp
#include <erbsland/cterm/all.hpp>

using namespace erbsland::cterm;

auto main() -> int {
    auto terminal = Terminal{Size{80, 25}};
    auto session = TerminalSession{terminal};

    if (!terminal.isInteractive()) {
        terminal.setOutputMode(Terminal::OutputMode::Text);
    }

    terminal.printLine(fg::BrightGreen, "Ready.");
    terminal.flush();
    return 0;
}
```

Rules:

- Create one long-lived `Terminal` per app.
- Prefer `TerminalSession` over manual `initializeScreen()` and `restoreScreen()` pairing.
- Call `isInteractive()` after initialization when output may be redirected.
- Keep `OutputMode::FullControl` for cursor or screen control. Use `OutputMode::Text` only for plain text fallback.
- If the host already manages signal handling or terminal restoration, look at `TerminalFlag` and `TerminalFlags`
  before bolting on custom cleanup logic.
- If you use `ui::Application`, let the framework manage the terminal lifecycle instead of wrapping it yourself.

## When Manual Lifecycle Control Still Makes Sense

Use explicit `initializeScreen()` and `restoreScreen()` only when the task needs a very precise control window around
interactive setup and teardown, such as specialized host integration or tests that intentionally exercise that boundary.

For manual redraw loops, continue with `rendering-and-layout.md` or `interactive-and-advanced.md`.
