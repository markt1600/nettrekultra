# NetTrek Ultra

A browser-based, single-player space-combat game inspired by **Netrek** (1988) and **NetTrek** (1985). You command one ship against three rival AI fleets across a 300,000 × 300,000-unit galaxy of 40 planets. Capture planets to grow your team stronger. Stay alive — one life, no respawns.

No build step. Pure HTML + JS + CSS, ready to deploy to Vercel as a static site.

## Run locally

Any static file server works. From this folder:

```
# python
python3 -m http.server 8000

# or node
npx serve .
```

Open <http://localhost:8000>.

## Deploy to Vercel

```
npm i -g vercel
vercel              # follow prompts for first deploy
vercel --prod       # promote to production
```

Vercel auto-detects this as a static site (the included `vercel.json` only sets cache headers). You can also drag-and-drop the folder into the Vercel dashboard. Every JS/CSS reference in `index.html` carries a `?v=N` cache-busting query string — bump it when you change game code.

## Pick a ship, pick a difficulty

Five ship classes (slowest/tankiest → fastest/glassiest):

| Class      | Speed | Hull | Shield | Energy  | Torps | Notes |
|------------|-------|------|--------|---------|-------|-------|
| Scout      | 12    | 80   | 80     | 5,000   | 4     | Hit-and-run |
| Destroyer  | 10    | 95   | 95     | 7,000   | 6     | Balanced |
| Cruiser    | 9     | 110  | 110    | 10,000  | 8     | Default all-rounder |
| Battleship | 7     | 150  | 150    | 14,000  | 10    | Tanky line ship |
| Demi God   | 7     | 300  | 300    | 280,000 | 200   | 3× size, 20× ammo, 3× torp accuracy, 2× recharge, 5 shots/sec Auto-Fire — a flagship-tier "god mode" option |

Five difficulty tiers, easiest first:

- **Cadet** — same AI as Lieutenant but you get 10× shield HP, 3× torps, 3× flares, Auto-Defend ON by default. Enemy phasers slowed to 0.75s and enemy torpedoes capped at 1/sec. A quick gameplay briefing pops up on Engage. Recommended first run.
- **Lieutenant** (1.0×)
- **Commander** (1.15×)
- **Captain** (1.3×)
- **Admiral** (1.5×) — AI faster and more aggressive at higher difficulties.

## How to play

You start in orbit around your home planet with full magazines. The galaxy is split into four sectors; the other three sectors are occupied by enemy fleets. Survive, score points, capture planets.

### The five auto systems

Three top-bar toggles (also keyboard) let the ship fight, defend, and home in on targets without you driving every key:

- **AUTO-FIRE** (`F`) — when you have a target locked and it's in cone and range, your phasers fire on their own. Throttled to 0.5s between shots (0.2s for Demi God) so the energy reserve isn't dumped instantly. Torpedoes auto-fire in 2-per-2-second bursts.
- **AUTO-LOCK+NAV** (`L`) — full-auto attack. Continuously locks the nearest confirmed enemy in short-range radar and autopilots after them. Combined with AUTO-FIRE, this is "press one button and fight."
- **AUTO-DEFEND** (`V`) — raises shields when enemies are near, deploys flares against incoming torpedoes (up to 2 per torp, within 500u). When the ship is out of flares OR out of energy to deploy one, AUTO-DEFEND switches to **EVASION**: flees at warp 6 directly away from the torpedo centroid until the threat clears. Once the inbound torpedoes are gone, the existing autopilot or AUTO-LOCK+NAV resumes the attack run.
- **Auto-Attack Defenses** — autopilot to an enemy planet for capture, and on arrival your ship engages the planet's phaser turret automatically. Once the turret hits 0 hull, the capture sequence starts.
- **Auto-Refit at Home** — orbiting your home planet refills every resource (energy, shields, hull, torps, flares) pro rata over 5 seconds. So `0% → 100%` takes 5s; `80% → 100%` takes 1s. The home turret will fire on attackers during refit but you can still be killed mid-recharge.

### Capturing planets

Click an enemy/neutral planet on the radar and press `C` — your ship autopilots there at warp 4 (using a physics-based deceleration cap so high warp settings don't make you loop past), engages the turret if it's still alive, then holds orbit for 5 seconds with no enemies inside the danger zone. Each captured planet grants permanent team-wide bonuses to max hull, shield, energy, and repair rate.

The capture banner at the top of the tactical view shows the current state: PROCEEDING TO CAPTURE / DESTROYING DEFENSES / CAPTURING / CAPTURE STALLED (enemy in proximity) / REFITTING AT HOME.

### Home base mechanics

Your home planet:

- **Refits you** in 5 seconds when orbited.
- **Defends itself** with a phaser turret (150 hull, fires every 0.2s at 35 dmg, range ~1,000u). If destroyed, it slowly rebuilds over 30 seconds; once it hits 100% again, it comes back online.
- **Has two friendly AI defenders** that patrol it and intercept incoming enemies.
- **Answers SOS** — press `Q` within short-radar range of home and friendly ships break off to rescue you. Press `Q` again to cancel and a friendly chatter line goes out ("Crisis averted — stand down…"). Out of range or no friendlies available shows a red SOS-failure banner at the top of the screen.

### Cloak

Press `K` to engage cloak (needs ≥20% energy). Cloaked = invisible to sensors, can't be locked or torp-tracked. Phasers and torps go offline. Drains 20% of max energy per second. Note: cloak doesn't remove your physical presence — a torp that crosses your path can still hit you.

### Energy economy

Every system pulls from one shared pool. Phasers and flares cost fixed quanta (not a fraction of max), so a Demi God's huge reserve actually means more shots:

| | Phaser cost | Flare cost | Ambient recharge |
|-------|-------------|------------|------------------|
| SC | 200 | 100 | 12/s |
| DD | 300 | 100 | 14/s |
| CA | 400 | 100 | 16/s |
| BB | 500 | 100 | 14/s |
| DG | 500 | 100 | 28/s |

Sustained fire burns the reserve faster than ambient recharge can refill — short, decisive bursts followed by a return to home (5s refit) or peace (slow recharge in space). Dropping shields gives a 1.6× recharge bonus.

### Structural damage tiers

Hull below threshold takes systems offline:

- `< 50%` — torpedoes offline
- `< 25%` — phasers offline
- `< 10%` — warp drive offline
- `0%` — destroyed (game over)

## Controls

### Combat
| Key | Action |
|-----|--------|
| LMB on enemy ship | Lock target (queues phaser shot — fires when ship turns into the ±30° cone) |
| LMB on empty space | Manual course + queue phaser in that direction |
| `P` | Fire phaser without changing course |
| `T` | Fire torpedo in current heading (homes onto locked target if it's in the ±20° cone) |
| `D` | Deploy flare |
| `S` | Shields up / down |
| `K` | Cloak toggle |
| `F` / `L` / `V` | Auto-Fire / Auto-Lock+Nav / Auto-Defend toggles |

### Navigation
| Key | Action |
|-----|--------|
| Right-click | Manual course (warp 1 if stopped) |
| `0` – `9` | Set warp speed. Preserves orbit / autopilot — just changes how fast you get there. |
| `A` | Autopilot to selected target |
| `H` | Return home (warp 4, orbit on arrival, refit) |
| `R` | Toggle radar mode (SHORT 15,000u / LONG 60,000u) |
| `+` / `−` | Tactical zoom (1× / 2× / 4× / 8×) |

### Planetary
| Key | Action |
|-----|--------|
| `O` | Orbit selected planet (auto-decelerates if needed) |
| `C` | Capture selected enemy/neutral planet (auto-flies + auto-engages turret + orbits + holds) |

### Gameplay
| Key | Action |
|-----|--------|
| `Q` | SOS (call friendlies from home) — second press cancels |
| `M` | Mute / unmute |
| `Esc` | Pause / close help |
| `?` or `F1` | Open the full help overlay |

### On-screen buttons (left side of tactical canvas)

A panel sits over the bottom-left half of the canvas:

- **SHIELDS** — toggles shields. Lit blue when UP, dim grey when DOWN.
- **FLARE** — deploy a flare. Flashes green when fired (manual or Auto-Defend).
- **PHASER** — fire phaser. Flashes orange (manual fire, queued shot resolution, Auto-Fire dispatch, or attacking planet defenses).
- **TORP** — fire torpedo. Flashes blue.
- **Speed meter** — vertical 0–9 scale with clickable warp-level rows. Live fill tracks actual current warp; click any level to snap to it.

### Radar widget (right side)

Toggle between SHORT (15,000u — full IFF colors, enemies red) and LONG (60,000u — galactic view, all dots uniform) by clicking the segmented `SHORT | LONG` switch or pressing `R`.

## Architecture

All files sit in the project root — drop the whole directory into Vercel.

- `index.html` — three screens (start, game, end), help overlay, Cadet briefing overlay.
- `style.css` — black-CRT-style HUD layout, responsive (drops console + keys panel on small viewports, square-canvas centering on widescreens).
- `constants.js` — ship classes, planet flags, weapon ranges, energy quanta, capture/refit/rebuild timing.
- `galaxy.js` — 40-planet galaxy generation across 4 sectors with a Mulberry32 PRNG.
- `ship.js` — ship physics, damage, refit/orbit/capture/SOS/cloak/evasion, planet defense tick.
- `weapons.js` — phasers (instant ray, ship + planet-turret collision), torpedoes (homing, probabilistic hit by distance, flare divert).
- `ai.js` — enemy state machine (HUNT, CAPTURE, FLEE, DEFEND, RESCUE), throttled phaser + torp fire, Cadet slowdown.
- `input.js` — mouse + keyboard binding, weapons panel + speed meter wiring.
- `render.js` — tactical and galactic canvas rendering, ship sprites, planet defense ring, capture progress, hover tooltips.
- `ui.js` — HUD, scoreboard, message log + chatter feed, top-bar mode indicators, capture/SOS banners.
- `audio.js` — Web Audio synthesized SFX (original sci-fi fanfare, ambient drone, weapon sounds, torp alarm) — all synthesized, no copyrighted material.
- `main.js` — bootstrap, world initialization, game loop, victory check, help / Cadet briefing wiring.
- `vercel.json` — short-cache headers.

## Credits & inspiration

- **Netrek** — Kevin Smith, Scott Silvey, Terence Chang, and many others (1988). <https://en.wikipedia.org/wiki/Netrek>
- **NetTrek** — Randy Carr (Macintosh, 1985–1989). <https://fatlion.com/nettrek/>

This game is not affiliated with either project; it's a small homage built for fun and easy deployment to a static host. All audio is original — no copyrighted material is reproduced.
