# Preparing Snippets

This guide covers the handoff between terminal capture and the Sphinx directive.

## Start With Static ANSI, Not Raw Terminal Behavior

`.. erbsland-ansi::` is best when the input is already a static ANSI-colored text block.

Good input:

- colored command output
- build logs with SGR styling
- already-flattened terminal snapshots

Bad direct input:

- spinners and progress bars that rely on `\r`
- cursor positioning or erase sequences that change earlier text
- alternate-screen/full-screen application captures

For the second group, normalize first with `$erbsland-ansi-convert`.

## Recommended Pipeline For Captured Output

1. Capture the terminal output properly, ideally from a real PTY.
2. Normalize it into static ANSI with `$erbsland-ansi-convert`.
3. Replace real ESC bytes with `␛`.
4. Paste the result into `.. erbsland-ansi::` with `:escape-char: ␛`.

Example conversion flow:

```bash
erbsland-ansi-convert capture.txt > snippet.ansi
```

If the capture was already normalized text and needs the collapse heuristic:

```bash
erbsland-ansi-convert --format ansi --collapse normalized-log.txt > snippet.ansi
```

If you need the capture workflow itself, also use `$erbsland-ansi-convert`.

## Prefer `␛` In Documentation Sources

Real ESC bytes are awkward in editors, hard to review in diffs, and easy to break when copied around. Replace them with
the visible `␛` character and declare it explicitly in the directive:

```rst
.. erbsland-ansi::
    :escape-char: ␛

    ␛[31merror␛[0m
```

This keeps the documentation source readable while still rendering the same ANSI styling.

## Converting ESC To `␛`

Use the method that best fits the task:

- Replace the character in your editor after generating the final static ANSI snippet.
- Use a tiny scripted replace in a build helper if the snippet is generated repeatedly.
- Keep the replacement step after normalization, not before, so ANSI-processing tools still receive real escape bytes.

Example in Python:

```python
from pathlib import Path

snippet = Path("snippet.ansi").read_text(encoding="utf-8")
Path("snippet.rstfrag").write_text(snippet.replace("\x1b", "␛"), encoding="utf-8")
```

## Example End-To-End Embedding

```rst
.. erbsland-ansi::
    :escape-char: ␛

    ␛[32m[tool]␛[0m Build complete
    ␛[36m[tool]␛[0m Output written to ␛[1m_build/html␛[0m
```

## Sanity Checks

Before committing the snippet:

- Verify the source contains `␛`, not invisible ESC bytes.
- Verify the snippet is already static and no longer depends on cursor motion.
- Build HTML and inspect the rendered output.
- Build a non-HTML target if relevant and confirm the plain-text fallback still reads well.
