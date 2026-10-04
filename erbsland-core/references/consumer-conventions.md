# Consumer includes and namespaces

Read this for public-header selection, namespace exceptions, or adapting Core tests into application code.
The ordinary defaults are short public includes and `el::Type`; investigate configuration only when the project
indicates a departure from them. Paths below are relative to the Core dependency, usually `erbsland/core`.

## Public headers

Include every declaration used directly. Prefer `<erbsland/String.hpp>` and `<erbsland/DateTime.hpp>` over their
longer domain paths when flat headers exist. Not every nested API has a flat header; inspect `include/erbsland`
when the needed path is unfamiliar. A domain-qualified include can still expose a flattened type name.

Avoid `all.hpp` for a narrow dependency unless the project deliberately uses an umbrella header. Generated Core
forwarding headers are dependency artifacts to include and inspect, rather than application files to edit.

The dependency contains:

- `include/erbsland`: generated public forwarding headers.
- `src/erbsland`: original declarations and implementation, including aliases and inherited methods.
- `doc/topics` and `demos`: task-oriented explanations and consumer examples.
- `doc/reference`: contracts and API details.
- `test/unittest`: behavioral and edge-case evidence.

## Namespace exceptions

Public include flattening and namespace flattening are separate mechanisms. Selected domains are imported into
`erbsland`, aliased as `el`; domain namespaces remain available, and specialized namespaces can remain explicit.

| Category | Consumer spelling |
| --- | --- |
| Explicit domains | `el::cterm::Terminal`, `el::conf::Parser`, `el::re::RegEx`, `el::block::Rectangle` |
| Specialized text APIs | `el::text::literals`, `el::text::pattern`, `el::text::fuzzy`, `el::text::base_n` |
| Data-format namespaces | `el::json`, `el::bson`, `el::cbor`, `el::xml` |
| Convenience aliases | `el::io` for standard-stream helpers; `el::sys_info` for machine-information queries |

For an explicitly nondefault consumer configuration, inspect target definitions: `ERBSLAND_CORE_DO_NOT_FLATTEN_NS`
keeps domain qualification, `ERBSLAND_NO_SHORT_NAMESPACE` disables `el`, and `ERBSLAND_SHORT_NAMESPACE` selects another
alias. The actual setup is in `src/erbsland/core/Definitions.hpp` and `src/erbsland/core/MakeOneNamespace.hpp`.

Avoid broad `using namespace erbsland` directives as a substitute for public spelling. For `_el` literal scope,
follow [the string conventions](strings.md#literal-scope).

## Adapt discovered examples

Core's implementation and unit tests use internal includes and explicit namespaces. Inspect them for behavior,
then translate to the consumer API: a test's `erbsland::time::DateTime` normally becomes `el::DateTime`, included
through `<erbsland/DateTime.hpp>`. Do not copy test helpers or `impl` types into the application.

For a declaration in `erbsland::cterm`, retain `el::cterm::...`; that domain remains explicit. Core's embedded terminal
API differs from the standalone Color Terminal library, so use the dependency's own documentation and demos.
