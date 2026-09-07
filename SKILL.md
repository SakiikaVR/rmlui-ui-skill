---
name: rmlui-ui
description: Design, implement, refactor, or visually debug RmlUi interfaces in C++ applications, especially DAW-style timelines, piano rolls, mixers, and dense desktop layouts. Use when RML/RCSS, RmlUi data binding, fonts, DPI, clipping, scrolling, input handling, or broken layouts are involved.
---

# RmlUi UI

Build RmlUi interfaces whose structure, appearance, state, and interaction remain understandable and independently testable. Preserve the product's chosen visual direction and framework unless the user requests a redesign or migration.

## Read the relevant guidance

- Read [references/layout-and-rendering.md](references/layout-and-rendering.md) before changing layout, fonts, DPI behavior, scrolling, or dynamically generated UI.
- Read [references/verification.md](references/verification.md) before visual debugging and before claiming a UI change is complete.
- In the KituneTone repository, also read [references/kitunetone.md](references/kitunetone.md) before editing its DAW or VST UI.

## Working method

1. Inspect the loaded RML, every linked RCSS file in cascade order, the C++ code that mutates the document, renderer dimensions, and the supplied screenshot. Locate an exact visible label when possible and identify whether RmlUi, host-native drawing, or a plug-in-owned surface renders it before changing RCSS. Do not diagnose from one stylesheet alone.
2. State the layout invariants that must survive resizing: fixed chrome, flexible work area, scroll ownership, minimum usable sizes, overlay coordinate space, and text truncation.
3. Keep responsibilities explicit:
   - RML owns semantic structure and stable IDs/classes.
   - RCSS owns static appearance and layout.
   - a C++ view-model/data model owns observable state.
   - controllers own commands and pointer gestures.
   - renderer/platform code owns framebuffer size, DPI ratio, input translation, and scissoring.
4. Prefer stable elements and targeted property/text/data-model updates. Rebuilding a large subtree during playback, dragging, metering, or text entry is a last resort because it loses hover/focus/capture/scroll state and can flicker.
5. For dynamic collections, give items stable application IDs. Preserve selection, focus, pointer capture, and scroll position across necessary rebuilds.
6. Keep UI work, document mutation, renderer calls, and presentation off the real-time audio thread. A graphics present call may wait for vertical sync even when the document is unchanged. Transfer snapshots or commands across a defined non-blocking boundary and let the audio side defer UI changes instead of waiting for a UI-owned lock.
7. Build, launch the real executable, exercise the affected interaction, and inspect screenshots at the required states and sizes. A successful compile is not visual verification.

## Non-negotiable layout checks

- A column flex chain that contains a scroller has definite outer dimensions and `min-height: 0` on flexible ancestors.
- A row flex chain allows content to shrink with `min-width: 0`; fixed sidebars use explicit `flex-basis`/width.
- Exactly one intended owner scrolls on each axis. Headers, rulers, lanes, playheads, notes, and clips use an explicitly documented shared coordinate space.
- Use axis-specific overflow when only one axis should scroll, and assert that the other axis has no accidental overflow.
- Absolutely positioned content does not reliably establish a scroll extent. Give the scroller a deliberate normal-flow extent element or a definite in-flow canvas size.
- Absolute children have a positioned containing block. If they must be clipped, use `overflow: hidden` with `clip: always` where RmlUi's positioned-element clipping limitation applies.
- Controls have explicit dimensions and styles; RmlUi has no browser default stylesheet to rely on.
- Long Japanese text, empty content, dense content, and narrow windows cannot overlap adjacent controls. Use minimum sizes, ellipsis, scrolling, or a deliberate responsive state.
- Use only properties supported by RmlUi's RCSS property index. RML is strict XML, not forgiving browser HTML.
- Human-scale chrome uses `dp` where DPI scaling is intended. One-pixel separators and application-computed timeline geometry may use `px` only when the coordinate conversion is consistent.

## Completion rule

Do not report a layout as fixed until the verification matrix relevant to the change passes. If the environment cannot launch or capture the application, report the unverified states explicitly and do not infer visual correctness from source inspection.
