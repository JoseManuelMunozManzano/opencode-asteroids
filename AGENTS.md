# AGENTS.md — asteroids

Zero-dependency static game. No build, bundler, tests, lint, package manager.

## Run

Open `index.html` directly, or: `npx serve .` → `http://localhost:3000`

## Structure

- `index.html` — 800×600 canvas `#canvas`, loads `game.js`. No modules.
- `game.js` — whole game, `'use strict'`, globals (`ship, bullets, asteroids, particles, score, lives, level, state`). Fixed `W=800, H=600`.
- `favicon.svg` — static only.

## Game logic (`game.js`)

- Loop: `initGame()` + `requestAnimationFrame(loop)`, dt clamped to 0.05s.
- Input via `keys` (held) + `justPressed`/`pressed(code)` (one-shot, consumed on read). Uses `e.code`: `ArrowLeft/Right/Up`, `Space`.
- Toroidal space: all ship/bullet/asteroid positions use `wrap(v, max)`.
- Tuning lives in code: ship `ROT=3.5, THRUST=260, DRAG=0.987`, shoot cooldown 0.2s, bullet `SPEED=520, ttl=1.1`; asteroid `RADII=[0,16,30,50]`, `SPEEDS=[0,85,55,32]`, `POINTS=[0,100,50,20]` indexed by size 1–3.
- Splitting: size 3→2×size2, size 2→2×size1, size 1 dies. Level `n` spawns `3+n` size-3 asteroids (level 1 = 4).
- Collisions: circle `dist()`; ship uses `ship.radius + a.radius * 0.82` (forgiving hitbox — keep). Respawn invincibility 3s with blink; safe spawn radius 130px from center.
- States: `playing | dead (2s timer) | gameover (Space restarts)`.

## Conventions

- README describes power-ups/shooting star — not implemented. Don't treat as spec.
- Spanish UI strings (`NIVEL`, `PUNTAJE`, `GAME OVER` overlay). Keep.
- Canvas monochrome white-on-black + orange thrust flame; monospace HUD.
