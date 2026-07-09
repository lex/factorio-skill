---
name: factorio
description: >-
  Factorio 2.0 + Space Age reference for modding and game mechanics. Use when
  writing control.lua, scenarios, map/surface generation, prototypes or
  data.raw, or mods (Factorio runtime API, LuaSurface, LuaEntity, events), and
  when reasoning about game mechanics — recipes, ratios, entity stats,
  terminology, limits, quirks, quality, spoilage, planets. Targets Factorio 2.0
  and the Space Age expansion.
---

# Factorio (2.0 + Space Age)

> Version: Factorio **2.0** with the **Space Age** expansion. Facts predating
> 2.0 are out of scope unless explicitly labeled historical. Unreleased **2.1**
> features are tracked separately in `references/upcoming-2.1.md` and must not
> be treated as current behavior.

## What you get here

Curated essentials below (terminology + the highest-value gotchas); deeper
detail lives in `references/`. When a fact isn't covered here or in
`references/`, WebFetch the right page via `references/url-map.md` — do not
answer Factorio 2.0 / Space Age facts from memory.

## Core terminology

- **Prototype** — a data-stage definition (recipe, entity, item, technology,
  etc.) stored in `data.raw`; frozen once the data stage ends.
- **`data.raw`** — the global table holding every prototype, keyed first by
  type then by name: `data.raw["type"]["name"]`.
- **Entity** — a placed, live instance of an entity prototype on a surface;
  accessed at runtime as a `LuaEntity`.
- **Surface** — a `LuaSurface`: the map/planet/platform that entities and
  tiles live on, looked up via `game.surfaces`.
- **Chunk** — a fixed-size area of a surface; generation is checked with
  `is_chunk_generated`, and pollution accumulates/spreads per chunk.
- **Tick / UPS** — the simulation step; `game.speed = 1.0` means normal speed,
  60 UPS (updates/ticks per second).
- **`storage`** — the per-mod persistent Lua table, serialized into the save
  and restored on load; called `global` before Factorio 2.0. Can hold data
  and `LuaObject` references, but not functions.
- **Data stage vs. control stage** — the data stage (`data.lua` →
  `data-updates.lua` → `data-final-fixes.lua`) defines prototypes and its Lua
  state is discarded when it ends; the control stage (`control.lua`) is the
  persistent runtime with `game`, `script`, and `storage`.
- **`on_init` / `on_load` / `on_configuration_changed`** — the three
  lifecycle handlers; `storage` is read-only and `game` is unavailable inside
  `on_load()`.
- **Recipe** — a `type = "recipe"` prototype defining `ingredients`,
  `results`, and `energy_required`.
- **Assembler / crafting machine** — entities such as
  `AssemblingMachinePrototype`, built on the abstract `CraftingMachinePrototype`,
  that execute recipes.
- **Force** — the faction/ownership grouping (e.g. the player force) that
  entities belong to, via `game.forces`.
- **LuaObject validity** — handles like `LuaEntity`/`LuaPlayer`/`LuaSurface`
  can go stale; check `.valid` before using a stored reference.
- **Quality** — Space Age feature: 5 tiers (Normal/Uncommon/Rare/Epic/
  Legendary) whose bonus scaling follows *tier strength* (0/1/2/3/5), not
  ordinal position.
- **Spoilage** — Space Age feature: many items decay into a spoilage/spoil
  product on a fixed timer that keeps running everywhere (machines, belts,
  even an inserter's hand).
- **Planet** — a distinct surface with its own hazards/resources (Vulcanus,
  Fulgora, Gleba, Aquilo); a `LuaPlanet`'s surface isn't auto-generated —
  it must be created or associated explicitly.
- **Space platform** — a self-contained orbital factory anchored by a space
  platform hub, built via ghost/remote construction and moved by thrusters.

## Cross-cutting quirks (read before scripting)

- **`storage`, not `global`.** Factorio 2.0 renamed the persistent per-mod
  data table; port any `global.foo` snippet to `storage.foo`.
- **Check `.valid` before touching a stored `LuaObject`.** A destroyed
  entity/player/surface leaves a stale handle that errors on access.
- **Don't do real work in `on_load()`.** `game` isn't available there and
  `storage` is read-only; only re-hook metatables/handlers on data already in
  `storage`, or risk desyncs and broken replays.
- **Plain Lua locals don't survive a save/load.** Only `storage` is
  serialized — put anything that must outlive a tick boundary there.
- **`math.random()` is deterministic and seeded from the map seed;
  `math.randomseed()` does nothing.** Use a separate `LuaRandomGenerator` for
  an independent random stream.
- **Prototypes are frozen after the data stage, and stage order matters.**
  `data.lua` → `data-updates.lua` → `data-final-fixes.lua` run as rounds
  across *all* mods; something defined late may miss other mods' logic that
  already ran (e.g. barrelling recipes).

See `references/quirks.md` for the full list, including mechanics and Space
Age quirks (quality-ingredient exact-tier matching, recycler material loss,
solar priority order, circuit signal overflow, and more).

## When to fetch deeper

- Need a specific ratio, limit, or numeric constant (belt throughput,
  inserter speed, power/pollution figures, resistance formula)? →
  `references/mechanics.md`, or the wiki page it cites.
- Need a runtime API signature (`LuaSurface`, `LuaEntity`, `game`, events,
  `script`)? → `references/modding-api.md`, or `lua-api.factorio.com`.
- Need a prototype field or `data.raw` shape (recipe/entity/item/technology
  definition)? → `references/data-raw.md`, or `prototypes.html`/`types.html`.
- Need Space Age specifics (quality, spoilage, planets, space platforms,
  elevated rails)? → `references/space-age.md`.
- Asked about an *upcoming/unreleased* 2.1 feature? → `references/upcoming-2.1.md`
  (clearly flagged as not-yet-shipped; don't state it as current behavior).
- Anything not covered above, or you're unsure which page has it? →
  `references/url-map.md` to pick the right URL, then WebFetch it.

## Reference files

- `references/modding-api.md` — runtime/scripting API.
- `references/data-raw.md` — prototype stage & data.raw.
- `references/mechanics.md` — ratios, limits, numbers.
- `references/space-age.md` — expansion mechanics.
- `references/quirks.md` — gotchas & common mistakes.
- `references/upcoming-2.1.md` — unreleased Factorio 2.1 features (subject to change).
- `references/url-map.md` — topic → URL for deep fetch.
