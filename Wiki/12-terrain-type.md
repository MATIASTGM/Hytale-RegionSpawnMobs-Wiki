## Terrain Type System

The terrain type controls **which block** a mob can spawn on top of inside the region.

By default, any solid block is valid. When you define a terrain type, the mob only appears if the ground block is in that terrain's list.

---

## How It Works

When the system finds a position to spawn a mob, it:

1. Picks a random column (X, Z) inside the boundary
2. Scans downward looking for the first solid block with air above it
3. If the mob has a terrain type configured, checks if that block is in the terrain's list
4. If not, tries another column — up to 100 attempts

If no valid position is found after 100 attempts, the spawn is skipped for that cycle.

---

## Default Terrain Types

The `terrain-types.json` file is created automatically on first run with three terrains:

| Terrain | Included blocks |
|---|---|
| `land` | All grass, dirt and sand variants |
| `cave` | Stone (`Rock_stone`) |
| `water` | Water (`Fluid_water`) |

---

## Configuring in JSON

The terrain type is set inside `spawnConditions` for each entity:

```json
"spawnConditions": {
  "terrainType": "land"
}
```

**Accepted values:**
- Name of a terrain defined in `terrain-types.json`
- `"ALL"` — no restriction, any solid block is valid (default)
- Case-insensitive: `"Land"`, `"LAND"`, `"land"` all work the same

---

## Editing terrain-types.json

The file is located in the plugin folder. You can add, remove or modify terrains:

```json
{
  "terrains": {
    "land": ["Soil_Grass", "Soil_Dirt", "Soil_Sand"],
    "cave": ["Rock_stone"],
    "water": ["Fluid_water"],
    "snow": ["Soil_Snow", "Soil_Snow_Full"]
  }
}
```

Block names are checked by **contains** (not exact match), so `"Soil_Grass"` accepts any block whose ID contains that string.

---

## Troubleshooting

**Mobs not spawning with terrain configured:**
- Enable debug with `/rsmdebug` and look for `[TERRAIN]` in the logs to see which blocks are being accepted or rejected
- Check that the terrain name in `spawnConditions` matches exactly the key in `terrain-types.json`
- If the region has no blocks from the configured terrain, all 100 attempts will fail — the spawn is silently skipped
