---
name: erbsland-color-term
description: Integrate and use the Erbsland Color Terminal C++ library (`erbsland-color-term`, `erbsland::cterm`) for colorful terminal output, cursor-oriented streaming, full-screen terminal UIs, Unicode-aware text layout, styles, scrollback buffers, input handling, animation, frames, bitmaps, scrollable views, and custom terminal backends. Use when adding the library to a CMake project, writing code against `<erbsland/cterm/...>` or `<erbsland/cterm/all.hpp>`, or implementing terminal apps with `Terminal`, `CursorWriter`, `Buffer`, `CursorBuffer`, `Text`, `String`, `CharStyle`, `Input`, `UpdateSettings`, `BufferView`, drawing helpers, or bitmap rendering.
---

# Erbsland Color Term

Use this skill to put agents on the library's pit-of-success path: choose the right output model, build around one
`Terminal`, reuse `CursorWriter` helpers across live terminals and retained buffers, render through persistent buffers
when the UI redraws, and reach for the higher-level APIs before inventing custom terminal plumbing.

If the project follows Erbsland's C++ conventions, also use `$erbsland-cpp-code-style`.

## Quick Workflow

1. Decide whether the task is plain terminal output, shared cursor-oriented output, retained scrollback/history, or a
   redraw-based full-screen UI.
2. Read `references/integration-and-lifecycle.md` and set up the build plus `Terminal` lifecycle correctly.
3. Read `references/cursor-output-and-styles.md` when the same output should work on `Terminal` and `CursorBuffer`, or
   when using `CharAttributes`, `CharStyle`, `printParagraph()`, or scrollback-style buffers.
4. For layout-heavy or interactive screens, render into a persistent `Buffer` and call `updateScreen()`.
5. Read `references/rendering-and-layout.md` when composing panels, wrapped text, frames, colors, fonts, bitmaps, or
   buffer-to-buffer composition.
6. Read `references/interactive-and-advanced.md` when adding key input, resize handling, scrolling, animation, or a
   custom backend.

## Choose the Right Level

- Use direct `Terminal::print()` / `printLine()` calls for short-lived tools, status output, prompts, and diagnostics.
- Use helpers that take `CursorWriter &` when the same streaming output should target a live `Terminal` and an
  in-memory `CursorBuffer`.
- Use `CursorBuffer` for retained terminal-style output such as logs, consoles, REPL history, and scrollback panes.
- Use `Buffer` plus `Terminal::updateScreen()` for dashboards, games, multi-panel layouts, redraw loops, or anything
  that reacts to resize or key input.
- Use `Text`, `String`, `CharStyle`, `ParagraphOptions`, `Rectangle`, `Margins`, and alignment helpers before writing
  manual cursor math.
- Use `BufferDrawOptions`, `BufferView`, `Bitmap`, `FrameDrawOptions`, and custom backends only when the simpler
  cursor/buffer workflow no longer fits.

## Core Rules

- Create exactly one `Terminal` for the application lifetime.
- Call `initializeScreen()` once near startup and `restoreScreen()` once on exit.
- Call `isInteractive()` after initialization when output may be redirected, and switch to a plain-text fallback when
  interactive control is unavailable.
- Keep one persistent `Buffer` in interactive apps and resize it when the terminal size changes.
- Prefer shared `CursorWriter` helpers over duplicating formatting logic for `Terminal` and `CursorBuffer`.
- Prefer `updateScreen()` over raw `write()` for full-screen rendering.
- Keep `OutputMode::FullControl` for UIs and cursor-aware output. Switch to `OutputMode::Text` only for plain text
  without cursor/screen control.
- Keep alternate-screen switching enabled for real TUIs. Disable it in `UpdateSettings` only when the output should
  remain in the normal shell history.
- Poll input with a timeout instead of busy waiting.
- Call `testScreenSize()` inside redraw loops.
- Use `CharAttributes::reset()` when you need to explicitly turn attributes back off.
- Use `Inherited` to preserve an underlying or active style component and `Default` to reset back to the terminal
  default.
- When relying on italic, blink, hidden, or strikethrough behavior, check `supportedCharAttributes()` and tolerate
  partial terminal support.
- Use `String::displayWidth()` when visible terminal width matters; `size()` counts stored characters, not cells.

## Default Implementation Patterns

Start from one of these shapes:

Shared cursor-oriented output helper:

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

Interactive full-screen app:

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

    if (!terminal.isInteractive()) {
        terminal.setOutputMode(Terminal::OutputMode::Text);
        terminal.printLine("Mode: plain-text fallback");
        terminal.flush();
        terminal.restoreScreen();
        return 0;
    }

    while (!quitRequested) {
        terminal.testScreenSize();
        buffer.resize(terminal.size());
        buffer.fill(Char{" ", Color{fg::Default, bg::Black}});

        if (const auto key = terminal.input().read(std::chrono::milliseconds{90}); key.valid()) {
            if (key == Key{Key::Character, U'q'}) {
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

Add a persistent `CursorBuffer` when the app also needs retained history, log panes, or a scrollback viewport that is
rendered into the main screen buffer.

## Decision Guide

- Adding the dependency or choosing between `add_subdirectory(...)` and `find_package(...)`:
  Read `references/integration-and-lifecycle.md`.
- Writing a simple CLI with colors and no redraw loop, or sharing the same print helper between a terminal and a
  buffer:
  Read `references/cursor-output-and-styles.md`.
- Writing direct terminal output with reusable styles or ANSI attributes:
  Read `references/cursor-output-and-styles.md`.
- Building a scrollback pane, log history, or retained text console:
  Read `references/cursor-output-and-styles.md` and `references/interactive-and-advanced.md`.
- Writing a simple CLI with colors and no retained output:
  Use direct terminal output from `references/integration-and-lifecycle.md`.
- Building a screen with panels, titles, wrapped text, Unicode, or animated headings:
  Read `references/rendering-and-layout.md`.
- Adding keyboard control, resize awareness, scrolling, viewports, procedural graphics, or test backends:
  Read `references/interactive-and-advanced.md`.

## Avoid These Mistakes

- Do not recreate `Terminal` or `Buffer` every frame unless the surrounding design forces it.
- Do not duplicate printing helpers for `Terminal` and `CursorBuffer` when a single `CursorWriter &` helper would do.
- Do not drive a full-screen app with repeated `printLine()` calls.
- Do not use a plain `Buffer` as a scrollback history when `CursorBuffer` matches the problem better.
- Do not hand-calculate terminal coordinates when `Rectangle`, `subRectangle()`, `insetBy()`, or `gridCells()` express
  the layout more clearly.
- Do not use `OutputMode::Text` and then expect `updateScreen()`, cursor control, or size detection to work.
- Do not treat `Inherited`, `Default`, and `CharAttributes::reset()` as synonyms.
- Do not assume every terminal emulator visibly renders every ANSI attribute even when the backend can emit it.
- Do not forget that wide and combining Unicode characters occupy terminal cells differently from character count.
