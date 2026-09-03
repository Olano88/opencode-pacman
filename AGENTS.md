# AGENTS.md

Vanilla JS/HTML/CSS Pac-Man MVP. No build system, no package manager, no tests, no lint. Verify changes by opening `src/index.html` in a browser and playing.

## Architecture

Scripts are plain globals loaded via `<script>` tags in `src/index.html`. The load order is load-bearing — don't reorder or add files without updating it:

1. `src/js/maze.js` — maze data; exposes `window.MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`
2. `src/js/game.js` — state/rules; exposes `window.createGame`, `window.update`, `window.DIRS`; depends on maze.js globals
3. `src/js/render.js` — canvas drawing; exposes `window.draw`; depends on `DIRS`
4. `src/js/main.js` — loop, keyboard, overlays; calls `createGame`/`update`/`draw` globals

Cross-file communication is via `window` globals, not ES modules.

## Maze data

- Grid is 28 cols x 31 rows, cell (x, y), origin top-left. `MAZE` in maze.js is the pristine copy; each game deep-copies rows into `game.grid` and mutates that (so dots don't destroy the original).
- Cell values: `0` traversable, `1` wall, `2` dot, `3` door. Door (3) blocks pacman but not ghosts — see `isWall` in game.js.
- Tunnel wrap row is `TUNNEL_ROW = 14`; ghost house door spans cols 13–14.
- Movement uses sub-cell float positions snapped by `aligned()`. `PACMAN_SPEED = 0.125` (aligns every 8 frames), `GHOST_SPEED = 0.1` (every 10 frames); keep speeds that align cleanly.

## Spec-driven workflow (this repo is a teaching project for it)

- Features are specified in `specs/NN-slug.md` before code (zero-padded, e.g. `03-levels`). No `specs/` folder exists yet.
- Use the `/spec` skill to write a spec (written as `Draft` by default), then `/spec-impl NN-slug` to implement it.
- `/spec-impl` only proceeds when the spec's status header means "Approved" (e.g. `Approved` / `Aprobado`); it creates/switches to branch `spec-NN-slug`, controlled by `AutoCreateBranch` in `specs/.spec-config.yml` (default `true`).
- The spec workflow never commits automatically; the human commits.

## Conventions

- Code comments and README are in Spanish. Reply in the user's language and match existing specs' language.