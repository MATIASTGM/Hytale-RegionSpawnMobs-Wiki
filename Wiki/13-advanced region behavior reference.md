# Advanced Region Behavior Reference

This page lists what a configuration can **make happen inside a region**.
It is not a list of JSON examples. It is a runtime behavior reference based on
the current code flow.

Scope of this guide:

- regions (`SpawnRegion`)
- presets (`SpawnPreset`)
- entities inside a region (`EntitySpawnConfig`)
- region SpawnPoints (`SpawnPoint`)
- terrain types (`terrain-types.json`)
- reconciliation and containment caused by those configurations
- region areas (`RegionArea`): cuts (`CUT`) and zones (`ZONE`)

Out of scope: markers. Markers have their own system.
For markers, see the notes in `02-how-region-spawning-works.md`,
`06-mob-containment-system.md` and `07-files-json-and-data-storage.md`.
Since version 1.1.8, markers also release the tracker from tamed/retyped mobs
and use a fixed 48-block leash during combat.

Important note: this document describes the behavior of the current code.
When an old comment or old guide diverges from the current code, this guide marks it as `Attention`.

---

## 1. Real Event Order Inside a Region

Every second, for each world with an online player:

1. The player activates nearby regions through the global `playerRangeSpawnChunks` radius.
2. Each region is processed at most once during that second.
3. The region must pass the basic gates.
4. The effective preset defines interval, quantity, batch distribution and density overflow.
5. The regional mode (`CONTINUOUS`, `WAVE`, `THRESHOLD`) decides whether this cycle can spawn.
6. `spawnPasses` defines how many attempt rounds can happen in the same valid cycle.
7. Region conditions can block the entire region.
8. The system scans already living mobs from the region.
9. The reconciler can remove mobs that violate current rules.
10. If there is still capacity, the system runs up to `spawnPasses` spawn rounds.
11. Each round rolls a quantity from the preset and respects the remaining space.
12. Each mob candidate must pass weight, conditions, limits and terrain.
13. If the entity spawns, it receives a `RegionTrackerComponent`.
14. Containment can teleport back mobs that left the area.

---

## 2. Gates That Spawn Nothing

These cases stop the action before any real spawn attempt happens.

| Configuration/scenario | Generated action |
|---|---|
| No online players | No region processes spawning. |
| Player outside `playerRangeSpawnChunks` | The region does not enter the cycle. |
| Region in another world | The region is ignored in that world. |
| `isActive = false` | Does not spawn new mobs. Living mobs are not removed automatically. |
| Boundary smaller than 4x4x4 | Region is skipped with a warning. |
| Boundary larger than 320 in X/Z or 128 in Y | Region is skipped. |
| Region without parent and without `presetId` | Region is skipped with a periodic warning. |
| Region without parent and without `spawnMode` | Region is skipped with a periodic warning. |
| `presetId` is set but does not exist in the repository | The region does not stop; it uses an in-memory default preset (`30s`, `1-3`). |
| No configured entities and no inherited entities | Region is skipped. |
| Region conditions do not match time/weather | The entire region is skipped; living mobs are not removed for that reason during this cycle. |
| No core chunk of the region is loaded | Region is skipped. |
| Total capacity is full | Region does not attempt to spawn, except for reconciliations before the block. |
| Preset interval has not passed yet | Region does not attempt to spawn in this cycle. |
| No eligible entity | No mob spawns in this cycle. |
| No valid position found | No mob spawns for that spawn slot. |

---

## 3. What Each Region Field Can Cause

### Main Region Fields

| Field | Possible action |
|---|---|
| `isActive` | Enables/disables new spawns for the region. |
| `boundary` | Defines where to search for positions, which chunks count toward capacity and which mobs can be contained. |
| `worldName` in the boundary | Ensures the region only runs in the correct world. |
| `presetId` | Chooses interval, quantity per pass, batch positioning and density overflow. |
| `spawnMode` | Chooses regional behavior: continuous refill, wave or threshold. |
| `spawnPasses` | Defines how many attempt rounds the region can execute in the same valid cycle. It is clamped between `1` and `10`; default is `1`. |
| `maxPerChunk` | Defines the per-chunk limit and total capacity: `region chunks * maxPerChunk`. |
| `ignoreChunkDensity` | Allows position search even in a full chunk. Total region capacity still limits the region. |
| `dynamicConditionReconcile` | If enabled, removes living mobs whose entity time/weather condition no longer matches. |
| Region `spawnConditions` | Blocks or allows the whole region by time/weather. `terrainType` here is not used by the validator. |
| `entities` | Defines which mobs enter the roll, with weight, limit and conditions. |
| Empty `spawnPoints` | Uses random positions inside the boundary. |
| Non-empty `spawnPoints` | Uses only SpawnPoints; random region search is not used. |
| `parentRegionPath` | Can provide entities, preset and conditions when the child does not define them locally. See inheritance notes. |

### Area Fields (`CUT` and `ZONE`)

| Field | Possible action |
|---|---|
| Empty `areas` | Region uses the entire boundary for spawning. |
| Area with `mode = CUT` | Removes that sub-area from the base region spawn space. Random positions never land inside the cut. |
| Area with `mode = ZONE` | Creates its own spawn pool inside the region with separate entities and `maxAlive` from the base. |
| Zone without entities | Zone is ignored during the spawn cycle. |
| Zone without rectangles | Zone is ignored during the spawn cycle. |
| Zone `name` | Used in the log display name: `region/zone_name`. If empty, the zone UUID is used. |
| Base region without entities, but with zones that have entities | Region is processed; only the zones spawn. |
| Base region with entities and zones with entities | Base and zones spawn in the same cycle, sharing total capacity and chunk density. |

---

## 4. Area Combinations (`CUT` and `ZONE`) with the Spawn Cycle

Cuts and zones are calculated once per cycle through `RegionSpawnMask`.

### CUT

- The cut is subtracted from the base rectangle of the region before any spawn attempt.
- Random positions never land inside a cut.
- SpawnPoints inside a cut can still be attempted, but the exact position and fallback must be outside the cut to pass mask validation.
- Cuts do not affect total capacity or chunk density; they only affect where positions can be chosen.
- Multiple cuts are subtracted in sequence; the result is the remaining area.
- If cuts cover the entire region area, the mask becomes empty and no random position is found.

### ZONE

- Each zone with entities and rectangles creates a separate `SpawnArea` in the cycle.
- The zone uses its own entity pool and its own per-entity `maxAlive` (`areaId + entityId` key).
- The zone shares these with the base: total region capacity, chunk density, preset interval, `spawnPasses`, regional mode and containment.
- The zone mask is the intersection between the zone rectangles and the region boundary, minus the region cuts.
- SpawnPoints do not exist per zone; zones always use random positions inside the zone mask.
- Areas are processed in the order of the region `areas` list.
- If total capacity is already full before a zone is processed, the zone is skipped for that cycle.

### CUT + ZONE Matrix

| Configuration | Action |
|---|---|
| Region without areas | Spawns across the whole boundary, minus the 2-block internal margin. |
| Only cuts | Spawns in the boundary minus the cut areas. |
| Only zones | Base does not spawn if there are no base entities; zones spawn in their own masks. |
| Base + zones | Base spawns in the boundary minus cuts and minus zone areas; zones spawn in their own masks minus cuts. |
| Zone whose mask becomes empty after cuts | Zone cannot find a position; no mob spawns from it during that cycle. |
| Same entity in base and in a zone | `maxAlive` counts are separate by `areaId`; the entity can have up to `maxAlive` in the base AND up to `maxAlive` in the zone at the same time. |

---

## 5. Regional `spawnMode` + Preset + `spawnPasses` Combinations

The preset always works together with the regional mode and `spawnPasses`.
The preset controls:

- when to attempt (`spawnIntervalSeconds`)
- how much to attempt (`minimumSpawnQuantity` to `maximumSpawnQuantity`)
- how to position the batch (`batchSpawnPlacement`)
- whether per-chunk density can overflow inside the batch (`allowChunkDensityOverflow`)

The regional mode controls:

- which cycles allow the region to generate new mobs
- when it waits, resets or blocks

`spawnPasses` controls:

- how many spawn rounds the region attempts in the same valid cycle
- how many times the preset quantity can be rolled again before ending the cycle

### Main Matrix

| `spawnMode` | Condition before preset | Action when the interval has passed |
|---|---|---|
| `CONTINUOUS` | `living mobs < max capacity` | Allows spawn passes to refill continuously. |
| `WAVE` | Wave has not yet completed max capacity | Allows spawn passes, limited by the remaining wave amount. |
| `WAVE` | Wave is already complete and there are still living mobs | Does not spawn. Waits for all mobs in the wave to die. |
| `WAVE` | Wave is already complete and `living mobs == 0` | Resets the wave. Spawning happens in a later valid cycle. |
| `THRESHOLD` | `living mobs > floor(capacity * 0.4)` | Does not spawn. Population is still above the threshold. |
| `THRESHOLD` | `living mobs <= floor(capacity * 0.4)` | Allows spawn passes. |

### Combination with `spawnPasses`

`spawnPasses` is a region configuration, not a preset configuration. It multiplies
the attempt rounds inside a cycle that has already passed the mode, interval,
condition and capacity gates.

| Effective `spawnPasses` | Action |
|---:|---|
| `1` | Default behavior: one spawn round per valid cycle. |
| `2` to `10` | Runs multiple passes in the same cycle, rolling a new preset quantity on each pass. |
| Less than `1` in the model/JSON | Adjusted to `1`. |
| Greater than `10` in the model/JSON | Adjusted to `10`. |
| Invalid value through the GUI | Change is rejected on save. |

In the current GUI, the dropdown shows `1x` to `10x`, and the text recommends
higher values for large regions, such as 8x8 chunks or larger.

Pass rules:

- each pass calculates remaining space before rolling quantity
- in `WAVE`, each pass also respects the remaining amount needed to complete the wave
- if one pass spawns nothing, the system can still try the next pass
- passes stop early if there is no capacity or if the calculated quantity is `0`
- the region interval is only marked if at least one mob spawned across all passes
- if many configured passes remain empty, the system emits a 60-second rate-limited warning
- `batchSpawnPlacement` and `allowChunkDensityOverflow` apply inside each pass

### Combination with `batchSpawnPlacement`

`batchSpawnPlacement` decides how the slots of the same rolled quantity are
positioned inside each spawn pass, when the region is not using SpawnPoints.

Effective values:

| Preset value | Action |
|---|---|
| `Distributed` | Each batch slot chooses an eligible entity and searches for its own valid position. This is the default. |
| `Grouped` | Legacy alias. Resolves as `GroupedMixed`. |
| `GroupedMixed` | The first successful spawn in the batch locks a position. Each slot can roll different entities and try to spawn at that same position. |
| `GroupedSameType` | The first slot rolls an entity; all following slots try to use the same entity and the same locked position. |
| null, empty or invalid value | Resolves as `Distributed`. |

In the current GUI, the explicit choices are `Distributed`, `GroupedMixed` and
`GroupedSameType`. `Grouped` remains accepted for compatibility with older presets.

Grouped mode rules:

- only works when the region has no SpawnPoints
- if the region has SpawnPoints, the batch uses the SpawnPoint flow and grouping is ignored
- the first slot still needs to find a valid position normally
- each following slot revalidates the same position for the rolled entity
- all grouped mobs enter the same chunk, so chunk density increases after every success
- without overflow, `maxPerChunk` can block part of the group
- with `allowChunkDensityOverflow`, after the first success the next slots can ignore chunk density
- still respects total capacity, `maxAlive`, time/weather, terrain and spawnable type
- with `spawnPasses > 1`, each grouped pass can lock its own position

Difference between grouped modes:

| Mode | Specific action |
|---|---|
| `GroupedMixed` | Recalculates eligible entities on each slot and rolls by weight again. Can mix mob types at the same point. If one type does not fit the locked position, that slot fails and the batch continues. |
| `GroupedSameType` | Chooses one type once and tries to repeat that type for the whole batch. If that type stops being eligible because of `maxAlive` or conditions, the batch stops. |

### Combination with `allowChunkDensityOverflow`

| Preset overflow | Action |
|---|---|
| `false` | Every spawn slot must find a chunk with density lower than `maxPerChunk`, unless the region uses `ignoreChunkDensity`. |
| `true` | The first successful spawn in the batch must pass normal density. After that, the other slots in the same batch can ignore per-chunk density. |

Important overflow effects:

- does not ignore total region capacity
- does not ignore `maxAlive`
- does not ignore `maxSpawn`
- does not ignore terrain
- does not help if no first spawn can pass normal density
- resets every pass/batch

### Quantity Generated by the Preset

| Preset | Action |
|---|---|
| `minimumSpawnQuantity = maximumSpawnQuantity` | Fixed quantity per attempt. |
| `minimumSpawnQuantity < maximumSpawnQuantity` | Inclusive random quantity between min and max. |
| Quantity greater than available space | System reduces it to the remaining space. |
| In `WAVE`, quantity greater than remaining wave amount | System reduces it to complete the wave without exceeding the target. |
| `spawnPasses > 1` | Quantity is rolled again on each pass, always limited by remaining space. |

Attention: the code assumes that `maximumSpawnQuantity >= minimumSpawnQuantity`.
If a manual JSON edit inverts these values, the roll can break at runtime.

---

## 6. `terrainType` + Terrain Spawn Mode Combinations

`terrainType` lives in the entity conditions, not in the preset.
It has two different effects:

1. Resolving the block list in `terrains`.
2. Resolving the physical mode in `spawnModes`: `FLOOR`, `FLUID` or `EMPTY`.

### Full Terrain Matrix

| Effective JSON case | Resolved mode | Block list | World action |
|---|---:|---:|---|
| `terrainType = "ALL"` | all valid `spawnModes` keys | no whitelist | Attempts broad rules for each configured mode. With the current default, accepts `FLOOR` and `FLUID`. |
| `terrainType = "ALL"` without valid `spawnModes` | `FLOOR` | no whitelist | Accepts any solid ground with empty space above. |
| `terrainType = null`/empty reaching the finder raw | `FLOOR` | none | Can fail because whitelist is missing; in the normal `EntitySpawnConfig` flow, null tends to be normalized to `ALL`. |
| Terrain does not exist in `terrains` and is not in `spawnModes` | `FLOOR` | none | Does not find a valid position. |
| Terrain exists in `terrains`, but is not in `spawnModes` | `FLOOR` | yes | Spawns on a solid block that matches the whitelist. |
| Terrain exists in `terrains` and is in `spawnModes.FLOOR` | `FLOOR` | yes | Spawns on a solid block that matches the whitelist. |
| Terrain is in `spawnModes.FLOOR`, but does not exist in `terrains` | `FLOOR` | none | Does not find a valid position. |
| Terrain is in `spawnModes.FLUID` | `FLUID` | loaded as `validFluids`, but not applied yet | Spawns inside fluid, with a non-solid head block. |
| Terrain is in `spawnModes.EMPTY` | `EMPTY` | ignored | Spawns in empty/dry air, without requiring ground and without accepting fluid. |
| Terrain is in an invalid `spawnModes` key | falls back to `FLOOR` | depends on `terrains` | If it has a whitelist, acts as FLOOR; if not, position fails. |
| Same terrain listed in more than one mode | depends on map order at runtime | depends | Avoid this. The first entry found wins, but this is not a safe contract. |

### FLOOR

The entity only spawns if:

- there is a solid block below
- the block where the mob spawns is empty
- there is no fluid at the mob position
- when the rule uses a whitelist, the whitelist exists and is not empty
- when the rule uses a whitelist, the ground block ID equals or contains one whitelist item
- when the rule does not use a whitelist, any solid ground works
- the chunk at the position has available density, unless density bypass applies

Generated action: terrestrial mob spawns on top of blocks accepted by the terrain.

In the current code, `terrainType = "ALL"` generates a `FLOOR` rule without a
whitelist when `FLOOR` exists in `spawnModes`, so it accepts any solid ground.

### FLUID

The entity only spawns if:

- there is fluid at the mob position (`fluidId > 0`)
- the head block is not solid, unless it is also fluid
- density/capacity allow it

Generated action: aquatic mob spawns inside fluid.

Attention: the code already loads the list as `validFluids`, but the current
validation still does not check that list. What matters is the world's `fluidId`.

### EMPTY

The entity only spawns if:

- the block at the mob position is empty
- there is no fluid at the position
- no ground is required
- density/capacity allow it

Generated action: flying/free mob spawns in the air inside the region.

Attention: in the current code, the terrain block list is not checked for `EMPTY`.

---

## 7. Time and Weather Condition Combinations

Time and weather can exist at two levels:

- region `spawnConditions`: blocks the entire region
- entity `spawnConditions`: blocks only that entity

Terrain exists inside `spawnConditions`, but it does not participate in
`SpawnConditionValidator`; it only participates in position finding.

### Time/Weather Matrix

| Conditions | Action |
|---|---|
| Conditions `null` | Allows. |
| `timeOfDay` null/empty and `weather` null/empty | Allows. |
| `timeOfDay` contains `ALL` | Time allows. |
| `weather` contains `ALL` | Weather allows. |
| Only time configured | Time must match. Weather does not restrict. |
| Only weather configured | Weather must match. Time does not restrict. |
| Time and weather configured | Time AND weather must match. |
| Invalid value | Logs a warning. In practice, it does not match unless another value in the list matches. |
| Time/weather resource unavailable or error | Checker returns `true` to avoid blocking spawning. |

Accepted periods in code:

| Value | Approximate time |
|---|---|
| `DAWN` | 5 to before 7 |
| `MORNING` | 7 to before 12 |
| `AFTERNOON` | 12 to before 17 |
| `DUSK` | 17 to before 19 |
| `NIGHT` | 19 to before 5 |
| `Day` | 5 to before 19 |
| `ALL` | Any time |

Accepted weather:

- `CLEAR`
- `ALL`
- exact registered Hytale weather asset name, compared case-insensitively

---

## 8. Entity Combinations

Each configured entity can generate these effects.

| Entity field | Action |
|---|---|
| Positive `spawnWeight` | Enters the weighted roll. Higher weight increases relative chance. |
| `spawnWeight = 0` | In practice, it is not chosen if there are other positive weights. If all are 0, nothing is chosen. |
| Negative `spawnWeight` | Not a safe use; can break the weight sum. |
| `maxAlive` null or less/equal to 0 | No entity-specific limit. |
| `maxAlive > 0` | Entity stops being eligible when `maxAlive` living mobs already exist. |
| `maxAlive` reduced below the current living amount | Reconciler removes excess mobs of that entity. |
| False time/weather conditions | Entity does not enter the roll. |
| Conditions pass, but terrain cannot find a position | Entity can be rolled, but the attempt fails. |
| Invalid or non-spawnable NPC type | Attempt fails before entity creation. |
| Entity `presetId` | Attention: in the current region flow, this is not used to control entity spawning. |

When `dynamicConditionReconcile = true` in the region:

| Condition change | Action on living mobs |
|---|---|
| Entity was valid and no longer matches time/weather | Living mobs of that entity are removed during the cycle. |
| Entity still matches time/weather | Living mobs remain. |
| Terrain changed or no longer matches | Does not remove for this reason; terrain is not validated by the condition reconciler. |

When `dynamicConditionReconcile = false`:

| Condition change | Action on living mobs |
|---|---|
| Entity no longer matches time/weather | New spawns stop, but living mobs remain. |

---

## 9. SpawnPoint Combinations

If `spawnPoints` is empty, the region chooses random positions.
If at least one SpawnPoint exists, the region uses only SpawnPoints.

### SpawnPoint Matrix

| SpawnPoint configuration | Action |
|---|---|
| `guaranteed = true` | The point is attempted before normal points. It does not guarantee a spawn if rules fail. |
| `guaranteed = false` | The point enters the normal group, which is shuffled on each attempt. |
| `maxSpawn` null or less/equal to 0 | No point-specific limit. |
| `maxSpawn > 0` | Point is skipped when it already has that number of living mobs originating from it. |
| `maxSpawn` reduced below the existing amount | Reconciler removes excess mobs from that point. |
| Empty `allowedEntities` | The point accepts any eligible entity from the region. |
| `allowedEntities` with values | The point only accepts those entities, after global eligibility filters. |
| Empty SpawnPoint `presetId` | Uses only the region/regional preset interval. |
| Filled SpawnPoint `presetId` | Applies an extra point-specific interval for that SpawnPoint. |
| Region preset with grouped `batchSpawnPlacement` | Ignored while SpawnPoints exist; the flow uses fixed points. |

### How the SpawnPoint Position Is Used

1. Tries the exact SpawnPoint position.
2. If it fails, tries local fallback within radius 2.
3. Tested vertical offsets are `0`, `+1`, `-1`, `+2`, `-2`.
4. The position still needs to be inside the boundary.
5. The position still needs to pass terrain and density, unless density bypass applies.

### SpawnPoint + Terrain Combination

| Resolved terrain | Action at the SpawnPoint |
|---|---|
| `FLOOR` with valid whitelist | The point must be at a position with valid ground, or have a nearby valid fallback. |
| `FLOOR` without whitelist, from `ALL` | The point must be on any solid ground with empty space above. |
| `FLOOR` without whitelist, from missing specific terrain | The point and fallbacks fail. |
| `FLUID` | The point/fallback must be in fluid. |
| `EMPTY` | The point/fallback must be in an empty block without fluid. |

---

## 10. Density and Capacity Combinations

There are two different concepts:

- per-chunk density: `maxPerChunk`
- total capacity: `total region chunks * maxPerChunk`

| Configuration/scenario | Action |
|---|---|
| Chunk below `maxPerChunk` | Can receive a new spawn. |
| Chunk at `maxPerChunk` | Position in that chunk is rejected. |
| `ignoreChunkDensity = true` | Position can be accepted even in a full chunk. |
| Preset with overflow already unlocked in the batch | Position can be accepted even in a full chunk. |
| Grouped `batchSpawnPlacement` | Several slots can try to occupy the same position/chunk, quickly increasing local density. |
| `spawnPasses > 1` | Several rounds in the same cycle can fill capacity faster; local density is updated after each successful spawn. |
| Total capacity full | Region does not attempt to spawn new mobs. |
| Total living count exceeded capacity | Reconciler tries to remove excess, prioritizing chunks above the per-chunk limit. |
| `maxPerChunk <= 0` | Capacity becomes zero or negative; in practice, the region does not spawn. |

Attention: mobs in containment margin chunks also enter the region scan when they
still have the tracker for that region. This can affect count, density and capacity
until containment acts.

Attention: `ignoreChunkDensity` changes position validation, but mobs still enter
the region total count and density scan.

Attention: raw chunk scans can now be cached during the same world spawn cycle.
The result is still separated per region, and the cache is invalidated when removals
or successful spawns change the living state.

---

## 11. Reconciliation Actions

Before attempting new spawns, the system can correct already living mobs.

| Detected rule | Action |
|---|---|
| Living mob no longer matches entity time/weather and `dynamicConditionReconcile = true` | Removes that mob. |
| There are more mobs of an entity than `maxAlive` | Removes excess mobs of that entity. |
| There are more mobs from a SpawnPoint than `maxSpawn` | Removes excess mobs from that SpawnPoint. |
| Region total exceeded max capacity | Removes excess, prioritizing chunks above `maxPerChunk`. |
| Mob was tamed or retyped | Removes the `RegionTrackerComponent`; the mob stops counting, being contained or being removed by region rules. |
| Mobs were removed by reconciliation | The region performs a new scan before calculating spawn. |

Group removals (`maxAlive` and `maxSpawn`) sort by `spawnTime` and remove the
newest mobs first.

---

## 12. Containment Actions

Containment runs every 5 seconds when there is a player with a transform in the world.

| Configuration/scenario | Action |
|---|---|
| `containMobs = false` | Does not contain mobs outside the region. |
| `containMobs = true` | Attempts to contain tracked mobs that left into margin chunks. |
| Mob outside, not in combat | Can be teleported back. |
| Mob outside, in combat and still inside the margin | Does not teleport in this cycle. |
| Mob outside, in combat and beyond the margin | Can be teleported back. |
| Recent teleport | Cooldown prevents another teleport for 3 seconds. |
| Original spawn position is still inside the region | Teleports to the original position. |
| Mob came from a SpawnPoint and the point still exists | Can teleport to the SpawnPoint. |
| No original position/SpawnPoint | Searches for a safe position using the entity terrain. |
| Failed to find a safe position 3 times in a row | Removes the mob. |

Attention: `containmentMode` exists in `GlobalSpawnConfig`, but the current
containment flow does not differentiate `OFF`, `RETURN` and `TELEPORT`.
What actually enables/disables it is `containMobs`.

---

## 13. Inheritance through `parentRegionPath`

The model has resolver methods to inherit some fields from the parent, but the
current flow only uses these methods partially.

| Field | Current behavior in the spawn flow |
|---|---|
| `entities` | If the child has no entities, it uses the parent's entities. |
| `presetId` | Can come from the parent. If the child has `presetId = "default"` and has a parent, the code treats it as if it can inherit the parent. |
| Region `spawnConditions` | If the child does not have them, it uses the parent's conditions. |
| `spawnMode` | Attention: inheritance method exists, but the controller uses `getSpawnMode()` in the final decision. Child without local mode tends to fall back to `CONTINUOUS`, even with parent `WAVE`/`THRESHOLD`. |
| `ignoreChunkDensity` | Attention: inheritance method exists, but position search uses `isIgnoreChunkDensity()` directly. In practice, treat it as local. |
| `maxPerChunk` | Local to the child region. |
| `spawnPasses` | Local to the child region. There is no parent resolution in the current flow. |
| `spawnPoints` | Always local to the child region. |
| `isActive` | Local to the child region. |
| `boundary` | Local to the child region. |

---

## 14. Compact List of All Possible Actions

This is a direct list of events/actions that configurations can make happen:

1. Do not process the region because there is no online player.
2. Do not process the region because no player is nearby.
3. Do not process the region because the world does not match.
4. Do not process the region because `isActive = false`.
5. Reject the region because the boundary is too small.
6. Reject the region because the boundary is too large.
7. Block a region without a required preset.
8. Block a region without a required spawn mode.
9. Use inherited parent configuration for entities.
10. Use inherited parent configuration for preset.
11. Use inherited parent configuration for region conditions.
12. Block the whole region by time/weather.
13. Allow the whole region by time/weather.
14. Skip region without eligible entity.
15. Skip region without loaded chunks.
16. Scan living mobs from the region and build per-chunk density.
17. Scan living mobs by entity.
18. Scan living mobs by SpawnPoint.
19. Remove mobs that no longer match dynamic conditions.
20. Remove mobs above `maxAlive`.
21. Remove mobs above `maxSpawn`.
22. Remove mobs above total capacity.
23. Rescan the region after removals.
24. Block spawn because capacity is full.
25. Block spawn because preset interval has not passed.
26. Run continuous refill (`CONTINUOUS`).
27. Run wave until capacity is completed (`WAVE`).
28. Wait for complete wave cleanup.
29. Reset the wave when all mobs are dead.
30. Block by threshold above 40%.
31. Allow by threshold at 40% or below.
32. Roll quantity between preset min and max.
33. Reduce quantity by available space.
34. Reduce quantity by remaining wave amount.
35. Roll entity by weight.
36. Block entity by `maxAlive`.
37. Block entity by time/weather.
38. Fail entity because of invalid NPC type.
39. Fail entity because of non-spawnable role.
40. Search random position inside the boundary.
41. Use only SpawnPoints when they exist.
42. Prioritize `guaranteed` SpawnPoints.
43. Shuffle normal SpawnPoints.
44. Block SpawnPoint by `maxSpawn`.
45. Block SpawnPoint by its own interval.
46. Filter entity by `allowedEntities`.
47. Try exact SpawnPoint position.
48. Try local SpawnPoint fallback.
49. Validate `FLOOR` terrain.
50. Validate `FLUID` terrain.
51. Validate `EMPTY` terrain.
52. Expand `terrainType = "ALL"` to broad rules from configured spawnModes.
53. Fail specific `FLOOR` terrain without whitelist.
54. Load `validFluids` in `FLUID`, but still validate by `fluidId`.
55. Ignore whitelist in `EMPTY`.
56. Block position because chunk is full.
57. Allow full chunk through `ignoreChunkDensity`.
58. Allow full chunk through preset overflow after the first valid spawn in the batch.
59. Spawn entity in the world.
60. Remove created entity if tracker fails.
61. Attach tracker with region, entity, chunk, origin position and SpawnPoint.
62. Update local batch density after spawn.
63. Mark region interval after successful spawn.
64. Mark SpawnPoint interval after successful spawn through the point.
65. Offer border mobs for containment.
66. Teleport mob outside the region to original position.
67. Teleport mob outside the region to SpawnPoint.
68. Teleport mob outside the region to safe position by terrain.
69. Skip teleport because of combat inside the margin.
70. Skip teleport because of cooldown.
71. Remove mob outside the region after repeated safe-position failures.
72. Resolve invalid `batchSpawnPlacement` as `Distributed`.
73. Resolve legacy `Grouped` as `GroupedMixed`.
74. Distribute each batch slot to its own position with `Distributed`.
75. Lock the first valid batch position with grouped mode.
76. Mix entities at the same point with `GroupedMixed`.
77. Repeat the same entity at the same point with `GroupedSameType`.
78. Ignore grouped mode when the region has SpawnPoints.
79. Adjust `spawnPasses` lower than `1` to `1`.
80. Adjust `spawnPasses` greater than `10` to `10`.
81. Reject invalid `spawnPasses` value when saving through the GUI.
82. Run multiple spawn passes in the same valid cycle.
83. Roll preset quantity again on each pass.
84. Reduce each pass by the remaining region space.
85. Reduce each `WAVE` pass by the remaining wave amount.
86. Continue to the next pass when one pass spawns nothing.
87. Stop passes early when there is no capacity or no positive quantity.
88. Emit a warning when many configured passes remain empty.
89. Cache raw chunk reads during one world spawn cycle.
90. Invalidate scan cache after removals or successful spawns.
91. Subtract cut area (`CUT`) from the base region spawn space.
92. Create a dedicated spawn pool for a zone (`ZONE`) with separate entities and maxAlive from the base.
93. Share total capacity and per-chunk density between base and zones in the same cycle.
94. Ignore zone without entities or without rectangles during the spawn cycle.
95. Process base and zones sequentially in the same cycle, stopping if capacity is exhausted.
96. Use `areaId + entityId` key to separate maxAlive count between base and zones.

---

## 15. Behavior Recipes

These recipes are not JSON; they are effect combinations.

| Desired behavior | Combination |
|---|---|
| Region that refills whenever possible | `spawnMode = CONTINUOUS` + preset with desired interval. |
| Large region that needs to populate faster | `spawnPasses` between `2` and `10`, while keeping `maxPerChunk` and capacity coherent. |
| Dungeon by waves | `spawnMode = WAVE` + well-defined capacity through `maxPerChunk` and region size. |
| Arena that only respawns when mostly emptied | `spawnMode = THRESHOLD`. |
| Single boss | Preset `min=max=1` + `maxAlive=1` on the boss + small capacity or dedicated SpawnPoint. |
| Compact group mixing types | Preset with `batchSpawnPlacement = GroupedMixed`, no SpawnPoints, and quantity greater than 1. |
| Compact group of a single type | Preset with `batchSpawnPlacement = GroupedSameType`, no SpawnPoints, and quantity greater than 1. |
| Compact group that can exceed the chunk limit in the same batch | `GroupedMixed` or `GroupedSameType` + `allowChunkDensityOverflow = true`. |
| Mob that uses all configured physical rules | Entity with `terrainType = "ALL"`. |
| Terrestrial mob by biome/block | Entity with terrain in `spawnModes.FLOOR` and whitelist in `terrains`. |
| Aquatic mob | Entity with terrain in `spawnModes.FLUID`. |
| Flying mob | Entity with terrain in `spawnModes.EMPTY`. |
| Spawn controlled by fixed points | Add `spawnPoints`; the region stops using random positions. |
| Priority boss SpawnPoint | SpawnPoint with `guaranteed = true`, boss filter and `maxSpawn = 1`. |
| Avoid accumulation of one entity | `maxAlive` on the entity. |
| Avoid accumulation at one point | `maxSpawn` on the SpawnPoint. |
| Keep mobs trapped inside the region | `containMobs = true` globally. |
| Day mobs disappear at night | Entity condition + `dynamicConditionReconcile = true`. |
| Day mobs stop spawning at night, but living ones remain | Entity condition + `dynamicConditionReconcile = false`. |
| Region with a forbidden area in the middle | Add a `CUT` area covering the forbidden area. |
| Room with its own mobs inside a larger region | Add a `ZONE` with room rectangles and its own entities. |
| Same entity with different limits in different areas | Add the entity to the base and to the zone with different `maxAlive`; counts are independent. |

---

## 16. Attention Points Found in the Current Code

1. `terrainType = "ALL"` now expands to the valid spawn modes configured in `terrain-types.json`; it does not use a block whitelist.
2. If `spawnModes.EMPTY` exists, `terrainType = "ALL"` can also accept dry-air spawning.
3. `FLUID` loads the list as `validFluids`, but the current validation still uses `fluidId` and does not check that list.
4. `EMPTY` does not accept fluid; it must be an empty/dry-air block.
5. `terrainType` inside region conditions does not block the entire region; terrain only affects entity position.
6. `presetId` inside `EntitySpawnConfig` exists, but does not control current regional spawning.
7. `containmentMode` exists, but the current system enables/disables through `containMobs`; the modes are not differentiated.
8. Inheritance of `spawnMode` and `ignoreChunkDensity` exists in the model, but is not applied at the main spawn-flow decision point.
9. `maxPerChunk` controls both per-chunk density and total capacity. Ignoring density does not ignore total capacity.
10. SpawnPoints are not extras: when they exist, they replace the region's random position search.
11. `Grouped` is a legacy alias of `GroupedMixed`.
12. Grouped modes do not act when the region has SpawnPoints; in that case, fixed points continue to control the flow.
13. `GroupedMixed` revalidates the locked position for each rolled entity, so mixing entities with different physical terrains can make part of the batch fail.
14. `GroupedSameType` chooses an entity once; if that entity hits `maxAlive` or loses its condition, the batch stops.
15. `spawnPasses` increases attempt rounds in the same cycle, but does not increase total capacity.
16. `spawnPasses` is local to the region; it is not inherited through `parentRegionPath` in the current flow.
17. `allowChunkDensityOverflow` and the grouped locked position reset on every pass/batch.
18. The scan cache improves repeated reads in the same cycle, but is invalidated after removals or successful spawns.
19. Zones do not have their own SpawnPoints; SpawnPoints always belong to the base region and only act on the base pool.
20. Cuts only affect where random positions can land; they do not affect total capacity or chunk density.
21. The same entity can exist in the base and in zones with independent `maxAlive` because the density key uses `areaId + entityId`.
22. The zone mask is calculated as the intersection between the zone rectangles and the region boundary, minus cuts; if the result is empty, the zone does not spawn during that cycle.
23. Zones share the region preset interval; there is no zone-specific interval.
