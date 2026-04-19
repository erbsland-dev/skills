---
name: erbsland-color-term
description: Integrate and use the Erbsland Color Terminal C++ library (`erbsland-color-term`, `erbsland::cterm`) for direct terminal output, `CursorWriter` and `CursorBuffer`, `Buffer` and `updateScreen()`, rich text via `text::HtmlRenderer`, and the beta `ui::Application` framework. Use when wiring CMake, choosing headers, or implementing terminal apps with `TerminalSession`, `Terminal`, `String`, `Text`, `BufferView`, drawing helpers, HTML rendering, or UI surfaces and layouts.
---

# Erbsland Color Term

Targets `erbsland-color-term` version `1.8.0` (`2026-04-17`).

If the checked-out project uses a later version, treat this skill as the default map, then verify newer APIs in the
headers and changelog before assuming a feature does not exist. Later versions may add higher-level helpers or change
beta APIs.

This skill is intentionally compact. Do not read every reference up front. Start with the one reference that matches
the task, then open a second one only if the implementation crosses layers.

If the project follows Erbsland's C++ conventions, also use `$erbsland-cpp-code-style`.

## Start With One Reference

- Build integration, header choice, `Terminal`, `TerminalSession`, direct output, plain-text fallback:
  `references/integration-and-lifecycle.md`
- Shared output helpers, `CursorWriter`, `CursorBuffer`, `CharStyle`, `CharAttributes`, retained scrollback:
  `references/cursor-output-and-styles.md`
- Manual redraw apps with `Buffer`, `updateScreen()`, `BufferView`, `Text`, geometry, frames, fonts, or bitmaps:
  `references/rendering-and-layout.md`
- Resize-aware loops, input polling, scrollable views, `UpdateSettings`, custom backends, or host integration:
  `references/interactive-and-advanced.md`
- Rich text, HTML, `text::HtmlRenderer`, `text::Style`, `text::TextNode`:
  `references/rich-text-and-html.md`
- Structured event-driven TUIs with pages, surfaces, layouts, timers, and worker threads:
  `references/ui-framework.md`

If unsure, start with `references/integration-and-lifecycle.md`.

## Choose The Highest-Level API That Fits

- Prefer `TerminalSession` over manual `initializeScreen()` and `restoreScreen()` pairing unless explicit lifetime
  control is required.
- Prefer direct `Terminal::print()` and `printLine()` for short-lived tools and simple colored output.
- Prefer helpers that take `CursorWriter &` when the same output should work on `Terminal` and `CursorBuffer`.
- Prefer `CursorBuffer` for retained terminal-style text such as logs, consoles, and scrollback panes.
- Prefer `Buffer` plus `Terminal::updateScreen()` for manual redraw apps, dashboards, games, and composed layouts.
- Prefer `text::HtmlRenderer` when the input is already structured HTML or document-like content.
- Prefer `ui::Application` when building a structured, event-driven TUI with focus handling, key bindings, timers, or
  worker threads.
- Treat `ui` as beta. Use it when it matches the task, but expect API drift across later releases.

## Core Rules

- Create one long-lived `Terminal` per app.
- If you use `ui::Application`, let the framework own the terminal lifecycle.
- Use `String` as the public UTF-8 boundary and `String::displayWidth()` when terminal cell width matters.
- If malformed UTF-8 handling matters, look for `EncodingErrors` instead of relying on implicit recovery.
- Use `Inherited` to preserve an existing color component and `Default` to reset to the terminal default.
- Use `CharAttributes::reset()` only when later text must explicitly clear attributes.
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
- `demo/ui-hello-world` for `ui::Application`, `ui::Stack`, `ui::TextBox`, and scheduled updates
