# Super Carry Bros.

A side-scroller where you run a $100M seed fund. You start with ten portfolio
companies. Nine of them are supposed to die.

**[Play it →](https://tmubashir.github.io/super-carry-bros/)**

## Controls

| Key | Does |
|---|---|
| `←` `→` or `A` `D` | Move |
| `Space` (hold it) | Conviction jump — bigger check, further to fall |
| `X` | Secondary sale. 50% of the mark, guaranteed, run over. |
| `B` | Skip straight to the hot deal |

## The rule that makes it not a platformer

Your score is your **single best exit**, not the sum of what you collected.
Losing companies is survivable. Whiffing the breakout is not.

TVPI is the enormous number in the top-left and it climbs on every pickup. DPI
is the small grey one underneath it. The end screen grades you on the small
grey one.

## The Tier 1 VC

He stands between you and the flag. Three hits and he walks, so you will always
get to sign. The only question is what you paid for it.

He has no health bar. He has a post-money, and it only goes up. Every hit you
land is a bid — plus $45M, and your ownership falls with it. But he ground-pounds,
and for two seconds afterward he is winded. Hit him *there* and it's free.

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
