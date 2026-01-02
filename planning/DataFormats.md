# **DATA FORMATS**

**Godot 4.x | Godot Resources (.tres) | Visual Map Editing**

---

## **DESIGN PHILOSOPHY**

### **1. Rapid Iteration**
- **Minimal required fields**: Most properties have sensible defaults
- **Copy-paste friendly**: Duplicate files and change only what's different
- **No code changes**: Add new units/maps without touching GDScript
- **Instant testing**: Reload scene in Godot editor, no recompilation needed

### **2. Visual Map Editing**
- Use Godot's native **TileMapLayer** system with built-in tile painting
- Paint maps directly in Godot editor
- Visual feedback as you design
- Save/load maps as `.tscn` scenes with TileMapLayer data

### **3. Godot Resources (.tres)**
- **Editor-friendly**: Auto-completion, type checking, inspector editing
- **Text-based**: Easy to version control and copy-paste
- **Inspector-driven**: Edit properties in Godot's visual editor

### **4. Granular Unit Files**
- Each unit type gets its own `.tres` file
- No variations within a single file
- Easy to browse and modify individual units

---

## **UNIT DATA FORMAT**

### **Resource Class Definition**

```gdscript
# scripts/resources/unit_data.gd
extends Resource
class_name UnitData

@export var display_name: String = "Unit"
@export var description: String = ""

# Core stats
@export var move: int = 1
@export var attack: int = 4
@export var defense: int = 6
@export var range: int = 2

# Special abilities (all default to false)
@export var is_heavy: bool = false              # Must deploy/pack
@export var is_relay: bool = false               # Acts as relay
@export var is_always_active: bool = false       # Cars, armored cars
@export var has_cavalry_charge: bool = false     # +3 attack when adjacent
@export var ignores_terrain: bool = false       # Balloon sees over obstacles

# Transport capacity
@export var can_be_carried: bool = false
@export var can_carry_units: bool = false
@export var capacity: int = 0                    # Number of units can carry

# Railway properties
@export var railway_locked: bool = false         # Must follow tracks
@export var is_train_engine: bool = false        # Leads train cars
@export var is_train_car: bool = false           # Attached to engine

# Visual properties
@export var marker_shape: String = "circle"      # circle, square, triangle, hexagon
@export var marker_size: int = 1                 # 1-3 scale multiplier
@export var show_direction: bool = true         # Show forward-facing indicator

# Colors for each army
@export var team_color_red: Color = Color(0.9, 0.2, 0.2)
@export var team_color_white: Color = Color(0.95, 0.95, 0.95)
```

---

### **Unit Files** (`resources/units/`)

Each unit type gets its own `.tres` file:

#### **infantry.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://b0k1m2n3o4p5"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Infantry"
description = "Standard infantry unit."
move = 1
attack = 4
defense = 6
range = 2
marker_shape = "circle"
marker_size = 1
```

#### **cavalry.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://c1k2m3n4o5p6"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Cavalry"
description = "Fast cavalry with charge bonus."
move = 2
attack = 4
defense = 5
range = 2
has_cavalry_charge = true
marker_shape = "triangle"
marker_size = 1
```

#### **artillery.tres** (Field Gun)
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://d2k3m4n5o6p7"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Artillery"
description = "Field gun with long range."
move = 1
attack = 5
defense = 8
range = 3
marker_shape = "square"
marker_size = 2
```

#### **swift_artillery.tres** (Tachanka)
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://e3k4m5n6o7p8"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Swift Artillery"
description = "Mounted machine gun (tachanka)."
move = 2
attack = 5
defense = 8
range = 3
marker_shape = "square"
marker_size = 1
```

#### **relay.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://f4k5m6n7o8p9"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Relay"
description = "Communication relay unit."
move = 1
attack = 0
defense = 1
range = 0
is_relay = true
marker_shape = "hexagon"
marker_size = 1
```

#### **swift_relay.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://g5k6m7n8o9p0"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Swift Relay"
description = "Mobile communication relay."
move = 2
attack = 0
defense = 1
range = 0
is_relay = true
marker_shape = "hexagon"
marker_size = 1
```

#### **machine_gun_nest.tres** (Maxim)
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://h6k7m8n9o0p1"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Machine Gun Nest"
description = "Heavy Maxim machine gun. Must deploy."
move = 1
attack = 5
defense = 10
range = 2
is_heavy = true
marker_shape = "square"
marker_size = 2
```

#### **observation_balloon.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://i7k8m9n0o1p2"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Observation Balloon"
description = "Balloon for scouting over obstacles."
move = 1
attack = 0
defense = 0
range = 0
is_heavy = true
is_relay = true
ignores_terrain = true
marker_shape = "circle"
marker_size = 3
```

#### **armored_tram.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://j8k9m0n1o2p3"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Armored Tram"
description = "Armored tram on tram lines."
move = 4
attack = 4
defense = 6
range = 2
railway_locked = true
can_carry_units = true
capacity = 2
marker_shape = "rectangle"
marker_size = 2
```

#### **armored_train_engine.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://k9l0m1n2o3p4"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Armored Train Engine"
description = "Locomotive for armored train."
move = 4
attack = 0
defense = 0
range = 0
railway_locked = true
is_train_engine = true
marker_shape = "rectangle"
marker_size = 3
```

#### **armored_train_gun_car.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://l0m1n2o3p4q5"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Armored Train Gun Car"
description = "Artillery car attached to train engine."
move = 0
attack = 6
defense = 8
range = 3
railway_locked = true
is_train_car = true
marker_shape = "rectangle"
marker_size = 2
```

#### **armored_train_troop_car.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://m1n2o3p4q5r6"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Armored Train Troop Car"
description = "Troop transport attached to train engine."
move = 0
attack = 0
defense = 0
range = 0
railway_locked = true
is_train_car = true
can_carry_units = true
capacity = 2
marker_shape = "rectangle"
marker_size = 2
```

#### **car.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://n2o3p4q5r6s7"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Car"
description = "Civilian vehicle."
move = 2
attack = 4
defense = 3
range = 2
is_always_active = true
marker_shape = "rectangle"
marker_size = 1
```

#### **armored_car.tres**
```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format=3 uid="uid://o3p4q5r6s7t8"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Armored Car"
description = "Military armored vehicle."
move = 2
attack = 6
defense = 5
range = 2
is_always_active = true
marker_shape = "rectangle"
marker_size = 2
```

---

### **Quick Unit Variation Workflow**

```bash
# Want to add "Heavy Cavalry"?
# 1. Copy cavalry.tres → heavy_cavalry.tres
# 2. Edit 2 properties:
display_name = "Heavy Cavalry"
defense = 7  # Up from 5
# 3. Done! No code changes needed.
```

---

## **TILE DATA FORMAT**

### **Resource Class Definition**

```gdscript
# scripts/resources/tile_data.gd
extends Resource
class_name TileData

@export var display_name: String = "Tile"
@export var description: String = ""

# Movement properties
@export var is_passable: bool = true

# Line of effect (LoE) - determines if units can "see" through
@export var blocks_line_of_effect: bool = false

# Defense bonus for occupying unit
@export var defense_bonus: int = 0

# Infrastructure
@export var has_rail: bool = false
@export var rail_type: String = ""              # "train", "tram", ""
@export var has_road: bool = false
@export var road_bonus: int = 0                 # Extra movement on road
@export var has_telegraph: bool = false

# Special tiles
@export var is_station: bool = false            # Source for network
@export var is_telegraph_office: bool = false   # Relay node
@export var is_turntable: bool = false          # Train can reverse direction
@export var is_intersection: bool = false      # Train can turn (not reverse)

# Visual properties
@export var base_color: Color = Color.WHITE
@export var icon_code: String = ""              # "" or "mountain", "building", "tracks", etc.
@export var has_border: bool = false
@export var border_color: Color = Color.BLACK
```

---

### **Tile Files** (`resources/tiles/`)

#### **grass.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_grass"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Grass"
is_passable = true
blocks_line_of_effect = false
base_color = Color(0.3, 0.7, 0.3)
```

#### **snow.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_snow"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Snow"
is_passable = true
blocks_line_of_effect = false
base_color = Color(0.95, 0.95, 0.98)
```

#### **mountain.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_mountain"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Mountain"
is_passable = false
blocks_line_of_effect = true
base_color = Color(0.5, 0.5, 0.5)
icon_code = "mountain"
```

#### **city_block.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_city_block"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "City Block"
is_passable = false
blocks_line_of_effect = true
base_color = Color(0.3, 0.3, 0.3)
icon_code = "building"
```

#### **road.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_road"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Road"
is_passable = true
has_road = true
road_bonus = 1
base_color = Color(0.6, 0.5, 0.4)
```

#### **rail_train_ns.tres** (North-South rail)
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_rail_ns"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Rail (N-S)"
is_passable = true
has_rail = true
rail_type = "train"
base_color = Color(0.2, 0.2, 0.2)
icon_code = "tracks_ns"
```

#### **rail_tram_ew.tres** (East-West tram)
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_tram_ew"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Tram (E-W)"
is_passable = true
has_rail = true
rail_type = "tram"
base_color = Color(0.3, 0.3, 0.3)
icon_code = "tracks_ew"
```

#### **station.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_station"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Railway Station"
is_passable = true
is_station = true
defense_bonus = 2
base_color = Color(0.7, 0.7, 0.8)
icon_code = "station"
```

#### **telegraph_office.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format=3 uid="uid://tile_telegraph"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Telegraph Office"
is_passable = true
is_telegraph_office = true
base_color = Color(0.8, 0.8, 0.6)
icon_code = "telegraph"
```

#### **plaza.tres** (City Plaza - Defensible)
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format="3" uid="uid://tile_plaza"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "City Plaza"
is_passable = true
defense_bonus = 4
base_color = Color(0.6, 0.7, 0.8)
has_border = true
border_color = Color(0.4, 0.5, 0.6)
```

#### **turntable.tres**
```gdscript
[gd_resource type="Resource" script_class="TileData" load_steps=2 format="3" uid="uid://tile_turntable"]

[ext_resource type="Script" path="res://scripts/resources/tile_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "Turntable"
is_passable = true
has_rail = true
rail_type = "train"
is_turntable = true
base_color = Color(0.6, 0.6, 0.6)
icon_code = "turntable"
```

---

## **MAP FORMAT**

### **Visual Map Editing with Godot's TileMap**

Maps are created as **.tscn scenes** using Godot's **TileMapLayer** system:

```
scenes/maps/
├── map1_iron_horse.tscn
├── map2_red_petrograd.tscn
└── map3_woods_bryansk.tscn
```

### **Map Scene Structure**

Each map scene contains:

```
map1_iron_horse.tscn (Node2D)
├── MapData (Resource)               # Map metadata
├── TileMapLayer                     # Main tile grid
│   ├── BaseTerrain                  # Grass, snow
│   ├── Obstacles                    # Mountains, city blocks
│   ├── Infrastructure               # Rails, roads
│   ├── Buildings                    # Stations, telegraph offices
│   └── Special                      # Turntables, intersections
└── StartingUnits (Node2D)           # Initial unit placements
    ├── RedArmy (Node2D)
    │   ├── Infantry (Vector2i position)
    │   └── ArmoredTrainEngine (Vector2i position)
    └── WhiteArmy (Node2D)
        └── Infantry (Vector2i position)
```

### **Map Metadata Resource** (`resources/maps/`)

#### **map1_iron_horse_data.tres**
```gdscript
[gd_resource type="Resource" script_class="MapData" load_steps=2 format=3 uid="uid://map1_data"]

[ext_resource type="Script" path="res://scripts/resources/map_data.gd" id="1"]

[resource]
script = ExtResource("1")
map_name = "The Iron Horse"
map_id = "map1_iron_horse"
description = "The Whites control the central rail hub. Capture it to establish supply lines."
historical_context = "In early 1919, the Red Army launched an offensive to secure key railway junctions in western Russia. The White Army's armored trains dominated the flatlands, but the Reds utilized sabotage tactics to disrupt supply lines."

victory_condition = "capture_station"
victory_target = Vector2i(12, 10)

# Starting setup
red_army_setup = {
    "infantry": 3,
    "artillery": 1,
    "armored_train": true
}

white_army_setup = {
    "infantry": 3,
    "swift_artillery": 1,
    "armored_train": true
}
```

### **MapData Resource Class**

```gdscript
# scripts/resources/map_data.gd
extends Resource
class_name MapData

@export var map_name: String = "New Map"
@export var map_id: String = "new_map"
@export var description: String = ""
@export var historical_context: String = ""

# Victory conditions
@export var victory_condition: String = "annihilation"  # "annihilation", "capture_station", "destroy_balloon"
@export var victory_target: Vector2i = Vector2i(-1, -1)

# Army setups (for auto-generation or validation)
@export var red_army_setup: Dictionary = {}
@export var white_army_setup: Dictionary = {}
```

---

## **VISUAL MAP EDITING WORKFLOW**

### **Step 1: Create New Map**

```bash
# In Godot Editor:
# 1. Create new scene: map_new.tscn
# 2. Add root Node2D named "MapRoot"
# 3. Add TileMapLayer child
# 4. Add MapData resource (from template)
```

### **Step 2: Paint Tiles**

1. **Select TileMapLayer**
2. **Open TileSet** (Create new or reuse existing)
3. **Paint tiles** using Godot's built-in painting tools:
   - Select tile from TileSet palette
   - Click and drag on map
   - Visual feedback as you paint

### **Step 3: Place Units**

```bash
# Add StartingUnits node (Node2D)
# Add RedArmy node (Node2D)
# Add unit marker nodes as children:
#   - Infantry (Node2D with "position" Vector2i property)
#   - ArmoredTrainEngine (Node2D with "position" Vector2i property)
```

### **Step 4: Set Map Metadata**

```bash
# Select MapData resource in inspector
# Fill in:
#   - map_name
#   - description
#   - victory_condition
#   - historical_context
```

### **Step 5: Save and Test**

```bash
# Save scene (Ctrl+S)
# Click "Play" on map scene
# Test unit placement, tile interactions
```

---

## **TILESET ORGANIZATION**

### **TileSet Resource** (`resources/tilesets/game_tileset.tres`)

```
TileSet (game_tileset.tres)
├── Layer: BaseTerrain
│   ├── Atlas 1x1: grass
│   ├── Atlas 1x1: snow
│   └── Atlas 1x1: water
│
├── Layer: Obstacles
│   ├── Atlas 1x1: mountain
│   └── Atlas 1x1: city_block
│
├── Layer: Infrastructure
│   ├── Atlas 1x1: rail_ns (north-south)
│   ├── Atlas 1x1: rail_ew (east-west)
│   ├── Atlas 1x1: rail_ne (northeast)
│   ├── Atlas 1x1: tram_ns
│   ├── Atlas 1x1: tram_ew
│   ├── Atlas 1x1: road_ns
│   ├── Atlas 1x1: road_ew
│   └── Atlas 1x1: telegraph_line
│
├── Layer: Buildings
│   ├── Atlas 1x1: station
│   ├── Atlas 1x1: telegraph_office
│   └── Atlas 1x1: plaza
│
└── Layer: Special
    ├── Atlas 1x1: turntable
    ├── Atlas 1x1: intersection
    └── Atlas 1x1: rail_crossing
```

### **Creating Tile Icons**

Use Godot's built-in tile painting to create simple icons:
- **Grass**: Green square
- **Mountain**: Gray triangle
- **Rail**: Black parallel lines
- **Station**: Building silhouette
- **Telegraph**: Small dots

All icons are code-generated (no external assets needed).

---

## **SAVE GAME FORMAT**

Save games use simple JSON for easy inspection and editing:

```json
{
  "version": "1.0",
  "map_id": "map1_iron_horse",
  "turn": 15,
  "current_player": 0,
  "game_state": "planning_a",
  
  "units": [
    {
      "unit_type": "infantry",
      "owner_id": 0,
      "position": [7, 8],
      "is_deployed": true,
      "health": 100,
      "carried_unit": null
    },
    {
      "unit_type": "armored_train_engine",
      "owner_id": 0,
      "position": [5, 1],
      "is_deployed": true,
      "health": 100,
      "attached_cars": [
        {
          "unit_type": "armored_train_gun_car",
          "owner_id": 0,
          "position": [5, 2],
          "health": 80
        }
      ]
    }
  ],
  
  "tile_states": {
    "5,5": {
      "type": "station",
      "owner_id": 0,
      "is_destroyed": false
    },
    "8,3": {
      "type": "rail_ns",
      "owner_id": 1,
      "is_destroyed": true
    }
  },
  
  "current_orders": {
    "player_0": {
      "movements": [
        {
          "unit_id": 1,
          "from": [7, 8],
          "to": [8, 8]
        }
      ],
      "attacks": [
        {
          "attacker_id": 2,
          "target_pos": [12, 10]
        }
      ]
    }
  },
  
  "campaign_data": {
    "completed_levels": ["map1_iron_horse"],
    "current_level": "map2_red_petrograd",
    "victories": 1,
    "defeats": 0
  }
}
```

---

## **DATA VALIDATION**

### **Load-Time Validation**

```gdscript
# In DataManager.gd
func validate_unit_data(data: UnitData) -> bool:
    if data.move < 0 or data.move > 10:
        push_error("Unit %s has invalid move value: %d" % [data.display_name, data.move])
        return false
    
    if data.attack < 0 or data.attack > 10:
        push_error("Unit %s has invalid attack value: %d" % [data.display_name, data.attack])
        return false
    
    if data.is_train_engine and data.is_train_car:
        push_error("Unit %s cannot be both engine and car" % data.display_name)
        return false
    
    if data.railway_locked and data.rail_type == "":
        push_error("Unit %s is railway_locked but has no rail_type" % data.display_name)
        return false
    
    return true

func validate_map_data(data: MapData) -> bool:
    if data.map_id.is_empty():
        push_error("Map has empty map_id")
        return false
    
    if data.victory_target == Vector2i(-1, -1) and data.victory_condition != "annihilation":
        push_error("Map %s needs victory_target for condition: %s" % [data.map_name, data.victory_condition])
        return false
    
    return true
```

### **Inspector Validation**

Use `@export_range` and `@export_enum` for clamped values:

```gdscript
@export_range(0, 10) var attack: int = 4
@export_enum("circle", "square", "triangle", "hexagon", "rectangle") var marker_shape: String = "circle"
@export_enum("annihilation", "capture_station", "destroy_balloon") var victory_condition: String = "annihilation"
```

---

## **TEMPLATE FILES**

### **Unit Template** (`resources/units/_template_unit.tres`)

```gdscript
[gd_resource type="Resource" script_class="UnitData" load_steps=2 format="3 uid="uid://template_unit"]

[ext_resource type="Script" path="res://scripts/resources/unit_data.gd" id="1"]

[resource]
script = ExtResource("1")
display_name = "New Unit"
description = "Description of the unit."
move = 1
attack = 4
defense = 6
range = 2
marker_shape = "circle"
marker_size = 1
```

### **Map Data Template** (`resources/maps/_template_map_data.tres`)

```gdscript
[gd_resource type="Resource" script_class="MapData" load_steps=2 format="3 uid="uid://template_map"]

[ext_resource type="Script" path="res://scripts/resources/map_data.gd" id="1"]

[resource]
script = ExtResource("1")
map_name = "New Map"
map_id = "new_map"
description = "Map description."
historical_context = "Historical context here."
victory_condition = "annihilation"
victory_target = Vector2i(-1, -1)
```

---

## **QUICK REFERENCE: CREATING NEW DATA**

### **Add New Unit** (30 seconds)
```bash
1. Copy resources/units/_template_unit.tres → new_unit.tres
2. Open in Godot inspector
3. Edit display_name, stats, special abilities
4. Save
5. Use in map or campaign - no code changes needed!
```

### **Add New Map** (5 minutes)
```bash
1. Copy scenes/maps/_template_map.tscn → new_map.tscn
2. Open scene in Godot
3. Paint tiles using TileMapLayer
4. Add starting units
5. Edit MapData resource (name, description, victory condition)
6. Save
7. Test instantly - no code changes needed!
```

### **Modify Unit Stats** (10 seconds)
```bash
1. Open unit.tres in Godot inspector
2. Change values (attack, defense, etc.)
3. Save
4. Reload game - changes apply instantly!
```

---

## **DIRECTORY STRUCTURE**

```
resources/
├── units/
│   ├── _template_unit.tres
│   ├── infantry.tres
│   ├── cavalry.tres
│   ├── artillery.tres
│   ├── swift_artillery.tres
│   ├── relay.tres
│   ├── swift_relay.tres
│   ├── machine_gun_nest.tres
│   ├── observation_balloon.tres
│   ├── armored_tram.tres
│   ├── armored_train_engine.tres
│   ├── armored_train_gun_car.tres
│   ├── armored_train_troop_car.tres
│   ├── car.tres
│   └── armored_car.tres
│
├── tiles/
│   ├── base_tiles/
│   │   ├── _template_tile.tres
│   │   ├── grass.tres
│   │   └── snow.tres
│   ├── obstacles/
│   │   ├── _template_tile.tres
│   │   ├── mountain.tres
│   │   └── city_block.tres
│   ├── infrastructure/
│   │   ├── _template_tile.tres
│   │   ├── rail_ns.tres
│   │   ├── rail_ew.tres
│   │   ├── tram_ns.tres
│   │   ├── tram_ew.tres
│   │   ├── road_ns.tres
│   │   ├── road_ew.tres
│   │   └── telegraph_line.tres
│   ├── buildings/
│   │   ├── _template_tile.tres
│   │   ├── station.tres
│   │   ├── telegraph_office.tres
│   │   └── plaza.tres
│   └── special/
│       ├── _template_tile.tres
│       ├── turntable.tres
│       ├── intersection.tres
│       └── rail_crossing.tres
│
├── maps/
│   ├── _template_map_data.tres
│   ├── map1_iron_horse_data.tres
│   ├── map2_red_petrograd_data.tres
│   └── map3_woods_bryansk_data.tres
│
└── tilesets/
    └── game_tileset.tres

scenes/
└── maps/
    ├── _template_map.tscn
    ├── map1_iron_horse.tscn
    ├── map2_red_petrograd.tscn
    └── map3_woods_bryansk.tscn
```

---

## **BENEFITS OF THIS APPROACH**

### **1. Visual Map Editing**
- Paint tiles directly in Godot editor
- See results instantly
- No need to memorize tile codes
- Intuitive and fast

### **2. Rapid Unit Iteration**
- Copy-paste template
- Change 2-3 properties
- Test immediately
- No code compilation

### **3. Granular Control**
- Each unit is a separate file
- Easy to find and modify specific units
- Clear organization
- No nested variants to dig through

### **4. Version Control Friendly**
- All data in text-based .tres files
- Easy to diff and merge
- Clear change history
- No binary blobs

### **5. Inspector-Driven**
- Edit properties in visual inspector
- Type checking and validation
- Auto-completion
- No manual JSON editing

### **6. Instant Feedback**
- Reload scene → changes apply
- No restart needed
- Test balance on the fly
- Iterate quickly

---

**Last Updated**: January 2, 2026
**Version**: 1.0
