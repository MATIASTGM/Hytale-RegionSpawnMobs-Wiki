## Time of Day System

Time is divided into 4 periods based on a 24-hour Hytale cycle:

| Period | Hours | Alias |
|---|---|---|
| `Dawn` | 0 – 4.8h | part of `Night` |
| `Midday` | 4.8 – 12h | part of `Day` |
| `Dusk` | 12 – 19.2h | part of `Day` |
| `Midnight` | 19.2 – 24h | part of `Night` |

**Accepted values in JSON:**
- Periods: `Dawn`, `Midday`, `Dusk`, `Midnight`
- Aliases: `Day` (Midday + Dusk), `Night` (Dawn + Midnight)
- Special: `ALL` (any time)
- Case-insensitive: `night`, `NIGHT`, `Night` all work the same

### Examples

```json
"timeOfDay": ["Night"]           // Dawn + Midnight
"timeOfDay": ["Day"]             // Midday + Dusk
"timeOfDay": ["Dawn", "Dusk"]    // transitions only
"timeOfDay": ["Midnight"]        // midnight only
"timeOfDay": ["ALL"]             // no restriction
```

### Troubleshooting

**Mobs not spawning at expected time:**
- Enable debug with `/rsmdebug spawn true` and look for `[TIME] Current:` in the logs to see the current period
- If `timeOfDay` is `null` or empty, the system allows spawning at any time
