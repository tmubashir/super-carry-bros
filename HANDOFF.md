# Super Carry Bros. — handoff brief

Context for a fresh session. Everything here is measured, not assumed.

---

## 1. What this is

A VC-themed Mario-style side-scroller. You run a $100M seed fund. You start with
ten portfolio companies and **nine of them are supposed to die**.

The design rule that makes it not a normal platformer:

> **Your score is your single best exit, not the sum of what you collected.**
> Losing companies is survivable. Whiffing the breakout is fatal.

Secondary joke: **TVPI vs DPI.** TVPI is the enormous number top-left and it
climbs on every pickup. DPI is the small grey one underneath. The end screen
grades you on the small grey one, as a fund quartile.

Live: <https://tmubashir.github.io/super-carry-bros/>
Repo: `tmubashir/super-carry-bros` (public, GitHub Pages off `main`)

---

## 2. Layout

| File | What |
|---|---|
| `index.html` | The entire game. Single file, canvas, zero dependencies, no build step. |
| `harness.html` | Automated playtester. Loads the real game and measures whether it's fun. |
| `README.md` | Player-facing. Controls + the joke. |
| `.claude/launch.json` | Dev server (`python3 -m http.server 8777`). |

Run locally:

```bash
cd /Users/tahamubashir/super_carry && python3 -m http.server 8777
```

Then `http://localhost:8777/index.html` and `http://localhost:8777/harness.html`.

**Deploy = push.** Pages rebuilds in ~60s. Always verify after:

```bash
curl -s https://tmubashir.github.io/super-carry-bros/ | shasum -a 256
```

---

## 3. Controls

`←/→` move · `Space` (hold) conviction jump · `Enter` Daily Fund · `B` skip to
the boss · `X` secondary sale / burn dry powder in the boss room · `C` copy
share card on the end screen.

---

## 4. What is built

**Level 1-1 Demo Day** — bounce pads that launch you into sky routes, a
patrolling BRIDGE ROUND moving platform, Serial Pivot enemies that lunge and
rebrand mid-air (AI→B2B→D2C→WEB3), a founder conveyor, uncapped SAFEs up high.
168 tiles wide.

**The Tier 1 VC boss.** No health bar — a post-money that only goes up. Attacks
telegraph with a `!`, chosen by distance: volley (far) / dash (mid) / ground
pound (close). Each leaves a **winded** window. Bait the dash into a wall for a
long free-hit opening. Strolling contact **shoves** rather than kills; deaths
come only from attacks. 3 conviction pips, phases speed up as they drop.

The decision it exists to pose:
- **Free hits** (winded window) → cheap entry, 16.7% ownership, needs execution
- **Bids** (impatient stomps) → +$45M post AND +4s on the rival clock. Money buys time.
- **Standing off** → the founder reads distance as disinterest; the rival closes
  2.6× faster beyond 300px. Punished outright.
- **Dry powder** → `X` burns $8M reserves to hold the rival 3s.
- **The door** → stand still in it to pass. Goes to the anti-portfolio.

**Economy** — dilution shrinks your sprite each round, pro rata restores it,
reserves gate follow-ons at rising cost, 10-year fund clock, LP nags,
anti-portfolio sting on the end screen.

**Virality layer** — `Enter` runs the **Daily Fund**: 4 companies, ~2 min,
date-seeded so everyone plays the same fund today. `C` copies a share card:

```
SUPER CARRY BROS. #2
🚀🦄🦄
DPI 6.49x · TOP DECILE
best exit $204M (20.4x)
tmubashir.github.io/super-carry-bros
```

---

## 5. Current measured state

Latest harness run: **15 pass · 3 warn · 1 fail · 0 invariant violations**

Verified in the shipped page, same level and boss, differing only in boss play:

| Line | Grade | Best exit | DPI |
|---|---|---|---|
| Patient (free hits) | TOP DECILE | $325M | 4.97 |
| Bidder (stomp on sight) | SECOND QUARTILE | $69M | 1.77 |

L1 feel all green: coyote 83ms, jump buffer 83ms, input latency 1 frame,
acceleration 7 frames to top speed, landing shake 0.21, stomp shake 0.15.

**The one FAIL: `Content variety`.** One level layout, replayed up to 10× per
run, plus one boss room. A player has seen every tile the game owns in ~40
seconds. This is the flow channel failing on the boredom side and it is the
single biggest remaining lever. **This is the main job.**

Open WARNs worth knowing:
- **Secondary sale is dominated** — competent runs reach the flag 100% of the
  time, so selling at 50% is never right. Dead surface; it needs a state where
  it wins.
- **Outcome ceiling** — only two grade bands show up in practice.
- **Patience under human reaction** — at 133ms delay both lines survive; the
  patient line only breaks down at 233ms. The trade-off exists but is thin.

---

## 6. The harness — read this before changing the game

`harness.html` fetches `index.html`, extracts the `<script>`, and evaluates it
against a stubbed canvas + audio. **It instruments the real shipped game and
cannot drift from what you ship.** It never modifies `index.html`.

It probes four timescales, because a game fails at each independently and a
broken lower layer cannot be rescued from above:

- **L1 second-to-second — FEEL.** Is the verb pleasurable in a vacuum? Coyote
  time, jump buffer, input latency, acceleration, hit-stop, landing weight.
- **L2 second-to-minute — DECISIONS.** Competitive options, no dominant answer,
  legible outcomes.
- **L3 minute-to-hour — LOOP.** Reward cadence, difficulty tracking skill, variety.
- **L4 hour-to-hundreds — DEPTH.** Does skill keep finding headroom?
- Plus **FAIRNESS** (randomness before decisions, never after) and **BUGS**
  (invariant sweep across full runs, input-mashing, clock expiry edge cases).

It also runs **experiments**: it applies candidate patches to the game source
*in memory*, replays with seeded bots, and returns `ADOPT` / `REJECT` / `STALE`.
A recommendation is a measured result, not a suggestion. Experiments are allowed
to break the game — the shipped file is never touched and git holds the rollback.

### Harness gotchas that already cost time

- **`judge(b, v)` takes (baseline, variant).** Passing a one-arg predicate
  directly silently grades the *baseline*. This bug rejected good experiments
  for several runs.
- **A probe that matches nothing reports PASS.** The randomness probe matched
  zero call sites after a rename and gave a clean bill of health on no evidence.
  Probes now fail loudly when their own patterns go stale. Watch for `STALE`.
- **A perfect bot makes patience look free.** It dodges everything, so the
  patient line shows zero risk. There is a reaction-delay probe (0 / 133 / 233ms)
  that prices execution; dominance is judged against imperfect play. Use it for
  any "is this a real choice?" question.
- **Browser caches `harness.html`.** Append `?v=N` when re-running after edits.
- Bot policies: `bid` (rush), `wait` (stand off — meant to lose), `platform`
  (camp — meant to lose), `engaged` (mid-range, dodge, punish — the skilled line).

---

## 7. Non-negotiables

**Design guardrails** (from the original spec — these are what make it shareable):

1. **The mechanic carries the joke, not the label.** If a feature is only funny
   because of its name, cut it.
2. **Insider, not explainer.** No tooltips defining TVPI. The audience knows.
   Anyone who doesn't should still enjoy the platforming.
3. **Losing is normal.** Nine deaths per run is the design target.
4. **Never punch at founders.** The game is a VC laughing at VCs. That is what
   makes it shareable rather than annoying.

**Technical invariants** (the harness enforces these — don't break them):

- **No randomness after a decision.** All RNG is cosmetic. Rolls that decide
  whether an action succeeds remove credit and read as unfair.
- **Determinism.** Same seed + same inputs = same run. Bisecting depends on it.
- **Single file, no build step, no external requests.** The game must survive
  being emailed to someone. Fully self-contained is also why it's safe.
- **Verify by playing, not by force-setting state.** A real mistake made here:
  the boss economics were "verified" by setting `mode='winded'` directly, which
  proved the math while the line was unplayable in practice. Drive it with a bot
  or your own inputs, end to end.

---

## 8. The job for the new session

**Primary: kill the `Content variety` FAIL.** Build more levels, and make the
minute-to-hour loop hold up. The original spec's world map, roughly in value order:

| Level | Gimmick |
|---|---|
| **2-1 Competitive Process** | Auto-scrolling. Cannot go back, cannot slow down. 48 hours to sign. |
| **3-1 The AI Bubble** | Water level. Everything floats up regardless of quality. |
| **5-1 The Down Round** | Screen dims. Valuations rain from above, all lower than yours. |
| **4-1 Capital Markets Frozen** | Ice. No traction, sliding everywhere. |
| **6-1 The Anti-Portfolio** | Bonus level of companies you passed on, now enormous and untouchable. |
| **8-1 The IPO Window** | A door that is closed 80% of the time. You wait. That is the level. |

2-1 is the recommended first build: auto-scroll supplies escalating pressure for
free, which also addresses "difficulty doesn't track rising skill."

**Secondary: the virality loop.** The Daily Fund and share card exist and work.
What's untested is whether the *result* is worth posting — whether the run
produces a story worth telling in 6 emoji.

### Research worth doing

Go look at how other games solved these, then adapt — don't copy:

- **Daily-run virality**: Wordle (one puzzle, spoiler-free emoji grid, everyone
  on the same board), Balatro (run seeds, screenshot-worthy absurd numbers),
  Slay the Spire daily climbs. What makes a *result* postable?
- **Auto-scroll pressure**: Mario 1-1's design language for teaching without
  tutorials; Downwell / Jetpack Joyride for escalation curves.
- **Run-based depth from few rules**: Slay the Spire, Balatro, Into the Breach —
  simple rules interacting to produce a space bigger than the rules. Depth, not
  complexity. Go has five rules; chess has more rules and less depth.
- **Game feel**: Steve Swink's *Game Feel*; Vlambeer's "juice it or lose it"
  talk — hit-stop, screen shake, particles, why the verb matters most.
- **Fun as learning**: Raph Koster's *Theory of Fun* — fun ends when learning
  ends, which is exactly why one repeated level goes flat no matter how polished.
- **Comedy games that traveled**: Papers Please, Frog Fractions, Universal
  Paperclips — how a joke premise sustains past the first laugh.

Note: **novelty is not depth.** It papers over a shallow system for exactly as
long as content lasts. New levels must add *decisions*, not just new tiles.

### Workflow that works

1. Run the harness first. Read the FAILs and WARNs.
2. Write an experiment before hand-tuning. The harness disagreed with my
   instincts twice and was right both times — most recently, a slower dash fixed
   the patient line while a longer telegraph did nothing, which is the opposite
   of what I'd have shipped.
3. Build, then verify by actually playing it end to end.
4. Commit before applying a batch of recommendations, so there's a rollback.
   There's a `pre-harness-recs` tag from an earlier round.
5. Push and hash-verify the live deploy.

---

## 9. Ops notes

- Commits must use `tmubashir@users.noreply.github.com`. The personal address
  leaked into early commits and was scrubbed via history rewrite; global git
  config is set correctly now. Don't reintroduce it.
- `.claude/` is gitignored (local tool permissions — not for a public repo).
- Repo is public and has been audited: no secrets, no credentials, no outbound
  network calls, no cookies or storage. Keep it that way — the zero-dependency,
  zero-request property is both a feature and the security posture.
