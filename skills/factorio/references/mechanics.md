# Game Mechanics — Ratios, Limits & Numbers

Concrete numbers for reasoning about throughput, power, pollution, combat, and
circuits. Figures below are quoted or closely paraphrased from the wiki pages
listed in **Sources**; anything not confirmed on a fetched page is marked
**unverified** rather than guessed. Where a page itself ties a number to a
build (e.g. 2.0.26/2.0.67), that build is noted — treat un-tagged figures as
current-2.0/Space Age per the page as fetched.

## Belts & throughput

Belt tier throughput, per the wiki's belt table (*Belt transport system*):

| Belt | Speed (tiles/s) | Max throughput |
|---|---|---|
| Transport belt (yellow) | 1.875 | 15 |
| Fast transport belt (red) | 3.75 | 30 |
| Express transport belt (blue) | 5.625 | 45 |
| Turbo transport belt (Space Age) | 7.5 | 60 |

The wiki's table header labels the throughput column "stacks/second" but
footnotes that "the 'stacks' below refers to stacks created by a stack
inserter, not inventory stacks" — i.e. these are effectively items/second
figures (the commonly-quoted 15/30/45/60 items/s), not a stack-size-scaled
number. A fully filled straight belt holds 8 stacks in total. Both lanes
exist on every tier (2 lanes per belt).

Underground belts (base game): max gap is 4 / 6 / 8 tiles for
transport/fast/express respectively, storing up to 44 items at a 4-tile gap
and up to 72 items at an express 8-tile gap. Turbo (Space Age) underground
range was **not confirmed** on the fetched page — unverified.

Splitters divide input/output in a 1:1 ratio and preserve lane (left items
stay left, right items stay right) unless configured otherwise; if one
output is blocked, a splitter routes all input to the other output. The
fetched pages did not give a specific throughput-loss figure for
sideloading (merging a belt in from the side) or for belt curves/corners —
**unverified**, do not assume a specific fraction.

## Inserters

Chest-to-chest throughput in items/second, from the wiki's *Inserters* page
(data there is explicitly tied to Factorio 2.0.26, cross-checked against a
1.1 baseline — treat as current 2.0 figures):

| Inserter | No bonus | +2 stack-size bonus | +7 stack-size bonus |
|---|---|---|---|
| Burner | 0.79 | 1.58 | 2.37 |
| Yellow (regular) | 0.86 | 1.71 | 3.43 |
| Long-handed | 1.2 | 2.4 | 4.8 |
| Fast (red) | 2.5 | 5.0 | 10.0 |
| Bulk | 5.0 | 10.0 | 30.0 |
| Stack (blue) | 15.0 | 20.0 | 40.0 |

"Bonus" columns are the capacity-bonus research (stack-size) levels; they
raise how many items an inserter moves per swing, not swing speed. Bulk
inserters get an extra edge dropping onto splitter *sides* — the page cites
16.0 items/s there instead of the usual 14.4 for the equivalent no-side
case.

Rotation/extension speed (fraction of full turn or tile per tick):

| Inserter | Rotation speed | Extension speed | Ticks/full rotation |
|---|---|---|---|
| Burner | 0.013 | 0.035 | 76 |
| Yellow | 0.014 | 0.035 | 70 |
| Long-handed | 0.02 | 0.05 | 50 |
| Fast / Bulk / Stack | 0.04 | 0.1 | 24 |

Power draw has two components — a per-transfer-cycle energy cost, and a
continuous rotation/extension power draw while swinging. Re-querying the
page by exact type name (rather than the wiki's color coding, which an
earlier pass mis-mapped) gives:

| Inserter | Rotation power (kW) | Extension power (kW) |
|---|---|---|
| Burner | 39 | 105 |
| Inserter (yellow) | 4.2 | 10.5 |
| Long-handed | 6 | 15 |
| Fast | 16.8 | 42 |
| Bulk | 48 | 120 |
| Stack | 96 | 240 |

Per-transfer-cycle energy cost was only confirmed unambiguously for three
types: Burner 66.9 kJ, Inserter (yellow) 6.65 kJ, Stack inserter 46.4 kJ.
The page also showed figures of 7.0/8.12/23.2 kJ that would presumably
belong to Long-handed/Fast/Bulk, but a follow-up query asking for those by
exact name came back "not specified in tables" for that metric — the two
fetches disagree on whether/how those three numbers are labeled, so they
are **unverified** and omitted rather than guessed.

Inserters can pick up from the ground, a belt, or anything with storage
space. On perpendicular belts, an inserter prefers the near lane and only
reaches the far lane if the near one is empty.

## Power

Units: electricity is tracked in W/kW/MW (instantaneous power) and
J/kJ/MJ/GJ (stored/transferred energy).

Generation figures from the wiki's *Electric system* page:

| Source | Output |
|---|---|
| Steam engine | 900 kW (from stored steam) |
| Steam turbine | 5800 kW |
| Solar panel | 60 kW per panel |
| Accumulator | 5 MJ capacity; 300 kW max charge/discharge rate |
| Nuclear reactor | ~8 GJ released per fuel-cell burn (baseline; more with reactor bonuses) |

Solar panel average output across the day/night cycle (the commonly-cited
"effectively less than peak" figure) was **not confirmed** on the fetched
page — only the 60 kW peak-per-panel figure is sourced; treat any
day/night-averaged number as unverified.

**Priority order** — the page states electricity demand is satisfied in
this order: solar panels first ("they always work at maximum performance
available"), then steam engines/turbines/fusion generators (which "match
whatever demand solar panels cannot satisfy," dividing load equally among
themselves), and accumulators last, only discharging once the above can't
cover demand. Accumulators charge from any surplus once demand is met.

**Brownouts**: if a network's generation can't cover total demand, the
available power is "evenly spread across all machines in the network
(based on each machine's demand), and all machines will slow down
proportionally to the power available." Worked example from the page: an
assembling machine 3 (210 kW) and an electric mining drill (90 kW) demand
300 kW total; with only 180 kW available (3 solar panels at 60 kW), both
machines run at roughly 60% speed.

**Idle drain**: machines draw power even when not actively working — e.g.
"assembling machine 2 will consume 155 kW (150 kW energy consumption + 5 kW
drain)."

## Pollution

Pollution production, in pollution/minute at full power, per the wiki's
*Pollution* page (base game and Space Age buildings mixed together below —
figures are additive per building, not global caps):

| Source | Pollution/min |
|---|---|
| Boiler | 30 |
| Burner mining drill | 12 |
| Electric mining drill | 10 |
| Pumpjack | 10 |
| Oil refinery | 6 |
| Steel furnace | 4 |
| Chemical plant | 4 |
| Centrifuge | 4 |
| Stone furnace | 2 |
| Assembling machine 1/2/3 | 4 / 3 / 2 |
| Electric furnace | 1 |
| Heating tower (Space Age) | 100 |
| Big mining drill (Space Age) | 40 |
| Foundry / Cryogenic plant / Biolab (Space Age) | 6 / 6 / 8 |
| Electromagnetic plant (Space Age) | 4 |
| Recycler / Crusher (Space Age) | 2 / 1 |

Module effects compose as `final pollution = pollution_multiplier ×
energy_usage_multiplier × base_pollution`.

**Spread**: a chunk starts spreading pollution to its four cardinal
neighbors once it accumulates 15.0 pollution, spreading roughly "2% per 64
ticks" (64 ticks ≈ one game-second).

**Absorption/dissipation** (per minute, by tile, from the page's table):
water tiles ‑1.536, grass/dirt ‑1.10592, red desert/sand ‑0.9216, nuclear
ground ‑0.768, path/landfill 0, out-of-map ‑6.144. Trees also absorb
pollution passively (rate depends on growth stage, from ‑0.001/s at full
density down to 0 at minimum), and can absorb a burst of "10 pollution"
when damaged if local chunk pollution exceeds 60.

**Evolution and enemies**: the page is explicit that "evolution factor is
not increased by spreading/absorbed pollution, but by pollution produced by
all machinery at every tick" — i.e. containing/absorbing pollution locally
does not slow the global evolution factor, only reducing pollution produced
does. Biter spawners absorb pollution and use it to trigger attacks: each
spawner type has an absorption threshold before it musters an attack group
(small biter/spitter 4, medium spitter 12, medium biter 20, big spitter 30,
big biter 80, behemoth spitter 200, behemoth biter 400), and once a chunk's
pollution exceeds 20, spawners absorb "20 + 0.01 × [chunk's pollution]"
every 64 ticks. Attack groups muster on a random 1–10 minute timer, with
stragglers given up to 2 more minutes to join.

## Combat

*The wiki's `Combat` page returned 404 when fetched; `Damage` and
`Resistance` (both current, mutually consistent) were used instead — see
Sources.*

**Damage types** (from `Damage`): physical, physical+explosion (cannon/
artillery shells), impact (vehicle collisions), fire, acid, poison,
explosion, laser, and electric — nine categories total, each resisted
independently by an entity's per-type resistance values.

**Resistance formula** (confirmed identically on both `Damage` and
`Resistance`): each resistance is a flat component `R` and a percentage
component `P` (as a decimal). Given incoming damage `D`, final damage `F`
is:

- If `R + 1 < D`: `F = (D − R) × (1 − P)`
- Else if `D > 1`: `F = (1 − P) / (R − D + 2)`
- Else (`D ≤ 1`): `F = (1 − P) / (R + 1)`

Flat resistance is applied first, percentage resistance after. Flat
resistance alone can never reduce damage to zero (it asymptotically
approaches a floor above 0); **only 100% percentage resistance grants true
immunity** to that damage type. Worked example from the page: 25/10%
resistance against 45 incoming damage → `(45 − 25) × 0.9 = 18` damage
taken.

**Health**: player base health is 250 HP (before Health research);
entity/turret/biter HP values are per-entity and listed on their individual
wiki pages, not enumerated here.

**Piercing**: piercing ammunition (e.g. tank cannon rounds) can continue
through and hit additional targets as long as remaining piercing power
exceeds the damage already dealt to the target killed.

## Circuit network

From the wiki's *Circuit network* page (current as of 2.0.67):

- **Wire colors are independent networks.** Red and green wires never
  merge with each other even when landing on the same pole/device; two
  wires of the *same* color joined together do merge into one network.
- **A receiving device sums all signals from every wire connected to it**,
  red and green alike — it sees one combined set of channel values, not two
  separate ones.
- **Signals**: every item type and every fluid type in the game has its own
  channel automatically (a storage device broadcasts its contents on the
  matching channel). On top of that there are virtual signals for things
  with no physical item — 177 in the base game, 241 once Space Age content
  is included (exact base-vs-added split not itemized on the page — treat
  241 as the Space Age-inclusive total, not necessarily "177 + 241").
- **Signal values are signed 32-bit integers**, range
  −2,147,483,648 to 2,147,483,647 inclusive, and they wrap on overflow
  (e.g. `2147483647 + 10` becomes `-2147483639`).
- **Multiple senders on a network add together** — two storage tanks each
  holding 1000 crude oil produce a combined crude-oil signal of 2000 on
  that network.
- **Combinators**:
  - *Constant combinator* — broadcasts up to 20 fixed signal values.
  - *Arithmetic combinator* — takes input signal(s), applies a math
    operation, emits a result signal.
  - *Decider combinator* — compares input signal(s) against a condition
    and emits signal(s) conditionally (true/false-gated).
  - *Selector combinator* — picks a single signal out of its inputs (e.g.
    largest/smallest, a random one, or a signal's stack size/item count)
    and outputs just that one.
- **Logic conditions** on decider-style comparisons can be quantified as
  *everything* (true only if the condition holds for every input signal),
  *anything* (true if it holds for at least one), or *each* (evaluated
  per-signal, independently, with results summed on output).
- **Feedback loops**: wiring a combinator's output back into its own input
  creates a loop where the value changes every tick, so it "shoot[s]
  upward" quickly (tick-rate speed) but not instantaneously — useful
  deliberately as a tick-rate counter, dangerous by accident.
- Wire reach/length limits are governed by the connecting entity (pole or
  wire type), which the page does not enumerate numerically — **specific
  wire-reach distances are unverified here**; consult individual
  entity pages if a wire-length number is needed.

## Hard limits & magic numbers

Collected fixed constants from the sections above — treat any not listed
here as not yet confirmed:

- Belt throughput ceiling: 15 / 30 / 45 / 60 items-per-second for
  transport / fast / express / turbo belts.
- Underground belt max gap: 4 / 6 / 8 tiles for transport / fast / express
  (turbo's gap is unverified).
- Fully-loaded straight belt holds 8 stacks.
- Splitters split input/output 1:1.
- Accumulator capacity: 5 MJ; max charge/discharge: 300 kW.
- Circuit signal range: signed 32-bit integer, −2,147,483,648 to
  2,147,483,647 (wraps on overflow).
- Constant combinator: up to 20 signal values per combinator.
- Virtual signals: 177 (base game) / 241 (with Space Age).
- Player base health: 250 HP.
- Resistance: 100% percentage resistance is the only way to reach true
  damage immunity; flat resistance alone never reaches zero damage.
- Pollution spread trigger: a chunk must reach 15.0 pollution before it
  spreads to neighbors, then spreads at ~2%/64 ticks.
- Biter spawner attack-muster timing: random 1–10 minutes, stragglers get
  up to +2 minutes.

## Sources

- https://wiki.factorio.com/Transport_belts
- https://wiki.factorio.com/Belt_transport_system
- https://wiki.factorio.com/Inserters
- https://wiki.factorio.com/Electric_system
- https://wiki.factorio.com/Pollution
- https://wiki.factorio.com/Damage — substituted for `Combat`, which
  404'd at fetch time; no page at that URL currently exists.
- https://wiki.factorio.com/Resistance — fetched alongside `Damage` to
  cross-check the resistance formula.
- https://wiki.factorio.com/Circuit_network
