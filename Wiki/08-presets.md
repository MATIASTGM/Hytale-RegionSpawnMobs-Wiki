## Spawn Preset System

Presets are **reusable spawn configuration templates**. Apply a preset to multiple regions to maintain consistency and simplify updates.

### Commands

```bash
/rsmpresetlist                                   # List all presets
/rsmpresetinfo <name>                            # View preset details
/rsmpresetcreate <name> <interval> <min> <max>   # Create preset
/rsmpresetinterval <name> <seconds>              # Change interval
/rsmpresetminqty <name> <quantity>               # Change minimum quantity
/rsmpresetmaxqty <name> <quantity>               # Change maximum quantity
/rsmpresetdelete <name>                          # Delete preset
```

### Default Presets

The plugin automatically creates 4 presets on first startup:

| Preset | Interval | Min | Max |
|---|---|---|---|
| `default` | 30s | 1 | 3 |
| `dungeon_easy` | 10s | 1 | 2 |
| `dungeon_hard` | 3s | 2 | 5 |
| `boss_spawner` | 120s | 1 | 1 |

### File Format (`presets.json`)

```json
{
  "default": {
    "spawnIntervalSeconds": 30,
    "minimumSpawnQuantity": 1,
    "maximumSpawnQuantity": 3
  },
  "dungeon_easy": {
    "spawnIntervalSeconds": 10,
    "minimumSpawnQuantity": 1,
    "maximumSpawnQuantity": 2
  },
  "dungeon_hard": {
    "spawnIntervalSeconds": 3,
    "minimumSpawnQuantity": 2,
    "maximumSpawnQuantity": 5
  },
  "boss_spawner": {
    "spawnIntervalSeconds": 120,
    "minimumSpawnQuantity": 1,
    "maximumSpawnQuantity": 1
  }
}
```

### Preset Fields

- `spawnIntervalSeconds` — time between spawn attempts (default: `30`)
- `minimumSpawnQuantity` — minimum mobs per attempt (default: `1`)
- `maximumSpawnQuantity` — maximum mobs per attempt (default: `3`)

### How to Apply to a Region

The `presetId` field in the region JSON references the preset by name:

```json
{
  "regionName": "wolf_arena",
  "presetId": "dungeon_hard",
  ...
}
```

### Troubleshooting

**Preset not applied:** check that the name is exact (case-sensitive) and use `/rsmreload` after editing the file.

**Region using wrong preset:** edit the `presetId` in the region JSON and reload with `/rsmreload`.
