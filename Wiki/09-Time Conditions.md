## Time of Day System

Time conditions define when a mob is allowed to spawn. They are configured in each entity's `spawnConditions.timeOfDay`.

### Current Periods

| Period | Approx. hours | Notes |
|---|---|---|
| `Dawn` | 05-07 | start of day |
| `Morning` | 07-12 | day |
| `Afternoon` | 12-17 | day |
| `Dusk` | 17-19 | end of day |
| `Night` | 19-05 | full night period |

### Accepted Values

- Specific periods: `Dawn`, `Morning`, `Afternoon`, `Dusk`, `Night`
- Alias: `Day` (`Dawn` + `Morning` + `Afternoon` + `Dusk`)
- Special: `ALL` (any time)
- Case-insensitive: `night`, `NIGHT`, `Night` all work the same

### Examples

```json
"timeOfDay": ["Night"]              // full night
"timeOfDay": ["Day"]                // Dawn + Morning + Afternoon + Dusk
"timeOfDay": ["Dawn", "Dusk"]       // transitions only
"timeOfDay": ["Morning"]            // morning only
"timeOfDay": ["ALL"]                // no restriction
```

### Reconcile by Time

If dynamic condition reconciliation is enabled, tracked region mobs can be removed when their time/weather conditions no longer match. This keeps time-based regions from leaving old mobs behind after the period changes.

Starting in version 1.1.8, marker mobs that are tamed or retyped release the marker tracker before marker reconciliation can count, teleport or remove them.

### Troubleshooting

**Mobs not spawning at expected time:**
- Enable debug with `/rsmdebug spawn true` and look for `[TIME] Current:` in the logs to see the current period
- If `timeOfDay` is `null` or empty, the system allows spawning at any time
- Confirm the value is current: `Midday` and `Midnight` were old names and should not be used in new configs
