## Weather System

### How Weather Works in Spawn Conditions

You can control when mobs appear based on the current Hytale weather. The `weather` field in `spawnConditions` accepts an array of weather names.

### Special Weather Values
- `ALL` - Spawn in any weather (ignores the weather condition)

Weather comparison is case-insensitive, but using the exact asset names keeps configs easier to maintain.

### Examples

#### Example 1: Spawn only during rain
```json
"skeleton": {
  "spawnWeight": 50,
  "spawnConditions": {
    "timeOfDay": ["ALL"],
    "weather": ["Zone1_Rain", "Zone1_Rain_Light"]
  }
}
```

#### Example 2: Spawn during storms
```json
"wolf_black": {
  "spawnWeight": 30,
  "spawnConditions": {
    "timeOfDay": ["Night"],
    "weather": ["Zone1_Storm", "Zone2_Thunder_Storm"]
  }
}
```

#### Example 3: Spawn in any weather
```json
"wolf_white": {
  "spawnWeight": 40,
  "spawnConditions": {
    "timeOfDay": ["ALL"],
    "weather": ["ALL"]
  }
}
```

#### Example 4: Multiple weather conditions
```json
"ice_creature": {
  "spawnWeight": 60,
  "spawnConditions": {
    "timeOfDay": ["ALL"],
    "weather": [
      "Zone3_Snow",
      "Zone3_Snow_Heavy",
      "Zone3_Snow_Storm",
      "Zone3_Northern_Lights"
    ]
  }
}
```

### Available Weather Types

**Note:** Weather names are case-sensitive and must match exactly!

#### Global/Special
- `Blood_Moon`
- `Creative_Hub`
- `Forgotten_Temple`
- `Void`

#### Zone 1 (Plains/Forest)
- `Zone1_Sunny`
- `Zone1_Sunny_Fireflies`
- `Zone1_Cloudy_Medium`
- `Zone1_Foggy_Light`
- `Zone1_Rain`
- `Zone1_Rain_Light`
- `Zone1_Storm`
- `Zone1_Autumn_Forest_Windy`
- `Zone1_Oak_Forest_Windy`
- `Zone1_Azurewood_Fireflies`
- `Zone1_Swamp`
- `Zone1_Swamp_Foggy`
- `Zone1_Swamp_Rain`
- `Zone1_Gully`

#### Zone 2 (Desert)
- `Zone2_Sunny`
- `Zone2_Blazing_Light`
- `Zone2_Cloudy_Light`
- `Zone2_Cloudy_Heavy`
- `Zone2_Desert_Haze`
- `Zone2_Sand_Storm`
- `Zone2_Thunder_Storm`
- `Zone2_Corrupted_Oasis`
- `Zone2_Corrupted_Oasis_Night`
- `Zone2_Story_Dungeon_Exterior`
- `Zone2_Story_Dungeon_Interior1`
- `Zone2_Story_Dungeon_Interior_Corrupted`

#### Zone 3 (Taiga/Snow)
- `Zone3_Cloudy_Light`
- `Zone3_Cloudy_Medium`
- `Zone3_Cloudy_Boreal`
- `Zone3_Rain`
- `Zone3_Snow`
- `Zone3_Snow_Heavy`
- `Zone3_Snow_Storm`
- `Zone3_Northern_Lights`
- `Zone3_Hedera`
- `Hedera_Forest`
- `Zone3_Cave_Shallow`
- `Zone3_Cave_Deep`
- `Zone3_Cave_Volcanic`

#### Zone 4 (Volcanic/Void)
- `Zone4_Light`
- `Zone4_Storm`
- `Zone4_Spooky`
- `Zone4_AshWastes`
- `Zone4_AshWastes_Storm`
- `Zone4_Wastes`
- `Zone4_Wastes_Rain`
- `Zone4_Wastes_Rain_Heavy`
- `Zone4_Lava_Fields`
- `Zone4_Geyser`
- `Zone4_GhostForest`
- `Zone4_GhostForest_Rain`
- `Zone4_Forest_Burnt`
- `Zone4_Forest_Root`
- `Zone4_Ruined_City`
- `Zone4_Swamp_Gloomy`
- `Zone4_Swamp_Storm`
- `Zone4_Underground_Jungle`
- `Zone4_Underground_Jungle_Pink`
- `Zone4_Underground_Jungle_Red`
- `Zone4_Caves_Jungles`

#### Cave
- `Cave_Shallow`
- `Cave_Deep`
- `Cave_Fog`
- `Cave_Goblin`
- `Cave_Volcanic`

#### Skylands
- `Skylands_Sunny`
- `Skylands_Light`
- `Skylands_Mushroom_Forest_Cloudy`
- `Skylands_Mystic_Forest_Cloudy`
- `Skylands_Rapid_Marsh_Cloudy_Medium`
- `Skylands_Rapids_Marsh_Cloudy_Medium`
- `Skylands_Rapid_Marsh_Stormy`

#### Poisonlands
- `Poisonlands_Heavy`
- `Poisonlands_PurpleTwirl`
- `Poisonlands_Red`
- `Poison_Test`

#### Dungeon
- `Dungeon_Cursed_Crypt`
- `Dungeon_Cursed_Crypt_Graveyard`

#### Portal
- `Portals_Void_Event_Light`
- `Portals_Void_Event_Intense`

#### Minigame
- `Terror_Weather`
- `Default_Flat`
- `Default_Void`

### Tips

1. **Use `ALL` for simplicity**: If you don't care about weather, use `["ALL"]`
2. **Match your zone**: Use Zone1 weather for plains, Zone2 for desert, etc.
3. **Test in-game**: Weather names must match exactly (case-sensitive)
4. **Combine conditions**: Mix weather + time for specific spawns (e.g. skeletons during night storms)
5. **Multiple weathers**: You can list multiple weather types in the arrays
