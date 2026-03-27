# Rendering and Layout

Use this reference when rendering screens with `Buffer`, laying out panels with `Rectangle`, styling content with
`Color`, `CharStyle`, and `String`, or adding text, frames, fonts, bitmaps, and composed sub-buffers.

## Contents

- Compose the screen in memory
- Prefer geometry helpers over manual coordinates
- Use the right text type
- Use color and style semantics correctly
- Compose buffers deliberately
- Frames, panels, and decorative drawing
- Fonts, animation, and bitmap rendering
- Useful convenience APIs

## Compose the Screen in Memory

For non-trivial UIs, treat the screen as a rendered frame:

1. Resize the persistent `Buffer` to `terminal.size()`.
2. Clear it with `fill(...)`.
3. Derive rectangles from a main canvas.
4. Draw text, frames, and other elements into the buffer.
5. Call `terminal.updateScreen(buffer, settings)`.

This is the library's intended workflow and scales far better than manual cursor movement.

## Prefer Geometry Helpers Over Manual Coordinates

Use these types first:

- `Rectangle` for canvas areas and derived regions
- `Margins` for padding and inset calculations
- `Anchor` and `Alignment` for placement without cursor math
- `Size` and `Position` for explicit dimensions and coordinates

Common patterns:

```cpp
const auto canvas = Rectangle{0, 0, buffer.size().width(), buffer.size().height()}; // or just `buffer.rect()`
const auto titleRect = canvas.subRectangle(Anchor::TopCenter, Size{0, 6}, Margins{1, 2, 0, 2});
const auto bodyRect = canvas.insetBy(Margins{7, 2, 2, 2});
const auto cells = bodyRect.gridCells(2, 2, 2, 1);
```

Use `gridCells()` for dashboards or menu grids instead of hand-splitting coordinates.

## Use the Right Text Type

- Use plain string overloads when one color/alignment is enough.
- Use `String` when different characters need different colors or attributes.
- Use `Text` when content also needs a rectangle, alignment, font, animation, or paragraph spacing.
- Use `Char` when working at single-cell precision.

Examples:

```cpp
auto footer = String{};
footer.append(
    bg::BrightBlack,
    fg::BrightYellow,
    "[Q]",
    fg::BrightWhite,
    " quit");

auto title = Text{String{"COLOR TERM"}, titleRect, Alignment::Center};
title.setFont(Font::defaultAscii());
title.setColorSequence(ColorSequence{
    {Color{fg::BrightBlue, bg::Black}, 4},
    {Color{fg::BrightCyan, bg::Black}, 3},
    {Color{fg::BrightMagenta, bg::Black}, 2},
    {Color{fg::BrightYellow, bg::Black}, 3},
});
title.setAnimation(TextAnimation::ColorDiagonal);
```

Visible width rules:

- `String::size()` counts stored terminal characters.
- `String::displayWidth()` counts terminal cells.
- Use `displayWidth()` when alignment or truncation depends on what the user actually sees.

## Use Color and Style Semantics Correctly

- Use `Default` to reset a foreground/background component to the terminal default.
- Use `Inherited` to keep the component from what is already below the current layer.
- Use `CharAttributes` for bold, underline, italic, reverse, and related ANSI attributes.
- Use `CharStyle` when color and attributes belong together as a reusable style fragment.
- Use `withOverlay()` / `withBase()` or character recoloring helpers to derive related styles without rebuilding
  everything.

Typical examples:

```cpp
auto base = Color{fg::BrightWhite, bg::Blue};
auto warning = base.overlayWith(Color{fg::BrightYellow, bg::Red});
auto recolored = Char{U'X', fg::Green, bg::Blue}.withColorOverlay(Color{fg::BrightWhite, bg::Inherited});

auto emphasis = CharAttributes{};
emphasis.setBold(true);
emphasis.setUnderline(true);
auto headingStyle = CharStyle{Color{fg::BrightWhite, bg::Blue}, emphasis};
auto mutedHeading = headingStyle.withOverlay(CharStyle{Color{fg::BrightBlack, bg::Inherited}});
```

Use `CharAttributes::reset()` when a later fragment must explicitly clear attributes that were turned on earlier.

## Compose Buffers Deliberately

For large screens, treat sub-buffers as reusable render artifacts.

- Use `drawBuffer(view, rect)` for straightforward placement into a target rectangle.
- Use `BufferDrawOptions` when placement needs source cropping, target alignment, color overwrite control, or character
  combination rules.
- Use `BufferView` / `BufferConstRefView` when the logical content is larger than the visible panel.

Pattern:

```cpp
auto options = BufferDrawOptions{};
options.setTargetRect(panelRect);
options.setSourceRect(Rectangle{0, topRow, source.width(), panelRect.height()});
options.setOverwriteColors(false);
target.drawBuffer(source, options);
```

This is the right tool when a screen mixes pre-rendered content, clipped history panes, or composed character artwork.

## Frames, Panels, and Decorative Drawing

Start simple:

- `drawFilledFrame(...)` for most panels
- `FrameStyle` for built-in box styles
- `drawText(...)` inside inset rectangles for titles and body copy

Reach for the advanced drawing APIs only when needed:

- `FrameDrawOptions` for reusable animated border styles and fill behavior
- `CharCombinationStyle` when intersecting frames must combine cleanly
- `Tile9Style` when a border or fill should stretch from corner/edge/center tiles

## Fonts, Animation, and Bitmap Rendering

Use `Font::defaultAscii()` for large ASCII titles. Because fonts plug into `Text`, alignment and animation still work.

Use `ColorSequence` plus `TextAnimation` for animated titles or highlights.

Use `Bitmap` and `drawBitmap(...)` when you need:

- Procedural icons
- Pixel masks
- Half-block or full-block rendering
- Line-art or circuit-style visuals driven by a bitmap

Use `BitmapDrawOptions` to control color sequences, scale mode, and neighbor-aware rendering.

## Useful Convenience APIs

- `Buffer::fromLinesInString(...)` for help panes or static text blocks
- `ReadableBuffer::countDifferencesTo(...)` for tests or frame analysis
- `ReadableBuffer::toMask(...)` when rendered cells need to become a bitmap mask
- `String::fromLines(...)` when a styled multi-line fragment is easier to build before rendering
