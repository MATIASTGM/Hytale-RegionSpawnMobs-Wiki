# How the Spawn System Works

This system has its own way of working.
Once you understand the core idea, everything else falls into place.

---

## The Core Idea in One Sentence

**Mobs appear inside regions. Players activate regions by being nearby. That's it.**

There is no "spawn around the player". The player is the trigger, not the target.

---

## The Three Controls

Everything in this system comes down to three numbers:

```
playerRangeSpawnChunks   How far a player needs to be to activate a region
maxPerChunk              How many mobs can exist per chunk inside the region
Region Size              How large the region is = how many mobs it can hold in total
```

That's it. Three controls. No hidden behavior.

---

## Visual: How It Works Step by Step

```
Every second:

STEP 1 — Find active regions
┌─────────────────────────────────────────┐
│                                         │
│   [ Region A ]                          │
│                                         │
│              ★ Player                   │  ← Player is here
│                  └──────────────────┐   │
│                  playerRangeChunks  │   │
│                  = search radius    │   │
│                                     ▼   │
│                           [ Region B ] │  ← Inside radius, will be processed
│                                         │
│   [ Region C ]                          │  ← Outside radius, ignored
└─────────────────────────────────────────┘

STEP 2 — Validate each found region
  ✓ Region is active?
  ✓ Boundary is at least 4x4x4 blocks?
  ✓ Has mobs configured?
  ✓ Weather/time conditions met?
  ✓ Has available capacity?
  ✓ Spawn interval has passed?
  → All yes? Spawn happens.
  → Any no? Skip this cycle.

STEP 3 — Find a position inside the boundary
  Random (X, Z) inside the boundary
       ↓
  Scans downward for solid ground
       ↓
  Checks chunk density < maxPerChunk
       ↓
  Spawns mob here
```

---

## Visual: Capacity — How Region Size Controls Mob Count

The maximum number of mobs a region can hold is calculated automatically:

```
Max mobs = Total chunks in boundary × maxPerChunk

Example A — Small region (4x4 blocks = 1 chunk):
  1 chunk × 3 maxPerChunk = 3 mobs maximum

Example B — Medium region (64x64 blocks = 4 chunks):
  4 chunks × 3 maxPerChunk = 12 mobs maximum

Example C — Large region (128x128 blocks = 16 chunks):
  16 chunks × 3 maxPerChunk = 48 mobs maximum
```

**You never set a fixed mob count. You set the density. The region size does the rest.**

---

## Visual: Chunk Density — How Mobs Distribute

```
Region boundary (top view):
┌────────────────────────────┐
│  Chunk 1  │  Chunk 2       │
│  [mob]    │  [mob][mob]    │
│  [mob]    │                │
│  ─────────┼────────────    │
│  Chunk 3  │  Chunk 4       │
│           │  [mob]         │
│           │                │
└────────────────────────────┘

maxPerChunk = 3

Chunk 1: 2 mobs → can spawn 1 more ✓
Chunk 2: 2 mobs → can spawn 1 more ✓
Chunk 3: 0 mobs → can spawn 3 more ✓
Chunk 4: 1 mob  → can spawn 2 more ✓
```

Mobs distribute naturally across the region because full chunks are skipped.
No configuration needed — this happens automatically.

---

## What Happens with Multiple Players

```
Player A ──→ finds Region X ──→ Region X is processed once
Player B ──→ finds Region X ──→ already processed this second, skipped
Player C ──→ finds Region Y ──→ Region Y is processed once
```

Each region is processed **at most once per second**, regardless of how many
players are nearby. 10 players near the same region = same cost as 1 player.

---

## FAQ

**"I set playerRangeSpawnChunks = 3 but mobs are spawning far from me"**

That is the correct behavior. The player activates the region, but mobs appear
anywhere inside the boundary — not necessarily near you. If you want mobs near
you, make the region smaller and position it around where players usually stand.

---

**"My small region (4x4x4) never spawns mobs"**

Check two things:
1. Are you within `playerRangeSpawnChunks` of the region?
2. Is the region already at max capacity? (`/rsminfo <name>` shows the current count)

A 4x4x4 region has 1 chunk. With `maxPerChunk = 3`, it holds 3 mobs maximum.
If there are already 3 mobs there, nothing will spawn until one dies.

---

**"I want more mobs in my region"**

Three options:
- Enlarge the region (more chunks = more capacity automatically)
- Increase `maxPerChunk` globally with `/rsmglobalmaxperchunk <value>` (affects all regions)

---

**"I want mobs to spawn faster"**

Decrease the spawn interval in the preset assigned to the region:
```
/rsmpresetinterval <preset_name> <seconds>
```

---

**"I disabled a region but the mobs are still there"**

Disabling a region stops new spawns. Existing mobs remain until they die
or you remove them manually with `/rsmkill <region_name>`.

---

## Spawn Modes

Each region has a `spawnMode` that controls how it decides to spawn mobs. You configure it through the edit menu (`/rsmeditmenu`).

There are three modes:

### CONTINUOUS
The default mode. The region replenishes mobs continuously as long as there is space and the configured interval has passed. This is the same behavior that existed before version 1.1.2.

```
Every cycle:
  adjustedCount < maxCapacity?
  Interval passed?
  → Yes to both? Spawn happens.
```

### WAVE
The region spawns mobs gradually until it reaches maximum capacity. Once the wave is complete, no new mobs appear — even if some die. Only when **all mobs from the wave die** does the next wave begin.

```
Every cycle:
  Wave already complete?
  → Yes + adjustedCount == 0? Reset wave, next wave can start.
  → Yes + adjustedCount > 0? Wait. No spawn.
  → No? Spawn up to the remaining wave target.
```

Ideal for dungeons, catacombs and challenge areas where you want to control the flow of enemies.

---

## SpawnPoints

When a region has SpawnPoints configured, the system uses exclusively those positions to spawn mobs instead of random positions inside the boundary.

Create a SpawnPoint by standing inside the region and running:
```bash
/rsmaddspawnpoint [name]
```

Each SpawnPoint can have:
- an entity filter (`allowedEntities`)
- `maxSpawn`

If the filter is empty, the SpawnPoint uses the full region mob pool. If it has mobs configured, only those mobs appear at that point.

`maxSpawn` limits how many living mobs originating from that point can exist at the same time.
If it is empty, the SpawnPoint has no local limit.

SpawnPoints are managed through the region edit menu (`/rsmeditmenu`).

---

## maxAlive per Entity

Each entity in the region can also have `maxAlive`.

`maxAlive` limits how many living mobs of that type can exist in the whole region,
regardless of how many SpawnPoints accept that entity.

Example:
- `guard` with `maxAlive = 4`
- SpawnPoint `gate_a` with `maxSpawn = 2`
- SpawnPoint `gate_b` with `maxSpawn = 2`

Result:
- the system can distribute up to 4 guards in the region
- never more than 2 at `gate_a`
- never more than 2 at `gate_b`

---

### THRESHOLD
The region only resumes spawning when the mob population drops below 40% of maximum capacity. Does not require a full clear like WAVE, but does not replenish continuously like CONTINUOUS.

```
threshold = floor(maxCapacity * 0.4)

Every cycle:
  adjustedCount > threshold? → Blocked. No spawn.
  adjustedCount <= threshold? → Spawn normally.
```

Ideal for strong areas where you want constant pressure without excessive spam.

> **Note:** If a region using THRESHOLD has a very low maximum capacity, the server will log a one-time warning informing that the behavior will be close to WAVE.

---

## Parent System — Configuration Inheritance

A region can inherit configurations from another region through the `parentRegionPath` field.

When an inheritable field is empty in the child region, the system automatically uses the parent's value. If there is a local value, the local value is used.

**Inheritable fields:** `entities`, `presetId`, `spawnMode`, `spawnConditions`, `ignoreChunkDensity`

**Always local fields:** `boundary`, `spawnPoints`, `isActive`, `regionName`, `folderPath`, `maxPerChunk`

This allows creating a "template" region with all mobs configured and linking multiple regions to it, each with its own boundary and SpawnPoints.

Configured through the region edit menu — **Parent** field in the REGION section.

---

## Preset and Mode — Required Fields

When creating a region, `preset` and `spawnMode` start as `(none)` — no silent default value.

- **Without parent:** preset and mode are required. The system blocks the save in the menu if they are `(none)` and logs a warning in the console every 15 seconds while the region is incomplete
- **With parent:** can leave as `(none)` — the values will be inherited from the parent

This ensures no region operates in an undefined state without the admin noticing.

---

## What Does NOT Exist in This System

Some features admins commonly ask about are intentionally not included:

| Feature | Status |
|---|---|
| Spawn mobs at a fixed distance from the player | ❌ Does not exist in this system |
| Spawn mobs only in front of the player | ❌ Does not exist in this system |
| Mobs disappear when the player leaves | ❌ Mobs persist — use containment |
| Sub-regions | ❌ Removed in 1.1.2 — use folders for organization or the parent system to inherit configurations |

What now exists as additional fine control:
- `maxSpawn` per SpawnPoint
- `maxAlive` per entity

These features sit on top of the current system without replacing the core region/chunk/capacity logic.
