## Data Saving

- Regions are saved automatically as JSON
- Base path: `mods/RegionSpawnMobs/regions/`
- Global config: `mods/RegionSpawnMobs/global_config.json`
- Markers: `mods/RegionSpawnMobs/markers.json`
- Each region is saved at `regions/<folderPath>/<name>.json`
- Regions at root (no folder) are saved at `regions/<name>.json`
- **Automatic backup:** on every save, the system creates `<name>.json.bak` automatically
- **Manual backup:** copy the `RegionSpawnMobs/` folder

### Folder Structure

```
mods/RegionSpawnMobs/
├── regions/
│   ├── wolf_arena.json                # Root region
│   ├── wolf_arena.json.bak            # Automatic backup
│   ├── dungeons/
│   │   ├── entrance.json              # Region in folder
│   │   ├── entrance.json.bak
│   │   ├── boss_room.json
│   │   └── boss_room.json.bak
│   └── world/
│       └── forest/
│           └── wolves.json            # Region in nested folder
├── global_config.json
├── presets.json
└── terrain-types.json
```

> Empty folders are removed automatically when all regions inside them are deleted.

### JSON Format Examples

#### Region JSON (`wolf_arena.json`)
```json
{
  "regionId": "425d0f39-0098-4d00-bcb2-27af5bd59589",
  "regionName": "wolf_arena",
  "folderPath": "",
  "presetId": "default",
  "isActive": true,
  "maxPerChunk": 5,
  "spawnMode": "CONTINUOUS",
  "ignoreChunkDensity": false,
  "dynamicConditionReconcile": true,
  "spawnPasses": 1,
  "boundary": {
    "worldName": "default",
    "firstPosition": {
      "x": -256.0,
      "y": 119.0,
      "z": -96.0
    },
    "secondPosition": {
      "x": -225.0,
      "y": 135.0,
      "z": -65.0
    }
  },
  "entities": {
    "wolf_black": {
      "spawnWeight": 40,
      "maxAlive": 4,
      "spawnConditions": {
        "timeOfDay": ["ALL"],
        "weather": ["ALL"],
        "terrainType": "ALL"
      }
    },
    "skeleton": {
      "spawnWeight": 30,
      "spawnConditions": {
        "timeOfDay": ["NIGHT", "DUSK"],
        "weather": ["ALL"],
        "terrainType": "ALL"
      }
    }
  },
  "spawnPoints": [
    {
      "spawnPointId": "a1b2c3d4-0000-0000-0000-000000000001",
      "name": "entrance",
      "x": -240.0,
      "y": 120.0,
      "z": -80.0,
      "maxSpawn": 2,
      "allowedEntities": []
    }
  ],
  "areas": []
}
```

#### Region with Folder JSON (`dungeons/boss_room.json`)
```json
{
  "regionId": "7a3c1f20-1234-4b00-aaaa-000000000001",
  "regionName": "boss_room",
  "folderPath": "dungeons",
  "presetId": "dungeon_hard",
  "isActive": true,
  "maxPerChunk": 3,
  "spawnMode": "WAVE",
  "boundary": { ... },
  "entities": { ... },
  "spawnPoints": []
}
```

#### Global Config JSON (`global_config.json`)
```json
{
  "playerRangeSpawnChunks": 2,
  "maxPerChunkDefault": 3,
  "containMobs": true,
  "containmentMode": "TELEPORT",
  "containmentMarginChunks": 3,
  "markerReconcileEnabled": true,
  "markerReconcileIntervalSeconds": 10,
  "markerReconcileMaxRefsPerPass": 200
}
```

#### Markers JSON (`markers.json`)
```json
[
  {
    "markerId": "ea9ed2e4-b423-4da5-96e5-501d55dfda0c",
    "name": "wolf_marker",
    "worldName": "default",
    "x": -240.0,
    "y": 120.0,
    "z": -80.0,
    "active": true,
    "presetId": "Default_Markers",
    "spawnMode": "CONTINUOUS",
    "activationDistance": 32,
    "leashRadius": 24,
    "entities": {
      "wolf_black": {
        "spawnWeight": 100,
        "maxAlive": 1,
        "spawnConditions": {
          "timeOfDay": ["ALL"],
          "weather": ["ALL"],
          "terrainType": "ALL"
        }
      }
    }
  }
]
```

### JSON Field Reference

**Region Fields:**
- `regionId` - Unique region UUID (generated automatically)
- `regionName` - Region name (must match the file name)
- `folderPath` - Folder path where the region is organized (empty = root)
- `presetId` - Preset template ID (default: `"default"`)
- `isActive` - Whether the region is enabled (`true`/`false`)
- `maxPerChunk` - Max mobs per chunk in this region
- `spawnMode` - Spawn mode: `CONTINUOUS`, `WAVE` or `THRESHOLD`
- `ignoreChunkDensity` - Ignores per-chunk density when this region spawns
- `dynamicConditionReconcile` - Removes tracked mobs when their conditions no longer match
- `spawnPasses` - Number of spawn attempts the region can run per cycle
- `boundary` - 3D area definition with world name and two corner positions
- `entities` - Map of mob types with spawn weights, limits and conditions
- `spawnPoints` - List of fixed spawn positions (can be empty)
- `areas` - Zones and cuts used to refine valid spawn positions

**SpawnPoint Fields:**
- `spawnPointId` - Unique SpawnPoint UUID (generated automatically)
- `name` - Optional name for identification
- `x`, `y`, `z` - Exact position in the world
- `maxSpawn` - Optional limit of living mobs originating from this SpawnPoint (`null` = unlimited)
- `allowedEntities` - List of allowed mobs at this point. If empty, uses the full region mob pool

**Entity Fields (`EntitySpawnConfig`):**
- `spawnWeight` - Relative spawn weight (default: `100`)
- `maxAlive` - Optional limit of living mobs of this type inside the region (`null` = unlimited)
- `presetId` - Entity-specific preset (optional, overrides the region's preset)
- `spawnConditions` - Spawn conditions (time, weather and terrain)

**Spawn Conditions:**
- `timeOfDay` - When to spawn:
  - **Specific periods**: `Dawn` (05-07), `Morning` (07-12), `Afternoon` (12-17), `Dusk` (17-19), `Night` (19-05)
  - **Aliases**: `Day` (Dawn + Morning + Afternoon + Dusk), `Night`
  - **Special**: `ALL` (any time)
- `weather` - Weather conditions (see Weather System)
- `terrainType` - Terrain type (see Terrain Type System)

**Global Config Fields:**
- `playerRangeSpawnChunks` - Player distance for spawn (in chunks, default: `2`)
- `maxPerChunkDefault` - Default max mobs per chunk (default: `3`)
- `containMobs` - Enable/disable containment system (default: `true`)
- `containmentMode` - How to contain mobs: `OFF`, `RETURN`, `TELEPORT` (default: `TELEPORT`)
- `containmentMarginChunks` - Containment safety margin in chunks (default: `3`)
- `markerReconcileEnabled` - Enable/disable automatic marker mob reconciliation
- `markerReconcileIntervalSeconds` - Interval between marker reconcile scans
- `markerReconcileMaxRefsPerPass` - Maximum refs inspected per marker reconcile pass

**Marker Fields:**
- `markerId` - Unique marker UUID
- `name` - Marker name
- `worldName`, `x`, `y`, `z` - Marker world and position
- `active` - Enable/disable the marker
- `presetId` - Preset used by the marker, usually `Default_Markers`
- `spawnMode` - Marker refill mode
- `activationDistance` - Distance in blocks for a player to activate the marker
- `leashRadius` - Normal out-of-combat return distance
- `entities` - Mobs the marker can spawn

Starting in version 1.1.8, marker mobs that are tamed or retyped release the marker tracker. They stop counting against marker capacity, stop teleporting to the marker and are not removed by the marker's time/weather conditions.

### folderPath Normalization

- If an old file has no `folderPath`, the system infers it from the real folder and fixes the JSON automatically
- If `folderPath` is wrong, the system auto-corrects it based on the file location
- The real folder inside `regions/` is the source of truth for organization

### Legacy Sub-Region Migration

If you had sub-regions from versions prior to 1.1.2, the system migrates them automatically:

- Files with `subRegionName` are detected as legacy format
- Each sub-region becomes an independent region with `folderPath` equal to the parent region name
- Example: sub-region `boss` of region `wolf_arena` becomes region `boss` in folder `wolf_arena`
- The legacy file is replaced with the new format and a warning is logged

No manual action is required.
