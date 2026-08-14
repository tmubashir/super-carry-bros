# Super Carry Bros. — handoff brief

Context for a fresh session on a new machine. Everything here is measured, not
assumed, and matches the shipped `index.html` as of the "Strip back to Mario"
commit.

---

## 1. What this is

Mario, but you are a VC. Three portfolio companies are your lives, term sheets
are your coins, and the joke is carried by the mechanics rather than the labels.

**The one rule: TVPI is your score and your health at the same time.** Sonic's
rings in a quarter-zip.

Live: <https://tmubashir.github.io/super-carry-bros/>
Repo: `tmubashir/super-carry-bros` (public, GitHub Pages off `main`)

---

## 2. Setup on a new machine

```bash
git clone https://github.com/tmubashir/super-carry-bros.git
cd super-carry-bros
git config --global user.email "tmubashir@users.noreply.github.com"
python3 -m http.server 8777
```

`localhost:8777/index.html` to play, `localhost:8777/harness.html` to run the
suite. `.claude/launch.json` is gitignored, so the dev-server shortcut has to be
recreated locally if you want it — the command above is the whole dependency.

**Deploy = push.** Pages rebuilds in ~60s. Always hash-verify after:

```bash
curl -s https://tmubashir.github.io/super-carry-bros/ | shasum -a 256
shasum -a 256 index.html
```

---

## 3. Layout

| File | What |
|---|---|
| `index.html` | The whole game. Single file, canvas, no build step, no external requests. |
| `harness.html` | Automated playtester. Loads the real game and measures whether it's fun. |
| `README.md` | Player-facing and **current**. |
| `SUPER_CARRY_BROS_research.md` | ~90-source design research. Historical — predates the strip. |
| `WORLD2_PROMPT.md` | The World 2 build spec. Historical — already implemented. |

---

## 4. The model

| Event | TVPI |
|---|---|
| SAFE | +0.05 |
| Term sheet | +0.10 |
| Uncapped SAFE | +0.50 |
| Stomp any enemy | +0.15 |
| Beat a boss | +1.00 |
| Pro rata pickup | +0.50 (heals what a down round took) |
| **Down Round Goomba contact** | **−0.40**, knockback, 1.5s mercy |

Start **1.00x**, **3 lives**. Death = TVPI ≤ 0, a pit, a Serial Pivot, a falling
anchor, a boss attack, or the runway expiring. Death costs a life and restarts
the current world.

Two enemy classes, and the split is the joke: `goomba` **drains** (a down round
marks you down), `pivot` and `anchor` **kill outright** (a pivot ends you).
`mover` is a harmless platform.

Power-ups keep their Mario jobs: **board seat** = 1-UP, **2&20** = star (7s),
**AI ACCELERATION** = 1.8x speed + run through NPCs for 9s — and deliberately
does *nothing* about anchors, because that's the capital structure, not
competition.

Controls: `←/→` or `A/D`, `Space`/`↑`/`W` (hold for conviction jump), `Enter`
daily fund, `B` skip to boss, `C` copy share card on an end screen.

---

## 5. Structure

Two worlds, each one level ending at a mandatory boss:

| World | Level | Runway | Boss |
|---|---|---|---|
| 1 | `1-1` Demo Day | 45s | **TIER 1 VC** |
| 2 | `2-1` Sand Hill Road | 55s | **DEBT HOLDERS** |

There is no flag inside a level — walking off the right edge *is* arriving at the
round. The boss stands in front of the signature; **three stomps** and he's out,
then the flag behind him ends the level. Beat both worlds → `YOU RETURNED THE
FUND`. Burn all three lives → game over. **No loop exists anywhere.**

Registries that make new content cheap:
- `WORLDS` — world order, level ids, runway seconds
- `LEVELS` — every playable map + a `mods` object (that's the hook new level
  rules hang off, e.g. `{theme:'valley', boss:'debt'}`)
- `NPC` — each enemy declares `legend` / `spawn` / `update` / `draw` in one entry;
  `spawnEntities()` iterates it, so a new enemy is one table entry, not three
  edits in three places
- `BOSSES` — every boss string and colour, keyed by `mods.boss`

Grading is on final TVPI against `BANDS`: 9.5 / 7.5 / 5.5 / 3.0. Measured ceiling
is ~12.8x if you take everything and stomp everything.

---

## 6. The harness — read before changing the game

`harness.html` fetches `index.html`, extracts the `<script>`, and evaluates it
against a stubbed canvas + audio. **It instruments the real shipped game and
cannot drift from what you ship.** It never modifies `index.html`.

Probes four timescales — L1 feel, L2 decisions, L3 loop, L4 depth — plus fairness
and an invariant sweep. It also runs **experiments**: applies candidate patches to
the source *in memory*, replays with seeded bots, returns `ADOPT`/`REJECT`/`STALE`.

**Current state: 14 pass · 3 warn · 0 fail · 0 invariant violations.**
Open warns are all "more game to build", not breakage: content variety (2
levels), skill spread 1.73x, outcome ceiling (2 bands seen).

### Gotchas that have already cost real time

- **`judge(b, v)` takes (baseline, variant).** Passing a one-arg predicate
  silently grades the *baseline* and rejects good experiments.
- **A probe that matches nothing reports PASS.** Watch for `STALE`. Fix the
  pattern; never accept the green.
- **Prune `EXPORTS` when you delete a function** — a stale symbol there breaks
  the whole harness on boot. This has bitten twice.
- **The harness never calls `render()`**, so drawing code is invisible to it.
  Verify visuals by hand in the page.
- **Browser caches `harness.html`** — append `?v=N` after edits.
- Bot tiers must differ *behaviourally*: skill 0 walks into enemies, 1 dodges and
  reads anchor tells, 2 also hunts stomps. A previous version had 0 and 1
  identical and 2 suiciding on high coins, which made the skill ladder meaningless.

---

## 7. Non-negotiables

**Design guardrails:**

1. **The mechanic carries the joke, not the label.** If a feature is only funny
   because of its name, cut it.
2. **Insider, not explainer.** No tooltips defining TVPI.
3. **Losing is normal.** Deaths are retries, not failure.
4. **Never punch at founders.** The game is a VC laughing at VCs.
5. **Mario mechanics, VC graphics.** If Mario doesn't have the system, question it.
   The last pass deleted reserves, rounds, dilution, hype, secondaries, DPI and
   exit pricing on exactly this test.

**Technical invariants (the harness enforces these):**

- **No randomness after a decision.** All RNG is cosmetic.
- **Determinism.** Same seed + inputs = same run.
- **Single file, no build step, no external requests.** It must survive being
  emailed to someone. That's also the security posture — audited clean, no
  secrets, no network calls, no storage.
- **Verify by playing, not by force-setting state.** A real mistake made here:
  boss economics were "verified" by setting `mode='winded'` directly, which
  proved the math while the line was unplayable. Drive it with a bot, end to end.

---

## 8. Where to go next

`Content variety` is the standing weakness — two levels. The registries above
make a third cheap: a map, a `LEVELS` entry, a `WORLDS` entry, optionally an `NPC`
table entry. Candidates from the original spec, each needing **one new decision**
rather than one new tile: 3-1 The AI Bubble (inverted gravity — ride the trash
upward), 4-1 Capital Markets Frozen (ice grants a *higher* top speed some gaps
require), 5-1 The Down Round (falling valuations are collectible light that
permanently marks you down).

One open question the bots can't answer: **is −0.40 the right sting for a down
round?** The harness measured "no change" in both directions because competent
bots rarely get hit, so it's a feel call. Play it and decide.

Suggested first move in a new session: run the harness, read the warns, then
pick one thing. Commit before a batch of changes so any one is revertible.
