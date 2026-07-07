# Prototype Stage & data.raw

Before a Factorio mod ever touches `game` or `script`, it runs through the
**prototype (data) stage** — a separate Lua pass, before the save/scenario
even exists, whose only job is to build the table of every entity, item,
recipe, technology, etc. that the game will know about. This file covers
that stage: `data.raw`'s shape, how mods add to and mutate it, the file
load order, and the prototype categories you'll actually touch. For the
runtime/control stage (`game`, `script`, events, `storage`), see
`modding-api.md`.

## What data.raw is

`data.raw` is the global table that holds **every prototype in the
game**, vanilla and mod-added, organized in two levels: first by
prototype **type**, then by prototype **name**. The wiki describes it as
"a complete listing of all the vanilla prototype's internal names under
their prototype types." Lookup follows directly from that shape:

```lua
data.raw["prototype-type"]["internal-name"]
```

For example, `data.raw["capsule"]["raw-fish"]` is the prototype for the
raw-fish capsule item, and `data.raw["recipe"]["small-electric-pole"]` is
the small-electric-pole recipe.

`data.raw` currently contains roughly 251 distinct prototype types (per
the wiki's listing, current as of Factorio 2.0/Space Age-era docs — this
count drifts release to release) — entities, items, recipes,
technologies, equipment, armor, fluids, achievements, sounds, sprites,
custom-inputs, tiles, rails, and more — each
holding all the named prototypes of that type. **You cannot add a
brand-new type** — the wiki is explicit that "it is not possible to add a
new type to Factorio, without asking the developers to do it," since types
are implemented in the closed-source engine. Mods work entirely within the
existing set of types, adding, editing, or (indirectly) disabling named
entries inside them.

*(Source: wiki `Data.raw`)*

## Defining prototypes

Prototypes are added with `data:extend{...}`, called with a table (array)
of one or more prototype tables:

```lua
data:extend{fireArmor, recipe}
```

Each prototype table needs, at minimum, a `type` (its prototype category,
e.g. `"recipe"`, `"armor"`, `"item"`) and a `name` (its internal
identifier), plus whatever fields that type requires — e.g. a recipe
needs `ingredients`/`results`/`energy_required`, an armor item needs
resistances, and so on:

```lua
local recipe = {
  type = "recipe",
  name = "fire-armor",
  enabled = true,
  energy_required = 8,
  ingredients = {...},
  results = {{type = "item", name = "fire-armor", amount = 1}},
}
```

A common pattern for a *new* prototype that's a variant of an existing one
is to copy the existing prototype out of `data.raw` with
`table.deepcopy()` and then edit the copy's fields before extending it in:

```lua
local fireArmor = table.deepcopy(data.raw["armor"]["heavy-armor"])
fireArmor.name = "fire-armor"
```

The full prototype API — every type and the fields it accepts — is
documented at `lua-api.factorio.com/latest/prototypes.html` (concrete
types) and `types.html` (the shared field-level data formats used across
prototypes, e.g. `IconData`, `BoundingBox`, `Sprite`, `Color`, `Energy`,
`LocalisedString`). Per that reference, "missing properties will either
fall back to a default value or give an error that indicates what is
missing. Extra properties that the game isn't looking for are ignored."

*(Source: wiki `Tutorial:Modding_tutorial/Gangsir`, lua-api
`data-lifecycle.html`, `prototypes.html`, `types.html`)*

## Overriding / removing existing prototypes

Because `data.raw` is a plain shared table during the whole data stage,
any mod that loads after another can reach into `data.raw` and mutate a
prototype the earlier mod (or the base game) already defined — no special
API needed, just ordinary table assignment:

```lua
data.raw["capsule"]["raw-fish"].stack_size = 200
data.raw["container"]["iron-chest"].max_health = 1000
```

The idiomatic way to effectively *remove* something without deleting the
type/name entry outright is to disable it via its own fields — for
example, a recipe can be taken out of the game by disabling it rather than
unlinking it from `data.raw`:

```lua
data.raw["recipe"]["small-electric-pole"].enabled = false
```

The lua-api docs note that this kind of cross-mod editing is tracked: "a
history of which mod has changed which prototype is recorded by the
game." If your mod needs to guarantee it edits a prototype *after*
another mod has defined or edited it, add that mod as a dependency (in
`info.json`) so the load-order rules below place your `data-updates.lua`
or `data-final-fixes.lua` after theirs.

*(Source: wiki `Data.raw`, lua-api `data-lifecycle.html`, wiki
`Tutorial:Modding_tutorial/Gangsir`)*

## Data stage files

Each mod can define up to three prototype-stage files, and — like the
settings stage — they run in three sequential rounds **across all mods**,
rather than one mod finishing all three files before the next mod starts:

1. Every enabled mod's `data.lua` runs, in load order.
2. Then every enabled mod's `data-updates.lua` runs, in load order.
3. Then every enabled mod's `data-final-fixes.lua` runs, in load order.

Per the lua-api docs: "Each mod's `stage`.lua file is run according to the
usual sequence, then each mod's `stage`-updates.lua is run, after which
each mod's `stage`-final-fixes.lua is run."

**Load order between mods**, per the wiki tutorial: "Within stages, mods
are loaded by dependency, then by alphabetical order... Required
dependencies are loaded first, always... Optional dependencies are loaded
first if present, but do not have to be present." So a mod's `data.lua`
is guaranteed to run after everything its `dependencies` list requires,
and ties are broken alphabetically by mod name.

Within the data stage, mods can read/write the shared `data` table (which
collects all prototypes), read a `mods` table mapping mod name → version
(so a mod can conditionally react to what else is installed), and read a
`settings` table already populated with the startup mod settings resolved
at the end of the earlier settings stage. Once the whole data stage
finishes, "the shared Lua state is discarded, meaning no variables or
functions defined in it will carry over" — and every prototype is then
frozen for the rest of that game.

Practical implication of the three-file split:
- **`data.lua`** — where a mod should define everything it adds outright.
- **`data-updates.lua`** — where a mod edits/extends prototypes that other
  mods (or the base game) may have just defined in their own `data.lua`.
- **`data-final-fixes.lua`** — a last-pass file for changes that must
  happen after *every* mod's `data-updates.lua` (e.g. compatibility
  patches), reserved for edits that genuinely need to go last.

*(Source: lua-api `data-lifecycle.html`, wiki
`Tutorial:Modding_tutorial/Gangsir`)*

## Key prototype categories

`data.raw` groups its ~251 types into a few functional families (from the
lua-api prototype index). Every concrete prototype has a `type` field
that names its category (e.g. `'accumulator'`, `'achievement'`); abstract
prototypes like `PrototypeBase` or `Prototype` are base classes in the
inheritance tree and are never instantiated directly.

| Category | Covers |
|---|---|
| Entities | Machines and placed objects: accumulators, beacons, assembling machines, furnaces, turrets, walls, etc. |
| Items & Equipment | Armor, tools, modules, consumables, equipment grids |
| Recipes & Crafting | `recipe` prototypes and recipe-category definitions |
| Technology & Research | `technology` prototypes and research definitions |
| Transport & Logistics | Belts, inserters, robots, trains, vehicles |
| Signals & Control | Combinators, virtual signals, custom inputs/events |
| Achievements | The various achievement-condition prototype types |
| Audio & Visual | Sprites, animations, sounds, fonts |
| Game Configuration | Map settings, difficulty, map-generation presets, utility prototypes |

Inheritance is real but shown structurally rather than with explicit
"inherits from" links in the docs — e.g. `AssemblingMachinePrototype`
inherits from the abstract `CraftingMachinePrototype`, and the reference
describes a full prototype tree that "can be visualized as a tree." When
in doubt about which fields a given type accepts, check its page under
`prototypes.html`, since inherited fields aren't always repeated on the
concrete type's own page.

*(Source: lua-api `prototypes.html`)*

## Common gotchas

- **Order within the data stage matters for what other code can see.**
  Because `data.lua`/`data-updates.lua`/`data-final-fixes.lua` run in
  three separate rounds across *all* mods, a prototype your mod defines
  in `data-final-fixes.lua` did not exist yet when other mods'
  `data-updates.lua` ran. The docs give a concrete example: "if you add a
  fluid in `data-final-fixes.lua`, it is created after the barrelling
  code runs in `data-updates.lua`, so no barrelling recipe gets created."
  Define new prototypes as early as possible (`data.lua`), and reserve
  `data-updates.lua`/`data-final-fixes.lua` for edits to things that must
  already exist.
- **Depend on a mod if you need to edit after it.** Cross-mod edits to
  `data.raw` only land in the right order if your `info.json`
  `dependencies` force your file to run after the mod you're patching —
  otherwise you're relying on alphabetical luck.
- **`data.raw` can't grow new *types*.** Mods can add/edit named
  prototypes inside existing types, but new prototype types are an
  engine-level (C++) concept and aren't something a mod can register.
- **The data-stage Lua state is thrown away.** Locals/functions defined
  while building prototypes do not survive into the control stage —
  don't expect data-stage helper variables to be usable later; see
  `modding-api.md` for what does persist at runtime (`storage`).

*(Source: wiki `Data.raw`, lua-api `data-lifecycle.html`, wiki
`Tutorial:Modding_tutorial/Gangsir`)*

## Sources

- https://wiki.factorio.com/Data.raw
- https://lua-api.factorio.com/latest/auxiliary/data-lifecycle.html
- https://lua-api.factorio.com/latest/prototypes.html
- https://lua-api.factorio.com/latest/types.html
- https://wiki.factorio.com/Tutorial:Modding_tutorial/Gangsir
