# Factorio 2.1 — what changed from 2.0

> **Released.** 2.1.7 was the first experimental release (2026-06-23); 2.1.23
> (2026-10-20) is the current stable, per factorio.com's `latest-releases`
> API. This file lists what 2.1 changed from 2.0.77, the last 2.0 release, so
> you can port 2.0 code and spot 2.0-era facts. Sources: the `changelog.txt`
> shipped in the 2.1.23 download, and the game's own prototype files
> (`data/base`, `data/quality`, ...) compared between 2.0.77 and 2.1.23. API
> facts marked *checked* were run on both headless servers.
>
> Servers and mods move to a new version at their own pace: before giving
> version-specific advice, find out which version the code runs on
> (`script.active_mods.base` at runtime, `mods.base` in the data stage).

## Runtime API: what breaks 2.0 scripts

Each line: what was removed or changed → what to use. **2.0?** says whether
the replacement already exists in 2.0.77, so one script can run on both.

| 2.0 | 2.1 | 2.0? |
|---|---|---|
| `entity.active = false` (write) | `entity.disabled_by_script = true`; reading `active` still works | yes (checked) |
| `entity.minable = false` (write) | `entity.minable_flag = false`; `minable` is read-only | yes (checked) |
| `entity.fluidbox[i]`, `LuaFluidBox` | `entity.get_fluid(i)`, `set_fluid(i, fluid)`, `get_fluid_capacity(i)`, `insert_fluid`, `clear_fluids`, `get_fluid_box_*`, `*_fluid_segment_*` | `get_fluid`, `set_fluid`, `get_fluid_capacity` yes (checked); `fluidbox` itself is gone in 2.1 |
| `entity.remove_fluid{...}` | `entity.extract_fluid{...}`. 2.1 also has a **new, different** `remove_fluid()`: rename old calls rather than assume they still mean the same | no |
| `entity.set_fluid(i)` to clear | second argument is mandatory (use `clear_fluid`); returns a `FluidAmount` | — |
| `defines.inventory.furnace_source`, `assembling_machine_input`, `rocket_silo_input` | `defines.inventory.crafter_input` (same value, 2) | yes (checked) |
| `furnace_result`, `assembling_machine_output`, `rocket_silo_output` | `crafter_output` (3) | yes (checked) |
| `furnace_modules`, `assembling_machine_modules`, `rocket_silo_modules` | `crafter_modules` (4) | yes (checked) |
| `assembling_machine_trash`, `furnace_trash` | `crafter_trash` (8) | yes (checked) |
| `player.get_quick_bar_slot(i)` → item prototype | `get_quick_bar_slot(page, slot)` → `QuickBarSlot` table (`type` = record/remote/filter/item, plus `record`/`filter`/`selection`/`item`); `set_quick_bar_slot(page, slot, filter)`; `player.quick_bar_width` | no: branch on version |
| `entity.neighbours` | `fluidbox_neighbours`, `underground_belt_neighbour`, `wall_neighbours`, `cliff_neighbours`, `neighbour_connectable_connections`, `belt_neighbours`, `heat_neighbours` | — |
| `entity.pump_rail_target` | `pump_input_rail_targets`, `pump_output_rail_targets` | — |
| `entity.mining_progress` | `LuaControl::mining_progress` (read/write) | — |
| `entity.display_panel_text = {localised}` | plain string only | — |
| `entity.inserter_spoil_priority` values with `_` | values use `-` | — |
| `recipe.category`, `additional_categories` (LuaRecipe and LuaRecipePrototype) | `recipe.categories` | — |
| `LuaRecipePrototype::is_parameter` | `LuaPrototypeBase::parameter` | — |
| `LuaRecipePrototype::show_amount_in_title`, `result_is_always_fresh`, `reset_freshness_on_craft`, `always_show_products` | removed | — |
| `LuaEntityPrototype::fixed_recipe` → string | → `LuaRecipePrototype` | — |
| `LuaEntityPrototype::fluid_usage_per_tick`, `max_power_output`, `pumping_speed` | `get_fluid_usage_per_tick()`, `get_max_power_output()`, `get_pumping_speed()` | — |
| `LuaEntityPrototype::building_grid_bit_shift` | `build_grid_size` | — |
| `LuaEntityPrototype::build_base_evolution_requirement` | removed | — |
| `LuaItemPrototype::inventory_size` | `get_inventory_size()` | — |
| `LuaItemPrototype::fuel_category` | `fuel_categories` | — |
| `LuaItemPrototype::tile_filter_slots`, `entity_filter_slots` | removed (planner filters are dynamic) | — |
| `LuaFluidBoxPrototype::volume` | `get_volume()` | — |
| `game.create_profiler()` | `helpers.create_profiler()` | — |
| `player.pipette_entity()` | `player.pipette()` | — |
| `player.spectator` | removed (it did nothing) | — |
| `LuaRecord::is_blueprint_preview` | `is_preview` | — |
| `LuaGroup` | `LuaItemGroup`, `LuaItemSubGroup` | — |
| `include_fuel` on assembler/furnace control behaviour | `read_fuel` | — |
| Logistic container / cargo landing pad `circuit_exclusive_mode_of_operation`, `defines.control_behavior.logistic_container.exclusive_mode`, `.cargo_landing_pad.exclusive_mode` | `set_requests` and `read_contents` (both can be on at once) | — |
| Display panel control behaviour `get_message`, `set_message`, `messages` | `add_record`, `get_record`, `set_record`, `remove_record`, `move_record`, `records` | — |
| `defines.control_behavior.type.storage_tank` | `.single_fluid_box` | — |
| `defines.default_icon_size` | `defines.constant.default_icon_size` | — |
| `defines.relative_gui_type.storage_tank_gui` | removed | — |
| `LuaSimulation::get_widget_position(data, data2)` | no data strings; 1-based indexing | — |
| `player.teleport()` left remote view | it no longer does | — |
| `set_fluid_filter`/`get_fluid_filter` with a `FluidFilter` (early 2.1) | a `FluidID` since 2.1.22 | — |
| Boiler fluid filters settable at runtime | no longer (2.1.21) | — |

Writing a removed attribute is a hard error ("LuaEntity::active is read
only.", "LuaEntity doesn't contain key fluidbox."). Code that ran fine on 2.0
stops the game on 2.1 the first time it reaches that line, so a mod or
scenario can load cleanly and crash minutes later. Grep for every row above
rather than trusting a test run, which only reaches the code it exercises.

### Notable runtime additions
- `LuaPlanet::associate_surface` works for planets that require heating
  (Aquilo) since 2.1.21; on 2.0 it refused them.
- `LuaSurface::override_pollution_type` read/write.
- `LuaElectricNetwork` / `LuaElectricSubNetwork`, `entity.electric_network(s)`.
- `LuaForce::add_alert`, `add_custom_alert`, `remove_alert`, `play_music`;
  `unlock_logistic_network`, `unlock_travel_to_space_platforms`,
  `cargo_landing_pad_limit`, `get_space_platforms()`.
- `LuaEntity::flip`, `protected`, `local_effect`, `potential_effects`,
  `get_signals_changed()`, tooltip fields, stored durability,
  `request_missing_construction_materials`, `providing_to_other_platforms`.
- `LuaQualityPrototype::roll_quality()`, `get_roll_chances()`.
- `LuaBootstrap::new_notification_queue()`, `get_event_name()`;
  `helpers.stage`.
- GUI element type `"inventory"` and `on_gui_inventory_action`.
- Events: `on_next_day_started`, `on_player_color_changed`,
  `on_blueprint_settings_pasted`, `on_player_music_changed`,
  `on_player_super_forced_selected_area`, and pre-selection events
  (`on_player_start_selected_area`, `on_player_change_selected_area`,
  `on_player_cancel_selected_area`, opt-in per selection tool).
- Recipes can raise an event when crafted (`RecipePrototype::raise_on_crafted`).

## Data stage: what breaks 2.0 mods

- **Recipe categories reworked.** `RecipePrototype::category` and
  `additional_categories` are gone: use `categories` (a list). New
  `hand-crafting` category; removed `basic-crafting`,
  `recycling-or-hand-crafting`, `chemistry-or-cryogenics`, `pressing`,
  `crafting-with-fluid-or-metallurgy`, `metallurgy-or-assembling`,
  `organic-or-hand-crafting`, `organic-or-assembling`, `organic-or-chemistry`,
  `electronics-or-assembling`, `electronics`, `electronics-with-fluid`,
  `cryogenics-or-assembling`. The `recycling` category moved to the recycler
  mod.
- **Science packs are plain items** (`type = "item"`; 2.0 had 12 `tool`
  items in base and Space Age, 2.1 has none). Labs and technologies accept
  any item as a research ingredient. Code that looks science packs up as
  `data.raw.tool[...]` breaks.
- **Renamed recipes** [Space Age]: `molten-iron` → `iron-ore-melting`,
  `molten-copper` → `copper-ore-melting`.
- **Quality values rescaled.** Quality effects in prototypes are 10× smaller
  (quality module 3: `quality = 0.25` → `0.025`), and each quality's
  `next_probability` went from 0.1 to 1. The repeat roll is the new
  `chain_probability` (0.1). In-game chances are unchanged; GUI and API
  values now match.
- Products: `ItemProductPrototype::probability`/`FluidProductPrototype::probability`
  → `independent_probability` (plus `shared_probability`);
  `result_is_always_fresh`, `reset_freshness_on_craft` moved from the recipe
  to `ItemProductPrototype` (`always_fresh`, `reset_freshness_on_craft`).
  `RecipePrototype::always_show_products`, `show_amount_in_title` removed.
- `EntityWithHealthPrototype::loot` is an array of `ItemProductPrototype`.
- `FluidBox::minimum_temperature`/`maximum_temperature` removed (2.1.22):
  temperature limits live on the energy source or machine
  (`minimum_input_temperature`, `maximum_input_temperature`).
  `FluidEnergySource::maximum_temperature` "unlimited" is `nil`, not 0.
- Fluid ingredient/product `fluidbox_multiplier` defaults to 3;
  `UtilityConstants::max_fluid_flow` removed (flow derives from volume).
- Renames and removals: `MapSettings::steering` → per-unit `steering`;
  `VehiclePrototype::braking_power`, `friction` → `braking_force`,
  `friction_force`; `LightningPrototype::damage` is `DamageParameters`;
  `MineEntityTechnologyTrigger::entity` → `entities`;
  `CraftingMachinePrototype::forced_symmetry` → `use_mirroring`;
  `AssemblingMachinePrototype::enable_logistic_control_behavior` → the
  `"no-logistic-connection"` flag; `AsteroidPrototype::mass` →
  `damage_per_hp`; `ItemPrototype::fuel_category` → `fuel_categories`;
  `DeconstructionItemPrototype::tile_filter_count`/`entity_filter_count`
  removed; `SpacePlatformHubPrototype::dump_container` removed;
  `UtilityConstants::rocket_lift_weight` → `default_rocket_lift_weight`;
  several circuit-connector layer fields → `CircuitConnectorSprites::render_layer`.
- `SelectionModeFlags` `blueprint`, `deconstruct`, `cancel-deconstruct`,
  `upgrade`, `cancel-upgrade`, `downgrade` no longer imply `any-entity` or
  `any-tile`: add those explicitly.
- `info.json`: dependency prefix `+` (optional but enabled by default), and
  `migrates_from` (take over a removed mod's storage, script inventories, GUI
  and render objects). Mods can add new languages through locale.

## Mods that ship with the game

- Space Age's dependency on **Quality** is now a recommendation: Space Age
  runs with Quality off.
- The **Recycler** is its own mod, `recycler` (in 2.0 it was part of
  `quality`); Fulgora needs it.
- New feature flag `expansion`.

## Gameplay changes (numbers from the prototype files)

- **Enemy expansion** (`base/prototypes/map-settings.lua`, 2.0.77 → 2.1.23):
  `max_expansion_distance` 7 → 5, new `min_expansion_distance` 3;
  `friendly_base_influence_radius` 2 → 6, `enemy_building_influence_radius`
  2 → 3; `building_coefficient` 0.1 → 0.5, `other_base_coefficient` 2 → 3,
  `neighbouring_base_chunk_coefficient` 0.4 → 0.5 (the score now counts
  *whether* a neighbouring chunk has buildings or bases, over Euclidean rather
  than Manhattan distance); `settler_group_max_size` 20 → 10, now scaled by
  `evolution_group_size_factor ^ evolution` (8.0); `min_expansion_cooldown`
  4 → 10 minutes (max stays 60); new `build_base_unit_dispatch_cooldown`
  30 s. Spawners spawn closer and emit a damaging acid cloud when blocked
  from spawning. An alert fires when an expansion base is built under radar
  coverage.
- **Steam** max temperature 5000 → 500 °C.
- **Centrifuge** stack size 50 → 10.
- **Shotguns:** no movement slowdown, 1 → 1.5 shots/s, pellets slow enemies
  briefly; piercing shotgun shell damage 8 → 12 and moved to military 3
  (combat shotgun to military 4); combat shotgun damage bonus 20% → 50%.
- [Space Age] Rocket turret health 400 → 1500, tesla turret 1000 → 2000,
  space platform hub 1000 → 5000 with 50% impact resistance; asteroid
  crushing chunk yield 20% → 30% (advanced 5% → 10%); asteroids take damage
  on collision and no longer slow down (retuned in 2.1.21); platform drag
  depends on weight, not width.
- [Space Age] Quality affects storage tank volume, cargo wagon slots
  (×1.25 at uncommon), locomotive power (×1.2 at uncommon) and rolling stock
  top speed (×1.03 at uncommon). Quality modules can't go in crushers running
  asteroid reprocessing.
- [Space Age] Rocket silo inventory is limited by weight only; platforms wait
  in orbit until all drop slots are empty; orbital requests can import from
  any planet or from other platforms; small requests share a rocket.
  Characters can't travel to platforms before the first planet technology.
- [Space Age] New **landing pad unloading bay** (a 4×5 cargo bay inserters
  can take from); elevated rails can run over cargo bays.
- **Circuits:** pipes, pipes-to-ground, storage tanks (contents and
  temperature), boilers, heat exchangers, heat pipes, labs and land mines can
  connect; each entity chooses red, green or both wires for input and for
  output; requester and buffer chests can set requests and read contents at
  once; radar "universe" channel; decider combinator else-output (2.1.9);
  selector combinator "Time" mode, index comparator and (Space Age) quality
  select mode (2.1.21); "Read launched" on silos, "Read empty slots" on
  containers.
- **Building:** pumpjacks, burner drills and inserters can be flipped;
  flamethrower turrets rotate 8 ways; pumps connect to fluid wagons on both
  ends, off-grid and on curves; rolling stock can be fast-replaced and
  upgraded by the upgrade planner; spidertrons can patrol.
- Technology price multiplier limit 1000 → 100000.

## Sources

- `data/changelog.txt` in the Factorio 2.1.23 headless download
  (`https://www.factorio.com/get-download/2.1.23/headless/linux64`).
- Prototype files under `data/` in the 2.0.77 and 2.1.23 downloads.
- https://lua-api.factorio.com/latest/ (2.1 API docs); 2.0.77's are at
  https://lua-api.factorio.com/2.0.77/
- https://factorio.com/api/latest-releases (current version numbers).
