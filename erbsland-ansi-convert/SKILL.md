---
name: erbsland-ansi-convert
description: Use `erbsland-ansi-convert` to postprocess captured ANSI terminal output, including cursor motion, erase sequences, alternate-screen redraws, and progress updates, into static ANSI, HTML, or plain text suitable for documentation. Use when a command or TUI needs to be captured from a real terminal/PTY and flattened into a reusable documentation artifact, or when working with Python `Terminal`, `writeFile()`, `write()`, `to_ansi()`, `to_html()`, `to_text()`, `--format`, or `--collapse`.
---

# Erbsland ANSI Convert

Use this skill when the goal is to turn dynamic terminal output into a stable artifact for documentation. The main
pitfall is usually capture, not conversion: redraw-heavy tools must be recorded from a real terminal session, not copied
from a log that already lost cursor movement.

## Quick Workflow

1. Decide whether the source is a raw terminal capture file or a normalized transcript/log.
2. For dynamic output like cursor moves, `\r`, `ESC[2K`, or alternate-screen redraws, capture through a PTY recorder
   such as `script`, not plain stdout redirection.
3. Convert the capture with `erbsland-ansi-convert` to `ansi`, `html`, or `text`.
4. Use `--collapse` only when carriage returns were already normalized to newlines before conversion.
5. For HTML docs, read `references/html-integration.md` and include matching CSS.

## Pick the Right Input Path

- Prefer CLI `erbsland-ansi-convert INPUT` for one-off conversions.
- Prefer Python `Terminal.writeFile(path)` for real capture files because it preserves raw carriage returns and
  auto-detects CR-only transcript files.
- Use `Terminal.write(text, collapse_capture_updates=True)` or CLI `--collapse` only when the input already became plain
  text and repeated progress-line updates were split across multiple lines.
- If the source was copied from a pager, CI log, browser, or clipboard, assume it may already be normalized and may
  have lost fidelity.

## Capture Rules That Matter

- If the program changes cursor position, rewrites one line in place, uses `ESC[2K`, or draws on the alternate screen,
  capture from a real TTY/PTY.
- Plain redirection like `command > out.txt` is only reliable for simple line-oriented programs that already emit the
  exact bytes you want.
- If a tool disables colors or cursor control when stdout is not a TTY, wrap it in a PTY capture.
- Keep the capture as bytes until conversion. Do not "clean it up" first.

## Default Capture Recipes

Read `references/capture-and-convert.md` for concrete commands. Default to:

- `script -q capture.txt your-command args...` for one command on systems with BSD `script`.
- `script -q capture.txt` and then run the interactive program manually for multi-step sessions.
- If the capture file is a transcript-style file from `script`, feed the file path directly to
  `erbsland-ansi-convert` or `Terminal.writeFile(...)` instead of piping through stdin.

## Choose the Output Target

- `ansi`: Keep colors and styles, but flatten all cursor motion into a static terminal snapshot. Best for terminal-native
  demos, fixtures, or later rendering by ANSI-aware tooling. If the target is Sphinx documentation using
  `.. erbsland-ansi::`, also use `$erbsland-ansi-for-sphinx`.
- `html`: Best target for websites or rich docs. The output is an HTML fragment, not a full page, so you must provide
  CSS.
- `text`: Best for Markdown code fences, diffs, searchable docs, or when style is not needed.

## Minimal Commands

```bash
pip install erbsland-ansi-convert

erbsland-ansi-convert capture.txt > final.ansi
erbsland-ansi-convert --format text capture.txt > final.txt
erbsland-ansi-convert --format html --output final.html capture.txt
erbsland-ansi-convert --format text --collapse normalized-log.txt > final.txt
```

## Python Pattern

```python
from erbsland.ansi_convert import Terminal

terminal = Terminal(width=120, height=40, back_buffer_height=2000)
terminal.writeFile("capture.txt")  # preferred for real terminal captures

static_ansi = terminal.to_ansi()
plain_text = terminal.to_text()
html_fragment = terminal.to_html(class_prefix="docs-terminal")
```

If the input text no longer contains raw carriage returns, switch to
`terminal.write(text, collapse_capture_updates=True)` instead of `writeFile()`.

## Limits and Checks

- Supported styles are the documented SGR set plus basic and bright 8 ANSI colors. If the source depends on 256-color
  or truecolor escapes, validate the result carefully.
- Unknown or unsupported escape sequences can change the rendered result. For debugging, use
  `Terminal(warn_unknown_sequences=True)` and reduce the capture to a minimal failing sample.
- HTML output contains CSS classes like `{prefix}-bold`, `{prefix}-red`, and `{prefix}-background-blue`. Read
  `references/html-integration.md` before embedding it into docs.

## Decision Guide

- Need concrete capture commands and when to use `script` vs redirection:
  Read `references/capture-and-convert.md`.
- Need HTML/CSS embedding guidance:
  Read `references/html-integration.md`.
- Need to embed the resulting static ANSI into Sphinx docs with `.. erbsland-ansi::`:
  Also use `$erbsland-ansi-for-sphinx`.
