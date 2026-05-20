## Visualization System

### Overview
RegionSpawnMobs offers two visualization tools to help you understand your regions and the chunk-based spawn system.

### `/rsmshow` - Region Boundaries

**Purpose:** Visualize ALL spawn regions with colored 3D boxes

**Commands:**
```bash
/rsmshow true   # Show all regions
/rsmshow false  # Hide all regions
```

**Features:**
- Shows **all regions** at once with colored bounding boxes
- Each region gets a **unique color** for easy identification
- Visualization lasts **10 minutes** and is then removed automatically
- Helps identify **overlapping regions**
- Useful for **planning new regions**

**Use Cases:**
- Check if regions overlap
- Confirm region size and position
- Plan placement of new regions
- Debug region boundaries
- Show regions to other admins

**Example Visual:**
```
[IMAGE: rsmshow_example.png]
(Screenshot showing multiple colored region boxes)
```

**What You See:**
- Colored wireframe boxes around each region
- Different colors for each region
- Clear boundaries of spawn areas

---

### `/rsmshowchunks` - Chunk Grid

**Purpose:** Visualize the chunk grid system used for spawning

**Commands:**
```bash
/rsmshowchunks true   # Show chunk grid
/rsmshowchunks false  # Hide chunk grid
```

**Features:**
- Shows **chunk boundaries** as a grid overlay
- Each chunk is **32x32 blocks** with a height of **320 blocks** (Hytale standard)
- Helps understand the **chunk-based spawn system**
- Visualizes **maxPerChunk** limits
- Shows where **chunk density** is calculated
- Useful for **optimizing spawn distribution**

**Use Cases:**
- Understand chunk-based spawning
- Optimize `maxPerChunk` settings
- See why mobs aren't spawning (chunk full)
- Plan region size in chunks
- Debug chunk density issues

**Example Visual:**
```
[IMAGE: rsmshowchunks_example.png]
(Screenshot showing chunk grid overlay)
```

**What You See:**
- Colored wireframe boxes aligned to chunk boundaries (32x32 blocks)
- Box height covers the entire world (320 blocks)
- Colors match the corresponding regions
- Helps count chunks in your region

---

### Comparison: `/rsmshow` vs `/rsmshowchunks`

| Feature | `/rsmshow` | `/rsmshowchunks` |
|---------|------------|------------------|
| **Shows** | Region boundaries | Chunk grid |
| **Purpose** | See spawn regions | Understand chunk system |
| **Duration** | 10 minutes | 10 minutes |
| **Use for** | Region planning | Spawn optimization |
| **Displays** | Colored boxes | Grid lines |
| **Best for** | Admins setting up regions | Debugging spawn issues |

### Combined Usage

**Recommended workflow:**
```bash
# 1. Show regions to see boundaries
/rsmshow true

# 2. Show chunks to understand density
/rsmshowchunks true

# 3. Analyze your region
# - Count how many chunks are in the region
# - Calculate capacity: chunks × maxPerChunk
# - Adjust settings if needed

# 4. Hide visualizations
/rsmshow false
/rsmshowchunks false
```

### Practical Examples

#### Example 1: Planning a New Region
```bash
# Step 1: Show existing regions
/rsmshow true

# Step 2: Find empty space
# (Look for areas without colored boxes)

# Step 3: Select new area
/pos1
/pos2

# Step 4: Create region
/rsmcreate new_arena
```

#### Example 2: Optimizing Chunk Density
```bash
# Step 1: Show chunks
/rsmshowchunks true

# Step 2: Count chunks in your region
# Example: 10 chunks wide × 10 chunks deep = 100 chunks

# Step 3: Calculate capacity
# 100 chunks × 3 maxPerChunk = 300 mobs maximum

# Step 4: Adjust if needed
/rsmglobalmaxperchunk 5  # Increase to 500 mobs maximum
```

#### Example 3: Debugging Spawn Issues
```bash
# Problem: Mobs are not spawning

# Step 1: Show region
/rsmshow true
# Check: Is the region where you think it is?

# Step 2: Show chunks
/rsmshowchunks true
# Check: Are chunks full? (maxPerChunk reached)

# Step 3: Check info
/rsminfo <region_name>
# Check: Current mobs vs capacity

# Step 4: Enable debug
/rsmdebug spawn true
# Check: Spawn attempt logs
```
