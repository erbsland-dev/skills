# Declare and validate command-line options

Read this when adding or reviewing options, choices, help text, or command-line validation in a Core consumer.
Confirm unfamiliar signatures against the dependency's headers. Paths below are relative to the Core root.

## Declare explicit, documented options

- Define each internal option identifier once as a named constant. Reuse it in the option declaration and every
  value lookup or diagnostic. Keep it independent of user-facing spellings such as `-o` and `--output-file`, so
  changing an alias does not require changing parsing code.
- Declare options explicitly with their aliases, type, constraints, and a meaningful value name when applicable.
  Do not generate `--<name>` and bare identifiers in a loop merely to shorten declarations: that does not remove
  duplicated lookup strings and tends to omit option-specific documentation.
- Give every option a help title and description. Explain its purpose in the title and its behavior, constraints,
  or interaction with configuration in the description, so generated help is useful without a separate manual.
- Use `setHelpTitle()`, `setHelpDescription()`, and other field-specific setters when composing help. `setHelp()`
  replaces the whole help object, including when passed only a title, and can erase previously set fields.
  Reserve it for intentional replacement with a complete help definition.
- For a finite set of accepted values, use `OptionChoices` and the `OptionChoiceEditor` returned by `addChoice()`
  to document each choice. See the example below; the editor's actual name is singular `OptionChoiceEditor`.

## Keep definitions and parsing with their owner

A component can return its own `OptionSet` and parse its values into typed member state. Apply the same principles
directly in a small `el::Application` subclass, or compose sets into an `OptionModule` for a larger command.
Do not introduce a component solely to match this example.

Use this sequence: declare options and constraints, parse and validate values into structured state, then run the
application's work. Perform application-specific validation before generating output or starting other side effects.
In an `el::Application` subclass, override `registerCommandLineOptions(const el::OptionsPtr &options)` to register
definitions and `handleCommandLineOptions(const el::OptionValuesPtr &values)` to read, convert, and validate parsed
values into application state. The latter runs only after successful parsing, before application-part option handling
and `main()`; help, version output, module overview, and parse errors skip it.
Do not override `parseCommandLine()` merely to declare options or consume their values. Override it only when the
parsing behavior itself must change. Some Core demos still use that method for ordinary setup; follow these lifecycle
hooks instead. `ApplicationPart::parseCommandLine(values)` is a separate part hook and does receive parsed values.
For function or command callbacks, validate at the start before doing work. See [application designs](applications.md).

Report invalid option values with `el::OptionError`: supply a title explaining what failed, a description explaining
why (and a remedy when useful), and `optionValues->value(identifier)` to preserve the offending option's context.

The excerpt below assumes the relevant public per-type headers and `el::text::literals` are in scope.

```cpp
/// Own the generator's command-line options and validated settings.
class MyComponent {
    constexpr static auto cOutputFileOption = "output-file"_el;
    constexpr static auto cCheckOption = "check"_el;

public: // command-line options
    /// Returns the option set for this component.
    [[nodiscard]] auto getOptionSet() -> el::OptionSetPtr;
    /// Parses the options from the given option values.
    /// @param optionValues Values produced by successful command-line parsing.
    /// @throws el::OptionError If the requested output file already exists.
    void parseOptions(const el::OptionValuesPtr &optionValues);

private:
    el::Path _outputFile;    ///< The C++ output file path.
    bool _checkOnly = false; ///< Flag if this is a check-only run (skips the actual generation).
};

auto MyComponent::getOptionSet() -> el::OptionSetPtr {
    auto optionSet = el::OptionSet::create();
    optionSet->addOption({"-o"_el, "--output-file"_el, "--output"_el, cOutputFileOption})
        .setType(el::OptionType::Text)
        .setValueName("file"_el)
        .setHelpTitle("Choose an output file"_el)
        .setHelpDescription("Override the configured output path. The selected file must not already exist."_el);
    optionSet->addOption({"-c"_el, "--check"_el, cCheckOption})
        .setHelpTitle("Check generated output"_el)
        .setHelpDescription("Fail if output differs, without writing files."_el);
    return optionSet;
}

void MyComponent::parseOptions(const el::OptionValuesPtr &optionValues) {
    if (auto text = optionValues->getText(cOutputFileOption); !text.isEmpty()) {
        _outputFile = el::Path{text};
        if (_outputFile.info().exists()) {
            throw el::OptionError(
                "Output file already exists"_el,
                "Please remove the existing file before generating the output."_el,
                optionValues->value(cOutputFileOption));
        }
    }
    _checkOnly = optionValues->getFlag(cCheckOption);
}
```

The output-file policy is illustrative; choose validation appropriate to the application's actual behavior.

## Document accepted choices

Build an `el::OptionChoices` collection and use each returned `el::OptionChoiceEditor` to set the choice's title and
description. Attach the collection through `OptionEditor::setChoices()`, which also sets the option type to
`OptionType::Choice`. This lets Core validate accepted values and generate choice-specific help, instead of leaving
the application to maintain a separate list of strings and manual validation.

For example, inside an option-registration method (`cModeOption` is an owner-level identifier constant reused
when reading the selected value):

```cpp
auto choices = el::OptionChoices::create();
choices->addChoice("append"_el)
    .setHelpTitle("Append data to the file"_el)
    .setHelpDescription("Keep existing content and write new data at the end of the file."_el);
choices->addChoice("overwrite"_el)
    .setHelpTitle("Overwrite the file"_el)
    .setHelpDescription("Replace existing file content with the newly generated data."_el);
options->addOption({"-m"_el, "--mode"_el, cModeOption})
    .setChoices(choices)
    .setValueName("mode"_el)
    .setHelpTitle("Choose how to write output"_el)
    .setHelpDescription("Select whether new data is appended to the output file or replaces its content."_el);
```

`OptionEditor::addChoice()` is a shorthand for adding bare choice values; use the collection/editor pattern above
when choices need their own help. Do not use bare choices merely to save lines at the expense of useful documentation.

## Source patterns

- Option definitions and choices: `doc/topics/options/options.rst` and `src/erbsland/options/OptionChoices.hpp`.
- Help setters: `src/erbsland/options/OptionEditor.cpp` and `OptionChoiceEditor.cpp`.
- Validation diagnostics: `src/erbsland/options/OptionError.hpp`; see also [error handling](errors.md).
- Application hooks: `src/erbsland/core/Application.hpp` and `doc/reference/core/application_framework.rst`.
