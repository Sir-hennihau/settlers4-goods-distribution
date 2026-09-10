# How Settlers IV Distributes Goods

A reverse-engineered account of the goods-distribution system in **The Settlers IV (History Edition)** —
what the percentage sliders in the sidebar actually do, why they so often don't do what you asked,
and what to set them to for tier-3 soldier production.

**Read it here → https://sir-hennihau.github.io/settlers4-goods-distribution/**

## What's in it

- **The two-stage solver.** Stage 1 picks the consumer building *type* from your sliders and a hidden
  delivery ledger; stage 2 picks *which* building of that type from stock and distance. Only stage 1
  looks at your sliders.
- **The shared ledger.** One 16-bit counter per building type at `CEcoSector+0x1C4`, shared across
  every good that has a slider, never reset for the whole match, and saved into your savegame.
  This is why the iron-bar slider silently spends the weaponsmith's coal budget.
- **Ore doesn't count.** Goods with no slider (iron ore, gold ore, stone, sulfur…) skip stage 1
  entirely and never touch the ledger — which is exactly why the smelters out-compete the smiths.
- **The dispatch ceiling.** 2 coal jobs + 1 of every other good per 16-tick cycle at 71 ms =
  **105.63 coal/min per economy sector**, or **26.4 tier-3 soldiers/min**. This reproduces the
  community's measured `Ausreizung Tragelimit` figures for all five builds.
- **The correct sliders,** derived and simulated, plus how mid-game tweaking incurs a computable
  catch-up debt.

## Method

Static analysis of `S4_Main.exe` (History Edition, md5 `153c49ab29946c21d50a3ae7a95c5cf8`).
The shipped `S4_MainR.pdb` is a public-symbol PDB carrying 125,554 named addresses, which anchored
the search; everything else was read out of the disassembly at the addresses cited in the page's
left margin. Recipes were confirmed against `GameData/buildingInfo.xml` and building names resolved
through `Txt/s4_texts.dat0`. Measured figures come from re-implementing the solver exactly as
disassembled.

No game file was modified at any point.

## Key addresses

| Address | What |
|---|---|
| `0x004D5D30` | `CEcoSector::DistributeGoods` — the whole dispatcher |
| `0x004D5F04` | the score: `(counter[type]*256 + 128) / sliderPercent` |
| `0x004D664C` | `inc word [ebx+eax*2+0x1C4]` — the only write to the ledger |
| `0x004D63AB` | stage 2: `262144000 * (16 - incoming - 2*inStock) / hexDistance` |
| `0x004C5330` | slider setter, `map<buildingType, map<goodType, percent>>` |
| `0x004F2770` | the `tick & 0xF` gate — the throughput ceiling |
| `0x00513C80` | main loop, `0x47` = 71 ms per tick |

## Credit

Production-speed tables and build plans come from the Settlers 4 community; the 71 ms tick derived
from the binary reproduces those tables to within 0.0007 units/min, and the dispatch ceiling
reproduces the measured `Tragelimit` percentages for every race.

Written with [Claude Code](https://claude.com/claude-code).
