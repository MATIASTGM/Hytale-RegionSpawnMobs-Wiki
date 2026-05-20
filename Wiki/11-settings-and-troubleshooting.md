## Settings and Troubleshooting

### Global Parameters

```bash
/rsmglobalmaxperchunk <number>   # Max mobs per chunk (default: 3, 0 = unlimited)
/rsmglobalspawnrange <chunks>    # Player distance to activate regions (default: 2 chunks)
/rsmcontainment <mode>           # Containment mode: OFF, RETURN, TELEPORT (default: TELEPORT)
```

> `/rsmsetinterval` and `/rsmsetquantity` exist but are **deprecated** — use presets to configure interval and quantity.

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

**Region won't create:**
- Make sure `/pos1` and `/pos2` have been set
- Minimum size: 4x4x4 blocks
- Check if the name already exists: `/rsminfo <name>`
