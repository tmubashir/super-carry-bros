# World 2 — implementation prompt

Paste this into a fresh Claude Code session in `/Users/tahamubashir/super_carry`.

---

Build World 2 in `index.html`. It is a single file, no build step, no external
requests, no dependencies — keep it that way. Read `HANDOFF.md` §7
(non-negotiables) before you start and honour all of it: no randomness after a
decision, determinism, never punch at founders, the mechanic carries the joke.

Everything below is anchored to code that exists today. Line numbers are from
the current `main` (`2693933`) and will drift as you edit — find by symbol.

## 0. Design decisions already made (do not re-litigate)

- Winning a boss is what advances the world. The fund carousel (10 companies,
  nine are supposed to die) stays exactly as it is; what changes is *which*
  level a company gets dropped into.
- Only a boss **killing you** ends the run. Losing the deal on the clock, or
  walking out the door, still costs nothing today — with one exception, the
  debt holders, where the maturity clock running out is also fatal because that
  is what maturity means.
- AI ACCELERATION makes you immune to NPCs, not to the capital structure.
  Anchors kill you while accelerated. That is the joke.
- Both bosses use the same three-hit state machine. World 2 re-skins and
  re-prices it; it does not rewrite it.

## 1. World progression (`LEVELS` registry, ~line 178)

Add above `PLAYLIST`:

```js
const WORLDS = [
  { level: '1-1', boss: 'boss'  },
  { level: '2-1', boss: 'boss2' },
];
```

Delete `PLAYLIST` (nothing else references it — `harness.html` does not) and:

- `newRun()` (~211): add `world: 0` to `G`.
- `spawnCompany()` (~252): `loadLevel(WORLDS[G.world].level)` instead of the
  `PLAYLIST[G.resolved % ...]` modulo.
- `spawnBoss()` (~276): `loadLevel(WORLDS[G.world].boss)`, and take the boss's
  identity from `G.mods.boss` rather than hardcoding TIER 1 VC.
- `winHotDeal()` (~863): after building `G.pendingHot`, advance —
  `G.world = Math.min(G.world + 1, WORLDS.length - 1)`. The very next
  `spawnCompany()` therefore lands the hot deal in 2-1. That is the feature:
  beat the boss, the world changes under you.
- `drawDealWon()` (~1052): add a line under the ownership number —
  `NOW ENTERING 2-1 · SAND HILL ROAD` (use `LEVELS[WORLDS[G.world].level].name`).
- `G.bossQueue` stays `[1, 5]` / daily `[1, 2]`, so the debt holders are the
  second boss you meet.

## 2. Boss death ends the run

`loseDeal(reason, passed)` (~874) gains a third argument, `fatal`.

Fatal call sites (all of them are the boss actually killing you):
- `updateBoss` dash contact, `'Run over mid-pre-empt.'` (~799)
- projectile hit (~815)
- shockwave hit (~820)
- fell off the map in boss mode, `moveAndCollide` caller (~578)

Non-fatal, unchanged: the door (`'You walked out of the room.'`, ~726) and —
for the Tier 1 VC only — the rival clock (~705). For `G.mods.boss === 'debt'`
the clock expiring **is** fatal (see §7).

When fatal:
- set `G.dealFatal = { headline, line }` from the boss's table entry,
- push `{t:'ko'}` onto `G.story` and map it to `☠️` in `storyEmoji()` (~956),
- in `update()`'s card branch (~516) route to `endRun()` instead of
  `nextScene()` when `G.dealFatal` is set,
- `drawDealLost()` (~1074) prints the headline in `#ff6b81`, the line under it,
  and `RUN OVER` where the "Costs you nothing today." line normally sits,
- `renderGameOver()` (~1512) shows the same headline above the grade.

Note in your commit message that this is a deliberate departure from "losing is
normal": it applies to boss rooms only, and regular level deaths still just
retire the company.

## 3. Level 2-1 map

New `const LEVEL_ROWS_2 = [...]` next to `LEVEL_ROWS` (~141). Constraints that
will bite you if you ignore them:

- **12 rows.** Ground on rows 10–11 (`#`), same as 1-1.
- **Spawn is hardcoded** at `P.x = TILE*2.5, P.y = TILE*8` — leave the first
  ~4 columns clear with solid ground under them.
- **The flag only spawns on row 7** — `spawnEntities()` has
  `if (ch === 'F' && ty === 7)`. Put `F` on row 7 or the level has no exit.
- Do not use `C` unless you want the founder conveyor; it spawns off the
  presence of a `C` anywhere in the map.
- Aim for ~180 columns — a touch longer than 1-1, since AI ACCELERATION wants
  a straight to open up on.

Two new legend characters:
- `A` — anchor rig, placed on a ceiling row (rows 2–5) above a path you must
  cross. Six to eight of them, spaced so a patient player can walk through and
  an accelerated player has to actually read them.
- `Z` — the AI ACCELERATION pickup. Two per level: one on the ground line early
  so everyone finds it, one on a high platform route.

Register both levels:

```js
'2-1':   { rows: LEVEL_ROWS_2,  name: 'Sand Hill Road', mods: { theme: 'valley' } },
'boss2': { rows: ARENA_ROWS_2,  name: 'The Recap',      mods: { theme: 'valley', boss: 'debt' } },
```

`ARENA_ROWS_2` is `ARENA_ROWS` (~158) with the `D` door tile removed — you
cannot walk out of your own cap table — and a slightly wider floor so the dash
still wall-slams.

## 4. Silicon Valley look

Drive it off `G.mods.theme`, so 1-1 is untouched.

- `render()` (~1012): theme the sky gradient. Valley is daytime, hazy and warm
  — roughly `#bcd9ef` at the top to `#f0dfbe` at the horizon, versus 1-1's
  night purple. It should read as a different *time of day*, not just different
  hues, so the world change is obvious in a screenshot.
- `drawSkyline()` (~1088): keep the existing body as `drawSkylineNight()`, add
  `drawSkylineValley()` at the same parallax rates (`cam.x*0.25` far,
  `cam.x*0.5` near). Procedural, no assets:
  - a band of golden-brown hills across the back (two overlapping sine ridges),
  - low glass office parks — wide, short rects with mirrored blue window grids,
    nothing taller than four floors,
  - palm trees: trunk plus five fronds, on the nearer parallax layer,
  - a 101 ribbon along the horizon with slow-moving car dots,
  - one roadside monument sign reading `SAND HILL RD`.
- `drawTiles()` (~1104): pull the `#`, `=`, `J` colours from a small per-theme
  palette object instead of the current hardcoded purples. Valley ground is
  concrete and beige; `=` platforms stay wood-gold so the one-way rule keeps
  reading the same way.

## 5. Falling anchors

Add an `anchor` entry to the `NPC` table (~335) with `legend: 'A'`. It gets
spawned by `spawnEntities()` (~460) and updated by the loop at ~626 for free —
do not add a branch anywhere else.

State machine, fully deterministic (fairness invariant — phase offset comes
from the spawn tile x, **never** from `RNG()`):

1. `hang` — at the ceiling, chain drawn down a few links, period ~2.2s.
2. `warn` — 0.55s: the rig shakes, a dashed drop-line paints the column below
   it, `blip(140,.12,'square',.05)`. This is the tell; it must be readable.
3. `fall` — gravity ~2000px/s², chain paying out behind it.
4. `smash` — on any `solidTop` tile or the ground: `G.shake = 0.25`,
   `freeze = 3`, a dust `burst`, rest 0.9s.
5. `retract` — winches back up at ~260px/s, then `hang` again.

Lethal only in `fall` and `smash`. Hitbox ~30×34 around the anchor head.
On contact:

```js
return killCompany('anchor', {
  headline: 'CRAMMED DOWN',
  line: "you were crammed down to commons and liquidation preference didn't clear",
});
```

`killCompany(why)` (~927) currently always picks a random line from
`PASS_LINES`. Give it an optional second argument that stores the override on
`G`, and have `drawDeathCard()` (~1448) print the override headline and line
verbatim when present — that exact sentence, unedited, is the payload of this
whole obstacle.

Draw it as a real ship's anchor: black shank, curved stock, two flukes, and a
chain of links up to the ceiling that shortens as it falls. Not a weight, not a
box — the silhouette is the joke.

## 6. AI ACCELERATION

`spawnEntities()`: `if (ch === 'Z') ents.push({t:'coin', kind:'accel', x, y, r:13, taken:false});`

`collect()` (~885), before the term-sheet branch:

```js
if (e.kind === 'accel') {
  e.taken = true; C.accel = 9;
  sfx.markup(); burst(e.x, e.y, 20, '#9ecbff', 260); freeze = 4;
  banner('AI ACCELERATION');
  return;
}
```

`banner(txt)` is a new one-liner alongside `toast()` (~477) that sets
`G.banner` / `G.bannerT = 1.4`; render it centre-screen in bold 40px gold with
a scale-in, above the HUD. The words `AI ACCELERATION` are the whole point of
the pickup — a small toast is not enough.

Effects while `C.accel > 0`:
- **Speed.** At the momentum line (~545): `const tgt = ax * RUN * (C && C.accel > 0 ? 1.8 : 1)`.
  Leave the jump alone — one changed variable keeps the state legible.
- **Immunity + lethality.** `goomba` (~358) and `pivot` (~404) already have a
  `C.fees > 0` branch that kills the enemy on contact. Extend the condition to
  `C.accel > 0 || C.fees > 0`, and when it is `accel`, add
  `burst(...); freeze = 3; G.shake = Math.max(G.shake, 0.12); C.paper += 1;`
  and a toast like `Ran straight through it. +$1M mark.` Running into an NPC at
  speed kills it; that is the power.
- **Anchors ignore it.** Do not add an `accel` check to the anchor. Getting
  crammed down while accelerating is the best version of this level.
- Tick `C.accel` down beside `C.proRata` / `C.fees` at ~536.
- HUD (~1311): an `AI ACCELERATION` pill with a draining bar. `drawPlayer()`
  (~1288): three fading afterimages behind the sprite while active.

## 7. The debt holders (boss 2)

Add a `BOSSES` table keyed by `mods.boss`, and read every string and number the
boss fight prints from it instead of from the current hardcoded literals
(`spawnBoss`, `updateBoss`, `hitBoss`, `drawBossIntro`, `drawDealLost`, the
boss branch of `drawEnts` ~1188, and the HUD boss readout ~1427).

`tier1` is exactly what ships today. `debt` is:

| Thing | Tier 1 VC (today) | Debt holders |
|---|---|---|
| Name | `TIER 1 VC` | `DEBT HOLDERS` |
| Pips | `conviction ●●●` | `covenants ●●●` |
| Clock | `APEX PARTNERS is closing it` | `MATURITY IN 16s` |
| Volley | "our platform team will call you" | `COVENANT BREACH` |
| Dash | "we're pre-empting" | `ACCELERATION CLAUSE` |
| Pound | "we'll lead" | `FORECLOSURE` |
| Winded | `WINDED — FREE HIT` | `WAIVER GRANTED — FREE HIT` |
| A bid costs | +$45M post-money | +$45M of debt that sits ahead of you |
| Defeated | "Sorry — I have a board thing." | "Fine. We'll amend and extend." |
| Palette | gold `#d4af37` / navy | ledger grey `#8a8f9c` / stamp red `#c0455a` |

Sprite: a grey suit built from a stack of loan documents rather than term
sheets, a briefcase instead of the vest, a ledger under one arm, and a red
`1x PREF` stamp across the chest that flashes on the telegraph.

**Every loss to the debt holders is fatal** — attack deaths, falling out of the
room, and the maturity clock expiring. There is no door in `ARENA_ROWS_2`. All
of them end the run with:

- headline: `DEBT HOLDERS TOOK OVER` (exactly that string)
- line: `the equity was worth nothing on the way out.`

Beating them still calls `winHotDeal()`; since `WORLDS` has no world 3,
`G.world` clamps and the run continues in 2-1 until the fund clock or the
companies run out.

## 8. Verify before you call it done

```bash
cd /Users/tahamubashir/super_carry && python3 -m http.server 8777
```

1. Play it end to end at `http://localhost:8777/index.html`: 1-1 → beat the
   Tier 1 VC → confirm you land in 2-1 and the sky is daytime → die to an
   anchor and read the message → take AI ACCELERATION and run through a Serial
   Pivot → reach the debt holders → lose to them (run over, exact string) →
   restart and beat them.
2. Also verify the paths that must *not* have changed: dying in 1-1 still just
   retires the company, walking out the Tier 1 VC door still costs nothing, the
   Daily Fund (`Enter`) and share card (`C`) still work, `B` still skips to the
   first boss.
3. Run `http://localhost:8777/harness.html?v=2`. Expect `Content variety` to
   move off FAIL. Watch for `STALE` probes — several of them pattern-match
   source that this change touches, and a probe matching nothing reports PASS on
   no evidence. Fix stale patterns rather than accepting the green.
4. Add a temporary debug key that jumps straight to world 2 while you build,
   and delete it before committing.

Commit in reviewable steps (registry + progression, then fatal deaths, then the
level and its art, then anchors, then the power-up, then the debt holders) so
any one of them can be reverted alone.
