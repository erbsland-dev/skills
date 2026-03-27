# Capture And Convert

This guide is for the part that usually goes wrong: preserving enough terminal behavior during capture that
`erbsland-ansi-convert` can reconstruct the final static view.

## Start With The Right Capture Strategy

Use the least complicated capture that still preserves behavior:

- Simple line-oriented output with no cursor movement:
  Plain stdout redirection can be enough.
- Output that uses `\r`, `CSI K`, cursor moves, progress bars, spinners, full-screen layouts, or alternate-screen
  redraws:
  Capture through a PTY recorder.
- Interactive sessions:
  Use a PTY recorder and capture the session as bytes, not as copied terminal text.

If the program detects "not a TTY" and disables styling or redraw logic, plain redirection is the wrong tool.

## Verify `script` Syntax On The Target Machine

`script(1)` differs between BSD/macOS and GNU/Linux. Before suggesting a command, inspect local usage with
`man script` or the command's built-in help and adapt the recipe to the actual platform.

Verified BSD/macOS syntax:

```bash
script -q capture.txt your-command --flag value
```

Interactive session capture:

```bash
script -q capture.txt
# run the TUI or the multi-step command sequence
exit
```

Use `-q` to suppress `script`'s own banner/trailer lines in the transcript.

## When Plain Redirection Is Good Enough

For simple output that already contains the exact bytes to preserve, the direct path is fine:

```bash
your-command > capture.txt
erbsland-ansi-convert --format text capture.txt > final.txt
```

Do not use this for full-screen tools or progress displays unless you already know the program writes ANSI output even
when stdout is not a terminal.

## Convert The Capture

Use file input whenever you have a real capture file:

```bash
erbsland-ansi-convert capture.txt > final.ansi
erbsland-ansi-convert --format text capture.txt > final.txt
erbsland-ansi-convert --format html --output final.html capture.txt
```

Why file input matters:

- The CLI uses `Terminal.writeFile(...)` for file paths.
- `writeFile(...)` preserves carriage returns.
- `writeFile(...)` auto-detects CR-only transcript files and enables collapse mode when appropriate.

If you pipe through stdin instead, the tool uses `write(...)`, so you lose that capture-file-specific auto-detection.

## Use `--collapse` Only For Normalized Text

Use `--collapse` when the input was already normalized to newline-delimited text before conversion and repeated
progress-line updates turned into duplicated lines.

Typical examples:

- pasted terminal output from a browser or chat tool
- logs exported from CI systems
- text copied from a recorder that already replaced `\r` with `\n`

Example:

```bash
cat normalized-log.txt | erbsland-ansi-convert --format text --collapse - > final.txt
```

Avoid `--collapse` for raw capture files unless you have confirmed the file was already normalized. Real capture files
should usually be passed as file paths and left to `writeFile(...)`.

## Python API Patterns

Real capture file:

```python
from erbsland.ansi_convert import Terminal

terminal = Terminal(width=120, height=40, back_buffer_height=2000)
terminal.writeFile("capture.txt")

ansi_output = terminal.to_ansi()
plain_text = terminal.to_text()
html_fragment = terminal.to_html(class_prefix="docs-terminal")
```

Already-normalized text:

```python
from erbsland.ansi_convert import Terminal

terminal = Terminal()
terminal.write(normalized_text, collapse_capture_updates=True)
final_text = terminal.to_text()
```

## Debugging A Bad Conversion

If the output looks wrong:

1. Check whether the source was captured from a real PTY or copied from somewhere that normalized it.
2. Re-run the flow with file input instead of stdin if you have a capture file.
3. Test a smaller sample that still reproduces the problem.
4. Enable `Terminal(warn_unknown_sequences=True)` in Python to expose unsupported escape sequences.
5. If the original output uses 256-color or truecolor sequences, expect partial fidelity because the documented support
   is basic plus bright 8 ANSI colors.

## Documentation-Oriented Output Choices

- Use `text` for Markdown code fences, searchable docs, and diffs.
- Use `html` for web docs that should preserve color and emphasis.
- Use `ansi` when the next tool in the pipeline understands ANSI and you want a flattened, static terminal snapshot
  rather than a live redraw transcript.
