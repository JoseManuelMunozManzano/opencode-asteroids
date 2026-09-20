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
- Power-up Velocidad (`SpeedPowerUp`): 15% drop on destroyed asteroid, pickup ttl 8s, effect 5s. `Ship.activateSpeedBoost()` doubles current `vx/vy` and thrust while `speedBoost > 0`; expiry halves `vx/vy`. Re-pickup resets timer without re-doubling. Cleared on `reset()` (death/respawn/level change); `powerUps` cleared in `initGame()`/`nextLevel()`. HUD shows `VELOCIDAD x2` + remaining seconds.
- Power-up Escudo (`ShieldPowerUp`): 10% independent drop on destroyed asteroid, pickup ttl 8s, effect 10s. `Ship.activateShield()` sets numeric `shield` timer (re-pickup resets, no stacking); `Ship.update()` decrements, expiry clears to 0. Cleared on `reset()`. `Ship.draw()` renders ring at `radius + 8` while `shield > 0`; HUD shows `ESCUDO` + remaining seconds. Destruction centralized in `destroyAsteroid()` / `destroyShootingStar()` via `dropPowerUps()` so bullet kills and shield absorbs share points, split, and drops (15% speed + 10% shield). Ship collision with shield destroys hazard with normal reward without consuming timer, filters `asteroids`/`shootingStars`; without shield calls `killShip()`. Invincibility retains priority over shield.
- Estrella fugaz (`ShootingStar`): bonus ocasional, no parte de nivel. Spawner por temporizador dt: primera en 3–6s, siguientes cada 10–18s (`shootingStarTimer`, `spawnShootingStar()`, `resetShootingStarTimer()`). Rápida (~300px/s), caduca en 4s, no se divide ni suelta power-up, no bloquea fin de nivel. Bala la destruye: +250pts + explosión. Colisión con nave mata (respeta invencibilidad). Sigue moviéndose y caduca en estado `dead`; `shootingStars` se limpia en `initGame()`/`nextLevel()`. Dibujo: cabeza + anillo + cola con alpha por `ttl`.
- States: `playing | dead (2s timer) | gameover (Space restarts)`.

## Conventions

- Spanish UI strings (`NIVEL`, `PUNTAJE`, `GAME OVER` overlay). Keep.
- Canvas monochrome white-on-black + orange thrust flame; monospace HUD.
