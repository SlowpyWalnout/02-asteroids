# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A vanilla-JavaScript clone of the arcade game *Asteroids*, rendered on an HTML5 canvas. No build step, no dependencies, no framework, no test suite. All game logic lives in a single file, `game.js` (~420 lines). User-facing strings and comments are in Spanish.

## Running

Open `index.html` directly in a browser, or serve the directory:

```bash
npx serve .
```

There is nothing to build, lint, or test. Verify changes by playing in the browser.

## Architecture

`game.js` runs top to bottom under `'use strict'` and is organized into labeled sections (`// ── Name ──`):

- **Input** — global `keys` (held) and `justPressed` maps populated by `keydown`/`keyup`. `pressed(code)` consumes an edge-triggered press (used for shooting and restart); `keys[...]` is polled for continuous actions (rotate, thrust). Arrow keys and Space have their default scroll behavior prevented.
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`, mutates its own state, draws directly to the module-global `ctx`, and marks itself removable via a `dead` flag. All positions are toroidal via `wrap(v, max)`.
- **Game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`). `state` is one of `'playing' | 'dead' | 'gameover'`. `initGame()` / `nextLevel()` / `killShip()` are the transitions.
- **`update(dt)`** — the single simulation step. Early-returns for `gameover` and `dead` states. Collision detection is O(bullets × asteroids) circle checks; asteroid fragments from `split()` are collected into `newAsteroids` and appended *after* the loop to avoid mutating the array mid-iteration. Clearing all asteroids calls `nextLevel()`.
- **`draw()`** — clears the canvas, draws every entity list, then HUD, then the game-over overlay.
- **Main loop** — `requestAnimationFrame`, with `dt` clamped to 50 ms so a backgrounded tab doesn't teleport entities.

### Tuning constants

Gameplay is driven by constants defined near their use: `RADII` / `SPEEDS` / `POINTS` arrays indexed by asteroid `size` (1–3, index 0 unused); `ROT` / `THRUST` / `DRAG` inside `Ship.update`; `SPEED` / `ttl` in `Bullet`; shoot cooldown and `NOSE` offset in `Ship.tryShoot`; spawn count and `SAFE_DIST` in `spawnAsteroids`.

### Adding an entity type

Follow the existing pattern: a class with `update(dt)` / `draw()` / `dead`, a module-level array, and calls wired into `update()` (spawn + collisions) and `draw()`. Canvas transforms use `ctx.save()` / `translate` / `rotate` / `ctx.restore()`.

## Note on the README

`README.md` advertises power-ups and special asteroid types ("estrella fugaz"). These are **not implemented** in `game.js` — treat them as aspirational, not existing features.
