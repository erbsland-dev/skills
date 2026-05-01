---
name: erbsland-color-term
description: Integrate and use the Erbsland Color Terminal C++ library (`erbsland-color-term`, `erbsland::cterm`) for terminal output, `CursorWriter`/`CursorBuffer`, `Buffer`/`updateScreen()`, rich text via `text::HtmlRenderer`, and beta `ui::Application`. Use when wiring CMake, choosing headers, or implementing terminal apps with `TerminalSession`, `Terminal`, `StringView`, `Text`, `BufferView`, drawing helpers, themes, actions, or UI layouts.
---

# Erbsland Color Term

Targets `erbsland-color-term` version `1.10.0` (`2026-04-30`).

For later checkouts, verify headers and changelog before assuming a feature does not exist.

Read one matching reference first; open a second only when the implementation crosses layers.

If the project follows Erbsland's C++ conventions, also use `$erbsland-cpp-code-style`.

## Start With One Reference

- Build integration, header choice, `Terminal`, `TerminalSession`, direct output, text fallback:
  `references/integration-and-lifecycle.md`
- Shared output helpers, `CursorWriter`, `CursorBuffer`, `CharStyle`, `CharAttributes`, retained scrollback:
  `references/cursor-output-and-styles.md`
- Manual redraw apps with `Buffer`, `updateScreen()`, `BufferView`, `Text`, geometry, grids, clipped buffers, frames:
  `references/rendering-and-layout.md`
- Resize-aware loops, input polling, scrollable views, `UpdateSettings`, custom backends, or host integration:
  `references/interactive-and-advanced.md`
- Rich text, HTML, `text::HtmlRenderer`, `text::Style`, `text::TextNode`:
  `references/rich-text-and-html.md`
- Structured event-driven TUIs with pages, actions, themes, surfaces, layouts, timers, and workers:
  `references/ui-framework.md`

If unsure, start with `references/integration-and-lifecycle.md`.

## Choose The Highest-Level API That Fits

- Prefer `TerminalSession` over manual `initializeScreen()`/`restoreScreen()` unless exact lifetime control is needed.
- Prefer direct `Terminal::print()` and `printLine()` for short-lived tools.
- Prefer helpers that take `CursorWriter &` when the same output should work on `Terminal` and `CursorBuffer`.
- Prefer `CursorBuffer` for retained terminal-style text such as logs, consoles, and scrollback panes.
- Prefer `Buffer` plus `Terminal::updateScreen()` for manual redraw apps, dashboards, games, and composed layouts.
- Prefer `GridLayout`/`FrameBorder` for table-like frames and `WriteClippedBuffer` for subsurface painting.
- Prefer `text::HtmlRenderer` when the input is already structured HTML or document-like content.
- Prefer `ui::Application` for structured, event-driven TUIs with focus, actions, timers, or workers.
- Treat `ui` as beta. Use it when it matches the task, but expect API drift across later releases.

## Core Rules

- Create one long-lived `Terminal` per app.
- If you use `ui::Application`, let the framework own the terminal lifecycle.
- Use `String` as the text owner, `StringView` for read-only APIs, and `displayWidth()` when terminal cells matter.
- If malformed UTF-8 handling matters, look for `EncodingErrors` instead of relying on implicit recovery.
- Use `Inherited` to preserve an existing color component and `Default` to reset to the terminal default.
- Use `CharAttributes::reset()` only when later text must explicitly clear attributes.
- In UI code, use `Action`/`Actions` and `Keys` instead of the old key-binding API; use themes over hard-coded styling.
- Keep `OutputMode::FullControl` for cursor- or screen-aware workflows. Switch to `OutputMode::Text` only for plain
  text.
- Do not build manual ANSI escape handling or a custom backend until the higher-level APIs clearly do not fit.
- Do not duplicate logic across `Terminal` and `CursorBuffer` when a `CursorWriter &` helper would work.

## When To Read A Second Reference

- Retained log pane inside a redraw-based app:
  combine `cursor-output-and-styles.md` with `rendering-and-layout.md`
- HTML rendered into a scrollable document pane:
  combine `rich-text-and-html.md` with `interactive-and-advanced.md`
- Event-driven TUI with timers or background work:
  read `ui-framework.md`
- Backend integration or terminal-host embedding:
  start with `interactive-and-advanced.md`

## Good Source Examples

If the library source is available, the most useful demos to mirror are:

- `demo/minimum-effort` for direct terminal output and `TerminalSession`
- `demo/html-viewer` for `text::HtmlRenderer`, `CursorBuffer`, and viewport rendering
- `demo/ui-html-viewer`, `demo/ui-world-view`, and `demo/ui-sections` for current `ui::Application` patterns
- `demo/grid-layout` for `FrameBorder`, `GridLayout`, and table-like drawing
