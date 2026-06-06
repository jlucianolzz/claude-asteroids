# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

No build step — open `index.html` directly in a browser, or serve locally:

```
npx serve .
```

Then visit `http://localhost:3000`. No package.json, no bundler, no dependencies.

## Architecture

Single-file vanilla JS game loop. Everything lives in two files:

- `index.html` — canvas setup (800×600), minimal CSS, imports `game.js`
- `game.js` — complete game implementation (~424 lines)

**game.js internal structure (in order):**

| Lines | Section |
|-------|---------|
| 8–24 | Keyboard input tracking |
| 26–30 | Utility functions (`wrap`, `dist`, `rand`, `randInt`) |
| 33–58 | `Bullet` class |
| 65–119 | `Asteroid` class — generation, size-based splitting, rotation |
| 122–204 | `Ship` class — thrust/rotation, shooting, 3s respawn invincibility |
| 207–236 | `Particle` class — explosion particles with fade |
| 239–290 | Global game state, spawn logic, death handling |
| 293–351 | Update loop — physics, collision detection (O(n²)), level progression |
| 354–409 | Render loop — HUD, overlays, game objects |
| 412–423 | `requestAnimationFrame` main loop with delta-time |

## Key Constants

```js
// Asteroids — indexed by size (1=small, 2=medium, 3=large)
RADII  = [0, 16, 30, 50]
SPEEDS = [0, 85, 55, 32]
POINTS = [0, 100, 50, 20]

// Ship
ROTATION_SPEED = 3.5    // rad/s
THRUST         = 260    // px/s²
DRAG           = 0.987  // per-frame velocity multiplier

// Bullets
BULLET_SPEED = 520      // px/s
BULLET_TTL   = 1.1      // seconds
```

## Behavioral Notes

- **Toroidal wrapping**: all entities use `wrap(x, w)` to tunnel through edges
- **Collision detection**: circle-circle via `dist()`, runs every frame against all bullets × asteroids and ship × asteroids
- **Asteroid splitting**: large → 2 medium, medium → 2 small, small → destroyed + particles
- **Game states**: `'playing'` → `'dead'` → (3 lives exhausted) → `'gameover'` → Space restarts
- **Delta-time**: all positions and velocities scaled by `dt` (seconds since last frame) for frame-rate independence
