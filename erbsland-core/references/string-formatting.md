# Format and assemble text

Read this when choosing `StringFormat`, escaping formatted fields, or assembling string fragments.
Paths below are relative to the Core dependency.

## Reuse parsed formats in repeated work

Constructing `el::StringFormat` parses and validates the pattern at runtime and stores compiled format data.
For a constant pattern used repeatedly, retain a `static const auto` format or an object-owned format. For a runtime
pattern, retain it for that pattern's lifetime; do not freeze the first call's input in a function-local static.
An inline `StringFormat{...}.build(...)` is fine on a one-off path, such as a fatal diagnostic. Avoid reconstructing
an unchanged format inside a hot loop.

Use Core's named specifications for new formatting code, such as `{:text:esc=html}` or
`{:number:hex,width=8,zero-fill}`. Plain `{}` fields remain useful for default formatting. Compact specifications
borrowed from `std::format` are a compatibility subset; do not use that syntax as the default for new applications
or assume the full standard grammar is supported.

With `<erbsland/StringFormat.hpp>` and `<erbsland/String.hpp>`, in a function body with the literal namespace in scope:

```cpp
const auto label = el::String{"<ready>"_el};
static const auto format = el::StringFormat{"<span>{:text:esc=html}</span>"_el};
const auto markup = format.build(label);
```

The result is `<span>&lt;ready&gt;</span>`. Escape the field in the pattern instead of calling
`label.toEscaped(...)` and passing that temporary to a plain `{}` field. Choose escaping for the destination syntax;
escaping does not add surrounding quotes or container markup. When output already uses an `AnyStringBuilder`,
`format.appendTo(builder, ...)` avoids an intermediate formatted string.

Sources: `src/erbsland/text/u8/U8Format.{hpp,cpp,tpp}`, `doc/topics/text_formatting/using_string_format.rst`, and
`doc/topics/text_formatting/format_specifications.rst`.

## Join fragments instead of repeatedly appending slices

For a fixed number of fragments, prefer `el::join(a, b, ...)` over `String::fromJoined({...})`.
Include `<erbsland/StringJoins.hpp>`; `el::join({a, b, ...})` is also supported. With `<erbsland/String.hpp>` and
the literal namespace in scope:

```cpp
const auto name = el::String{"north"_el};
const auto label = el::join("station: "_el, name, " ready"_el);
```

The result is `station: north ready`. The free function concatenates without a separator; every argument is a
fragment. It accepts one or more Core strings, editors, or `_el` literals of the same encoding and returns a
read-only `String`, `U16String`, or `U32String` accordingly. It does not accept mixed encodings, raw character
pointers, standard strings, runtime-width strings, or an empty argument list. A single input shares storage;
multiple inputs use the existing checked `fromJoined` implementation to plan final character storage once.
Inputs remain unchanged, and UTF-8 sensitivity is preserved. `fromJoined` remains available when explicitly
selecting the result string type is useful.

For fragments discovered while processing, collect shared string slices in `StringList`, then call
`fragments.join(separator)`. `StringList::join()` with no separator concatenates them directly. The free `el::join`
does not join runtime collections. Both construction paths compute the final length before filling the result
instead of repeatedly growing an editor for a simple concatenation.

Keep the cost claim precise: the fragment list can allocate/grow, transformations can allocate, and slices retain
the source storage. A single final character allocation does not mean the entire operation allocates only once.
Use `StringEditor` when the task actually needs dependent mutations or an existing mutable buffer, rather than
introducing a list only to replace useful editor operations.

See [strings](strings.md#choose-the-allocation-strategy) for construction examples and storage trade-offs, and
[splitting](string-splitting.md) for splitting, reordering, and joining fields.
Sources: `src/erbsland/text/StringJoins.hpp`, `src/erbsland/text/impl/{StringJoinTools.hpp,StringList.tpp}`,
`doc/reference/text/strings.rst`, and `doc/topics/text_formatting/building_strings.rst`.
