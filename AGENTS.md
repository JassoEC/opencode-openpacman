# AGENTS.md

Pac-Man clone in vanilla JS/HTML/CSS (no framework, no build). The whole project — README, code comments, specs — is written in **Spanish**: write code comments and spec documents in Spanish too.

## Running and verifying

- No package.json, bundler, tests, or linter. Serve is unnecessary: open `src/index.html` directly in a browser and test in the dev console (`requestAnimationFrame` loop, no errors = basic sanity check).
- Thread the canvas: it is 560×620 px = 28 cols × 31 rows at `TILE = 20` (in `src/js/render.js`).

## JS architecture (no modules — don't add `import`/`export`)

Scripts are plain globals loaded via `<script>` tags in `src/index.html`; **load order is load-bearing**:

1. `maze.js` → exports `window.MAZE`, `window.TUNNEL_ROW`, `window.PACMAN_START`, `window.GHOST_STARTS`
2. `game.js` → reads those globals at load; exports `window.createGame`, `window.update`, `window.DIRS`
3. `render.js` → renders `game.grid`; exports `window.draw` (takes `game` + `frame`)
4. `main.js` → entry point: `createGame()`, game loop, keyboard, overlays

If you add a JS file, you must add a `<script>` tag in `src/index.html` in dependency order, and expose its API on `window`. Missing that is an easy slip-up.

- `maze.js` encodes the 28×31 maze as strings: `#`=wall(1), `.`=dot(2), `-`=pen door(3), space=open. `createGame()` deep-copies `MAZE` into `game.grid` so eating dots doesn't mutate the pristine `MAZE`; new games call `createGame()` again.
- Collision/tile rules differ by actor: pipes in `isWall`/`canMove` (game.js): pacman is blocked by walls and pen doors (3); ghosts only by walls, so they pass through the door. Tunnel row is `TUNNEL_ROW` (14); leaving the sides there wraps via `wrapTunnel`.
- Ghost AI in `decideGhost` (`game.js`): `kind: 'hunter'` (greedy Manhattan chase) vs `kind: 'random'`; the 180° reverse is only allowed as a dead-end fallback.

## Code style

JS is formatted with spaces inside parens: `func( x, y )`, `( a === b )`, `grid[ y ][ x ]`. This differs from default formatters — match it or it will look wrong against the rest of the file.

## Spec-Driven Development workflow (the point of this repo)

- Feature work starts with the `/spec` skill (in `.agents/skills/spec/`, from `klerith/fernando-skills`; see `skills-lock.json`) → writes `specs/NN-slug.md` in state `Draft`, section order per `.agents/skills/spec/template.md`. `specs/` does not exist yet → next spec is `01-`.
- States are Spanish: `Borrador` → `Aprobado` (user flips this by hand) → `Implementado`. `/spec-impl` refuses anything not meaning "Approved".
- `/spec-impl` (`.agents/skills/spec-impl/SKILL.md`) creates branch `spec-NN-slug`, then implements the plan one step at a time, pausing after each step for diff review. Never commit unless asked.
- **Gotcha:** `/spec-impl` relies on git (`git checkout -b`, dirty-tree checks), but this directory is **not a git repository** (`git init` is needed before that skill can work).
- Keep specs consistent with existing conventions: single-sentence objective, boolean acceptance criteria, explicit "out of scope", decisions section.