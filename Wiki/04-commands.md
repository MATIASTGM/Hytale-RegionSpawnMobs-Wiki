## Commands

### Basic Management
- `/rsmcreate <region>` – Create a spawn region from pos1/pos2 selection (at root)
- `/rsmcreate <folder> <region>` – Create a region inside a folder
- `/rsmcreate <folder> <category> <region>` – Create a region inside a nested folder
- `/rsmdelete <region>` – Delete a region (accepts full path: `folder/name`)
- `/rsminfo <region>` – Display region information in chat
- `/rsmkill [region]` – Remove all mobs from the region (or all regions if not specified), including orphaned mobs

### Mob Configuration
- `/rsmaddmob <region> <mob> <weight>` – Add a mob type to the region (default conditions: ALL)
- `/rsmremovemob <region> <mob>` – Remove a mob type from the region

> `maxAlive`, weather, time and terrain type can also be configured through `/rsmeditmenu`.

### SpawnPoints
- `/rsmaddspawnpoint [name]` – Create a SpawnPoint at your current position, inside the region you are standing in. Name is optional

> `maxSpawn`, entity filters and SpawnPoint rename are configured through `/rsmeditmenu`.

### Menus (GUI)
- `/rsminfomenu [region]` – Open region info screen in the menu. If name is omitted, detects region by player position
- `/rsmeditmenu [region]` – Open region edit screen in the menu. If name is omitted, detects region by player position
- `/rsmpresetinfomenu` – Open preset info screen in the menu
- `/rsmpresetmenu` – Open preset edit screen in the menu
- `/rsmglobalmenu` – Open global config menu

### Presets
- `/rsmpresetlist` – List all spawn presets
- `/rsmpresetinfo <name>` – Display preset details in chat
- `/rsmpresetcreate <name> <interval> <minQty> <maxQty>` – Create a new preset
- `/rsmpresetdelete <name>` – Delete a preset
- `/rsmpresetinterval <name> <seconds>` – Change preset spawn interval
- `/rsmpresetminqty <name> <quantity>` – Change preset minimum quantity
- `/rsmpresetmaxqty <name> <quantity>` – Change preset maximum quantity

### Visualization
- `/rsmshow true` – Show all regions with colored boxes
- `/rsmshow false` – Hide all visualizations
- `/rsmshowchunks true` – Show regions aligned to chunk boundaries
- `/rsmshowchunks false` – Hide chunk visualization
- `/rsmselect <region>` – Load region into BuilderTools selection (for adjustment with pos1/pos2)

### Relocation
- `/rsmrelocate <region>` – Move region to current pos1/pos2 selection

### Global Settings
- `/rsmglobal` – Display current global settings in chat
- `/rsmglobalmaxperchunk <number>` – Set global max mobs per chunk (0 = unlimited)
- `/rsmglobalspawnrange <chunks>` – Set player distance to activate regions (in chunks)
- `/rsmcontainment <mode>` – Set containment mode: `OFF`, `RETURN` or `TELEPORT`

### Other
- `/rsmreload` – Reload regions, global config and presets from disk
- `/rsmdebug <type> <true/false>` – Enable or disable a debug type

**Available debug types:**
`spawn`, `spawn_verbose`, `containment`, `containment_verbose`, `region`, `player`, `tracker`, `policy`, `initialization`, `capacity`, `validation`, `all`

---

### Deprecated Commands
The commands below still exist but return a warning message. Use presets or the menu instead:

- `/rsmenable <name>` – ⚠️ Deprecated. Use the region edit menu
- `/rsmdisable <name>` – ⚠️ Deprecated. Use the region edit menu
- `/rsmsetinterval <name> <seconds>` – ⚠️ Deprecated. Use presets
- `/rsmsetquantity <name> <min> <max>` – ⚠️ Deprecated. Use presets

---

### Commands Removed in 1.1.2
The following commands were removed along with the sub-region system:

- `/rsmcreatesub` – Removed. Use `/rsmcreate <folder> <region>` to organize regions
- `/rsmdeletesub` – Removed. Use `/rsmdelete <path/name>`
- `/rsminfosub` – Removed. Use `/rsminfo <path/name>`
- `/rsmaddmobsub` – Removed. Use `/rsmaddmob <path/name> <mob> <weight>`
- `/rsmsubinfomenu` – Removed. Use `/rsminfomenu <path/name>`
- `/rsmsubeditmenu` – Removed. Use `/rsmeditmenu <path/name>`
- `/rsmrelocatesub` – Removed. Use `/rsmrelocate <path/name>`
