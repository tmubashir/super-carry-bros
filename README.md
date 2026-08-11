# Super Carry Bros.

A side-scroller where you run a $100M seed fund. Ten portfolio companies are
your lives, worlds are your companies' journeys, and every level ends at a
round with someone standing in front of the signature. Kill your way through
both worlds and you have returned the fund. Nine companies are still supposed
to die on the way.

**[Play it →](https://tmubashir.github.io/super-carry-bros/)**

## Controls

| Key | Does |
|---|---|
| `←` `→` or `A` `D` | Move |
| `Space` (hold it) | Conviction jump — bigger check, further to fall |
| `Enter` | **The Daily Fund** — 4 companies, ~2 minutes, same seed for everyone today |
| `R` | Raise a round. Costs reserves, marks you up 2.5x, and every cent of it is air. |
| `X` | Secondary sale — sells the *mark* at 50%. Sometimes that beats holding. |
| `B` | Skip straight to the hot deal |
| `C` (end screen) | Copy your fund card — emoji portfolio, DPI, quartile — and go paste it somewhere loud |

## The rule that makes it not a platformer

Your score is your **single best exit**, not the sum of what you collected.
Losing companies is survivable. Whiffing the breakout is not.

TVPI is the enormous number in the top-left and it climbs on every pickup. DPI
is the small grey one underneath it. The end screen grades you on the small
grey one.

And TVPI is **purchasable**. Press `R` to raise a round: it costs dry powder,
marks you up 2.5x, shrinks your sprite because you own less — and every cent of
the markup converts to actual cash at fifteen cents. Three rounds takes you to
a $156M mark that exits at $32M, while the pace line quietly reads FOURTH
QUARTILE. The capital you spent making the number beautiful is the same capital
that would have made it real.

Once a mark is mostly air, selling it to someone else at half price beats
holding it. That is not a bug.

## World 2: Sand Hill Road

Beat the Tier 1 VC and the world changes under you — daytime, golden hills,
palm trees, the 101 on the horizon. Two new things live there:

- **Anchors.** Liquidation preference hangs from the sky on chains. It
  telegraphs, it drops, and it does not care about your momentum or your
  markups. Getting hit reads you the sentence it exists for.
- **AI ACCELERATION.** A pickup that makes you faster and lets you run
  straight through founders' problems for nine seconds. It does nothing
  about the anchors, because it is immunity to competition, not to the
  capital structure.

At the end of it: **the debt holders.** Same fight, worse stakes — every hit
you land adds debt that sits ahead of you, and the clock is a maturity date.
Beat them, sign the recap, and you have returned the fund. That is the game.

## The Tier 1 VC

Every level ends at him — there is no flag until he is out of the deal, and
there is no door. Three hits and he folds; the flag is behind him. The only
question is what you paid for it: hits while he is WINDED are free, and any
other hit is a bid that comes straight out of your exit. Losing the room —
to his attacks or to the clock — writes the company off and the next one
retries the level.

He has no health bar. He has a post-money, and it only goes up. Every hit you
land is a bid — plus $45M, and your ownership falls with it. But every attack
telegraphs, every attack leaves him winded after, and if you bait his dash into
a wall he is winded for a long time. Hit him *there* and it's free. Brushing
past him while he strolls doesn't kill you — he just shoves you and says it's
great to see you.

- **Three free hits:** $60M entry, you own 16.7%, and this one can return the fund.
- **Three impatient ones:** $195M entry, you own 3.5%, and you have bought a
  perfectly decent company that cannot possibly matter.

Same three hits, same flag, same triumphant gold banner either way. Only the
ownership number changes colour.

## Also in there

Dilution that shrinks your sprite every round, pro rata that grows it back,
reserves you can strand entirely at seed, follow-on pricing that ratchets round
over round, a 10-year fund clock, an LP who will not stop emailing, and an
anti-portfolio that waits until the end screen to tell you what the ones you
missed are worth now.

## Running it locally

One file. No build step, no dependencies, no package manager.

```
open index.html
```

Everything is drawn on canvas and the audio is synthesized at runtime, so
nothing is fetched from anywhere. Works offline, works from a `file://` URL,
works if you email it to someone.

## Credits

Built with [Claude Code](https://claude.com/claude-code).
The game is a VC laughing at VCs. Founders come out of it fine.
