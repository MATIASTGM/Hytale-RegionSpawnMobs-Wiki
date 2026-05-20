## Containment System Explained

### How Chunk-Based Containment Works

The containment system uses **chunk coordinates** instead of exact block positions for performance reasons. This creates an important behavior you need to understand:

### Containment Margin

**Key Concept:** The system checks mobs using `containmentMarginChunks` (default: 3 chunks = 96 blocks)

```
Region Boundary (exact blocks)
┌─────────────────────────┐
│                         │  ← Your defined region
│    Actual Region        │
│                         │
└─────────────────────────┘

Containment Check Area (chunks)
╔═══════════════════════════╗
║  ┌─────────────────────┐  ║
║  │                     │  ║  ← Margin: 3 chunks
║  │  Actual Region      │  ║     (96 blocks)
║  │                     │  ║
║  └─────────────────────┘  ║
╚═══════════════════════════╝
     ↑
  Mobs can walk here before being teleported!
```

### Important: Small Region Behavior

**If your region is smaller than 1 chunk (32 blocks):**

Mobs will have **free space to walk outside** the region boundary before being teleported back!

**Why?** The system checks containment by chunks, not exact blocks.

**Example Visual:**
```
[IMAGE: containment_small_region.png]
(Screenshot showing small region with mob walking outside)
```

**Example:**
- Region size: 20x20 blocks (smaller than 1 chunk)
- Containment margin: 3 chunks (96 blocks)
- Result: Mobs can walk up to ~96 blocks before teleport

### Configuration

**In `global_config.json`:**
```json
{
  "containMobs": true,
  "containmentMode": "TELEPORT",
  "containmentMarginChunks": 3
}
```

**Modes:**
- `OFF` - No containment (mobs roam freely). Sets `containMobs` to `false` automatically
- `RETURN` - Pushes mobs back with velocity
- `TELEPORT` - Instant teleport back (recommended)

**Command to change mode:**
```bash
/rsmcontainment OFF
/rsmcontainment RETURN
/rsmcontainment TELEPORT
```



### Recommendations

**For Small Regions (< 32 blocks):**
```bash
# Reduce margin for more precise containment
# Edit global_config.json:
"containmentMarginChunks": 1    # 32-block margin
```

**For Large Regions (> 100 blocks):**
```bash
# Keep the default margin
"containmentMarginChunks": 3    # 96-block margin (default)
```

**For Boss Arenas:**
```bash
# Precise containment
"containmentMarginChunks": 0    # No margin (exact chunk boundary)
"containmentMode": "TELEPORT"   # Instant teleport
```

### How It Works

1. **Every 5 seconds**, the system checks mob positions (only if there are players in the world)
2. Converts mob position to **chunk coordinates**
3. Checks if the mob is outside the region + margin
4. If outside:
   - Skips if mob is in combat and still within the margin
   - Waits for cooldown (3 seconds between teleports per mob)
   - Finds a safe position inside the region
   - Teleports the mob back
5. Maximum of **5 teleports per tick** to avoid lag
6. Every 10 minutes, cleans up cooldown records for inactive mobs

### How the Safe Position is Found

When teleporting a mob, the system follows this priority order:

**Priority 1 — Mob's original SpawnPoint:**
- If the mob was spawned from a SpawnPoint, it returns to that exact position
- The `spawnPointId` is saved in the `RegionTrackerComponent` at spawn time and persists across server restarts
- The lookup is done by unique UUID — no ambiguity even if multiple SpawnPoints share the same name
- If the SpawnPoint was deleted since the mob spawned, falls through to Phase 1

**Phase 1 — 12 random attempts (fallback):**
- Used when the mob has no origin SpawnPoint
- Picks random positions distributed across the boundary (with 2-block internal margin)
- Scans downward for solid ground
- Checks for 2 empty blocks above the ground

**Phase 2 — Fixed anchor fallback:**
- If no random attempt works, tries: center + 4 corners of the region
- If no anchor works, the teleport is cancelled and logged in debug

### Combat Protection

**Important:** The containment system has smart combat detection!

**How it works:**
- Mobs **in combat** (with `LockedTarget`) can move freely through the containment margin
- Only teleports when the mob reaches or exceeds the **last chunk of the margin** (distance ≥ `containmentMarginChunks`)
- Mobs **not in combat** are teleported as soon as they leave the region (any distance > 0 chunks)

**Example with `containmentMarginChunks = 3`:**
```
Region: [====REGION====]
Margin: [1][2][3][====REGION====][3][2][1]

Mob NOT in combat:
  → Teleports immediately upon leaving the region

Mob IN combat:
  → Can fight in margin chunks 1 and 2
  → Teleports only upon reaching margin chunk 3 (border)
```

**Benefits:**
- Players can chase mobs near borders without constant teleports
- Combat feels natural and fluid
- Mobs still cannot fully escape (safety limit applied)

### Performance Notes

- Checks only **border mobs** (via `ChunkIterator.scanRegions`)
- Maximum of **5 teleports per tick** (avoids lag)
- **3-second cooldown** per mob (avoids spam)
- Skips check if **no players** are in the world
- Automatic cooldown cleanup every **10 minutes**

### Troubleshooting

**Problem:** Mobs escape too far
```bash
# Solution: Reduce the margin
# Edit global_config.json:
"containmentMarginChunks": 1
```

**Problem:** Mobs teleport during combat
```bash
# This should NOT happen — it's a bug!
# Enable debug and report:
/rsmdebug containment true
```

**Problem:** Mobs are not being contained
```bash
# Check if containment is enabled:
/rsmglobal

# Enable via command:
/rsmcontainment TELEPORT

# Or edit global_config.json manually:
"containMobs": true,
"containmentMode": "TELEPORT"
```

**Problem:** Mob cannot find a safe position for teleport
```bash
# The system tries 12 random positions + 5 fixed anchors
# If all fail, the teleport is cancelled
# Enable debug to see logs:
/rsmdebug containment true
# Look for: "[SKIP] <mob_uuid> — sem posição segura" (or similar in debug output)
```
