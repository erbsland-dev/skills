# Error handling and exceptions

Read this when choosing error contracts, validating input, catching failures, or reporting application errors.
Paths below are relative to the Core dependency; its current headers take precedence over older examples.

## Choose the contract

| Situation | Use |
| --- | --- |
| Expected outcome that the immediate caller can act on | `el::Result`, or a derived result with a few actionable states |
| The outcome also carries typed data | `el::ResultWithData<Data, Status>`; `Status` defaults to `el::Result` |
| An operation cannot fulfil its contract; context must cross layers | A runtime exception |
| An application failure needs a user-facing report and graceful exit | `el::ApplicationError` |
| A violated programming contract or broken invariant | A logic exception; do not convert it into an ordinary failure |

Prefer named results over `bool` for operation success/failure. Keep `bool` for predicates such as `exists()`.
Where both forms exist, Core commonly names the throwing variant `...OrThrow`; check the actual API contract.
Returning a result does not itself make a function `noexcept`.

## Choose the exception

Never throw `el::Exception` directly: the neutral base does not distinguish runtime failures from logic errors.

| Type | Role in consumer code |
| --- | --- |
| `el::RuntimeError` | Base for runtime failures; catch at a boundary that can handle all such failures |
| `el::ApplicationError` | Default for application-defined runtime failures; derive from it when useful |
| `el::ParseError` | Invalid syntax in a parser, with a position in the parsed text |
| `el::conf::ConfError` | Configuration validation with the offending configuration value and its source context |
| `el::OptionError` | Command-line validation with option context |
| `el::OverflowError` | Arithmetic overflow reported by Core |
| `el::LogicError` | Developer fault or broken invariant; fatal |
| `el::OutOfRangeError` | Caller accesses outside the API's valid bounds; a logic error |
| `el::ParameterError` | Caller violates a parameter contract; a logic error |

Malformed user input is a runtime failure, not a programming-contract violation. Use `ParseError` inside a parser;
use `ConfError` or `OptionError` when validating configuration or command-line values so source context survives.
Do not borrow unrelated Core domain errors for application failures. Catch errors such as `PathError` from their
own APIs and translate them when application context helps.

Include public per-type headers, for example `<erbsland/ApplicationError.hpp>` and `<erbsland/Result.hpp>`.
Configuration keeps its explicit namespace and header: `<erbsland/conf/ConfError.hpp>`.

## Catch only where there is a useful decision

- Catch by `const` reference, most specific first. Keep the `try` block close to the operation being handled.
- Recover only from the expected condition. A missing optional file can mean “not present”; a permission or I/O
  failure must not silently become the same answer. Inspect typed error context, not message text.
- Use `throw;` to propagate the original exception. Do not catch merely to log and rethrow at every layer.
- Translate technical failures into `ApplicationError` at a meaningful application boundary, preserving the cause
  with `std::current_exception()`. Do not replace it with a concatenated `what()` string.
- Let existing `ApplicationError`, `ConfError`, and `OptionError` reports propagate when already sufficient.
  If a broad `RuntimeError` handler would wrap them again, narrow the handler or rethrow these types first.
  An extra wrapper is useful only when it adds meaningful operation context.

Use `catch (const el::RuntimeError &)` when a boundary genuinely handles all Core runtime failures.
Do not use `catch (...)`, `catch (const el::Exception &)`, or `catch (const std::exception &)` as a recovery shortcut:
they also catch programming errors. At foreign-library boundaries, catch the documented recoverable types explicitly.

Catch-all handlers are appropriate only at boundaries that cannot propagate exceptions, such as destructors or
thread entry points, or for cleanup followed by rethrow. They must preserve/transport the failure or follow an
explicit fatal policy, not turn a logic error into success or a recoverable result. Keep cleanup non-throwing;
letting a fatal error terminate is safer than continuing after a broken invariant.

Use `noexcept` when the operation’s contract does not allow exceptions to escape. Termination on a logic error can
be intentional; it does not by itself make `noexcept` inappropriate.
Leave operations throwing when callers need to recover from runtime failures or preserve them for diagnostic reporting.

## Build a structured application report

Give a user-facing error a title explaining **what** failed and a description explaining **why**, with a useful
remedy when known. Put source names, paths, and locations in `el::ApplicationErrorContext`, rather than embedding
technical details in prose. Set `setExitCode()` when the default failure code is insufficient.
Source positions use `setCodeLocation()` with an `el::CodeLocation` containing zero-based `el::LineIndex` and
`el::ColumnIndex` values.

For example, inside a file-reading operation (`path` is an `el::Path`):

```cpp
try {
    return path.content().readTextOrThrow();
} catch (const el::PathError &) {
    auto context = el::ApplicationErrorContext{}
                       .setTitle("Failed to Read Input"_el)
                       .setDescription("The input file could not be read."_el)
                       .setSourcePath(path.toString());
    throw el::ApplicationError{std::move(context), std::current_exception()};
}
```

Let `el::Application::run()` perform final cleanup, diagnostic rendering including causes, and exit-code selection.
The current implementation reports `RuntimeError`; it cleans up and rethrows logic and other exceptions so they
remain fatal. Do not add a top-level recovery handler that defeats this distinction.

For a custom reporting boundary, use `el::DiagnosticHelper` to build the complete diagnostic with causes.
`reason()` is the supplied reason; `toString()` is a compact description; `what()` is the standard-library
compatibility interface. `diagnostic()` alone does not include the cause chain.

If recurring domain details exceed the standard context fields, derive application error/context types and
provide a matching diagnostic through `diagnostic()`. Keep the diagnostic semantic and renderer-neutral; a new
exception does not by itself require a custom terminal renderer. See `doc/topics/err/writing_custom_exceptions.rst`
and `doc/topics/err/diagnostics.rst` for this less common extension.

## Return and inspect named results

Include `<erbsland/Result.hpp>`. Return `el::Result::Success` or `el::Result::Failure` from an operation returning
`el::Result`, and make the caller's decision explicit:

```cpp
if (isSuccessful(myFunction())) {
    // Continue after success.
}

if (auto result = myFunction(); isFailure(result)) {
    // Handle failure; result is available for an exact-state comparison.
}
```

These are alternative call-site forms. Call the operation once when inspecting one outcome.
The unqualified functions `isSuccessful(result)` and `isFailure(result)` are found by argument-dependent lookup;
member forms such as `result.isSuccessful()` work too. Do not assume a Boolean conversion.

For multiple actionable states, derive from `el::Result`, expose constructors with `using Result::Result;`, and
introduce named constants using the inherited `Value::success<N>()` and `Value::failure<N>()` factories.
Use distinct numbers within each group, rather than raw numeric encodings. Group tests accept every success or
failure state; compare a named constant only when the exact state changes the caller's action. Avoid testing only
`result == el::Result::Success` if derived success states should also count. See
`doc/topics/err/reporting_with_result.rst` for the complete custom-type pattern.

## Carry data with the result

Include `<erbsland/ResultWithData.hpp>`. Construct `el::ResultWithData<Data, Status>` with **both** status and data.
For example, these are alternative return statements in a function returning `el::ResultWithData<el::String>`:

```cpp
return {el::Result::Success, "ready"_el};
return {el::Result::Failure, el::String{}};
```

Document what the payload means for each outcome; in this example the failure string is unused. At the call site:

```cpp
if (auto result = myFunctionWithData(); isSuccessful(result)) {
    auto text = result.takeData();
    // Consume text.
}
```

`ResultWithData` derives from its status type, so it supports the same group tests. `status()` returns the status
by value; `data()` returns a reference; `takeData()` moves the payload out. After taking it, do not assume the
payload retains its old contents. Both successful and failed results contain data: this is not an optional or
`expected`-style value/error union, and `data()` does not check success. Inspect status before using a payload
whose contract makes it valid only on success. A custom `Status` must derive from `el::Result`.

Keep expected failures local and actionable. Do not catch arbitrary exceptions and reduce them to `Failure`:
use exceptions when source locations, nested causes, or rich user-facing diagnostics must survive.

## Source patterns

- Consumer examples: `tool/src/tool/frame/ProjectRoot.cpp` (selective recovery and rethrow),
  `ProjectFiles.cpp` (cause-preserving translation), and `ToolApplication.cpp` (configuration versus logic errors).
- Exception handling: `doc/topics/err/handling_exceptions.rst`; confirm boundary behavior in
  `src/erbsland/core/Application.cpp` when documentation differs.
- Result contracts: `src/erbsland/util/Result.hpp` and `ResultWithData.hpp`.
