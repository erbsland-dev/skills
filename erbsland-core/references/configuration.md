# Configuration with Erbsland Configuration Language

Read this when consuming Core's ELCL parser, defining a configuration schema, or reading configuration values.
Use `el::conf` and `el::conf::vr`; these APIs are not interchangeable with the standalone configuration library.
Paths below are relative to the Core dependency. Check its headers for the version used by the application.

## Let the schema own validation and defaults

Use the loading sequence: configure the parser's required extensions, parse the document, validate it against the
application's schema version, then read typed values. Prefer validation rules for constraints the schema can express
instead of reproducing those checks in application code.

Use `el::conf::vr::RulesBuilder` for compact or programmatically generated schemas. For substantial schemas maintained
as ELCL, parse the rules document and call `el::conf::vr::Rules::createFromDocument(ruleDocument)`; consider compiled
resources for application-owned rules. Finalize rules once during setup and reuse them for configuration reloads.

Include meaningful `Title` and `Description` attributes when authoring application settings and sections: give the
setting a readable label and explain its purpose, units, or accepted meaning. They are optional to the validator,
but make the schema useful to diagnostics and configuration editors. Use `configureRoot()` for root metadata.
After validation, metadata is available through a value's `validationRule()->title()` and `description()`.

This fragment uses `<erbsland/conf/Parser.hpp>`, `<erbsland/conf/vr/RulesBuilder.hpp>`, and `<erbsland/String.hpp>`;
the builder header exposes its attribute types. Example:

```cpp
auto configurationSchema() -> el::conf::vr::RulesPtr {
    using namespace el::text::literals;
    using namespace el::conf::vr;
    using namespace el::conf::vr::builder;
    
    auto builder = RulesBuilder{};
    builder.addRule(
        "server"_el, RuleType::Section,
        Title{"Server"_el}, Description{"Settings for the listening server."_el});
    builder.addRule(
        "server.port"_el, RuleType::Integer,
        Title{"Listening port"_el}, Description{"TCP port used for incoming connections."_el},
        Default{8080}, Minimum{1}, Maximum{65535});
    return builder.takeRules();
}

void readConfiguration(const el::Path &configurationPath) {
    const auto document = el::conf::Parser{}.parseFileOrThrow(configurationPath);
    auto rules = configurationSchema();
    rules->validate(document, 1);
    const auto port = document->getIntegerOrThrow("server.port"_el); // 8080, inserted by validation.
}
```

`Default` supplies a missing value in the validated tree; it is not merely documentation or a getter fallback.
Do not manually insert that value or repeat the default in every getter. A supplied value remains subject to the
rule's constraints; an invalid supplied value is not replaced by the default. Choose a default that satisfies the rule.
`Value::isDefaultValue()` identifies inserted values when a configuration summary needs that distinction.

Use `IsOptional{}` when absence should remain absence. A rule cannot combine `IsOptional` with `Default`;
finalization rejects that combination. A child's default does not create a missing parent section.
Model an absent branch explicitly when it is allowed. Validation can mutate the tree and throws
`ConfError` on failure; interpret the document only after successful validation.

### Select constraints before writing custom checks

The following are the complete value-constraint families exposed by `vr::builder::Constraints.hpp` in the inspected
Core API. Names are in `el::conf::vr::builder`. Use this as a discovery map, then check the selected constraint's
accepted rule types and constructor arguments; not every constraint applies to every type.

| Constraint | Use for |
| --- | --- |
| `Minimum`, `Maximum` | Numeric/date bounds, text length, byte length, container counts, or matrix dimensions, according to rule type |
| `Equals` | Equality to a scalar, or an exact container count/matrix size where supported |
| `Multiple` | Numeric divisibility or a multiple of a length/count/dimension |
| `In` | Membership in a fixed set of integer, float, text, or byte values |
| `Chars` | Permitted characters in text; uses VR range definitions such as `"letters"`, `"(a-z)"`, or `"[._-]"` |
| `Starts`, `Ends`, `Contains` | Required text prefixes, suffixes, or contained parts |
| `Matches` | A regular-expression constraint on text |
| `ConfKey` | A reference to a key in an index declared by `KeyIndex` |

For text length constraints, counts are Unicode code points. `Chars` takes VR character-range syntax, so do not
assume an arbitrary `CharSet` pattern or regex has the same meaning. `ConstraintOptions` supplies `isNegated` and
`errorMessage` when a supported constraint needs inversion or a specific diagnostic.

Other builder attributes serve different purposes: `Title`, `Description`, `Default`, `IsOptional`, `IsSecret`,
`CaseSensitive`, `CustomError`, `Dependency`, and `KeyIndex`. `ConfVersion`, `MinimumVersion`, and `MaximumVersion`
select the schema versions in which a rule applies; they are not bounds on the setting's numeric value.
`IsSecret` marks handling policy; code displaying extracted values still needs to respect that marker.

Sources: `doc/topics/conf/validation-rules.rst`, `doc/topics/conf/validated-documents.rst`,
`doc/reference/conf/validation_rules.rst`, and `src/erbsland/conf/vr/builder/{Attributes,Constraints}.hpp`.

## Read values through paths and typed getters

Use a full value path for a known location:

```cpp
const auto host = document->getTextOrThrow("server.connection.host"_el);
```

Prefer this to chaining `value("server")->value("connection")->value("host")->asTextOrThrow()`, or even
`value("server.connection.host")->asTextOrThrow()`. `value()` can return null; typed getters combine lookup and
conversion and report failures with configuration context. Use the corresponding `getIntegerOrThrow`,
`getBooleanOrThrow`, or other typed getter for the expected type.

Paths resolve relative to the receiving node, so a retained section can use shorter paths for several related reads.
Use `valueOrThrow(path)` when the node itself is needed for iteration, metadata, or secret handling; ordinary scalar
reads do not need that intermediate object. Nonthrowing `getText(path, fallback)` and related getters return their
fallback on lookup/conversion failure. Use them only when that behavior is intended, not to hide invalid configuration
or bypass schema defaults.

### Construct a dynamic path as text

For a simple dynamic **text-name** segment, format the path and pass it directly to a getter accepting `NamePathLike`.
Include `<erbsland/StringFormat.hpp>` for this fragment:

```cpp
const auto dynamicName = "api.example"_el;
const auto valuePath = el::StringFormat{"server.\"{:text:esc=config}\""_el}.build(dynamicName);
const auto value = document->getTextOrThrow(valuePath);
```

This addresses the single text name `"api.example"` below `server`. The quotes delimit one text-name segment;
`esc=config` escapes its content but does not add those quotes. Bare interpolation of arbitrary text could turn dots
into path separators or produce invalid syntax. Escape only the dynamic segment, not the whole path.

Match the document's name kind: regular names are normalized, while quoted text names preserve spelling and case.
For a dynamic regular name, validate/normalize it as a regular name before interpolating it unquoted; quoting it
would select a different kind of name. Keep `Name`/`NamePath` objects when manipulating path components, preserving
name kinds, or reusing an already parsed path is the actual task. Avoid building `NameList`/`NamePath` scaffolding
solely to perform a simple textual lookup.

Sources: `src/erbsland/conf/{Value,Name,NamePath}.hpp`, `doc/topics/conf/value-trees.rst`,
and `doc/topics/text_formatting/format_specifications.rst`.

## Activate placeholders through the parser

Before writing custom `${...}` substitution, check the parser's built-in sources and filters. A new parser has
neither registered. Expansion starts when at least one source is registered; filters alone do not activate it.
Configure the parser before parsing (using the same literal namespace and parser/string headers as above):

```cpp
auto parser = el::conf::Parser{};
parser.setPlaceholderVariableSource({{{"deployment"_el, "  production  "_el}}});
parser.addPlaceholderTextFilters();
const auto document = parser.parseTextOrThrow(
    "[server]\n"
    "label: \"${var:deployment|trim|upper}\"\n"_el);
const auto label = document->getTextOrThrow("server.label"_el); // PRODUCTION
```

| Need | Parser registration / behavior |
| --- | --- |
| Application variables, `${var:name}` | `setPlaceholderVariableSource(...)`; calling it again replaces the entire variable map |
| Process environment, `${env:NAME}` | `addPlaceholderEnvironmentSource()`; reads the environment at expansion time |
| Built-in text filters | `addPlaceholderTextFilters()`; registers `safe`, `trim`, `slice`, `remove`, `replace`, `escape`, `if`, `error_if`, `default`, `required`, `lower`, `upper` |
| Application-specific lookup or transformation | `addPlaceholderSource(...)` / `addPlaceholderFilter(...)` with a custom provider |

Enable the sources needed by the application's configuration contract. Register environment sources and filter sets
once per parser; duplicate names are rejected. Filters run left to right. Expansion applies to quoted text values,
including multiline text and text list items, but not text names, `@include`, code, or regex values. Replacement text
is not recursively expanded. With no source registered, placeholder-looking text remains literal.

For a required environment value, `${env:APP_NAME,required|trim|required}` rejects both a missing variable and an
empty result after trimming. The `,required` source flag checks existence; the `required` filter checks empty text.
Without the flag, a missing environment variable yields the literal `undefined`. An unknown `var` name is an error.
The `default` filter substitutes for empty text only; it does not recover either of these missing-source outcomes
and is separate from a validation rule's `Default`.

Sources: `src/erbsland/conf/Parser.hpp`, `doc/topics/conf/placeholders.rst`, and
`doc/topics/text_placeholders/{built_in_sources,built_in_filters}.rst`. Consult these for filter parameters or a custom
provider instead of implementing another substitution engine.
