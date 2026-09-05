# Visual and interaction verification

Use the smallest matrix that covers the changed invariant, then broaden it when shared shell, fonts, renderer scaling, or reusable controls changed.

## Before editing

1. Reproduce the problem in the real executable.
2. Record window client size, display scale, active page, project/content state, input sequence, owning process/window, and rendering path (RmlUi, native host drawing, or plug-in owned).
3. Capture the whole window. Treat a user-provided crop as evidence of the defect, not as the only area that may be affected. Search an exact visible label before assuming which renderer produced it.
4. Inspect RmlUi log output when RmlUi owns the surface. In a development build, enable the Debugger plugin and use element outlines/element info when available. For native host drawing, inspect the selected font, quality, DPI source, and GDI object lifetime instead.

For a browser/RmlUi parity harness, keep the semantic element tree and main stylesheet single-sourced. Browser-only compatibility rules are acceptable for syntax RmlUi intentionally differs on, but do not maintain two independent mock-ups and call their visual similarity a parity test.

## Required states

For a shared DAW layout change, inspect at least:

- Arrange, piano roll, mixer, and any modified modal/popup;
- empty project and representative populated project;
- no selection and selected/active/hovered controls;
- resting and combined interaction states whose geometry differs, such as a pressed black key or selected resize handle;
- playback stopped and playing;
- timeline/playhead at the start and after horizontal scrolling;
- short ASCII labels and long Japanese labels;
- minimum supported window, a common laptop size, and a large desktop size;
- 100% and one non-100% Windows display scale when fonts, DPI, renderer, or top-level dimensions changed.

Recommended broad matrix when practical: 1280x720, 1600x900, and 1920x1080 at 100%; one of them at 125% or 150%. Add 200% for DPI-specific fixes.

## Interaction checks

- Scrub the ruler by holding the mouse and moving both slowly and quickly. The playhead and time display must move continuously without text blinking.
- Drag notes/clips across grid boundaries, resize them, scroll while editing if supported, and verify snapping uses musical coordinates.
- After every drag or scrub, release inside and outside the window, then send an additional mouse move. Geometry must remain unchanged after release, and any auditioned note must have received Note Off.
- For controls with semantic cursors, verify the native cursor at four points: hover on the handle, drag after leaving the handle, immediately after release over a non-handle area, and after one additional mouse move. On Windows, `GetCursorInfo` compared with `LoadCursor` handles can distinguish `IDC_SIZEWE`, `IDC_SIZENS`, `IDC_SIZEALL`, and `IDC_ARROW` without relying on screenshots.
- Exercise modifier-wheel zoom over both the timeline canvas and the visible percentage label. Confirm one wheel step changes the percentage once, preserves the intended cursor-centered anchor, and plain wheel retains its scrolling behavior.
- Type into every changed input; focus and caret must survive unrelated periodic updates.
- Scroll to both extremes and confirm fixed headers, ruler labels, keys, lanes, notes, grid, and overlays remain aligned.
- Open and close menus, dialogs, and independent VST windows. Check z-order, focus return, and clipping at window edges.
- For custom-painted native windows, exercise every tab that uses host drawing as well as the plug-in-owned GUI. Verify ASCII, Japanese, and numeric labels use the intended explicit font rather than a stock DC font.
- Resize custom-painted native windows to their declared minimum and inspect them at a non-100% DPI scale. Parameter names, values, sliders, tab controls, and topmost controls must use the same scaled geometry for rendering and hit testing.
- For staged routing controls, change the draft port/channel without applying it, then test Connect, Clear, persistence, and process-boundary IPC. The visible applied status must agree with the audio/MIDI routing model.
- Extend a loop clip beyond one source-pattern length and verify the repeated preview and emitted Note On/Off sequence. Shorten through the final note and verify the clip boundary produces Note Off.
- Populate enough tracks/strips/parameters to force each intended scrollbar. Confirm controls do not compress below usable size.
- For every one-axis scroller, force overflow on the intended axis and assert that the cross axis does not gain a scrollbar.

## Screenshot review

Inspect the whole image at native resolution, then zoom into:

- right and bottom window edges for overflow/gaps;
- the gap between bottom-anchored auxiliary lanes (such as velocity) and the horizontal scrollbar or viewport edge;
- boundaries between fixed and scrolling areas;
- Japanese glyph edges and text baselines;
- native companion-window labels at original resolution, especially dynamic parameter names that do not exist as source literals;
- mixed toolbar rows where plain labels, buttons, numeric fields, and icons must share one visual center;
- clipped/ellipsized labels;
- slider thumbs, meter bars, one-pixel grid lines, and the playhead;
- resize-handle hit areas and loop-boundary markers at native resolution;
- popup/modal edges and overlays;
- the first, middle, and last visible track/strip.

Do not stop at local parent-child containment. A meter can fit its strip while the strip, page, or flexible workspace still extends beyond the window. Audit the containment chain from fragile absolute controls through their component, page/viewport, workspace, and root client area. Also verify intended scroll canvases have real overflow; a missing definite canvas size can make a broken layout look deceptively tidy.

Audit internal insets as well as outer containment. A card can remain inside its rack while its contents begin at the rack border because padding was lost; compare the first content child's left/top offset with the component's declared padding.

For bottom-anchored lanes, assert a maximum bottom gap as well as containment. Allow only the known horizontal scrollbar thickness; otherwise a fixed-height inner canvas can pass containment while leaving a large unused region below the lane.

When browser and RmlUi images are available, create same-size contact sheets for the tested matrix. Use geometry and containment assertions for automation, then inspect the sheets for missing borders, unintended full-screen fills, off-screen lower controls, and engine-specific text expansion. Do not require pixel identity because browser and FreeType/RmlUi font rasterization can legitimately differ.

Box-geometry audits cannot prove that text is optically centered: a button and its text node can both fit while the line box remains top-aligned. After changing font size, control height, padding, or line height, inspect native-resolution crops of representative toolbars, compact track controls, piano keys/notes, mixer strips, and value labels in the real RmlUi renderer.

Compare against a known-good reference when one exists, but accept deliberate design changes. A screenshot is insufficient for drag smoothness, flicker, focus retention, audio-thread safety, or VST process behavior; test those dynamically.

## Completion evidence

Report:

- executable/configuration tested;
- window sizes and DPI scales inspected;
- interactions exercised;
- automated tests/build result;
- any state that could not be visually or interactively verified.

Treat a successful audit as necessary but not sufficient until its assertions cover the observed failure mode. When visual review finds a defect that passed, add a structural invariant (for example, root containment or absolute-child containment) before fixing it, and rerun the whole affected matrix.

Do not use phrases such as “fully fixed” or “no layout issues” if only one page, one crop, or one resolution was checked.
