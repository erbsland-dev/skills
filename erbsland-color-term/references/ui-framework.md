# UI Framework

Use this reference when building a structured event-driven TUI with pages, surfaces, layouts, focus handling, actions,
themes, timers, or background work.

The `erbsland::cterm::ui` module is beta in `1.10.0`. Prefer it when the task is a real surface tree, but expect API
drift in later versions.

## When To Prefer `ui::Application`

Use the UI framework instead of a manual `Buffer` loop when the app wants:

- a page and surface tree instead of ad-hoc rectangles everywhere
- built-in focus routing and shared action dispatch
- scheduled UI updates
- background worker threads that report results back to the UI thread
- reusable layouts, themes, surface types, modal pages, and scroll areas

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
- `ui::Action` / `ui::Actions`:
  shared commands with keys, enablement, help, and focus-chain dispatch
- `theme::Theme` / `theme::ThemeBuilder`:
  shared style, block, padding, state, and tag lookup for UI painting
- `ui::Panel`:
  simple container with background support
- `ui::TextBox`:
  single-string surface with built-in sizing and alignment
- `ui::ScrollArea`, `ui::Viewport`, `ui::Sections`, `ui::Buttons`, `ui::Button`, `ui::Choice`:
  current higher-level layout, button, scroll, and modal helpers

The umbrella include `<erbsland/cterm/ui/all.hpp>` re-exports the built-in `layout` and `surface` helpers into `ui`.

## Minimal Shape

```cpp
using namespace erbsland::cterm;
using namespace erbsland::cterm::ui;

auto page = Page::create();
auto root = Stack::create(Orientation::Vertical);
auto title = TextBox::create("Library Status", Alignment::Center);
auto body = Panel::create();

title->editLayoutMetrics().setFixedHeight(1);
body->addSurface(TextBox::create("Everything is running.", Alignment::Center));

root->addSurface(title);
root->addSurface(body);
page->addSurface(root);

auto app = Application{};
app.setTheme(theme::Theme::dark());
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
- create `ui::Action` objects, assign `Keys`, add them to the page or owning surface, and let focus-chain dispatch work
- use built-in `HeaderLine`, `FooterLine`, `ActionHelp`, `ScrollArea`, `Sections`, and `Buttons` before custom surfaces
- use `ThemeBuilder` or surface theme attributes instead of hard-coded colors in paint code

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
- start with `Stack`, `Frame`, `Panel`, `TextBox`, `Button`, `ScrollArea`, and `Sections` before deriving custom surfaces
- derive from `Surface` only when custom paint, layout, or key handling is really needed
- keep layout and paint invalidation explicit so the framework can update only what changed
