## Settings and Troubleshooting

### Global Parameters

```bash
/rsmglobalmaxperchunk <number>   # Max mobs per chunk (default: 3, 0 = unlimited)
/rsmglobalspawnrange <chunks>    # Player distance to activate regions (default: 2 chunks)
/rsmcontainment <mode>           # Containment mode: OFF, RETURN, TELEPORT (default: TELEPORT)
```

> `/rsmsetinterval` and `/rsmsetquantity` exist but are **deprecated** — use presets to configure interval and quantity.

> Marker reconciliation settings live in the global menu and in `global_config.json`: `markerReconcileEnabled`, `markerReconcileIntervalSeconds` and `markerReconcileMaxRefsPerPass`.

---

### Debug Mode

The `/rsmdebug` command requires a type and a state:

```bash
/rsmdebug <type> true    # Enable
/rsmdebug <type> false   # Disable
```

**Available types:**

| Type | What it logs |
|---|---|
| `spawn` | Spawn attempts and results |
| `spawn_verbose` | Detailed spawn info (very spammy) |
| `containment` | Teleports and boundary checks |
| `containment_verbose` | Detailed containment info (very spammy) |
| `region` | Region management |
| `player` | Player detection |
| `tracker` | Mob tracking |
| `policy` | Spawn throttle and cooldown |
| `initialization` | Region initialization |
| `capacity` | Capacity calculations |
| `validation` | Position validation |
| `all` | All types above |

---

### Common Issues

**Mobs not spawning:**
1. Check if the region is active: `/rsminfo <name>`
2. Confirm mobs have been added to the region
3. Check if there are players in the world (spawn only occurs with players present)
4. Enable debug: `/rsmdebug spawn true`

**Mobs escaping the region:**
- Check if containment is active: `/rsmglobal`
- Adjust the margin: edit `containmentMarginChunks` in `global_config.json`
- See the containment guide for details

**Too many mobs in an area:**
- Reduce the chunk limit: `/rsmglobalmaxperchunk 3`
- Adjust the region's preset with a smaller interval/quantity
- Check whether `spawnPasses` is higher than intended for that region

**Marker mobs teleporting during combat:**
- Outside combat, the marker uses the configured leash.
- In combat, the marker should only teleport after the mob passes 48 blocks.
- If it teleports earlier, enable `/rsmdebug containment true` and check if the mob actually has `LockedTarget`.

**Tamed marker mob keeps returning to marker or disappears by time:**
- Starting in version 1.1.8, the marker removes its tracker when the NPC role changes after taming/retype.
- If it still happens, confirm the mob still has `MarkerTrackerComponent` and send the marker config plus logs.

**Region won't create:**
- Make sure `/pos1` and `/pos2` have been set
- Minimum size: 4x4x4 blocks
- Check if the name already exists: `/rsminfo <name>`
