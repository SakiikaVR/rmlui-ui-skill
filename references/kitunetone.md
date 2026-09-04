# KituneTone RmlUi conventions

Read current repository files before acting; this reference records invariants, not frozen implementation details.

## Product constraints

- Keep the main DAW UI on RmlUi and keep the production application free of JUCE unless the user explicitly changes that decision. JUCE may be used only in isolated validation when authorized.
- Keep native VST3 editors in independent host windows. Their GUI/MIDI/parameter navigation belongs to that independent window; the DAW page provides an entry point rather than embedding a fake plug-in editor.
- Preserve Windows Explorer-style native open/save/export dialogs.
- Use the bundled LINE Seed JP faces for the main UI and verify Japanese coverage/fallback behavior before document load.
- Arrange supports audio and MIDI clips, a grid, snap choices, horizontal scrolling, and a full-height draggable playhead.
- Piano roll supports usable pitch/time navigation, note creation/selection/move/resize, velocity editing, snapping, and stable playback display.
- Track rows remain compact but controls must not overlap. Mixer strips scroll instead of collapsing.

## Files to inspect together

- `ui/daw.rml`, every RCSS file it links, and shared assets/fonts;
- `app/rmlui_daw.cpp` for generated markup, state refresh, event routing, pointer math, and runtime initialization;
- the RmlUi backend for resize, framebuffer, scissor, DPI, and input conversion;
- `docs/RMLUI_DAW_SPEC.md` and release/implementation notes for accepted behavior;
- VST process/runtime files when an RmlUi control opens or routes an independent VST window.
- `ui/design_lab/`, `app/rmlui_design_lab.cpp`, and `scripts/test_rmlui_design_lab.ps1` when changing shared shell, arrange, piano, or mixer layout behavior.

Do not assume a rule in the last-linked override is the whole design. Trace cascade conflicts back to their source. Prefer consolidating a stable rule into a clear page stylesheet when doing so is within scope.

## HTML/RmlUi parity lab

KituneTone includes a non-production comparison harness with Arrange, Piano, and Mixer views. It loads the same `design_lab.rml` body and `design_lab.shared.css` in Edge and the actual RmlUi DX11 runtime. Use it as a fast regression gate for shared layout rules; it does not replace interaction testing in `KituneTone.exe`.

The production DAW uses `design_lab.shared.css` as its base layout contract and adds only live-control behavior in `ui/daw.rcss`. Keep the production page skeleton aligned with the lab's `#topbar`, `#transport`, `#workspace`, `#arrange-page`, `#piano-page`, and `#mixer-page` structure. Port real IDs, event attributes, and dynamic containers into that skeleton; never copy the lab's sample clips, notes, meters, or channel names as production state.

For the mixer, keep `#mixer-viewport` horizontal-only, let `#mixer-scroll-extent` contribute the normal-flow width, and position `#mixer-strip-row` with explicit top/bottom bounds. Keep the FX rack outside that scroller and make `.fx-list` the rack's vertical scroll owner.

Build and run the Release matrix from the repository root:

```powershell
cmake --build build --config Release --target kitunetone_rmlui_design_lab --parallel
powershell -ExecutionPolicy Bypass -File scripts/test_rmlui_design_lab.ps1 -Configuration Release
```

The matrix covers Arrange, Piano, and Mixer at 800x500, 1280x720, and 1600x900 in both engines: 18 layout audits and 18 screenshots. Require the script's `DESIGN LAB PASS` result and inspect both generated contact sheets under `build/design_lab_results/`. For exact RmlUi element rectangles, run the lab executable with one view/size plus `--audit --metrics`.

## Refactoring direction

When a requested change reasonably permits refactoring, move toward:

- stable semantic RML instead of reconstructing whole pages from C++;
- page/component RCSS rather than static inline declarations;
- a `DawViewState`/RmlUi data model for visible state;
- small page controllers for arrange, piano, mixer, dialogs, and VST-window commands;
- centralized timeline and piano coordinate conversion;
- incremental meter, playhead, time-text, note, and clip updates.

Do not turn a focused bug fix into an unrequested full rewrite. Preserve working audio, project serialization, VST isolation, and existing user changes while improving the touched boundary.

## KituneTone visual acceptance

In addition to the general verification reference:

- Confirm the menu/transport/snap region remains readable at minimum width.
- Confirm arrange ruler marks line up with all track lanes after horizontal scrolling.
- Confirm the red arrange playhead reaches the intended bottom edge and moves smoothly while dragging.
- Confirm selected MIDI clips remain visible in Arrange and open the corresponding piano content.
- Confirm piano notes, keyboard rows, ruler, velocity bars, and playhead agree after zoom/scroll.
- Confirm the velocity lane stays immediately above the horizontal scrollbar at every tested height, while the synchronized key/note grid—not blank space below velocity—absorbs vertical resizing.
- Confirm time text does not blink during scrubbing or playback.
- Confirm Japanese text is crisp at the active Windows scale.
- Confirm the independent VST window presents GUI, MIDI, and parameter views without destabilizing the host process.
