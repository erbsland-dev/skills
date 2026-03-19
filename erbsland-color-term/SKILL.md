---
name: erbsland-color-term
description: Integrate and use the Erbsland Color Terminal C++ library (`erbsland-color-term`, `erbsland::cterm`) for colorful terminal output, full-screen terminal UIs, Unicode-aware text layout, input handling, animation, frames, bitmaps, scrollable views, and custom terminal backends. Use when adding the library to a CMake project, writing code against `<erbsland/cterm/...>` or `<erbsland/cterm/all.hpp>`, or implementing terminal apps with `Terminal`, `Buffer`, `Text`, `String`, `Input`, `UpdateSettings`, `BufferView`, drawing helpers, or bitmap rendering.
---

# Erbsland Color Term

Use this skill to put agents on the library's pit-of-success path: choose the right integration mode, build around one
`Terminal`, render through buffers when the UI redraws, and reach for the higher-level APIs before inventing custom
terminal plumbing.

If the project follows Erbsland's C++ conventions, also use `$erbsland-cpp-code-style`.

## Quick Workflow

1. Decide whether the task is plain colored output or a redraw-based terminal UI.
2. Read `references/integration-and-lifecycle.md` and set up the build plus `Terminal` lifecycle correctly.
3. For anything layout-heavy or interactive, render into a persistent `Buffer` and call `updateScreen()`.
4. Read `references/rendering-and-layout.md` when composing panels, wrapped text, frames, colors, fonts, or bitmaps.
5. Read `references/interactive-and-advanced.md` when adding key input, resize handling, scrolling, animation, or a
   custom backend.

## Choose the Right Level

- Use direct `Terminal::print()` / `printLine()` calls for short-lived tools, status output, prompts, and diagnostics.
- Use `Buffer` plus `Terminal::updateScreen()` for dashboards, games, multi-panel layouts, redraw loops, or anything
  that reacts to resize or key input.
- Use `Text`, `String`, `Rectangle`, `Margins`, and alignment helpers before writing manual cursor math.
- Use `BufferView`, `Bitmap`, `FrameDrawOptions`, and custom backends only when the simpler buffer workflow no longer
  fits.

## Core Rules

- Create exactly one `Terminal` for the application lifetime.
- Call `initializeScreen()` once near startup and `restoreScreen()` once on exit.
- Keep one persistent `Buffer` in interactive apps and resize it when the terminal size changes.
- Prefer `updateScreen()` over raw `write()` for full-screen rendering.
- Keep `OutputMode::FullControl` for UIs. Switch to `OutputMode::Text` only for plain text without cursor/screen
  control.
- Keep alternate-screen switching enabled for real TUIs. Disable it in `UpdateSettings` only when the output should
  remain in the normal shell history.
- Poll input with a timeout instead of busy waiting.
- Call `testScreenSize()` inside redraw loops.
- Use `Inherited` to preserve an underlying color and `Default` to reset back to the terminal default.
- Use `String::displayWidth()` when visible terminal width matters; `size()` counts stored characters, not cells.

## Default Implementation Pattern

Start from this shape for most interactive applications:

```cpp
#include <chrono>
#include <erbsland/cterm/all.hpp>

using namespace erbsland::cterm;

auto main() -> int {
    auto terminal = Terminal{Size{96, 28}};
    auto buffer = Buffer{terminal.size()};
    auto settings = UpdateSettings{};
    auto quitRequested = false;

    terminal.initializeScreen();
    terminal.setRefreshMode(Terminal::RefreshMode::Overwrite);
    terminal.input().setMode(Input::Mode::Key);
    settings.setMinimumSize(Size{40, 12});

    while (!quitRequested) {
        terminal.testScreenSize();
        buffer.resize(terminal.size());
        buffer.fill(Char{" ", Color{fg::Default, bg::Black}});

        if (const auto key = terminal.input().read(std::chrono::milliseconds{90}); key.valid()) {
            if (key == Key{Key::Character, 'q'}) {
                quitRequested = true;
            }
        }

        // Render the whole frame into buffer here.
        terminal.updateScreen(buffer, settings);
        terminal.flush();
    }

    terminal.restoreScreen();
    return 0;
}
```

Adjust this downward for simple output programs and upward only when the task genuinely needs advanced features.

## Decision Guide

- Adding the dependency or choosing between `add_subdirectory(...)` and `find_package(...)`:
  Read `references/integration-and-lifecycle.md`.
- Writing a simple CLI with colors and no redraw loop:
  Use direct terminal output from `references/integration-and-lifecycle.md`.
- Building a screen with panels, titles, wrapped text, Unicode, or animated headings:
  Read `references/rendering-and-layout.md`.
- Adding keyboard control, resize awareness, scrolling, viewports, procedural graphics, or test backends:
  Read `references/interactive-and-advanced.md`.

## Avoid These Mistakes

- Do not recreate `Terminal` or `Buffer` every frame unless the surrounding design forces it.
- Do not drive a full-screen app with repeated `printLine()` calls.
- Do not hand-calculate terminal coordinates when `Rectangle`, `subRectangle()`, `insetBy()`, or `gridCells()` express
  the layout more clearly.
- Do not use `OutputMode::Text` and then expect `updateScreen()`, cursor control, or size detection to work.
- Do not treat `Inherited` and `Default` as synonyms.
- Do not forget that wide and combining Unicode characters occupy terminal cells differently from character count.
