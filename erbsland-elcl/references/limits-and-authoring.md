# Limits and Human-Centric Authoring

Use this file when generating new files or normalizing style in existing files.

## Hard Limits

- Max line length: 4000 bytes.
- Max name length: 100 characters.
- Max name-path depth: 10 elements.
- Max include nesting: 5.
- Integer digit limits (64-bit baseline): decimal 19, hex 16, binary 64.

## Authoring Defaults for Readability

- Prefer human-readable section/value names (spaces are allowed).
- Use visual separators (`---[ Section ]---`) sparingly but consistently.
- Keep related keys grouped within one section location.
- Use empty lines between logical groups.
- Add comments for non-obvious constraints or units.
- Prefer explicit units (`ms`, `s`, `MiB`, etc.) where relevant.

## Safe Editing Pattern

When editing existing ELCL:

1. Preserve section/list structure first.
2. Apply minimal localized changes.
3. Re-check for name conflicts and relative-section context.
4. If `@signature` exists, remove/regenerate before finalizing.

