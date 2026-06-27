# Known issue: TUI leaves ghost/overlapping text when the terminal is scrolled

**Status:** issue-first (no confident fix yet) — this branch documents the bug; the fix
PR will follow once the repaint path is isolated and verified.

## Symptom
Scrolling the terminal window during a session leaves **ghost/overlapping text**: fragments
of longer lines bleed into the right side and line-number/content interleave
(e.g. `2d file`, `3ers/mikeChains…`). Reproduced in Ghostty (host `TERM=xterm-256color`);
not a terminal/locale issue (a non-UTF-8 VM locale produces *different*, box-drawing glitches —
see note below).

## Suspected root cause
Incomplete cell-clearing on **scroll repaint** in the ratatui TUI — vacated cells are not
blanked when the viewport scrolls, so stale glyphs from longer prior lines remain.

## Workarounds (confirmed)
- `Ctrl-L` forces a full redraw and clears the ghosts.
- Resizing the window a hair (`SIGWINCH` → full repaint) also clears them.
- Avoid mouse-wheel scrolling the host window.

## Proposed investigation / fix direction
1. Audit the draw path (`crates/tui/src/render.rs`) for partial-area redraws that don't
   `Clear`/blank the vacated region on scroll.
2. Repro harness: a long transcript + programmatic scroll; assert no residual cells.
3. Candidate fix: explicitly clear the scrolled-away region (or force a full-frame redraw on
   scroll) before repainting.

## Related (separate) — non-UTF-8 locale box-drawing
When run in a non-UTF-8 locale (`LC_CTYPE=POSIX`), Unicode box-drawing degrades independently
of this bug; pass `LC_ALL=C.utf8`. Not the same defect.
