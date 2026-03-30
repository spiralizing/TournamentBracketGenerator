# Tournament Bracket Generator

![Tournament Bracket Generator](preview.png)

A double-elimination bracket generator for tournaments of any size. Enter player names, shuffle, and run through the bracket stage by stage.

## Features

- Works for any number of players (4+), no byes needed
- Randomly shuffles seeding on each run
- Stage-by-stage interactive flow: select heat winners, advance, repeat
- Scrollable bracket overview diagram showing all stages, filling in as you play
- Live R1 heat preview on the input screen

## You can

**Open it locally.** Clone this repo and open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge). It loads React from a CDN on first open, so an internet connection is needed the first time.

**Host it or visit the associated webapp for this repo.** 

https://spiralizing.github.io/TournamentBracketGenerator/

---

## Bracket Format Design

### Overview

Races support 2, 3, or 4 players per heat. This section describes a general double-elimination bracket format for any number of players N >= 4, using heats of these sizes. The format guarantees that every participant plays at least two matches before being eliminated.

### Core Mechanism

Each heat advances the top half and drops the bottom half:

| Heat size | Advance | Drop |
|-----------|---------|------|
| 4 players | Top 2 | Bottom 2 |
| 3 players | Top 2 | Bottom 1 |
| 2 players | Winner | Loser |

The general rule is: a heat of size $k$ advances $\lceil k/2 \rceil$ and drops $\lfloor k/2 \rfloor$.

A round with $M$ active players always produces $\lceil M/2 \rceil$ advancers and $\lfloor M/2 \rfloor$ drops.

### Bracket Structure

The tournament has three zones:

**Winners Bracket (WB)** contains undefeated players. After each heat, the bottom half drops to the Losers Bracket.

**Losers Bracket (LB)** contains players with one loss. After each heat, the bottom half is eliminated (second loss). Each round, the LB receives fresh drops from the WB and merges them with existing LB survivors.

**Grand Finals** is a 2-player match between the WB champion and LB champion. The WB side holds an advantage (bracket reset, point lead, or similar).

#### LB Round Types

The Losers Bracket alternates between two types of rounds:

- **Reduction rounds**: only LB survivors play, no new WB drops enter. This shrinks the LB pool.
- **Merge rounds**: LB survivors combine with incoming WB drops, then play heats.

This staggering technique ensures that merge pools land on manageable sizes.

### The Partition Rule

Given $M$ players in a round, decompose into heats using the following table:

| $M \bmod 4$ | Heats of 4 | Heats of 3 | Heats of 2 | Byes |
|-------------|-----------|-----------|-----------|------|
| 0 | $M/4$ | 0 | 0 | 0 |
| 1 | $(M-5)/4$ | 1 | 1 | 0 |
| 2 | $(M-2)/4$ | 0 | 1 | 0 |
| 3 | $(M-3)/4$ | 1 | 0 | 0 |

**Zero byes are required, for any $M \geq 2$, always.** A single 3-player heat absorbs any odd remainder.

Special cases for small $M$: when $M = 2$, use one 2-player heat. When $M = 3$, use one 3-player heat. When $M = 5$, use one 3-player heat and one 2-player heat.

### Why This Works for Any N

The halving operation $M \mapsto \lceil M/2 \rceil$ could introduce problems when $M$ is odd, but the 3-player heat absorbs the remainder cleanly. When $M$ is odd, exactly one heat of 3 is used: two players advance, one drops. The rest of the round uses 4-player heats as usual. No player sits out, no byes are needed, and this holds for every $N \geq 4$.

#### Fairness Note

In a 3-player heat, the bottom player has a $1/3$ chance of dropping versus $1/2$ in a 4-player heat, making the 3-player heat slightly more forgiving for its participants. To mitigate this, assign 3-player heats to the lowest-seeded group so the advantage goes to the players who need it most, or rotate 3-player heat assignment randomly each round.

### Ideal Player Counts

With {2, 3, 4}-player heats, **every $N \geq 4$ works cleanly**. No player count is structurally disadvantaged. For practical purposes, the best counts are those that minimize 3-player heats:

| Preference | Player counts | Reason |
|------------|--------------|--------|
| Optimal | Multiples of 4 (4, 8, 12, 16, 20) | All heats are 4-player. Maximum balance. |
| Good | Even numbers (6, 10, 14, 18) | One 2-player heat per round, rest are 4-player. |
| Fine | Odd numbers (5, 7, 9, 11, 13) | One 3-player heat per round. Fully functional, zero byes. |

If you have the freedom to choose $N$, multiples of 4 give the tidiest brackets. But any number works.

### Worked Example: N = 8

All even intermediate sizes, no 3-player heats needed.

```
WB-R1:  [P1 P2 P3 P4]  [P5 P6 P7 P8]     ← 2 heats of 4
            ↓    ↓          ↓    ↓
WB-R2:  [Top2A  Top2B]                      ← 1 heat of 4
            ↓    ↓
WB-Final: [1st  2nd]                        ← 2-player

LB-R1:  [Bot2A  Bot2B]                      ← 1 heat of 4
            ↓
LB-R2:  [LB-R1 survs + WB-R2 drops]        ← 1 heat of 4
            ↓
LB-Semis: [2 survs + WB-Final drop]        ← 1 heat of 3
            ↓
LB-Final: [top 2]                           ← 2-player

Grand Final: WB Champion vs. LB Champion    ← 2-player
```

Minimum races per player: **2**. Maximum (LB champion path): **6**.

### Worked Example: N = 7

```
WB-R1:  [P1 P2 P3 P4]  [P5 P6 P7]         ← 1 heat of 4 + 1 heat of 3
            ↓    ↓        ↓    ↓
WB-R2:  [4 advancers]                       ← 1 heat of 4
            ↓    ↓
WB-Final: [1st  2nd]                        ← 2-player

LB-R1:  [2 drops + 1 drop = 3]             ← 1 heat of 3
            ↓
LB-R2:  [2 survs + 2 WB-R2 drops = 4]     ← 1 heat of 4
            ↓
LB-R3:  [2 survs + 1 WB-Final drop = 3]   ← 1 heat of 3
            ↓
LB-Final: [top 2]                           ← 2-player

Grand Final: WB Champion vs. LB Champion
```

Zero byes. Every player races at least twice.

### Worked Example: N = 13

$13 \bmod 4 = 1$, so the partition is: $(13-5)/4 = 2$ heats of 4, 1 heat of 3, 1 heat of 2. That accounts for $2(4) + 3 + 2 = 13$. ✓

```
WB-R1:  [4] [4] [3] [2]                    ← 13 players → 7 advance, 6 drop
WB-R2:  [4] [3]                            ← 7 players → 4 advance, 3 drop
WB-R3:  [4]                                ← 4 players → 2 advance, 2 drop
WB-Final: [2]                              ← 2 players → 1 WB champion, 1 drops

LB-R1:  [4] [2]                            ← 6 drops from WB-R1 → 3 survive, 3 eliminated
LB-R2:  [4] [2]                            ← 3 survs + 3 WB-R2 drops = 6 → 3 survive, 3 elim
LB-R3:  [3] [2]                            ← 3 survs + 2 WB-R3 drops = 5 → 3 survive, 2 elim
LB-R4:  [4]                                ← 3 survs + 1 WB-Final drop = 4 → 2 survive, 2 elim
LB-Final: [2]                              ← 1 LB champion

Grand Final: WB Champion vs. LB Champion
```

Clean routing, zero byes, 13 players fully accommodated.

### The Swiss Alternative

For very large $N$ (above ~16) or when scheduling simplicity matters more than elimination drama, a **Swiss system** is an alternative. Swiss rounds are non-eliminative: all players race every round, grouped by cumulative score, and elimination occurs only at a defined cut point. This can be simpler to organize for casual settings where tracking WB/LB routing is impractical.

### Summary

The partition rule decomposes any $M \geq 2$ into heats of 4, 3, and 2 with zero byes, using a single lookup table indexed by $M \bmod 4$. Combined with double elimination (WB/LB structure), this produces a universal tournament format that works for every $N \geq 4$, guarantees every player participates at least two times, and requires no special-case routing or scheduling.

#### AI Disclosure statement
I used `Claude Opus 4.6` to design and implement the code for the bracket generator in this repository, as well as its html design. I used gemini (nano banana) to generate the image for the README and thumbnails.