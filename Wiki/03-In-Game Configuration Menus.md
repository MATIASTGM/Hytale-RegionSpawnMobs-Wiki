# Configuration Menu

RegionSpawnMobs has a built-in GUI for Hytale to configure everything without editing JSON files or using commands. All menus are accessible directly in-game.

---

## Menu Overview

There are multiple menus in total:

| Menu | Command | What it does |
|------|---------|--------------| 
| Region | `/rsmeditmenu [name]` | Configure a region: mobs, SpawnPoints, spawn mode |
| Preset | `/rsmpresetmenu` | Create, edit and remove presets |
| Global Config | `/rsmglobalmenu` | Configure global options |
| Terrain Type | `/rsmterrainmenu` | Manage terrain types |
| Central Config | `/rsmconfigmenu` | Access the main configuration tools |
| Zone/Cut | commands and UI items | Create, edit and remove zones and cuts |
| Marker | commands and UI items | Create, edit and remove Spawn Markers |

> **Note:** The sub-region menu was removed in version 1.1.2. All region configuration is done through the single `/rsmeditmenu`.

---

## Region Menu

### Info Screen
Opened with `/rsminfomenu [name]` or the **Info** button inside the editor.

```
[ IMAGE: menu-region-info.png ]
```

Displays:
- Full region path (`folder/name`) and world
- Status (active/inactive)
- Assigned preset
- Region MaxPerChunk
- Total chunks and dimensions (e.g. `4 (2x2)`)
- Current mob count vs max capacity (e.g. `7/12`)
- List of configured mobs with weight, time, weather, terrain and `maxAlive` for each
- List of SpawnPoints with position, `maxSpawn` and entity filter

### Edit Screen
Opened with `/rsmeditmenu [name]` or the **Edit** button on the info screen.

If the name is omitted, the menu automatically detects the region by player position.

```
[ IMAGE: menu-region-edit.png ]
```

The menu is visually organized into main sections:
- `Region`
- `SpawnPoints`
- `Mobs`
- `Zones`

#### REGION Section
- **Name** — rename the region (cannot contain `/` or `\`)
- **Folder** — folder path (read-only on this screen)
- **Parent** — dropdown to select a parent region (see [Parent System](#parent-system))
- **World** — region world (read-only)
- **Chunks** — total chunks and dimensions (read-only)
- **Mob Count** — current mobs vs max capacity (read-only)
- **Active** — checkbox to enable or disable the region
- **Ignore Chunk** — checkbox to ignore chunk density when you want stronger manual control through SpawnPoints
- **Dynamic Condition Reconcile** — checkbox to remove tracked mobs when their time/weather conditions stop matching
- **Spawn Passes** — how many spawn attempts the region can run per cycle
- **Preset** — dropdown to select the assigned preset (`(none)` if not configured)
- **Max / Chunk** — mob limit per chunk for this region

#### SPAWN MODE Section
- **Mode** — dropdown with four options:
  - `(none)` — not configured (must configure or have a parent)
  - `Continuous` — replenishes mobs continuously (default)
  - `Wave` — waits for all mobs to die before the next wave
  - `Threshold` — only spawns when population drops below 40% of capacity

> **Important:** Preset and Mode are required for the region to function. If the region has no parent configured, the system blocks the save and displays an error message. If it has a parent, `(none)` is valid — the values will be inherited.

#### SPAWN POINTS Section
- **Selected** — dropdown to select the active SpawnPoint
- **Count** — total SpawnPoints in the region
- **Point ID** — short identifier of the selected SpawnPoint
- **Name** — editable name for the selected SpawnPoint
- **Max Spawn** — dropdown with `Unlimited` or `1..50`
- **Position** — SpawnPoint position (read-only)
- **Allow Mob** — dropdown to add a mob to the SpawnPoint filter
- **Remove Mob** — dropdown to remove a mob from the SpawnPoint filter
- **Save SpawnPoint** button — saves the name, `maxSpawn` and applies the `Allow/Remove` selections
- **Delete SpawnPoint** button — removes the selected SpawnPoint

> SpawnPoints are created with the `/rsmaddspawnpoint [name]` command while standing inside the region.

#### MOBS Section
- **Mob ID** — text field for the mob type
- **Spawn Weight** — spawn weight
- **Max Alive** — dropdown with `Unlimited` or `1..200`
- **Time of Day** — dropdown (ALL, Dawn, Morning, Afternoon, Dusk, Night, Day)
- **Weather** — weather dropdown
- **Terrain Type** — terrain type dropdown
- **Add / Update Mob** button — adds or updates the mob with the given values
- **Remove Mob** — dropdown to select the mob to remove
- **Remove Mob** button — removes the selected mob
- **Mob List** — list of all configured mobs with their parameters

Clicking **Save Region**, **Save SpawnPoint** or **Add / Update Mob** saves the corresponding section immediately.

#### ZONES / CUTS Section
- **Zones** — include sub-areas where spawn is allowed
- **Cuts** — excluded boxes where spawn should not happen
- They are edited through the Zone/Cut tools and menus

---

## Parent System

Any region can inherit configurations from another region through the **Parent** field.

### What it's for

Avoids reconfiguring the same mobs, preset and conditions across multiple regions. Create one region with everything configured and use it as the parent for the others.

**Practical example:**
```
Region "animals" (parent)
  → entities: cow, pig, chicken
  → preset: fast
  → mode: CONTINUOUS

Region "forest-north" (parent = animals)
  → own boundary
  → inherits everything from "animals"

Region "forest-south" (parent = animals)
  → own boundary
  → inherits everything from "animals"
```

### Inheritable fields

| Field | Inheritable |
|-------|-------------|
| `entities` (mob list) | ✅ |
| `presetId` | ✅ |
| `spawnMode` | ✅ |
| `spawnConditions` (time/weather/terrain) | ✅ |
| `ignoreChunkDensity` | ✅ |
| `boundary` | ❌ always local |
| `spawnPoints` | ❌ always local |
| `isActive` | ❌ always local |
| `regionName` / `folderPath` | ❌ always local |
| `maxPerChunk` | ❌ always local |

| `spawnPasses` | always local |

### Rules

- **1 level maximum** — a parent cannot have a parent
- **Cannot be itself** — the dropdown does not list the region being edited
- **Deletion blocked** — a region with children linked to it cannot be deleted
- **Automatic rename** — when renaming the parent through the menu, the `parentRegionPath` of all children is updated automatically
- **Moving files manually** — the admin's responsibility; the system attempts to resolve by name as a fallback

### How to configure

1. Open the child region menu with `/rsmeditmenu <name>`
2. In the **Parent** field, select the parent region from the dropdown
3. Click **Save Region**
4. Empty inheritable fields will now use the parent's values

To remove the link, select `(none)` in the Parent dropdown and save.

> **Note:** The parent is stored as a full path string (`folderPath/regionName`) in the JSON. If the parent file is moved manually outside the mod, the system attempts to resolve by name. If not found, the region operates as independent and logs a warning in the console.

---

## Preset Menu

### Info Screen
Opened with `/rsmpresetinfomenu`.

```
[ IMAGE: menu-preset-info.png ]
```

Displays:
- Total registered presets
- For each preset: name, interval in seconds and quantity range (min-max)

### Edit Screen
Opened with `/rsmpresetmenu` or the **Edit** button on the info screen.

```
[ IMAGE: menu-preset-edit.png ]
```

Allows in a single screen:
- **Add preset** — enter name, interval, minimum and maximum quantity
- **Edit existing preset** — enter name and new values
- **Remove preset** — enter the name to remove

---

## Global Config Menu

Opened with `/rsmglobalmenu`.

```
[ IMAGE: menu-global-config.png ]
```

Displays current values and allows editing:

| Field | Description |
|-------|-------------|
| `playerRangeSpawnChunks` | Player distance to activate regions (in chunks) |
| `maxPerChunkDefault` | Default mob limit per chunk (for all regions) |
| `containMobs` | Enable or disable the containment system (checkbox) |
| `containmentMode` | Containment mode: `OFF`, `RETURN` or `TELEPORT` |
| `containmentMarginChunks` | Containment safety margin in chunks |
| `markerReconcileEnabled` | Enable/disable automatic marker mob reconciliation |
| `markerReconcileIntervalSeconds` | Marker reconciliation interval |
| `markerReconcileMaxRefsPerPass` | Maximum marker refs inspected per reconcile pass |

#### Preset extra fields

Presets can also store:

| Field | Description |
|-------|-------------|
| `batchSpawnPlacement` | Keeps mobs in the same batch closer together when possible |
| `allowChunkDensityOverflow` | Allows a batch to finish even if the chunk reaches density during that batch |

Clicking **Apply** saves the configuration immediately without restarting the server.

---

## Navigation Between Screens

Info and edit menus are connected by buttons:

```
Info Screen  ──[ Edit ]──→  Edit Screen
Edit Screen  ──[ Info ]──→  Info Screen
```

You can close any menu at any time without saving — changes are only applied when clicking the save button for the corresponding section.
