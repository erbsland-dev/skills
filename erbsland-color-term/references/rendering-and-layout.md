# Rendering and Layout

Use this reference when composing redraw-based screens with `Buffer`, laying out panels with geometry helpers,
rendering `Text`, or drawing frames, grids, fonts, bitmaps, clipped regions, and composed sub-buffers.

If the task is really an event-driven surface tree, stop here and read `ui-framework.md` instead.

## Manual Redraw Workflow

For non-trivial screen updates, treat the screen as a rendered frame:

1. Keep one persistent `Buffer`.
2. Resize it to the terminal size when needed.
3. Clear it with `fill(...)`.
4. Derive rectangles from the main canvas.
5. Draw text, frames, bitmaps, and sub-buffers into the buffer.
6. Call `terminal.updateScreen(buffer, settings)`.

Pattern:

```cpp
auto terminal = Terminal{Size{96, 28}};
auto buffer = Buffer{terminal.size()};
auto settings = UpdateSettings{};

for (;;) {
    terminal.testScreenSize();
    buffer.resize(terminal.size());
    buffer.fill(Char{" ", Color{fg::Default, bg::Black}});

    // Draw the frame into buffer.

    terminal.updateScreen(buffer, settings);
    terminal.flush();
}
```

## Geometry First

Prefer geometry helpers over manual cursor math:

- `Rectangle` for regions and derived areas
- `Margins` for padding and inset calculations
- `Anchor` and `Alignment` for placement without coordinate juggling
- `Size` and `Position` for explicit dimensions and coordinates

Common patterns:

```cpp
const auto canvas = buffer.rect();
const auto titleRect = canvas.subRectangle(Anchor::TopCenter, Size{0, 6}, Margins{1, 2, 0, 2});
const auto bodyRect = canvas.insetBy(Margins{7, 2, 2, 2});
const auto cells = bodyRect.gridCells(2, 2, 2, 1);
```

## Use The Right Text Type

- Plain string overloads:
  one style, simple output
- `String`:
  per-character colors or attributes, width-aware text handling
- `Text`:
  content that also needs a rectangle, alignment, font, animation, or paragraph settings
- `Char`:
  single-cell precision

When visible width matters, use `String::displayWidth()`, not `String::size()`.

## Resizing And Composition

Prefer explicit APIs over ad-hoc resize logic:

- use `BufferResizeMode` when content-preserving resize behavior matters
- use `BufferView` or `BufferConstRefView` when logical content is larger than the visible panel
- use `BufferDrawOptions` when placement needs cropping, alignment, or color overwrite control
- use `WriteClippedBufferRef` for short-lived subsurface paint helpers that must write through a clipped rectangle

Pattern:

```cpp
auto options = BufferDrawOptions{};
options.setTargetRect(panelRect);
options.setSourceRect(Rectangle{0, topRow, source.width(), panelRect.height()});
options.setOverwriteColors(false);
target.drawBuffer(source, options);
```

## Frames, Fonts, And Bitmaps

Start with the highest-level drawing helper that fits:

- `drawFilledFrame(...)` or `FrameStyle` for most panel borders
- `FrameDrawOptions` for reusable or animated border styles
- `FrameBorder`, `GridLayout`, and `drawGridLayout(...)` for table-like layouts with independently styled grid lines
- `Text` plus `Font::defaultAscii()` for large ASCII headings
- `ColorSequence` and `TextAnimation` for animated text
- `Bitmap` and `drawBitmap(...)` for pixel-style visuals
- `BitmapDrawOptions` when bitmap color or scaling behavior needs control

Use `CharCombinationStyle` only when overlapping frame or block-art composition really needs it.

## Good Defaults

- Keep one persistent `Buffer` instead of recreating it every frame.
- Resize the buffer and redraw the full frame instead of mixing ad-hoc cursor writes into a redraw workflow.
- Use `UpdateSettings` for minimum-size handling and crop feedback instead of inventing custom too-small rendering.
- Put scrollback and streaming text in `CursorBuffer`, then render it into the main screen through a view.
