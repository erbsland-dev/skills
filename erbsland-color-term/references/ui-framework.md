# UI Framework

Use this reference when building a structured event-driven TUI with surfaces, layouts, focus handling, key bindings,
timers, or background work.

The `erbsland::cterm::ui` module is beta in `1.8.0`. Prefer it when the task is a real surface tree, but expect API
drift in later versions.

## When To Prefer `ui::Application`

Use the UI framework instead of a manual `Buffer` loop when the app wants:

- a page and surface tree instead of ad-hoc rectangles everywhere
- built-in focus routing and key binding dispatch
- scheduled UI updates
- background worker threads that report results back to the UI thread
- reusable layout containers and surface types

Stay with `Buffer` plus `updateScreen()` when the app is mostly a custom canvas or a tightly controlled render loop.

## Core Model

- `ui::Application`:
  owns the event-driven runtime and one terminal
- `ui::Page`:
  root of a visible page tree
- `ui::Surface`:
  base node type for paint, layout, and key handling
- `ui::Layout`:
  container surface with child layout behavior
- `ui::Stack`:
  built-in horizontal or vertical layout
- `ui::Panel`:
  simple container with background support
- `ui::TextBox`:
  single-string surface with built-in sizing and alignment

The umbrella include `<erbsland/cterm/ui/all.hpp>` re-exports the built-in `layout` and `surface` helpers into `ui`.

## Minimal Shape

```cpp
using namespace erbsland::cterm;
using namespace erbsland::cterm::ui;

auto page = Page::create();
auto root = Stack::create(Orientation::Vertical);
auto title = TextBox::create("Library Status", Alignment::Center);
auto body = Panel::create();

body->setBackground(Char{" ", Color{fg::Inherited, bg::Blue}});
body->addChild(TextBox::create("Everything is running.", Alignment::Center));

root->addChild(title);
root->addChild(body);
page->addChild(root);
page->focusTo(body);

auto app = Application{};
app.setMainPage(page);
return app.run();
```

## Geometry, Focus, And Invalidations

Use surface geometry instead of raw buffer math:

- `Geometry` stores minimum, preferred, and maximum size plus size policy
- `DimensionPolicy` and `SizePolicy` drive how layouts distribute space

Rules:

- call `setLayoutOutdated()` when geometry or layout dependencies changed
- call `setPaintOutdated()` when only the rendered appearance changed
- route focus intentionally with `Page::focusTo(...)`
- bind keys close to the owning surface with `keyBindings().bind(...)`

## Timers And Workers

Use the framework's scheduling helpers before inventing your own thread or timer layer:

- `surface->scheduler()` for UI-thread delayed or repeated actions
- `Application::invoke()` to post work back onto the UI thread
- `Application::createEventThread()` for worker-thread jobs
- `ui::StopToken` for cooperative worker shutdown
- `ui::EventDriver` and `ui::EventScheduler` only when embedding or testing below `Application`

Worker threads must not mutate surfaces directly. Post results back to the UI thread instead.

## Good Defaults

- let `ui::Application` own the terminal lifecycle
- start with `Stack`, `Panel`, and `TextBox` before deriving custom surfaces
- derive from `Surface` only when custom paint, layout, or key handling is really needed
- keep layout and paint invalidation explicit so the framework can update only what changed
