# Layout and rendering guidance

Use this reference for RML/RCSS implementation and layout diagnosis.

## RmlUi facts that change design decisions

- RML follows strict XML syntax. Close tags, quote attributes, escape generated text, and keep IDs unique.
- RCSS is based mainly on CSS2 with selected later features. Browser CSS knowledge is useful but not proof that a property or behavior exists; check the RmlUi property index.
- RmlUi supplies no default styles for form controls or scrollbars. Size and style every interactive control used by the product.
- Flex alignment centers an element's box, not the glyph baseline inside that box. For fixed-height buttons, labels, keys, rulers, and value fields, set an intentional `line-height` or center a definite inner text span. Derive it from the content height after borders and vertical padding instead of accumulating visual `padding-top` offsets. Give smaller control variants their own line height.
- RmlUi border shorthands contain width and color, not the browser `solid` keyword. Use forms such as `border: 1dp #445`; `border: 1px solid #445` can be rejected as a whole declaration. When one stylesheet must also render in a browser, add the corresponding `border-style` or side-specific `border-*-style` in a browser-only sheet.
- Normal boxes use the CSS content-box model unless `box-sizing` changes it. Replaced elements such as form controls calculate borders and padding differently, so measure the actual rendered control before aligning it with ordinary boxes.
- RmlUi flexbox has differences from browsers: `order`, `flex-basis: content`, and `visibility: collapse` are unsupported; inline-flex needs a definite width; baseline alignment is approximate; stretched items are not reformatted.
- Definite sizes or numeric `flex` values on expensive top-level flex items reduce repeated formatting.
- An absolutely positioned child may not be clipped solely because it visually crosses an `overflow: hidden` parent. Add `clip: always` when clipping positioned descendants is required.
- `position: relative` selects a containing block but does not replace an explicit layout role. Empty `span`-like rails, canvases, and fill tracks should declare `display: block` plus definite geometry before they own absolute children.
- Do not assume an absolutely positioned strip row or canvas contributes to its parent's scroll width or height. If the scroller contains absolute content, add an intentional normal-flow extent element sized from the same model, or keep the scrolling canvas in normal flow.
- Prefer `overflow-x` and `overflow-y` over `overflow: auto` when only one axis is intended to scroll. A horizontal scrollbar can reduce the available height and trigger an unwanted vertical scrollbar through a percentage-height or minimum-height cycle.
- Repeated cards, rails, and empty geometry elements should declare their layout role explicitly (`display: block`, definite width, and bounded padding). In RmlUi's no-default-style environment, implicit element display assumptions can change internal insets and containment.
- Any element can establish a stacking level with `z-index`. Keep a small documented layer scale instead of escalating arbitrary values.
- Font files must be loaded from C++ before documents using them are loaded. RCSS accepts one font family rather than a comma-separated browser fallback list.
- The host must obtain platform scaling and call `Context::SetDensityIndependentPixelRatio`. The context dimensions, renderer viewport, scissor conversion, and pointer coordinates must describe the same logical/physical geometry.

## Stable application shell

Prefer a flex shell over subtracting several magic heights:

```rcss
#app {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
}

.app-chrome { flex: 0 0 40dp; }
.workspace { display: flex; flex: 1 1 0; height: 0; min-height: 0; }
.sidebar { flex: 0 0 180dp; }
.viewport { flex: 1 1 auto; min-width: 0; min-height: 0; overflow: auto; }
```

Use `calc()` only when the subtracted metric is a genuine shared invariant. A flex shell is less likely to leave gaps or overflow when font metrics or toolbar content changes.

For a RmlUi column shell, `flex: 1 1 auto` may preserve a content-sized basis and let the workspace extend below the client area. When the workspace must consume only the space left after fixed chrome, use a numeric zero basis, `height: 0`, and `min-height: 0`, then audit that the workspace bottom remains inside the root. Keep this rule on the flexible workspace rather than zeroing the height of its fixed children.

## Scroll and overlay coordinate spaces

Before implementing a timeline or piano roll, name these spaces:

1. window/context coordinates;
2. viewport coordinates after fixed headers and sidebars;
3. scroll-content coordinates;
4. musical coordinates such as seconds, beats, ticks, and MIDI pitch.

Centralize conversions between them. Account for viewport origin, scroll offset, zoom, ruler height, keyboard width, and DPI exactly once. Use the same conversion for painting, hit testing, dragging, snapping, auto-scroll, and playhead placement.

Place content that must scroll together under one scroll owner. If a ruler or piano keyboard stays fixed, synchronize its offset from the same scroll state rather than reconstructing it independently. Preserve scroll offsets when regenerating content.

For a horizontal-only absolute strip row, a robust pattern is a positioned viewport with `overflow-x: auto; overflow-y: hidden`, a one-pixel-high normal-flow extent carrying the computed content width, and an absolute row constrained with `top` and `bottom`. Avoid `height: 100%` when scrollbar reservation would make that height feed back into overflow calculation.

Use absolute positioning for objects whose position is data—notes, clips, automation points, grid lines, playheads—not for ordinary toolbars and forms. Their containing canvas must have explicit dimensions, and the containing block must be positioned.

When sharing an RML body with an HTML comparison harness, do not insert XML-serialized self-closing non-void elements directly into HTML. For example, a serialized `<span />` can consume following siblings under HTML parsing rules. Expand such elements to explicit opening and closing tags before insertion, or author a format-neutral template that emits them explicitly.

## Data and event updates

Prefer RmlUi data models for view state and two-way form values:

- create/register/bind the model before loading its document;
- bind structs and arrays deliberately;
- mark C++-changed variables dirty before `Context::Update`;
- use `data-for`, `data-if`, `data-visible`, `data-value`, and `data-event-*` where they make ownership clearer;
- keep commands as named callbacks rather than parsing presentation text.

For rapidly changing playback position or meters, update stable element properties or a compact bound state. Do not call `SetInnerRML` on an enclosing page every frame. Recreating the node that owns text input, drag capture, hover, or scroll is a likely cause of blinking and interrupted gestures.

During a drag, capture the pointer if the backend supports it, retain a drag-session record, update continuously from mouse movement, and release on button-up/cancel/focus loss. Snap musical values after coordinate conversion; do not snap raw screen pixels.

## DPI and crisp text

- Load the intended regular/bold/italic files explicitly before document loading and verify each file contains the required Japanese glyphs.
- Match `font-weight` to a face that was actually loaded. Do not synthesize weight by scaling or drawing twice.
- Use the family name recognized by RmlUi and remember that RCSS does not accept a comma-separated fallback chain.
- On every DPI change or resize, update the context dimensions and density ratio, then ensure the renderer's viewport and scissor rectangles use the correct framebuffer scale.
- Render text at its final scale. Avoid rendering the whole UI to a small texture and enlarging it.
- Prefer integral logical positions for small text and hairlines when animation does not require sub-pixel motion.
- Use `dp` for scalable controls and spacing. Keep timeline pixels-per-beat in one explicit application coordinate system; convert at its boundary rather than mixing `dp` and `px` ad hoc.

## Styling discipline

- Define tokens with RCSS custom properties for background layers, text tiers, accent, warning, border, control heights, sidebar width, and track height.
- Keep base styles before page-specific overrides and know the link order. Remove obsolete override files once their rules can be safely merged; hidden competing cascades make diagnosis unreliable.
- Avoid inline styles for static appearance. Dynamic geometry may be set through targeted properties, but keep the formula and metric constants in one place.
- Use reusable classes for repeated controls. An ID is for identity or a unique layout role, not a substitute for component styling.
- Limit text by available width, not guessed character count. Combine `min-width: 0`, `overflow: hidden`, `white-space: nowrap`, and `text-overflow: ellipsis` when a single-line label must shrink.
- Treat typography metrics as component tokens: font size, line height, control height, border, and vertical padding must form one consistent box. Set a base body line height so nested text does not inherit an engine-dependent `normal` value, then override deliberate compact or multi-line components.

## DAW-specific invariants

- Track header height and lane height are one shared metric.
- Ruler width, every lane canvas width, empty-area width, grid extent, and playhead extent derive from one timeline length/zoom model.
- The playhead spans the intended arrange content, remains above clips/grid, and does not get recreated every tick.
- Piano keyboard rows, grid rows, note Y positions, and velocity panel offsets derive from the same row-height and pitch range.
- In a resizable piano editor, keep the ruler and velocity lane as definite flex children and let the key/note grid consume the remaining height. A short fixed-height canvas inside a taller viewport leaves false empty space below velocity; stretching only the velocity lane makes its editing scale inconsistent. Provide enough synchronized key/grid rows for large windows and clip or scroll the shared pitch area deliberately.
- Mixer strips have a fixed readable width; the strip viewport scrolls horizontally instead of compressing labels, buttons, meters, and faders into overlap.
- Meters update existing bars only. Layout and text nodes remain stable during audio callbacks.
- Native VST editor windows remain independent platform windows. RmlUi may control their launch/routing UI but must not fake or reparent plug-in views unless the host architecture explicitly supports it.

## Authoritative documentation

- RML: https://mikke89.github.io/RmlUiDoc/pages/rml.html
- RCSS and property index: https://mikke89.github.io/RmlUiDoc/pages/rcss.html and https://mikke89.github.io/RmlUiDoc/pages/rcss/property_index.html
- Flexbox differences: https://mikke89.github.io/RmlUiDoc/pages/rcss/flexboxes.html
- Positioning and clipping: https://mikke89.github.io/RmlUiDoc/pages/rcss/visual_formatting_model.html
- Fonts: https://mikke89.github.io/RmlUiDoc/pages/rcss/fonts.html
- High DPI: https://mikke89.github.io/RmlUiDoc/pages/faq.html
- Data binding: https://mikke89.github.io/RmlUiDoc/pages/data_bindings/model.html and https://mikke89.github.io/RmlUiDoc/pages/data_bindings/views_and_controllers.html
- Debugger: https://mikke89.github.io/RmlUiDoc/pages/cpp_manual/debugger.html
