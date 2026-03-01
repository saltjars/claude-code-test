# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

**GitHub:** https://github.com/saltjars/claude-code-test
**Stack:** Zero-dependency single-file HTML5 games. No build step, no package manager, no framework.

## Running the Games

Open any `.html` file directly in a browser:
```bash
start tictactoe.html
start shooter.html
```

There are no tests, no linters, and no build commands. Development cycle is: edit → save → refresh browser.

## Git Workflow

GitHub CLI is installed but **not on the bash PATH**. Always use the full path:
```bash
"/c/Program Files/GitHub CLI/gh.exe" <command>
```

Commit and push after every meaningful change:
```bash
git add <specific-files>        # never use git add -A or git add .
git commit -m "..."
git push
```

## Design System (shared across all games)

All games use the same visual language — enforce this in any new work:

| Role | Value |
|------|-------|
| Background | `#0a0a1a` (dark navy) |
| Accent red | `#e94560` |
| Accent cyan | `#a8dadc` |
| Panel/card bg | `#16213e` |
| Font | `monospace` (canvas games) / `'Segoe UI'` (DOM games) |

Art is always procedural — no image files. Canvas games use `fillRect` pixel grids at `PX = 2` scale.

## Game Architectures

### tictactoe.html (~210 lines)
Pure DOM game. No canvas. State lives in three variables (`board[]`, `current`, `over`) and a `scores` object. `checkWinner()` iterates the 8 hardcoded win-line arrays. Score persists across rounds in-memory only (no localStorage).

### shooter.html (~1150 lines)
HTML5 Canvas game. Script is organized into 11 numbered sections in dependency order:

1. **Constants** — `W=800`, `H=600`, `PX=2`, color palette object `C`
2. **Input** — singleton; tracks keys + mouse; `getMovement()` returns normalized dx/dy
3. **Particle / ParticleSystem** — physics particles for explosions, muzzle flash, hit sparks
4. **Bullet / BulletPool** — fixed pool of 60 `Bullet` objects (avoids GC churn)
5. **ENEMY_TYPES** — data object: color, size, speed, hp, score, shoot interval, keep-distance
6. **Pixel sprite functions** — one function per entity (`sprGrunt`, `sprTank`, etc.), returns 2-D color array; `drawSprite(ctx, spr, ox, oy)` renders it centered at the current transform origin
7. **Enemy** — moves toward player (or orbits at `keepDist`), shoots if `shootInterval > 0`, has HP bar; TANK triggers screen shake + health drop on death; SPLITTER spawns 2 GRUNTs
8. **Player** — WASD/arrow movement, mouse aim, auto-fire on hold, invincibility frames, walk animation (4 frames)
9. **LEVEL_DATA (`LEVELS[]`)** — 5 entries; spawn interval tightens to 0.85× at 33% kills and 0.70× at 66%
10. **Game** — owns the `requestAnimationFrame` loop; state machine (`MENU → PLAYING → LEVEL_COMPLETE → PLAYING | VICTORY`, `PLAYING → GAME_OVER`); circle-vs-circle collision everywhere; high score via `localStorage`
11. **Bootstrap** — `window.addEventListener('load', () => new Game())`

**Key implementation details:**
- Delta-time is capped at `50ms` (`dt = Math.min(dt, 0.05)`) to prevent spiral-of-death on tab restore
- All sprites are rotated `angle + π/2` so the "head" of the sprite (top row) points in the direction of travel
- Player collision radius (`PLAYER_RADIUS = 10`) is intentionally smaller than the sprite for a forgiving feel
- `Input.mouse.clicked` is reset to `false` at the end of every frame in `_tick()`; state transitions guard against same-frame click-through with `stateTimer > 0.5`
