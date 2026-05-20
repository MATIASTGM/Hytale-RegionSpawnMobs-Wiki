# RegionSpawnMobs – Admin Guide (BETA)

## Project Status: BETA
**Version:** 1.1.3

This plugin is under active development and currently in **BETA**.

- Core features are working and optimized
- Bugs may occur — please report issues
- Feedback and suggestions are welcome

**Recommendation:** Test on a development server before using in production.

---

## What is RegionSpawnMobs?
RegionSpawnMobs is a plugin for Hytale servers that allows admins to create **custom mob spawn regions** with control over:

- **Where** mobs appear (exact 3D area)
- **Which** mobs appear (multiple types per region)
- **When** they appear (configurable interval via presets)
- **How many** appear (dynamic capacity based on region size)
- **Player proximity** (spawn only when players are nearby — in chunks)
- **Chunk density** (limits mobs per chunk to avoid overcrowding)
- **Spawn mode** (CONTINUOUS, WAVE or THRESHOLD per region)
- **SpawnPoints** (fixed positions inside the region where mobs appear)
- **Smart containment** (keeps mobs in the region without breaking combat)
- **Folder organization** (group regions into folders and sub-folders)

---

## Use Cases
- **Combat Arenas:** continuous PvE spawns within a fixed area
- **Dungeons/Raids:** specific rooms with specific mobs and spawn rates
- **Farm Zones:** predictable spawns for grinding
- **Natural Habitats:** controlled spawns for themed areas
- **Special Events:** temporary regions you can enable/disable
- **Content Hierarchy:** organize regions in folders like `dungeons/boss_room` or `world/forest/wolves`

---

## Quick Start

### 1) Select the area
```bash
/pos1
/pos2
```

### 2) Create the region
```bash
/rsmcreate <region_name>
```
Or with a folder:
```bash
/rsmcreate <folder> <region_name>
/rsmcreate <folder> <category> <region_name>
```
Example:
```bash
/rsmcreate wolf_arena
/rsmcreate dungeons wolf_arena
/rsmcreate dungeons bosses wolf_arena
```

### 3) Add mobs (with weights)
```bash
/rsmaddmob <region_name> <mob_type> <weight>
```
Example:
```bash
/rsmaddmob wolf_arena wolf_black 70
/rsmaddmob wolf_arena wolf_white 30
```
(70/30 weight split)

### 4) (Optional) Add SpawnPoints
Stand inside the region and run:
```bash
/rsmaddspawnpoint [name]
```
When SpawnPoints exist, mobs spawn exclusively at those positions.

---

## Folder Organization

Regions can be grouped into folders to make managing large servers easier:

```
regions/
├── wolf_arena.json          ← root
├── dungeons/
│   ├── entrance.json
│   └── boss_room.json
└── world/
    ├── forest/
    │   └── wolves.json
    └── caves/
        └── skeletons.json
```

To reference a region with a folder, use the full path:
```bash
/rsminfo dungeons/boss_room
/rsmeditmenu world/forest/wolves
```

If the name is unique across all folders, it can be used without the path:
```bash
/rsminfo boss_room
```

---

## Support
When reporting a bug, include:

- Plugin version (currently 1.1.3)
- Hytale server version
- Commands used
- Error messages (if any)
- Server logs (with `/rsmdebug` enabled if possible)
- Region configuration (`/rsminfo <name>`)
- Global settings (`/rsmglobal`)
