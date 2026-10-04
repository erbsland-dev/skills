# Choose an application design

Read this when starting an application from scratch or changing its control flow. Use the simplest design that fits
its work; these are five ways to organize one `el::Application` lifecycle, not five unrelated base classes.
Paths below are relative to the Core dependency.

## Select the entry point

| Design | Best fit | Entry point |
| --- | --- | --- |
| Function based | Small, single-file utility with a short body | `setInitializeFn()` for setup, `setMainFn()` returning `el::ExitCode` for work |
| Event driven | Server, monitor, or client reacting to callbacks/timers | Application subclass arranges events; keep base `main()` for the main event loop |
| Procedural | Converter, generator, or batch tool with sequential work and shared state | Application subclass overrides `main()` returning `el::ExitCode` |
| Command style | CLI with actions such as `list`, `add`, or `export` | `el::OptionModule` callbacks; keep base `main()` to dispatch the selected command |
| Application parts | Larger program with independently started services and dependencies | Register concrete part classes before `run()`; base `main()` starts parts and enters the event loop |

Choose a function callback for a genuinely small synchronous body. Use a procedural subclass when setup, options,
helpers, or cleanup need separate methods. Choose event-driven control for asynchronous completion; command modules
for action selection; parts when independently managed service lifecycles and dependencies are required.

## Shared lifecycle and dispatch

Create one application object in the process entry point and return `app.run()`. `run()` returns `int`, while application
work callbacks and overridden `main()` return `el::ExitCode`.

The lifecycle initializes the application, prepares registered parts, registers/parses options, runs application work,
and cleans up. Help, version, and parse errors finish before main work. Handled Core exceptions use the framework's
cleanup/reporting boundary; `cleanup()` must not block or throw.

The default `Application::main()` chooses in this order:

1. The selected option module's main callback, if present.
2. The callback registered with `setMainFn()`, if present.
3. Automatic part startup followed by the main event loop.

An overridden `main()` replaces this dispatch. Call `Application::main()`, `partManager()->start()`, or `runEventLoop()`
explicitly when the chosen design needs their behavior. A command callback or `setMainFn()` callback also bypasses
automatic part startup; part registration alone does not start services in those designs.

## Function based: minimal complete application

```cpp
#include <erbsland/Application.hpp>
#include <erbsland/ExitCode.hpp>
#include <erbsland/StandardStreams.hpp>
#include <erbsland/String.hpp>

/// Run a short utility inside the normal application lifecycle.
auto main(const int argc, char *argv[]) -> int {
    using namespace el::text::literals;

    auto app = el::Application{argc, argv};
    app.setMainFn([]() -> el::ExitCode {
        el::io::printLine("ready"_el);
        return el::ExitCode::success();
    });
    return app.run();
}
```

Use `setInitializeFn()` for metadata, options, or terminal setup. Read `optionValues()` in the main callback after
successful parsing. Capture `app` by reference only when it is used; other captured objects must also outlive `run()`.
Move to a subclass when separate lifecycle hooks or persistent callback state make the relationships clearer.
Source: `doc/topics/core/function_based_applications.rst`.

## Event driven: let the base main enter the loop

Derive from `el::Application` and inherit its constructors with `using Application::Application;`.
Use `initialize()` for metadata/services and queue initial work through `events()->invoke()`.
For work depending on parsed arguments, override `parseCommandLine()`, call `Application::parseCommandLine()` first,
and arrange the work after it succeeds. Keep base `main()`; finish through `quit(exitCode)`.

Queued events are not dispatched for help/version output. Avoid starting unmanaged background work during initialization.
If a custom `main()` performs preparation, return `runEventLoop()` to enter managed dispatch explicitly.
Source: `doc/topics/core/event_driven_applications.rst`.

## Procedural: override one synchronous method

Derive from `el::Application`, inherit its constructors, and override
`auto main() -> el::ExitCode`. Keep that method as the top-level sequence, with details in helpers or collaborators.
Use `initialize()` for metadata/services and `registerCommandLineOptions(const el::OptionsPtr &options)` for definitions.
The overridden main reads validated `optionValues()`; it does not manually parse help or errors.

Keep ordinary resources local where possible; use `cleanup() noexcept` for application-lifetime state.
This design intentionally replaces command dispatch, callback dispatch, part startup, and event-loop entry.
Source: `doc/topics/core/procedural_applications.rst`.

## Command style: modules own action selection

Create each action with `el::OptionModule::create("action"_el)`, define its options, and install a callback using
`module->setMainFn()`. The callback takes `el::OptionValuesPtr` and returns `el::ExitCode`.
Add modules through `options->addModule()` in `registerCommandLineOptions()` and retain base `Application::main()`.
The framework selects the action, validates its arguments, and generates root/module help.

For substantial commands, keep each action's options and behavior in an action object owned by the application.
Objects captured by callbacks must survive the complete run. A command requiring application parts must explicitly
start them and account for asynchronous readiness before using them.
Sources: `doc/topics/core/command_style_applications.rst` and `doc/topics/options/option_modules.rst`.

## Application parts: compose services with dependencies

Give each service a small public interface with a stable `partIdentifier()` and implement it using
`el::ApplicationPartWithInterface<Interface>`. Concrete parts provide a static `create()` returning their concrete
shared-pointer type; declare dependencies through static `dependencies()` when needed.
Register implementations with `app.registerPart<ConcretePart>()` before `run()` and retain the default main dispatch.

The application prepares the graph before option registration without starting part threads. Part option hooks run
on the application thread; service lifecycle hooks run on dedicated part threads. Dependencies reach `Running`
before dependent initialization; shutdown proceeds in reverse dependency order.
After preparation, resolve a service through `app.part<Interface>()`. Its methods must be thread-safe or enqueue work
on its event target: startup ordering does not synchronize ordinary calls. Parts are one-shot services.

Keep metadata, root options, terminal setup, and final exit handling in the application; service state belongs in parts.
For custom main/command startup, use manager callbacks or safe waits outside manager-controlled threads.
Sources: `doc/topics/core/applications_from_parts.rst` and `doc/topics/core/application_parts.rst`.

## Further design details

- Comparison and dispatch: `doc/topics/core/choosing_an_application_design.rst`.
- Service/daemon integration when requested: `doc/topics/core/service_lifecycle.rst`.
- Parts managed outside an application: `doc/topics/core/detached_application_parts.rst`.
