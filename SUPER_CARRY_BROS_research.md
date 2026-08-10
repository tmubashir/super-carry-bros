# Super Carry Bros. — Deep Research & Design Direction

*Research doc. No code changed. Grounded in the shipped `index.html` (1,437 lines, staged from `/Users/tahamubashir/super_carry`), the handoff brief, and ~90 sources across game design, virality, and comedy-game longevity.*

---

## 0. The verdict, in one paragraph

**`Content variety` is the symptom. The disease is that your run only ever asks one question, and that question is "can you platform?"** Building the six planned levels will buy you roughly six minutes of novelty and will not move the loop, because a new tile is not a new decision. The premise you actually have — a fund where nine of ten die and only the best exit counts — is a **portfolio-allocation game wearing a platformer costume**, and right now the costume is doing all the work. The highest-leverage change is not more levels. It is **making capital allocation the game and the platformer the resolution mechanic**: a small node map of parallel companies, finite reserves you must concentrate or spread, and a TVPI number that is *purchasable* and therefore actively lies to you. That is a ~2-day change that turns your existing 1,437 lines from "the whole game" into "the combat system," and it makes every level you subsequently build worth 3–5× more. Levels then get built for a reason: each one is a different *market condition* you chose to enter.

If you build only one thing this week: **Level 2-1 with a speed-vs-diligence trade, plus finite reserves, plus a share card that shows regret.** Those three fix the FAIL, the two-band outcome ceiling, and the untested virality loop respectively, and none of them requires a rewrite.

---

## 1. Diagnosis: why six levels won't fix it

### 1.1 Depth vs. content (the load-bearing distinction)

Josh Bycer's definition is the cleanest in the literature: **depth is "the number of viable options at any given moment"; complexity is the count of systems and mechanics.** A game with 200 items and 4 viable builds is complex and shallow. ([Game Developer](https://www.gamedeveloper.com/design/defining-depth-in-game-design))

Your harness measured *content variety*. But if replay #7 asks the same question as replay #1, level #11 changes nothing. Count decisions, not tiles.

**Where you are today, honestly counted, the run contains roughly three real decisions:**

1. Sky route vs. ground route in 1-1 (weakly consequential — both reach the flag)
2. Free hits vs. bids vs. dry powder in the boss room (this one is genuinely good)
3. Take the follow-on or don't

Everything else is execution. Ten repetitions of three decisions is the boredom you measured.

### 1.2 Koster's prediction, and the trap on the other side

Raph Koster: *"fun is just another word for learning"*; *"boredom is the opposite of learning"*; and the operative test — **"a good game is one that teaches everything it has to offer before the player stops playing."** ([quotes](https://www.goodreads.com/work/quotes/19639-a-theory-of-fun-for-game-design))

Your game teaches its whole pattern in ~40 seconds and then asks for nine more repetitions. Koster predicts exactly your FAIL.

But note the failure mode on the *other* side, which is the one a "just add randomness / add levels" fix walks into: **"noise is any pattern we don't understand."** If you fix boredom with variance, you get unlearnable, and unlearnable is equally boring — and it also breaks your own fairness invariant. The unit of content you need is **a pattern to be learned**, targeting roughly **one new learnable pattern per two minutes of run**.

### 1.3 The container mismatch — this is the real finding

A side-scroller is **sequential, complete-or-fail, and sum-scored**. A fund is **simultaneous, partially observed, and max-scored**.

That mismatch explains all three of your open WARNs at once, and I think this is the single most useful sentence in this document:

| Symptom | Root cause |
|---|---|
| Secondary sale is dominated (competent runs reach the flag 100%) | There is no state where selling wins, because there is no capital hunger, no volatility, and no binding clock. The game never *needs* liquidity. |
| Only two grade bands appear | Linear sequential play produces linear outcome distributions. You cannot get a heavy tail out of ten independent identical trials that a skilled player passes every time. |
| Content variety FAIL | One question asked ten times. |

Fixing these one at a time via level content is treating three symptoms of one disease.

### 1.4 What the code says about cost

From the shipped file — this is good news:

- Levels are **string arrays** (`LEVEL_ROWS`, `ARENA_ROWS`) swapped by `setMap(rows)`. Adding map data is trivial.
- Entities spawn from a **tile-character legend** in `spawnEntities()` — adding a spawn character is 1 line.
- But **enemy behaviour is a hardcoded `if (e.t === 'goomba')` chain** in `update()` and `drawEnts()`. Each new NPC costs ~3 edits in 3 places, ~30 lines.
- There is **no level registry** — just two module-level constants. `nextScene()` always calls `spawnCompany()` → `setMap(LEVEL_ROWS)`.
- The legend comment already declares tiles that don't exist yet (`b` brick, `^` spike). Dead legend entries.

**Two small refactors unlock everything else and should happen first:**

```js
// 1. Level registry — replaces the two constants
const LEVELS = {
  '1-1': { rows: L11_ROWS, name:'Demo Day',           mods:{} },
  '2-1': { rows: L21_ROWS, name:'Competitive Process', mods:{ autoscroll: 34 } },
  '3-1': { rows: L31_ROWS, name:'The AI Bubble',      mods:{ buoyancy: -540 } },
  // ...
};
// G.levelId drives setMap; spawnCompany() picks from a per-run playlist.

// 2. Enemy behaviour table — replaces the if-chain
const NPC = {
  goomba: { w:26, h:26, init(e){...}, update(e,dt){...}, draw(e){...} },
  pivot:  { ... },
};
// update(): NPC[e.t].update(e, dt)
```

Neither changes behaviour, so the harness should read identical before and after — which makes them safe to land first and gives you a clean bisect point.

---

## 2. The design spine

Five principles, each sourced, that everything below is derived from.

**P1 — One new *decision* per level, not one new *tile*.**
Nintendo's rule, via Koichi Hayashida: a level carries **one core concept "absolutely all the way,"** structured as *kishōtenketsu* — introduce → develop → twist → conclude. Mark Brown's formalization adds the safety property: **step 1 must be somewhere failure costs nothing.** ([Game Developer](https://www.gamedeveloper.com/design/the-structure-of-fun-learning-from-i-super-mario-3d-land-i-s-director), [GMTK/Nintendo Life](https://www.nintendolife.com/news/2015/03/video_nintendos_four_step_stage_design_is_why_you_love_super_mario_games_so_much))
→ **Your six planned levels currently list six tiles. Every one needs its decision written before its map.**

**P2 — A modifier level must trade, not subtract.**
The reason water and ice levels are universally hated is that they make the base verb *strictly worse* — slower, less precise, tools disabled — and grant nothing. The fix is to give the modified state a capability the normal state lacks. ([Orange Juice Liberation Front](https://orangejuiceliberationfront.com/why-underwater-levels-suck-so-often/), [RetroGameTalk](https://retrogametalk.com/threads/are-ice-levels-always-the-worst.17093/))
→ Corollary: **never stack two subtractions.** Dark + slippery + timer is three, and each alone already drops accuracy.

**P3 — Each NPC is a question, written in one sentence before it is drawn.**
Jason de Heras' Hollow Knight breakdown: enemies build incrementally, each introducing one tweak — Husk Warrior's *block* converts the question from "when do I hit" to "when does he recover." Rules he extracts: **form foreshadows function** (silhouette predicts attack direction), **environment changes behavior**, **aggro range calibrated just outside your attack range** so baiting is a skill. ([jasondeheras.com](https://www.jasondeheras.com/gamedesign/2021/3/6/early-game-hollow-knight-enemies))
→ If two enemies share a question, cut one. Chase **combinatorial** variety: three enemies whose questions compose beat eight that all ask "when?"

**P4 — Make one resource serve two opposed goals.**
Downwell's gunboots are simultaneously weapon and brake; ammo refills only on landing; the combo counter punishes caution. One button, four live considerations. ([Kill Screen](https://killscreen.com/previously/articles/the-tricky-brilliance-of-downwells-gunboots/), [GDC](https://gdcvault.com/play/1023533/Polishing-the-Boots-Designing-Downwell))
→ Your pickups currently only go up. **Money that is both reserve-for-follow-on and score** is the single cheapest depth injection in this document.

**P5 — Charge the joke; don't label it.**
Lucas Pope: *"Don't tell or show something when you can make the player do it themselves instead."* Bureaucratic tedium in Papers, Please isn't described — it's billed to you in seconds and rubles. Universal Paperclips stops being funny at exactly the moment the exponential actually eats the world. ([Reason/Pope](https://reason.com/2013/09/26/papers-please-politics-in-games-and-the/), [IF50](https://if50.substack.com/p/2017-universal-paperclips))
→ **TVPI vs DPI is currently a caption.** It should be a cost. See §5.2 — this is the best idea in the doc.

---

## 3. Step-changes: the container

You said "whatever the research says wins," so here it is straight. Three container options, ranked. They are alternatives, not additions.

### Option A — **Node map + platformer-as-resolution** ⭐ recommended

Replace the linear "10 companies in a row" with a **Slay-the-Spire-style branching map of quarters**. Ten companies are alive *simultaneously* as rows on a portfolio dashboard. Each quarter you may personally work **3 of them**; the rest run on autopilot and drift. "Working" a company = playing a 30–45 second side-scroll segment whose outcome sets its state.

- **New decision:** founder attention as a zero-sum resource — which seven companies do I neglect this quarter? That is literally the seed-fund job.
- **Why it fixes everything:** parallelism produces the heavy-tailed outcome distribution the grade bands need; capital hunger creates the state where secondary sale wins; and every level you build becomes a *choice* rather than a *queue position*.
- **Reuses:** all existing level code as the micro-game. `spawnCompany()` becomes `enterCompany(id)`.
- **Effort:** medium (~2 days). **Risk:** medium — HUD/readability is the hard part.
- **Precedent:** Into the Breach's design core — *perfect information, tiny board, and a win condition that is protecting the grid rather than killing everything, so every turn is explicit triage: which building do I let die?* ([Game Developer](https://www.gamedeveloper.com/design/-i-into-the-breach-s-i-designers-explain-how-to-follow-up-from-a-hit-game), [GDC postmortem](https://gdcvault.com/play/1026333/-Into-the-Breach-Design))

### Option B — Escalating ante (cheapest real step-change)

Keep everything. Add a **visible, exponentially rising DPI bar** you must clear at the end of each fund cycle to raise Fund II, Fund III, Fund IV. Balatro's structure.

- **New decision:** concentrate early or die to the ante later.
- **Effort:** small (a number and a UI bar). **Risk:** low.
- **This is the highest ratio of tension-created to code-written in the whole document.** Balatro's entire tension comes from a fixed, visible, exponentially rising bar. Worth doing *regardless* of which container you pick.

### Option C — Split-lane board meeting

Three companies run simultaneously in stacked lanes; you control one at a time, the other two degrade on autopilot.

- **New decision:** attention as zero-sum, expressed in real time.
- **Highest comedy-to-mechanic fidelity of the three** — the joke *is* the control scheme.
- **Effort:** medium-high. **Risk:** high (readability, control confusion). A great prototype, a risky ship.

**My call: A + B.** C is a fantastic 2-hour experiment to run in the harness before committing.

### The meta-progression guardrail

If you add persistence: **unlock knowledge and pool-width, never raw power.** Balatro deliberately has almost no power meta-progression — unlocks only widen the pool ([Steam](https://steamcommunity.com/app/2379780/discussions/0/4346606879506457345/)). The consensus on stat-based meta-progression is that it corrodes roguelite tension ([ResetEra](https://www.resetera.com/threads/im-starting-to-feel-that-stat-based-meta-progression-is-starting-to-ruin-roguelites-generally-speaking.1509337/)). Unlock new **founder archetypes**, never more capital.

---

## 4. New levels — specs, not gimmicks

Each spec states the ONE decision, then the four *kishōtenketsu* beats. Build the beats or don't build the level.

> **Sequencing rule (P1 corollary):** alternate the pressure axis — execution → patience → execution → attention → execution. Two low-agency levels back to back (auto-scroll then waiting) will read as the game getting worse.
>
> **Recombination rule:** level N's gimmick must reappear as a *supporting* element under level N+2's rules. The cheapest new content is old content in new company.
>
> **And Tezuka's correction: build 1-1 last.** He confirms SMB 1-1 was authored at the very end, once the game was known, "while balancing the overall difficulty." Rebuild Demo Day as exam-prep for everything after it. ([shmuplations](https://shmuplations.com/supermariobros/))

---

### 2-1 — Competitive Process *(build this first)*

**Decision: speed vs. diligence.**

The auto-scroll *is* the 48-hour clock. The level is stuffed with optional **data-room documents** placed just behind the safe line. Grab them and you drift toward the left edge; skip them and you sign a worse deal — a materially weaker starting state at the flag (lower ownership → smaller sprite → harder next round).

**Why auto-scroll is normally hated, and the fixes:** it removes agency and decouples skill from speed — you can't go faster by being better, only die by being slower ([TV Tropes](https://tvtropes.org/pmwiki/pmwiki.php/Main/AutoScrollingLevel), [Giant Bomb](https://www.giantbomb.com/auto-scrolling-levels/3015-2299/forums/does-anyone-actually-like-auto-scrolling-levels-1470420/)). Four required mitigations:

1. **Give speed back.** Touching the right edge *accelerates* the camera. Mastery buys diligence time. This is the single biggest fix for the agency complaint — and it makes the diligence trade real rather than scripted.
2. **Diegetic justification.** The camera moves because *you* are attached to something — the deal process. DKC's Mine Cart Carnage works because "Motion = Tension… Stopping = Contact = Defeat" ([accordion sprout](https://www.accordionsprout.com/blog/donkey-kong-country-amp-mine-cart-carnage-the-joy-of-movement-1)).
3. **One-screen readable threat.** Place nothing lethal within `scroll_speed × 400ms` of the right edge.
4. **Checkpoint ~3× denser than a normal level.** The redo cost here is wall-clock you already spent, not skill you already expressed. Plus **3–4 deliberately empty rest beats** in a 90-second scroll.

**Beats:** *ki* — scroll starts slow, one doc on the safe line, free. *shō* — docs drift behind the line, over a pit. *ten* — **the scroll reverses**: a competing term sheet drags you left, and the back half is the same geometry read right-to-left. *ketsu* — max speed, no docs, pure execution to the flag.

---

### 3-1 — The AI Bubble

**Decision: when to be heavy.**

**Do not build a swim level.** Build an **inverted-gravity** level. Buoyancy is a constant upward force; the ceiling is the hazard, not the floor. You collect **revenue weights** that let you descend to the genuinely good stuff but slow you down. Everything floats up regardless of quality — so garbage platforms rise too, and the smart play is *riding trash upward*. That is the joke, charged rather than labelled.

**No oxygen timer.** ([Water-level complaints and fixes](https://culturedvultures.com/developers-struggle-get-water-levels-right-video-games/))

**Beats:** *ki* — float up a clear shaft, nothing to hit. *shō* — ceiling spikes; you need a weight to duck under. *ten* — a weight you're carrying is *itself* rising (it was fake revenue) and stops working mid-jump. *ketsu* — a vertical gauntlet alternating heavy and light.

---

### 4-1 — Capital Markets Frozen

**Decision: commit or brake.**

Ice grants a **higher top speed than the game otherwise allows**, and several gaps are *only* clearable at ice speed. Sliding is the tool, not the tax (P2). **Twist:** a section where friction abruptly returns and your muscle memory over-brakes you into a pit.

**Beats:** *ki* — flat ice, no hazards, learn the slide. *shō* — ice over a pit. *ten* — friction returns mid-run. *ketsu* — a gap that requires a full-length ice run-up with a Serial Pivot on it.

---

### 5-1 — The Down Round

**Decision: what do you accept.**

Falling valuations are **collectible**. Taking one restores light for ~6 seconds but *permanently marks down* your company's valuation. So the level is legible-and-poorer, or dark-and-heroic. That fixes darkness-being-just-subtraction (P2), and it is a beautiful, self-explanatory piece of the joke.

Never require blindness and precision simultaneously — lighting-design writing is blunt that realism is no defence, players still need to see ([Game Design Snacks](https://game-design-snacks.fandom.com/wiki/Overly_Dark_Lighting_May_Look_More_Realistic_%E2%80%93_But_Players_Still_Need_to_See)).

**Beats:** *ki* — dim, wide, safe; one valuation falls in front of you. *shō* — dark over a pit, valuation available. *ten* — the valuations start being *lethal on contact* unless you're standing still (you have to *accept* it, not catch it). *ketsu* — a stretch where the only light source is the mark-down you refuse to take.

---

### 6-1 — The Anti-Portfolio *(bonus / palate cleanser)*

**Decision: how greedy.**

Companies you passed on, now enormous and untouchable, are **moving terrain** — you platform *on* the thing you passed on. **No combat verb at all.** This is the deliberate low-intensity beat between execution levels (P1 sequencing). **Twist:** one of them notices you.

Only reachable if your anti-portfolio has ≥3 entries — which makes passing on deals a *route unlock*, retroactively adding weight to boss-room passes.

---

### 8-1 — The IPO Window

**Decision: what do you do while it's closed.**

Waiting is not a level; it's a frame. The door opens 20% of the time. **While it's shut, the screen is a small survival arena that gets denser** — and there are optional value-building pickups in the dangerous half. So the choice is *build value in the closed window (risky) vs. camp safe and walk through with nothing*.

**Requires a visible window-state indicator** — a countdown ring on the door. That is the entire difference between this being your best level and your worst: the wait must be **anticipation**, never confusion.

---

### One more level the brief doesn't have, which I think beats two of the above

### 7-1 — **The Board Meeting** *(attention level)*

Three lanes stacked vertically, three companies, one player. You're in one lane; the other two visibly degrade. Switch lanes with ↑/↓ at fixed junction points. Nothing kills you — you simply cannot save all three.

**Decision: which company do you let die.** It is Option C at level scope instead of game scope, which makes it a cheap, low-risk test of the highest-fidelity idea in this document. If it plays well, promote it to the container.

---

## 5. Game-design improvements

### 5.1 Fix the dominated secondary sale — by manufacturing its state, not buffing it

Sirlin: a dominant move *"is strictly better than any other you could do, so its very existence reduces the strategy of the game"*; viable options must be *"materially different from each other, not worthless, and not dominated."* Crucially, options need **not** be equal — they can be situational, so long as a state exists where they win. The fix is counterpicking: **create the matchup where the weak thing is right.** ([GDC 2009 handout, PDF](https://static1.squarespace.com/static/50f14d35e4b0d70ab5fc4f24/t/53ef1dbae4b0a6d424125a6f/1408179642248/GDC+2009+sirlin+handout6.pdf))

Add three states; secondary becomes correct under any one:

1. **Capital hunger** — reserves below one cheque and a follow-on is live *now*.
2. **A binding fund clock** — DPI must be *realised* before the clock ends; unrealised marks score zero. (Currently your clock only ends the run.)
3. **Per-company volatility** — a visible `σ` on each company. High σ + it's your only DPI = sell.

Under none of these, secondary should stay dominated. That's fine and correct.

### 5.2 Make TVPI actively lie ⭐ *the best single idea in this document*

Right now TVPI vs DPI is a **caption**. Make it a **cost**:

- Markups come from **rounds raised**, and you can *cause* rounds by spending reserves.
- So the big fun number top-left is **directly purchasable**, and buying it **directly consumes the capital that could have produced DPI**.
- P4 satisfied: one resource, two opposed goals. P5 satisfied: the joke is charged, not labelled.
- The end screen's sting writes itself — you watched a number climb, every action was locally correct, and almost none of it converted.

This is the Papers, Please move and it costs maybe 40 lines.

### 5.3 Make losing a *played move*

Slay the Spire's deck-thinning is instructive because it is **negative acquisition** — paying gold to have *less*. Players must learn that refusing is a strategy. That is exactly the emotional shape of "nine are supposed to die." Add an explicit **SHUT IT DOWN** action that is affirmatively good: returns a fraction of reserve, frees an attention slot, banks a token DPI. ([Giovannetti interview](https://justingarydesign.substack.com/p/anthony-giovannetti-crafting-slay))

And: **every dead company must print a one-line post-mortem the player can act on.** Loss is fun when the *player* gets better; it is not fun when it is opaque or unattributable ([Game Developer](https://www.gamedeveloper.com/design/what-do-you-mean-losing-is-fun-), [counter-view](https://game-wisdom.com/critical/debunking-losing-is-fun-game-design)). Nine deaths with no readable cause is nine units of noise.

### 5.4 Fix the two-band outcome ceiling

Publish **fixed vintage-benchmark quartile thresholds at fund start** (e.g. DPI 1.0x / 1.8x / 3.0x), visible in the HUD mid-run. Bands must be **aimed at**, not fallen into. The moment the player can see "I'm on track for second quartile; going for top decile means concentrating everything into #7 right now," you have created the game's best decision for free.

Supporting theory: Geoff Engelstein argues games feel good when outcomes are power-law shaped, and recommends **power-law distribution of decision importance** — mostly inconsequential choices punctuated by rare critical ones ([GameTek](https://gametek.substack.com/p/power-laws-and-game-design)). Your run should contain ~3 decisions that matter enormously; the skill is **identifying which**.

### 5.5 Founder archetypes — the cheapest real variety in the game

Eight archetypes with **multiplicative** traits, drafted 10-from-a-shuffled-deck each run:

| Archetype | Trait |
|---|---|
| Repeat Founder | ×2 follow-on efficiency |
| Fake It | ×3 TVPI, ×0.2 DPI |
| Technical Founder | slow markup, no cap on exit multiple |
| Sales Founder | fast markup, hard exit ceiling |
| The Grinder | immune to one death per run |
| Momentum Chaser | scales with *other* companies' markups |
| Solo Founder | half the reserve cost, half the ceiling |
| Ex-FAANG | huge seed valuation, so your ownership starts small |

LocalThunk built Balatro's depth from **joker × joker interaction**, not joker count — he started with *no jokers at all* ([Rogueliker](https://rogueliker.com/balatro-interview/)). Eight traits that multiply beat forty companies that don't interact. **This is n² variety from n content, and it is a data table, not a system.**

### 5.6 Boss legibility and boss #2

Mike Stout's canonical boss structure: Build-Up → Intro → **Business As Usual** → **Escalation** → **Midpoint (false victory / transformation)** → It's ON! → Kill → Victory ([Game Developer](https://www.gamedeveloper.com/design/boss-battle-design-and-structure)).

Your no-health-bar VC is a strong idea that is currently **illegible**. A health bar is a *legibility* device, not a combat device — removing it is only good if something else communicates progress. Bosses without bars all substitute readable physical tells ([discussion](https://www.neogaf.com/threads/bosses-without-health-bars.544668/)).

- **Keep** the "post-money only goes up" gag. **Add** state tells: he visibly sheds associates; the arena fills with term sheets; the `!` telegraph shortens each phase. The player must answer "am I winning?" from the screen alone.
- **Add the midpoint.** He announces the round is closed, appears beaten — then returns, and *every previous attack gains one modifier* (leaves a hazard on landing).
- **Boss 2 must change the verb, not the numbers.** Boss 1 asks "can you read a telegraph?" Boss 2 (**The Crossover Fund** — invulnerable, arrives late, prices you out) should ask **"can you use the level against it?"** Boss 3 (**Your Own LP**) should ask **"can you resist?"** — it offers you power-ups that are traps.
- **Every boss teaches the next world's gimmick.** Stout's Build-Up beat, inverted.

### 5.7 Game feel — you're in good shape, two things left

Your L1 numbers are all green (coyote 83ms, buffer 83ms, 1-frame latency). Two additions worth the bytes:

- **Hit-stop on the *exit*, not just the stomp.** You have `freeze = 5` on exit; consider 8–10 frames plus a brief zoom. The exit is the payoff of the entire thesis and should feel disproportionate.
- **Layered audio on the auto-scroll level** — DKC's mine cart works partly because the wheel rumble *cuts* on jump and the landing has a distinct reward tone. Cheap, and it does most of the flow work in 2-1.

---

## 6. New NPCs — twelve questions, ship five

Written as questions per P3. **Roster discipline: ship 4–5, not 12.** #8 alone multiplies your existing roster more than five new sprites would.

| # | NPC | Question it asks |
|---|---|---|
| 1 | **The Ghost Tier-2** | *Can you move backward under pressure?* Follows only while the camera scrolls right; freezes when you back up. (Boo verb, VC skin.) |
| 2 | **The Acquihire** | *Spend it now or carry it?* Invulnerable, but stomping it yields a carryable block/projectile. |
| 3 | **The Board Observer** | *Can you keep moving while doing something else?* Hovers offscreen-top, drops governance hazards on your exact X. (Lakitu.) |
| 4 | **The Bridge Round** | *Can you commit to close or far?* Arcing term sheets — safe directly beneath and far away, lethal in the middle band. (Hammer Bro.) |
| 5 | **The Follow-On** | *Do you deal with problems early?* Harmless on first contact; lethal and faster on the second, anywhere later in the level. |
| 6 | **The Zombie Unicorn** | *Can you make room?* Huge, slow, unkillable — only pushable. Doubles as movable terrain. |
| 7 | **The Secondary Buyer** | *Can you plan a path that kills your own reflection?* Mirrors your inputs on a 1s delay across a vertical axis. |
| 8 | **The Fund of Funds** ⭐ | *Which threat do you triage?* Spawns two smaller copies of **whatever type is already on screen**. Makes the existing roster combinatorial for free. |
| 9 | **The Vesting Cliff** | *Can you read a clock while jumping?* Platform solid 3s, gone 1s, with a visible four-year progress ring. |
| 10 | **The Pro-Rata** | *Can you resist rushing?* Sits on a collectible; shields if approached fast, open if approached slow. |
| 11 | **The Ratchet** | *Is avoidance worth a detour when damage isn't visible?* Contact doesn't hurt — it permanently raises scroll speed / lowers your jump. |
| 12 | **The LP** | *Does the player change behaviour when observed?* Non-combat, stands at fixed points. Every hit taken in its sight closes an exit later in the level. |

**My five:** #8 (Fund of Funds), #4 (Bridge Round — you have the spacing gap), #9 (Vesting Cliff — pure platforming, zero new AI), #12 (The LP — best joke-to-code ratio in the list), #1 (Ghost Tier-2, for 2-1's reverse-scroll twist).

**Also apply de Heras' free-variety rules to what you already have:** give the Serial Pivot a *silhouette that changes with its form* (form foreshadows function), and calibrate its aggro range just outside your stomp range so baiting becomes a skill. Both are a few lines and both make an existing enemy feel new.

---

## 7. New challenges (modes, not levels)

1. **The Ante Run** — Funds I→IV with an exponentially rising DPI bar. §3 Option B.
2. **The Anti-Portfolio Gauntlet** — unlocked by passing on ≥3 deals; every company you passed on returns as terrain. Makes passing a *route*.
3. **Vintage Year** — a seeded market regime announced *before* the run (ZIRP / Correction / AI Mania) that modifies archetype traits. **Telegraphed, not random** — Into the Breach's perfect-information rule. Removes noise while adding depth, and satisfies your "no randomness after a decision" invariant cleanly.
4. **The LP Meeting** — a 60-second end-of-fund boss where you *defend* your decisions; the questions are generated from your actual run log.
5. **Conviction Mode** — you may write only three cheques, ever.
6. **Committee Mode (asynchronous multiplayer)** — you see the three most common decisions of everyone who played today's Daily *after* you commit. Cheap, and it makes the shared substrate visible.

---

## 8. Virality: the share card is currently a receipt

Scored against the principles below, your current card rates **17/50**. Worst lines: **`TOP DECILE`** (self-conferred status with no denominator — the purest brag anti-pattern) and **`DPI 6.49x`** headlining a *sum* metric in a game whose entire thesis is that only the best exit counts. **The card contradicts the design.**

### What actually makes a result spread

1. **The artifact must be a story, not a score.** The Wordle grid wasn't Wardle's idea — a player in New Zealand was hand-typing emoji grids; he automated it. His framing: it lets you *"share your story, the journey you took to get to the answer"* without revealing it. **Process is discussable; outcome is not.** ([Syntax #430](https://syntax.fm/show/430/creator-of-wordle-josh-wardle/transcript), [TechCrunch](https://techcrunch.com/2022/01/12/josh-wardle-interview-wordle/))
2. **Spoiler-preservation converts a brag into an invitation.** Withholding the answer means posting costs the reader nothing and opens a curiosity gap.
3. **No link — the grid *is* the ad.** Wardle: *"Not including a link was… the opposite of what you're meant to do… Just removing it made all that simpler. They were sharing for themselves."* The instant a card looks like a growth loop, readers discount it and the sharer feels like an affiliate. ([Slate](https://slate.com/culture/2022/01/wordle-game-creator-wardle-twitter-scores-strategy-stats.html)) **You currently append `SITE_URL`. Cut it.**
4. **A shared substrate, made visible.** Dailies work because permadeath sets stakes and a shared seed means players *"inhabit the same gameworld"* — a ritualization of play ([Playing Software](https://playingsoftware.substack.com/p/the-ritual)). Clones failed because they fragmented the audience: posting a niche variant's result *"just feels like shouting into the void."*
5. **Variance must be dramatic *and attributable*.** This is why most dailies *don't* spread. Slay the Spire's Daily Climb is structurally perfect but outputs a *leaderboard row*, not a portable artifact. Isaac's dailies are worse — players say scores are dominated by RNG and exploit knowledge, so nobody respects the number and nobody posts it. **Score legitimacy is a precondition of sharing.**
6. **Losing must be postable — ideally more so than winning.** X/6 spread as widely as 2/6; Connections' cultural energy is the shared grievance about purple. Near-misses are stickier than wins.
7. **Social currency ≠ looking skilled; it means looking *interesting*.** A result that says something about *you* — your temperament, your taste — beats one that says only how well you did.

**Anti-patterns you're currently hitting:** self-graded tier with no denominator; a URL in the card; no journey; a wipeout card that's a naked humiliation with no shape — which matters enormously here, because **by design most of your players wipe out**.

### The redesign

```
SUPER CARRY BROS.
FUND #2 "THE COLD WINTER"

🌱🟩🟩💀
🌱💀
🌱🟩💰🦄
🌱✋ … 🦄 $1.2B

best exit $204M (20.4x)
beat 71% of 4,180 funds today
CONVICTION BET · 6% held #3
```

Same length. Twelve moves in it:

1. **Rows, not a row.** One row per company, one column per stage. `🌱🟩🟩💀` reads as *"looked great, died at B."* The row is the story.
2. **Encode decisions, not just outcomes.** `💰` follow-on, `✋` pass, `…` held. "Here's what I *did*" is defensible in a reply; "here's what happened to me" isn't.
3. **Lead with best exit.** Kill DPI from the headline — the card must not imply the sum matters.
4. **The counterfactual line** — highest-leverage single addition. `passed on: 🦄 → $1.2B`. Regret is the most conversational output a VC game can produce, and Bessemer's anti-portfolio is already a beloved real-world ritual. **Half your cards should have a heartbreak line.**
5. **Replace `TOP DECILE` with a real denominator.** `beat 71% of 4,180 funds today`. This is the Slay-the-Spire property delivered in a copyable format Spire never had. (Needs a tiny counter endpoint — weigh against your zero-request invariant; a fixed published benchmark from §5.4 is the zero-dependency substitute.)
6. **A divergence marker — your "purple."** `6% held #3`. Rarity-of-correct-decision is a better flex than magnitude, and it produces the best possible reply: *"wait, YOU held that one?"*
7. **Archetype from decision pattern, not score.** `CONVICTION BET`, `SPRAY & PRAY`, `TOURIST`, `ZOMBIE LANDLORD`, `DOWN-ROUND DENIER`, `MOMENTUM CHASER`. This is the identity payload, and it gives losers a postable card.
8. **Make the zero card the funniest artifact in the game.** `DPI 0.00x · 🪦🪦🪦🪦 · returned $0 of $100M · archetype: DILIGENT`. If nine in ten die by design, the wipeout *must* be your most shareable output or most players never post.
9. **Name the day.** `FUND #2 "THE COLD WINTER"`. A proper noun gives conversation a handle; a number doesn't.
10. **Never name the companies.** Sector glyphs only — preserves spoiler-safety so early-risers can post without social cost, which is what lets the cascade build through the day.
11. **Field histogram.** `field: ▁▃█▅▂▁ ▲you` — shows *where today was hard*, which is what everyone wants to talk about.
12. **Discipline:** ≤5 lines, ≤16 emoji, **no URL, no hashtag**, and one plain-prose line at the end. Screen readers render every square verbatim — *"it takes my iPhone over a minute to read it"* — and colorblind users can't parse grids ([Slate](https://slate.com/culture/2022/02/wordle-word-game-results-accessibility-twitter.html)).

**Net:** the current card contains zero replies waiting to happen. The redesign contains five — a journey, a heartbreak, a real denominator, a rare decision, and an identity.

---

## 9. Build order

Ordered by (fun delivered ÷ effort), with harness acceptance criteria. Commit before each phase.

### Phase 0 — Refactors (½ day, zero behaviour change)
- Level registry (`LEVELS` map, `G.levelId`, per-run playlist)
- NPC behaviour table (`NPC[type].update/draw`)
- Purge dead legend entries (`b`, `^`)
- **Accept when:** harness output is byte-identical to the pre-refactor run.

### Phase 1 — The cheap step-changes (1 day) ⭐ *biggest fun-per-hour*
- **§5.2 TVPI actively lies** (markups purchasable with reserves)
- **§5.4 published quartile thresholds**, visible mid-run
- **§5.1 the three states** that rescue secondary sale
- **§3-B escalating ante** bar
- **Accept when:** `Secondary sale is dominated` WARN clears at 133ms reaction delay; ≥3 grade bands appear across seeded bot lines; no new invariant violations.

### Phase 2 — Level 2-1 Competitive Process (1 day)
Full four-beat build, all four auto-scroll mitigations, reverse-scroll twist.
- **Accept when:** `Content variety` FAIL downgrades to WARN; difficulty-tracks-skill probe improves; the `engaged` bot's completion rate sits in 55–80% (not 100%, not 20%).
- **Experiment first, per your own workflow:** patch in the "right edge accelerates the camera" rule alone and judge `(baseline, variant)` — remembering `judge(b, v)` takes both args. My prediction is it dominates the fixed-speed variant; if the harness disagrees, it's right and I'm wrong.

### Phase 3 — Five NPCs + founder archetypes (1 day)
Fund of Funds, Bridge Round, Vesting Cliff, The LP, Ghost Tier-2. Eight archetypes as a data table.
- **Accept when:** `Content variety` clears; L4 depth probe finds new headroom.

### Phase 4 — Share card redesign (½ day)
All twelve moves; cut `SITE_URL`; make the zero card the funniest.
- **Accept when:** a wipeout run and a top-decile run both produce cards you'd actually post.

### Phase 5 — Levels 3-1, 4-1, 5-1, 7-1 (2–3 days)
One per day, four beats each, recombining prior gimmicks.

### Phase 6 — The container (§3 Option A, 2 days)
Only after Phases 1–4 prove the decisions are good. Node map + attention budget.

**Cumulative: ~8 days to a genuinely different game; ~2.5 days (Phases 0–2) to kill the FAIL.**

---

## 10. What NOT to build

- **Six levels before fixing the decision layer.** Novelty papers over a shallow system for exactly as long as content lasts. You already wrote this in your own brief; the research agrees emphatically.
- **A literal swim level, a literal slippery-ice level, or a literal dark level.** Every one is a subtraction. Trade or don't ship (P2).
- **Randomness to create variety.** Noise is any pattern we don't understand. It breaks your fairness invariant *and* Koster's condition simultaneously.
- **Stat-based meta-progression.** Corrodes run-to-run tension. Knowledge and pool-width only.
- **More enemies that ask "when do I jump?"** You have two. That question is answered.
- **A URL in the share card.**

---

## Sources

**Level & enemy design:** [Hayashida on kishōtenketsu / Game Developer](https://www.gamedeveloper.com/design/the-structure-of-fun-learning-from-i-super-mario-3d-land-i-s-director) · [The secret to Mario level design](https://www.gamedeveloper.com/design/the-secret-to-i-mario-i-level-design) · [GMTK four-step, via Nintendo Life](https://www.nintendolife.com/news/2015/03/video_nintendos_four_step_stage_design_is_why_you_love_super_mario_games_so_much) · [SMB developer interview, shmuplations](https://shmuplations.com/supermariobros/) · [de Heras on Hollow Knight enemies](https://www.jasondeheras.com/gamedesign/2021/3/6/early-game-hollow-knight-enemies) · [GameDesignSkills: Enemy Design](https://gamedesignskills.com/game-design/enemy-design/) · [Celeste level design, Thorson](https://www.gamedev.net/news/level-design-in-celeste-as-told-by-matt-thorson-r1022/) · [Auto-Scrolling Level, TV Tropes](https://tvtropes.org/pmwiki/pmwiki.php/Main/AutoScrollingLevel) · [DKC Mine Cart Carnage](https://www.accordionsprout.com/blog/donkey-kong-country-amp-mine-cart-carnage-the-joy-of-movement-1) · [Downwell GDC: Polishing the Boots](https://gdcvault.com/play/1023533/Polishing-the-Boots-Designing-Downwell) · [Endless runner difficulty curves](https://www.orcunnisli.com/post/2015/11/22/endless-runners-procedural-map-generation-and-difficulty-curves) · [Why underwater levels suck](https://orangejuiceliberationfront.com/why-underwater-levels-suck-so-often/) · [Overly dark lighting](https://game-design-snacks.fandom.com/wiki/Overly_Dark_Lighting_May_Look_More_Realistic_%E2%80%93_But_Players_Still_Need_to_See) · [Stout: Boss Battle Design and Structure](https://www.gamedeveloper.com/design/boss-battle-design-and-structure) · [Bosses without health bars](https://www.neogaf.com/threads/bosses-without-health-bars.544668/)

**Depth, structure, loss:** [Defining Depth in Game Design](https://www.gamedeveloper.com/design/defining-depth-in-game-design) · [Tyroller / Thronefall on minimalism](https://www.gamedeveloper.com/design/mastering-minimalism-and-layering-complexity-with-strategy-game-thronefall) · [Into the Breach designers](https://www.gamedeveloper.com/design/-i-into-the-breach-s-i-designers-explain-how-to-follow-up-from-a-hit-game) · [ItB GDC postmortem](https://gdcvault.com/play/1026333/-Into-the-Breach-Design) · [Downwell gunboots, Kill Screen](https://killscreen.com/previously/articles/the-tricky-brilliance-of-downwells-gunboots/) · [LocalThunk / Balatro interview](https://rogueliker.com/balatro-interview/) · [Engelstein: Power Laws and Game Design](https://gametek.substack.com/p/power-laws-and-game-design) · [Giovannetti on Slay the Spire](https://justingarydesign.substack.com/p/anthony-giovannetti-crafting-slay) · [Koster, A Theory of Fun](https://www.goodreads.com/work/quotes/19639-a-theory-of-fun-for-game-design) · [What do you mean, losing is fun?](https://www.gamedeveloper.com/design/what-do-you-mean-losing-is-fun-) · [Debunking losing-is-fun](https://game-wisdom.com/critical/debunking-losing-is-fun-game-design) · [Sirlin, GDC 2009 (PDF)](https://static1.squarespace.com/static/50f14d35e4b0d70ab5fc4f24/t/53ef1dbae4b0a6d424125a6f/1408179642248/GDC+2009+sirlin+handout6.pdf) · [Meta-progression design](https://bugnet.io/blog/how-to-design-a-roguelite-meta-progression)

**Comedy premises:** [Lucas Pope interview, Reason](https://reason.com/2013/09/26/papers-please-politics-in-games-and-the/) · [Universal Paperclips, IF50](https://if50.substack.com/p/2017-universal-paperclips) · [Orteil on Cookie Clicker](https://www.vice.com/en/article/cookie-clicker-wasnt-meant-to-be-fun-why-is-it-so-popular-8-years-later/) · [Reigns and the loop](https://www.gamedeveloper.com/design/why-being-trapped-in-a-loop-makes-i-reigns-her-majesty-i-so-satisfying) · [Crawford, A Dark Room GDC talk](https://vimeo.com/91436410)

**Virality:** [Slate / Wardle on stripping the link](https://slate.com/culture/2022/01/wordle-game-creator-wardle-twitter-scores-strategy-stats.html) · [TechCrunch / Wardle](https://techcrunch.com/2022/01/12/josh-wardle-interview-wordle/) · [Syntax #430 transcript](https://syntax.fm/show/430/creator-of-wordle-josh-wardle/transcript) · [Why Wordle won](https://nishad.substack.com/p/why-wordle-won) · [Playing Software: The ritual](https://playingsoftware.substack.com/p/the-ritual) · [StS Daily Climb](https://slaythespire-archive.fandom.com/wiki/Daily_Climb) · [Isaac daily scoring complaints](https://steamcommunity.com/app/250900/discussions/0/133256689833429234/) · [Balatro virality breakdown](https://bysolopreneurs.com/how-balatros-turned-into-an-indie-hit/) · [Why these games went viral](https://dawnosaur.substack.com/p/why-did-these-games-go-viral) · [Clone fragmentation, TheGamer](https://www.thegamer.com/wordle-clones-communal-phase-ended-framed-heardle-worldle-dordle/) · [Connections and purple, Slate](https://slate.com/life/2024/07/connections-nyt-today-wordle-wyna-liu.html) · [Share-card accessibility](https://slate.com/culture/2022/02/wordle-word-game-results-accessibility-twitter.html) · [Berger, STEPPS](https://cronkitehhh.jmc.asu.edu/blog/2017/04/jonah-berger-reveals-secret-contagiousness) · [Near-miss effect](https://en.wikipedia.org/wiki/Near-miss_effect)
