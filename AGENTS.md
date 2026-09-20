# AGENTS.md — asteroids

Zero-dependency static game. No build, bundler, tests, lint, package manager.

## Run

Open `index.html` directly, or: `npx serve .` → `http://localhost:3000`

## Structure

- `index.html` — 800×600 canvas `#canvas`, loads `game.js`. No modules.
- `game.js` — whole game, `'use strict'`, globals (`ship, bullets, asteroids, particles, powerUps, shootingStars, score, lives, level, state, deadTimer, shootingStarTimer`). Fixed `W=800, H=600`.
- `favicon.svg` — static only.

## Game logic (`game.js`)

- Loop: `initGame()` + `requestAnimationFrame(loop)`, dt clamped to 0.05s.
- Input via `keys` (held) + `justPressed`/`pressed(code)` (one-shot, consumed on read). Uses `e.code`: `ArrowLeft/Right/Up`, `Space`.
- Toroidal space: all ship/bullet/asteroid/shooting-star positions use `wrap(v, max)`.
- Tuning lives in code: ship `ROT=3.5, THRUST=260 (x2 with boost), DRAG=0.987`, shoot cooldown 0.2s, bullet `SPEED=520, ttl=1.1`; asteroid `RADII=[0,16,30,50]`, `SPEEDS=[0,85,55,32]`, `POINTS=[0,100,50,20]` indexed by size 1–3; shooting star `SPEED=300 (±30/40), ttl=4, radius=14, POINTS=250`.
- Splitting: size 3→2×size2, size 2→2×size1, size 1 dies. Level `n` spawns `3+n` size-3 asteroids (level 1 = 4).
- Collisions: circle `dist()`; ship uses `ship.radius + a.radius * 0.82` (forgiving hitbox — keep, also applies to shooting stars). Power-up pickup uses `ship.radius + p.radius`. Respawn invincibility 3s with blink; safe spawn radius 130px from center (150px vs ship for shooting stars).
- Power-ups (`powerUps`): 15% drop on destroyed asteroid, 50/50 `SpeedPowerUp` vs `TripleShotPowerUp`, pickup ttl 8s, radius 12, pickup uses `ship.radius + p.radius`. Both effects last 5s, re-pickup resets timer, cleared on `Ship.reset()` (death/respawn/level change); `powerUps` cleared in `initGame()`/`nextLevel()`.
- Power-up Velocidad (`SpeedPowerUp`): `Ship.activateSpeedBoost()` doubles current `vx/vy` and thrust while `speedBoost > 0`; expiry halves `vx/vy`. Re-pickup resets timer without re-doubling. Draw: circle + `>>`. HUD shows `VELOCIDAD x2` + remaining seconds.
- Power-up Triple Shot (`TripleShotPowerUp`, `Ship.tripleShot`, `activateTripleShot()`): while `tripleShot > 0`, `tryShoot()` spawns 3 simultaneous `Bullet`s from nose (`NOSE=21`), same speed (`SPEED=520`), fan spread: center + `±SPREAD(0.18rad)`. Cooldown stays 0.2s. Draw: circle + 3 dots in fan. HUD shows `TRIPLE SHOT` + remaining seconds, stacked below `VELOCIDAD` when both active. Stacks with speed boost.
- Estrella fugaz (`ShootingStar`): bonus ocasional, no parte de nivel. Spawner por temporizador dt: primera en 3–6s, siguientes cada 10–18s (`shootingStarTimer`, `spawnShootingStar()`, `resetShootingStarTimer()`). Rápida (~300px/s), caduca en 4s, no se divide ni suelta power-up, no bloquea fin de nivel. Bala la destruye: +250pts + explosión. Colisión con nave mata (respeta invencibilidad). Sigue moviéndose y caduca en estado `dead`; `shootingStars` se limpia en `initGame()`/`nextLevel()`. Dibujo: cabeza + anillo + cola con alpha por `ttl`.
- States: `playing | dead (2s timer) | gameover (Space restarts)`.

## Conventions

- Spanish UI strings (`NIVEL`, `PUNTAJE`, `GAME OVER` overlay). Keep.
- Canvas monochrome white-on-black + orange thrust flame; monospace HUD.
