# **TECHNICAL SPECIFICATIONS**

**Godot 4.x | Core Systems | Algorithms | Integration Points**

---

## **DESIGN PHILOSOPHY**

### **1. Separation of Concerns**
Each system has a single, well-defined responsibility
- No system does more than one thing
- Clear interfaces between systems
- Easy to test in isolation

### **2. Data-Driven Design**
All game logic is driven by data, not hardcoded
- Unit behavior defined in UnitData resources
- Tile behavior defined in TileData resources
- System behavior configurable via settings

### **3. Event-Driven Architecture**
Systems communicate via signals, not direct method calls
- Loose coupling between components
- Easy to add/remove systems
- Clean dependency management

### **4. Performance First**
Optimize for smooth gameplay on target hardware
- Efficient algorithms (O(n) or better)
- Minimal memory allocations during gameplay
- Cache frequently accessed data

---

## **SYSTEM ARCHITECTURE**

```
TurnStateMachine
    ├── OrderManager
    ├── MovementSystem
    │   └── Pathfinding
    ├── CollisionSystem
    ├── BattleSystem
    ├── NetworkSystem
    │   └── TerrainSystem
    └── RailwaySystem
```

### **Data Flow**

```
Player Input → GameBoard → OrderManager → TurnStateMachine
                                           │
                                           ├── MovementSystem → Pathfinding
                                           ├── CollisionSystem
                                           ├── BattleSystem
                                           ├── NetworkSystem → TerrainSystem
                                           └── RailwaySystem
```

---

## **1. TURN STATE MACHINE**

**File**: `scripts/game/turn_state_machine.gd`

### **Responsibilities**
- Manage turn phases (Planning, Resolution)
- Coordinate system execution order
- Handle game mode differences (Hot Seat, AI, Online)
- Emit state change events

### **Core Data Structures**

```gdscript
enum TurnPhase {
    QUEUE_ORDERS_A,
    QUEUE_ORDERS_B,
    IRON_CURTAIN,
    REVEAL_ORDERS,
    CALCULATE_MOVEMENT,
    CALCULATE_BATTLES,
    PLAY_BATTLE_ANIMS,
    RESOLVE_BATTLES,
    CLEANUP,
    CHECK_WIN_CONDITION,
    RESET_TURN
}

enum GameMode {
    HOT_SEAT,
    VS_AI,
    ONLINE
}

# State
var current_phase: TurnPhase = TurnPhase.QUEUE_ORDERS_A
var current_player: int = 0  # 0 = Red, 1 = White
var turn_number: int = 1
var game_mode: GameMode = GameMode.HOT_SEAT
var player_ready_status: Dictionary = {0: false, 1: false}  # For online
```

### **Key Algorithms**

#### **Phase Transition Validation**

```gdscript
func can_transition(from: TurnPhase, to: TurnPhase) -> bool:
    var transitions: Dictionary = {
        TurnPhase.QUEUE_ORDERS_A: [
            TurnPhase.IRON_CURTAIN,
            TurnPhase.CALCULATE_MOVEMENT
        ],
        TurnPhase.IRON_CURTAIN: [TurnPhase.QUEUE_ORDERS_B],
        TurnPhase.QUEUE_ORDERS_B: [TurnPhase.REVEAL_ORDERS],
        TurnPhase.REVEAL_ORDERS: [TurnPhase.CALCULATE_MOVEMENT],
        TurnPhase.CALCULATE_MOVEMENT: [TurnPhase.CALCULATE_BATTLES],
        TurnPhase.CALCULATE_BATTLES: [TurnPhase.PLAY_BATTLE_ANIMS],
        TurnPhase.PLAY_BATTLE_ANIMS: [TurnPhase.RESOLVE_BATTLES],
        TurnPhase.RESOLVE_BATTLES: [TurnPhase.CLEANUP],
        TurnPhase.CLEANUP: [TurnPhase.CHECK_WIN_CONDITION],
        TurnPhase.CHECK_WIN_CONDITION: [
            TurnPhase.RESET_TURN,
            MetaState.VICTORY_SCREEN,
            MetaState.DEFEAT_SCREEN
        ],
        TurnPhase.RESET_TURN: [TurnPhase.QUEUE_ORDERS_A]
    }
    
    return to in transitions.get(from, [])
```

#### **Game Mode Branching**

```gdscript
func on_player_ready(player_id: int) -> void:
    match game_mode:
        GameMode.HOT_SEAT:
            _handle_hot_seat_ready(player_id)
        GameMode.VS_AI:
            _handle_ai_ready(player_id)
        GameMode.ONLINE:
            _handle_online_ready(player_id)

func _handle_hot_seat_ready(player_id: int) -> void:
    if player_id == 0:
        # Player A ready → Iron Curtain
        transition(TurnPhase.IRON_CURTAIN)
    elif player_id == 1:
        # Player B ready → Reveal Orders
        transition(TurnPhase.REVEAL_ORDERS)

func _handle_ai_ready(player_id: int) -> void:
    if player_id == 0:
        # Human ready → Generate AI orders for Player B
        ai_manager.generate_orders_for_player(1)
        # Skip Iron Curtain and Reveal Orders
        transition(TurnPhase.CALCULATE_MOVEMENT)

func _handle_online_ready(player_id: int) -> void:
    player_ready_status[player_id] = true
    # Both ready → Start resolution
    if player_ready_status[0] and player_ready_status[1]:
        transition(TurnPhase.CALCULATE_MOVEMENT)
```

### **Integration Points**

**Inputs:**
- `player_ready(player_id: int)` from GameBoard
- `undo_clicked()` from GameBoard
- `ready_clicked()` from GameBoard

**Outputs:**
- `state_changed(from: TurnPhase, to: TurnPhase)` to all UI components
- `phase_started(phase: TurnPhase)` to systems for initialization
- `turn_started(player_id: int, turn_number: int)` to UI
- `turn_ended(player_id: int)` to UI

---

## **2. ORDER MANAGER**

**File**: `scripts/game/order_manager.gd`

### **Responsibilities**
- Store and validate movement orders
- Store and validate attack orders
- Track order counts (5 movements, 1 attack max)
- Provide undo functionality
- Clear orders on turn reset

### **Core Data Structures**

```gdscript
class MovementOrder extends RefCounted:
    var unit_id: int
    var unit_data: UnitData
    var source_pos: Vector2i
    var target_pos: Vector2i
    var path: Array[Vector2i] = []  # Calculated by Pathfinding

class AttackOrder extends RefCounted:
    var attacker_id: int
    var attacker_data: UnitData
    var attacker_pos: Vector2i
    var target_pos: Vector2i
    var target_unit_id: int = 0

# Order state
var movement_orders: Dictionary = {0: [], 1: []}  # player_id -> MovementOrder[]
var attack_orders: Dictionary = {0: [], 1: []}  # player_id -> AttackOrder[]
var undo_stack: Array[Dictionary] = []  # Stack for undo
```

### **Key Algorithms**

#### **Movement Order Validation**

```gdscript
func can_add_movement_order(unit: Unit, target_pos: Vector2i) -> bool:
    # Check if unit belongs to current player
    if unit.owner_id != current_player:
        return false
    
    # Check movement count
    var player_moves: int = movement_orders[current_player].size()
    if player_moves >= 5:
        return false
    
    # Check if unit already has movement order
    for order in movement_orders[current_player]:
        if order.unit_id == unit.unit_id:
            return false
    
    # Check if target is passable (deferred to MovementSystem)
    if not movement_system.is_tile_passable(target_pos):
        return false
    
    # Check path validity (deferred to Pathfinding)
    var path = pathfinding.find_path(unit.grid_position, target_pos)
    if path.is_empty():
        return false
    
    return true
```

#### **Attack Order Validation**

```gdscript
func can_add_attack_order(attacker: Unit, target_pos: Vector2i) -> bool:
    # Check if attacker belongs to current player
    if attacker.owner_id != current_player:
        return false
    
    # Check attack count
    var player_attacks: int = attack_orders[current_player].size()
    if player_attacks >= 1:
        return false
    
    # Check if attacker already has attack order
    for order in attack_orders[current_player]:
        if order.attacker_id == attacker.unit_id:
            return false
    
    # Check if target is within range
    var distance = abs(attacker.grid_position.x - target_pos.x) + \
                 abs(attacker.grid_position.y - target_pos.y)
    if distance > attacker.data.range:
        return false
    
    return true
```

#### **Undo Implementation**

```gdscript
func undo_last_order() -> void:
    if undo_stack.is_empty():
        return
    
    # Pop last order state
    var last_state = undo_stack.pop_back()
    
    # Restore movement orders
    movement_orders = last_state.movements.duplicate(true)
    
    # Restore attack orders
    attack_orders = last_state.attacks.duplicate(true)
    
    # Emit signal for UI update
    orders_changed.emit()

func save_order_state() -> void:
    var state = {
        "movements": movement_orders.duplicate(true),
        "attacks": attack_orders.duplicate(true)
    }
    undo_stack.push_back(state)
    
    # Limit stack size (optional, e.g., 100)
    if undo_stack.size() > 100:
        undo_stack.pop_front()
```

### **Integration Points**

**Inputs:**
- `add_movement_order(unit: Unit, target: Vector2i)` from GameBoard
- `add_attack_order(attacker: Unit, target: Vector2i)` from GameBoard
- `undo_order()` from GameBoard
- `clear_orders(player_id: int)` from TurnStateMachine

**Outputs:**
- `orders_changed()` to GameBoard UI (update counters)
- `orders_valid_changed(is_valid: bool)` to GameBoard (enable/disable Ready button)
- `movement_orders` to MovementSystem (for collision detection)
- `attack_orders` to BattleSystem (for battle calculation)

---

## **3. MOVEMENT SYSTEM**

**File**: `scripts/game/movement_system.gd`

### **Responsibilities**
- Execute all movement orders
- Update unit positions
- Coordinate with CollisionSystem
- Handle embark/disembark
- Update transport relationships

### **Core Data Structures**

```gdscript
# Movement queue
var movement_queue: Array[MovementOrder] = []

# Transport relationships
var carried_units: Dictionary = {}  # unit_id -> carrier_unit_id
var carrier_capacity: Dictionary = {}  # unit_id -> carried_unit_ids[]
```

### **Key Algorithms**

#### **Movement Execution**

```gdscript
func execute_movements(player_movements: Dictionary) -> void:
    # Clear previous queue
    movement_queue.clear()
    
    # Flatten all player movements into single queue
    for player_id in player_movements:
        for order in player_movements[player_id]:
            movement_queue.push_back(order)
    
    # Sort queue to maintain consistency (optional)
    # Order by unit_id to ensure deterministic behavior
    
    # Execute movements
    for order in movement_queue:
        _execute_single_movement(order)

func _execute_single_movement(order: MovementOrder) -> void:
    var unit = unit_manager.get_unit(order.unit_id)
    if not unit:
        return  # Unit was destroyed
    
    # Calculate path (already calculated when order was added)
    var path = order.path
    
    # Handle embark/disembark
    if _should_embark(unit, order.target_pos):
        _embark_unit(unit, order.target_pos)
    elif _should_disembark(unit):
        _disembark_unit(unit)
    else:
        # Normal movement
        unit.grid_position = order.target_pos
        unit.global_position = _grid_to_world(order.target_pos)
    
    # Emit signal
    unit.moved.emit(order.source_pos, order.target_pos)
```

#### **Transport Handling**

```gdscript
func _should_embark(unit: Unit, target_pos: Vector2i) -> bool:
    # Check if target tile has carrier
    var carrier = unit_manager.get_unit_at(target_pos)
    if not carrier or not carrier.data.can_carry_units:
        return false
    
    # Check if carrier has capacity
    var carried = carried_units.get(carrier.unit_id, [])
    if carried.size() >= carrier.data.capacity:
        return false
    
    return true

func _embark_unit(unit: Unit, target_pos: Vector2i) -> void:
    var carrier = unit_manager.get_unit_at(target_pos)
    
    # Set carrier relationship
    carried_units[unit.unit_id] = carrier.unit_id
    carrier_capacity[carrier.unit_id].push_back(unit.unit_id)
    
    # Set unit position to carrier's position
    unit.grid_position = target_pos
    unit.global_position = _grid_to_world(target_pos)
    
    # Hide carried unit visually
    unit.visible = false

func _disembark_unit(unit: Unit) -> void:
    var carrier_id = carried_units.get(unit.unit_id)
    var carrier = unit_manager.get_unit(carrier_id)
    
    # Find valid disembark tile (adjacent, passable)
    var valid_tiles = _find_adjacent_passable(carrier.grid_position)
    var target_pos = valid_tiles[0]  # First valid tile
    
    # Clear carrier relationship
    carried_units.erase(unit.unit_id)
    carrier_capacity[carrier_id].erase(unit.unit_id)
    
    # Set unit position
    unit.grid_position = target_pos
    unit.global_position = _grid_to_world(target_pos)
    
    # Show carried unit visually
    unit.visible = true
```

### **Integration Points**

**Inputs:**
- `movement_orders` from OrderManager
- `get_units_at(pos: Vector2i)` from UnitManager

**Outputs:**
- `movement_executed(unit: Unit, from: Vector2i, to: Vector2i)` to NetworkSystem
- `movement_executed()` to CollisionSystem (for collision detection)
- `unit_positions_changed()` to BattleSystem (for attack target resolution)

---

## **4. COLLISION SYSTEM**

**File**: `scripts/game/collision_system.gd`

### **Responsibilities**
- Detect overlapping movements
- Apply collision rules
- Handle train collision exceptions
- Resolve "squish rule" for trains

### **Core Data Structures**

```gdscript
# Collision detection data
var movement_targets: Dictionary = {}  # target_pos -> MovementOrder[]

# Collision results
var collisions: Array[Dictionary] = []

# Modified movements (for early stops)
var modified_movements: Array[Dictionary] = []
```

### **Key Algorithms**

#### **Collision Detection**

```gdscript
func detect_collisions(movement_orders: Array[MovementOrder]) -> void:
    # Build target mapping
    movement_targets.clear()
    
    for order in movement_orders:
        var target = order.target_pos
        if not target in movement_targets:
            movement_targets[target] = []
        movement_targets[target].push_back(order)

func resolve_collisions() -> void:
    collisions.clear()
    modified_movements.clear()
    
    # Check each target for multiple movements
    for target_pos in movement_targets:
        var orders = movement_targets[target_pos]
        
        if orders.size() > 1:
            # Collision detected
            var collision = {
                "target_pos": target_pos,
                "orders": orders,
                "type": _determine_collision_type(orders)
            }
            collisions.push_back(collision)
            
            # Apply early stops
            _apply_early_stops(collision)

func _determine_collision_type(orders: Array) -> String:
    var has_train = false
    var has_regular = false
    
    for order in orders:
        if order.unit_data.railway_locked:
            has_train = true
        else:
            has_regular = true
    
    if has_train and has_regular:
        return "train_vs_unit"
    elif has_train:
        return "train_vs_train"
    else:
        return "unit_vs_unit"
```

#### **Early Stop Application**

```gdscript
func _apply_early_stops(collision: Dictionary) -> void:
    var type = collision.type
    
    match type:
        "unit_vs_unit", "train_vs_train":
            # Both units stop one tile before collision
            for order in collision.orders:
                var stop_pos = _find_stop_position(order)
                order.path[order.path.size() - 1] = stop_pos
                modified_movements.push_back({
                    "unit_id": order.unit_id,
                    "new_target": stop_pos
                })
        
        "train_vs_unit":
            # Regular unit stops one tile early, train continues
            for order in collision.orders:
                if order.unit_data.railway_locked:
                    # Train continues (no change)
                    continue
                else:
                    # Regular unit stops
                    var stop_pos = _find_stop_position(order)
                    order.path[order.path.size() - 1] = stop_pos
                    modified_movements.push_back({
                        "unit_id": order.unit_id,
                        "new_target": stop_pos
                    })

func _find_stop_position(order: MovementOrder) -> Vector2i:
    # Find last tile before collision target
    var path = order.path
    if path.size() <= 1:
        return order.source_pos
    
    # Stop at second-to-last tile
    return path[path.size() - 2]
```

#### **Squish Rule**

```gdscript
func apply_squish_rule() -> void:
    # After early stops, check if regular units still in train path
    for collision in collisions:
        if collision.type != "train_vs_unit":
            continue
        
        # Find train order and regular unit order
        var train_order = null
        var regular_order = null
        
        for order in collision.orders:
            if order.unit_data.railway_locked:
                train_order = order
            else:
                regular_order = order
        
        if not train_order or not regular_order:
            continue
        
        # Get regular unit's new position (after early stop)
        var regular_pos = regular_order.target_pos
        
        # Check if regular_pos is still on train path
        if _is_on_path(train_order.path, regular_pos):
            # Apply forced retreat
            var retreat_pos = _find_valid_retreat_tile(regular_pos, train_order.target_pos)
            
            if retreat_pos != regular_pos:
                # Move regular unit one tile away
                modified_movements.push_back({
                    "unit_id": regular_order.unit_id,
                    "new_target": retreat_pos
                })
            else:
                # No valid retreat → destroy unit
                unit_manager.destroy_unit(regular_order.unit_id)

func _is_on_path(path: Array[Vector2i], pos: Vector2i) -> bool:
    for tile in path:
        if tile == pos:
            return true
    return false

func _find_valid_retreat_tile(unit_pos: Vector2i, train_pos: Vector2i) -> Vector2i:
    # Find tiles one away from unit_pos that are NOT towards train_pos
    var adjacent_tiles = _get_adjacent_tiles(unit_pos)
    
    for tile in adjacent_tiles:
        # Calculate direction from unit_pos to tile
        var direction = tile - unit_pos
        
        # Calculate direction from unit_pos to train_pos
        var train_direction = train_pos - unit_pos
        
        # Check if tile is away from train (dot product < 0)
        if direction.dot(train_direction) < 0:
            # Check if passable
            if movement_system.is_tile_passable(tile):
                return tile
    
    # No valid retreat found
    return Vector2i(-1, -1)
```

### **Integration Points**

**Inputs:**
- `movement_orders` from OrderManager
- `unit_data` from UnitManager

**Outputs:**
- `collision_detected(collision: Dictionary)` to MovementSystem (for early stops)
- `squish_applied(unit_id: int, new_pos: Vector2i)` to MovementSystem
- `collision_results` to BattleSystem (for battle participant determination)

---

## **5. BATTLE SYSTEM**

**File**: `scripts/game/battle_system.gd`

### **Responsibilities**
- Identify all battles
- Group participants
- Calculate outcomes (Attack vs Defense)
- Determine forced retreat or destruction
- Handle simultaneity

### **Core Data Structures**

```gdscript
class Battle extends RefCounted:
    var battle_id: int
    var target_pos: Vector2i
    var attackers: Array[int] = []  # unit_ids
    var defenders: Array[int] = []  # unit_ids
    var total_attack: int = 0
    var total_defense: int = 0
    var terrain_bonus: int = 0
    var outcome: String = "fail"  # "fail", "retreat", "destroy"

# Battle state
var battles: Array[Battle] = []
var participant_cache: Dictionary = {}  # unit_id -> bool (participating this turn)
```

### **Key Algorithms**

#### **Battle Identification**

```gdscript
func identify_battles(attack_orders: Array[AttackOrder]) -> void:
    battles.clear()
    participant_cache.clear()
    
    for attack_order in attack_orders:
        var target_pos = attack_order.target_pos
        var attacker_id = attack_order.attacker_id
        
        # Find target unit
        var target_unit = unit_manager.get_unit_at(target_pos)
        var target_id = target_unit.unit_id if target_unit else 0
        
        # Find all participants in range
        var attackers = _find_attackers_in_range(target_pos)
        var defenders = _find_defenders_in_range(target_pos)
        
        # Mark participants
        for attacker_id in attackers:
            participant_cache[attacker_id] = true
        for defender_id in defenders:
            participant_cache[defender_id] = true
        
        # Create battle
        var battle = Battle.new()
        battle.battle_id = battles.size()
        battle.target_pos = target_pos
        battle.attackers = attackers
        battle.defenders = defenders
        
        battles.push_back(battle)

func _find_attackers_in_range(target_pos: Vector2i) -> Array[int]:
    var attackers: Array[int] = []
    
    # Get all units of current player
    var player_units = unit_manager.get_units_for_player(current_player)
    
    for unit in player_units:
        # Check if unit already in this battle
        if unit.unit_id in participant_cache:
            continue
        
        # Check if target is in range
        var distance = abs(unit.grid_position.x - target_pos.x) + \
                     abs(unit.grid_position.y - target_pos.y)
        
        if distance <= unit.data.range:
            # Check if path is not blocked
            if _is_path_clear(unit.grid_position, target_pos):
                attackers.push_back(unit.unit_id)
    
    return attackers

func _find_defenders_in_range(target_pos: Vector2i) -> Array[int]:
    var defenders: Array[int] = []
    var opponent_id = 1 - current_player  # 0 ↔ 1
    
    # Get all units of opponent
    var opponent_units = unit_manager.get_units_for_player(opponent_id)
    
    for unit in opponent_units:
        # Check if unit is at target tile
        if unit.grid_position == target_pos:
            defenders.push_back(unit.unit_id)
            continue
        
        # Check if target is in range
        var distance = abs(unit.grid_position.x - target_pos.x) + \
                     abs(unit.grid_position.y - target_pos.y)
        
        if distance <= unit.data.range:
            # Check if path is not blocked
            if _is_path_clear(unit.grid_position, target_pos):
                defenders.push_back(unit.unit_id)
    
    return defenders

func _is_path_clear(from: Vector2i, to: Vector2i) -> bool:
    # Check if path is blocked by enemy units or impassable terrain
    var path = pathfinding.find_path(from, to)
    
    for tile in path:
        # Check terrain
        var tile_data = terrain_system.get_tile_data_at(tile)
        if not tile_data.is_passable:
            return false
        
        # Check enemy units
        var unit = unit_manager.get_unit_at(tile)
        if unit and unit.owner_id != current_player:
            # Enemy unit blocks path
            return false
    
    return true
```

#### **Outcome Calculation**

```gdscript
func calculate_outcomes() -> void:
    for battle in battles:
        # Sum attack values
        var total_attack = 0
        for attacker_id in battle.attackers:
            var attacker = unit_manager.get_unit(attacker_id)
            
            # Apply cavalry charge bonus
            var attack = attacker.data.attack
            if attacker.data.has_cavalry_charge:
                if _is_adjacent_to_defender(attacker, battle):
                    attack += 3
            
            total_attack += attack
        
        battle.total_attack = total_attack
        
        # Sum defense values + terrain bonus
        var total_defense = 0
        var terrain_bonus = 0
        
        var tile_data = terrain_system.get_tile_data_at(battle.target_pos)
        terrain_bonus = tile_data.defense_bonus
        
        for defender_id in battle.defenders:
            var defender = unit_manager.get_unit(defender_id)
            total_defense += defender.data.defense
        
        battle.terrain_bonus = terrain_bonus
        battle.total_defense = total_defense + terrain_bonus
        
        # Determine outcome
        if total_defense >= total_attack:
            battle.outcome = "fail"
        elif total_attack > total_defense:
            var margin = total_attack - total_defense
            if margin == 1:
                battle.outcome = "retreat"
            else:
                battle.outcome = "destroy"

func _is_adjacent_to_defender(attacker: Unit, battle: Battle) -> bool:
    # Check if attacker is adjacent to any defender
    for defender_id in battle.defenders:
        var defender = unit_manager.get_unit(defender_id)
        var distance = abs(attacker.grid_position.x - defender.grid_position.x) + \
                     abs(attacker.grid_position.y - defender.grid_position.y)
        
        if distance <= 1:
            return true
    
    return false
```

#### **Battle Resolution Application**

```gdscript
func resolve_battles() -> void:
    # First, mark all affected units
    var to_destroy: Array[int] = []
    var to_retreat: Dictionary = {}  # unit_id -> retreat_pos
    
    for battle in battles:
        match battle.outcome:
            "destroy":
                for defender_id in battle.defenders:
                    to_destroy.push_back(defender_id)
            
            "retreat":
                for defender_id in battle.defenders:
                    var retreat_pos = _find_retreat_position(defender_id, battle)
                    if retreat_pos != Vector2i(-1, -1):
                        to_retreat[defender_id] = retreat_pos
                    else:
                        to_destroy.push_back(defender_id)
    
    # Apply destructions (simultaneous)
    for unit_id in to_destroy:
        unit_manager.destroy_unit(unit_id)
        # Also destroy carried units
        if unit_id in carried_units:
            # This unit was carrying others
            pass  # Carrier destruction handles carried units
    
    # Apply retreats
    for unit_id in to_retreat:
        var unit = unit_manager.get_unit(unit_id)
        if unit:
            unit.grid_position = to_retreat[unit_id]
            unit.global_position = _grid_to_world(to_retreat[unit_id])

func _find_retreat_position(unit_id: int, battle: Battle) -> Vector2i:
    var unit = unit_manager.get_unit(unit_id)
    var battle_pos = battle.target_pos
    
    # Find adjacent tiles away from battle
    var adjacent = _get_adjacent_tiles(unit.grid_position)
    var valid_retreats: Array[Vector2i] = []
    
    for tile in adjacent:
        # Check if tile is away from battle (not towards attackers)
        var direction = tile - unit.grid_position
        var battle_direction = battle_pos - unit.grid_position
        
        if direction.dot(battle_direction) < 0:
            # Check passable
            if movement_system.is_tile_passable(tile):
                valid_retreats.push_back(tile)
    
    # Return first valid retreat
    if valid_retreats.is_empty():
        return Vector2i(-1, -1)
    else:
        return valid_retreats[0]
```

### **Integration Points**

**Inputs:**
- `attack_orders` from OrderManager
- `collision_results` from CollisionSystem
- `unit_data` from UnitManager
- `tile_data` from TerrainSystem

**Outputs:**
- `battles_calculated(battles: Array[Battle])` to TurnStateMachine
- `battle_result(unit_id: int, outcome: String)` to UnitManager
- `unit_destroyed(unit_id: int)` to RailwaySystem (for train detachment)

---

## **6. NETWORK SYSTEM (COMMUNICATIONS)**

**File**: `scripts/game/network_system.gd`

### **Responsibilities**
- Calculate lines of effect
- Determine active/inactive unit states
- Handle relay units
- Handle station sources
- Handle telegraph propagation

### **Core Data Structures**

```gdscript
# Unit states
var active_units: Dictionary = {}  # unit_id -> bool

# Network graph
var network_graph: Dictionary = {}  # tile_pos -> NetworkNode
var source_tiles: Array[Vector2i] = []  # Station positions

class NetworkNode extends RefCounted:
    var position: Vector2i
    var is_source: bool
    var is_relay: bool
    var owner_id: int
    var connected_nodes: Array[int] = []  # tile_pos keys
```

### **Key Algorithms**

#### **Line of Effect Calculation**

```gdscript
func calculate_lines_of_effect() -> void:
    # Build network graph
    _build_network_graph()
    
    # Determine active units via BFS from sources
    active_units.clear()
    
    for source_pos in source_tiles:
        _propagate_activity_from_source(source_pos)

func _propagate_activity_from_source(source_pos: Vector2i) -> void:
    var visited: Dictionary = {}  # tile_pos -> bool
    var queue: Array[Vector2i] = [source_pos]
    
    while not queue.is_empty():
        var current_pos = queue.pop_front()
        
        if current_pos in visited:
            continue
        
        visited[current_pos] = true
        
        # Check if this tile has a unit
        var unit = unit_manager.get_unit_at(current_pos)
        if unit:
            # Mark as active
            active_units[unit.unit_id] = true
        
        # Add adjacent nodes to queue
        var node = network_graph.get(current_pos)
        if node:
            for connected_pos in node.connected_nodes:
                if connected_pos not in visited:
                    # Check if path is clear (no enemy units, no impassable terrain)
                    if _is_connection_clear(current_pos, connected_pos):
                        queue.push_back(connected_pos)

func _is_connection_clear(from: Vector2i, to: Vector2i) -> bool:
    # Check if connection is blocked by impassable terrain
    var to_tile = terrain_system.get_tile_data_at(to)
    if not to_tile.is_passable or to_tile.blocks_line_of_effect:
        return false
    
    # Check if connection is blocked by enemy units
    var to_unit = unit_manager.get_unit_at(to)
    if to_unit:
        var from_unit = unit_manager.get_unit_at(from)
        if from_unit and to_unit.owner_id != from_unit.owner_id:
            # Enemy unit blocks connection
            return false
    
    return true
```

#### **Station Source Identification**

```gdscript
func identify_station_sources() -> void:
    source_tiles.clear()
    
    # Find all stations
    var stations = terrain_system.get_tiles_by_type("station")
    
    for station_pos in stations:
        var station = terrain_system.get_tile_data_at(station_pos)
        if station.is_station:
            # Check who controls it (last infantry to move onto it)
            var controlling_player = _get_station_owner(station_pos)
            
            if controlling_player >= 0:
                # Controlled by a player
                source_tiles.push_back(station_pos)
                
                # Mark node as source
                var node = network_graph.get(station_pos)
                if node:
                    node.is_source = true
                    node.owner_id = controlling_player

func _get_station_owner(station_pos: Vector2i) -> int:
    # Check which player has infantry on station
    var units = unit_manager.get_units_at(station_pos)
    
    for unit in units:
        if unit.data.display_name == "Infantry":
            return unit.owner_id
    
    # No controlling infantry
    return -1
```

#### **Relay Handling**

```gdscript
func calculate_relays() -> void:
    # Find all relay units
    var relay_units = unit_manager.get_units_by_ability("is_relay", true)
    
    for relay in relay_units:
        # Check if relay is active (connected to source or another active relay)
        if _is_relay_active(relay):
            # Mark relay as active
            active_units[relay.unit_id] = true
        else:
            # Mark relay as inactive
            active_units[relay.unit_id] = false

func _is_relay_active(relay: Unit) -> bool:
    # Check if relay is connected to an active unit
    var adjacent = _get_adjacent_units(relay.grid_position)
    
    for unit in adjacent:
        if unit.owner_id == relay.owner_id:
            if active_units.get(unit.unit_id, false):
                # Connected to active unit
                return true
    
    return false
```

#### **Telegraph Propagation**

```gdscript
func propagate_telegraph() -> void:
    # Find all telegraph stations
    var telegraph_stations = terrain_system.get_tiles_by_type("telegraph_office")
    
    # Mark controlled telegraph stations as active
    var active_telegraphs: Array[Vector2i] = []
    
    for station_pos in telegraph_stations:
        var station = terrain_system.get_tile_data_at(station_pos)
        var owner = _get_station_owner(station_pos)
        
        if owner >= 0:
            # Controlled telegraph acts as source for other telegraphs
            active_telegraphs.push_back(station_pos)
    
    # Propagate activity to connected telegraphs
    for source_pos in active_telegraphs:
        _propagate_telegraph_from_source(source_pos)

func _propagate_telegraph_from_source(source_pos: Vector2i) -> void:
    var visited: Dictionary = {}
    var queue: Array[Vector2i] = [source_pos]
    
    while not queue.is_empty():
        var current_pos = queue.pop_front()
        
        if current_pos in visited:
            continue
        
        visited[current_pos] = true
        
        # Get tile at current position
        var tile = terrain_system.get_tile_data_at(current_pos)
        
        # If telegraph station, mark as active source
        if tile.is_telegraph_office:
            # Mark as source for Network System
            var node = network_graph.get(current_pos)
            if node:
                node.is_source = true
        
        # Add adjacent telegraph connections to queue
        var adjacent = _get_adjacent_telegraph_stations(current_pos)
        for telegraph_pos in adjacent:
            if telegraph_pos not in visited:
                queue.push_back(telegraph_pos)

func _get_adjacent_telegraph_stations(pos: Vector2i) -> Array[Vector2i]:
    var telegraphs: Array[Vector2i] = []
    var adjacent = _get_adjacent_tiles(pos)
    
    for tile_pos in adjacent:
        var tile = terrain_system.get_tile_data_at(tile_pos)
        if tile.is_telegraph_office:
            telegraphs.push_back(tile_pos)
    
    return telegraphs
```

### **Integration Points**

**Inputs:**
- `unit_positions` from UnitManager
- `tile_data` from TerrainSystem
- `movement_executed` from MovementSystem (recalculate after movement)

**Outputs:**
- `active_units` to UnitManager (for visual state update)
- `active_units` to BattleSystem (participation requires active state)

---

## **7. RAILWAY SYSTEM**

**File**: `scripts/game/railway_system.gd`

### **Responsibilities**
- Handle train movement (track-locked)
- Manage train composition (engine + cars)
- Handle turntables and intersections
- Handle train collisions (destroy vs. squish)
- Detach cars when engine destroyed

### **Core Data Structures**

```gdscript
# Train composition
var train_compositions: Dictionary = {}  # engine_unit_id -> car_unit_ids[]

# Track graph
var track_graph: Dictionary = {}  # tile_pos -> ConnectedTrackNodes

class ConnectedTrackNodes extends RefCounted:
    var position: Vector2i
    var rail_type: String  # "train" or "tram"
    var directions: Array[String] = []  # "N", "S", "E", "W", "NE", etc.

# Special track tiles
var turntables: Dictionary = {}  # tile_pos -> Engine on it
var intersections: Dictionary = {}  # tile_pos -> Engine on it
```

### **Key Algorithms**

#### **Track Graph Construction**

```gdscript
func build_track_graph() -> void:
    track_graph.clear()
    
    # Find all rail tiles
    var rail_tiles = terrain_system.get_tiles_by_rail()
    
    for tile_pos in rail_tiles:
        var tile_data = terrain_system.get_tile_data_at(tile_pos)
        var node = ConnectedTrackNodes.new()
        node.position = tile_pos
        node.rail_type = tile_data.rail_type
        
        # Find connected directions
        for direction in ["N", "S", "E", "W", "NE", "SW", "NW", "SE"]:
            if _has_track_in_direction(tile_pos, direction, tile_data.rail_type):
                node.directions.push_back(direction)
        
        track_graph[tile_pos] = node

func _has_track_in_direction(pos: Vector2i, direction: String, rail_type: String) -> bool:
    var adjacent_pos = _get_tile_in_direction(pos, direction)
    var tile_data = terrain_system.get_tile_data_at(adjacent_pos)
    
    return tile_data and tile_data.has_rail and tile_data.rail_type == rail_type

func _get_tile_in_direction(pos: Vector2i, direction: String) -> Vector2i:
    match direction:
        "N": return pos + Vector2i(0, -1)
        "S": return pos + Vector2i(0, 1)
        "E": return pos + Vector2i(1, 0)
        "W": return pos + Vector2i(-1, 0)
        "NE": return pos + Vector2i(1, -1)
        "NW": return pos + Vector2i(-1, -1)
        "SE": return pos + Vector2i(1, 1)
        "SW": return pos + Vector2i(-1, 1)
        _: return pos

func _get_adjacent_tiles(pos: Vector2i) -> Array[Vector2i]:
    return [
        _get_tile_in_direction(pos, "N"),
        _get_tile_in_direction(pos, "S"),
        _get_tile_in_direction(pos, "E"),
        _get_tile_in_direction(pos, "W")
    ]
```

#### **Train Movement Validation**

```gdscript
func validate_train_movement(engine: Unit, target_pos: Vector2i) -> bool:
    # Check if unit is train engine
    if not engine.data.is_train_engine:
        return false
    
    # Check if target is on track
    var target_tile = terrain_system.get_tile_data_at(target_pos)
    if not target_tile.has_rail or target_tile.rail_type != "train":
        return false
    
    # Check if path is valid (connected tracks)
    var path = _find_train_path(engine.grid_position, target_pos)
    if path.is_empty():
        return false
    
    return true

func _find_train_path(from: Vector2i, to: Vector2i) -> Array[Vector2i]:
    # BFS to find connected path on tracks
    var visited: Dictionary = {}
    var queue: Array[Array] = [[from]]
    var came_from: Dictionary = {}
    
    while not queue.is_empty():
        var current_path = queue.pop_front()
        var current = current_path[-1]
        
        if current == to:
            return current_path
        
        if current in visited:
            continue
        
        visited[current] = true
        
        # Get valid track directions
        var node = track_graph.get(current)
        if not node:
            continue
        
        # Check special tiles (turntable, intersection)
        var can_change_direction = _can_change_direction(current, engine.grid_position)
        
        for direction in node.directions:
            var next_pos = _get_tile_in_direction(current, direction)
            
            # Check if we can go in this direction
            if not can_change_direction and direction not in _get_direction_from_to(current, from):
                # Not allowed to turn around
                continue
            
            if next_pos not in visited:
                var new_path = current_path.duplicate()
                new_path.push_back(next_pos)
                queue.push_back(new_path)
                came_from[next_pos] = current
    
    return []

func _get_direction_from_to(from: Vector2i, to: Vector2i) -> String:
    var diff = to - from
    if abs(diff.x) > abs(diff.y):
        return "E" if diff.x > 0 else "W"
    else:
        return "S" if diff.y > 0 else "N"

func _can_change_direction(current_pos: Vector2i, engine_pos: Vector2i) -> bool:
    # Can change direction at turntable or intersection
    var current_tile = terrain_system.get_tile_data_at(current_pos)
    
    # Not at special tile → continue in current direction
    if not current_tile.is_turntable and not current_tile.is_intersection:
        return false
    
    # At special tile → can turn or continue
    return true
```

#### **Train Car Synchronization**

```gdscript
func sync_train_movement(engine: Unit, movement_path: Array[Vector2i]) -> void:
    # Get attached cars
    var car_ids = train_compositions.get(engine.unit_id, [])
    
    if car_ids.is_empty():
        return
    
    # Move each car to follow engine
    for i in range(car_ids.size()):
        var car = unit_manager.get_unit(car_ids[i])
        if not car:
            continue
        
        # Car should be at appropriate offset from engine
        var car_index = i + 1  # 1-based
        var target_pos_in_path = min(car_index, movement_path.size() - 1)
        var target_pos = movement_path[target_pos_in_path]
        
        # Update car position
        car.grid_position = target_pos
        car.global_position = _grid_to_world(target_pos)

func update_train_composition(engine: Unit) -> void:
    # Update composition when engine moves onto tile with car
    var adjacent_units = _get_adjacent_units(engine.grid_position)
    
    for unit in adjacent_units:
        if unit.data.is_train_car and unit.owner_id == engine.owner_id:
            # Attach car to engine
            if not engine.unit_id in train_compositions:
                train_compositions[engine.unit_id] = []
            
            if unit.unit_id not in train_compositions[engine.unit_id]:
                train_compositions[engine.unit_id].push_back(unit.unit_id)

func detach_train_cars(engine: Unit) -> void:
    # Detach all cars when engine destroyed
    if engine.unit_id in train_compositions:
        var car_ids = train_compositions[engine.unit_id]
        
        for car_id in car_ids:
            # Check if car was destroyed (already removed)
            var car = unit_manager.get_unit(car_id)
            if car:
                # Car is still alive, now independent
                pass  # Cars become independent units
        
        train_compositions.erase(engine.unit_id)
```

### **Integration Points**

**Inputs:**
- `movement_orders` from OrderManager
- `unit_data` from UnitManager
- `tile_data` from TerrainSystem

**Outputs:**
- `train_moved(engine: Unit, cars: Array[Unit])` to MovementSystem
- `train_collided(engine_a: Unit, engine_b: Unit)` to CollisionSystem
- `car_detached(car: Unit)` to UnitManager

---

## **8. PATHFINDING**

**File**: `scripts/utils/pathfinding.gd`

### **Responsibilities**
- Find shortest path between two tiles
- Check path validity (passable, no enemies)
- Calculate Manhattan distance (for range checks)

### **Core Data Structures**

```gdscript
# Cache for performance
var path_cache: Dictionary = {}  # "from,to" -> path

# Grid size
var grid_width: int = 20
var grid_height: int = 25
```

### **Key Algorithms**

#### **A* Pathfinding**

```gdscript
func find_path(from: Vector2i, to: Vector2i) -> Array[Vector2i]:
    # Check cache
    var cache_key = str(from) + "," + str(to)
    if cache_key in path_cache:
        return path_cache[cache_key].duplicate()
    
    # A* algorithm
    var open_set: Array[PathNode] = []
    var closed_set: Dictionary = {}
    var came_from: Dictionary = {}
    
    var start_node = PathNode.new(from, 0, _heuristic(from, to))
    open_set.push_back(start_node)
    
    while not open_set.is_empty():
        # Get node with lowest f_score
        var current = _get_lowest_f_score(open_set)
        open_set.erase(current)
        
        # Check if reached target
        if current.position == to:
            var path = _reconstruct_path(came_from, current.position)
            path_cache[cache_key] = path.duplicate()
            return path
        
        closed_set[current.position] = true
        
        # Expand neighbors
        var neighbors = _get_passable_neighbors(current.position)
        for neighbor in neighbors:
            if neighbor in closed_set:
                continue
            
            var tentative_g_score = current.g_score + 1  # Cost is 1 per tile
            var neighbor_node = PathNode.new(neighbor, tentative_g_score, _heuristic(neighbor, to))
            
            # Check if this path to neighbor is better
            var in_open = _find_in_open(open_set, neighbor)
            if not in_open or tentative_g_score < in_open.g_score:
                if in_open:
                    open_set.erase(in_open)
                open_set.push_back(neighbor_node)
                came_from[neighbor] = current.position
    
    # No path found
    return []

func _heuristic(a: Vector2i, b: Vector2i) -> int:
    # Manhattan distance
    return abs(a.x - b.x) + abs(a.y - b.y)

func _get_lowest_f_score(nodes: Array[PathNode]) -> PathNode:
    var lowest = nodes[0]
    for node in nodes:
        if node.f_score < lowest.f_score:
            lowest = node
    return lowest

func _find_in_open(nodes: Array[PathNode], position: Vector2i) -> PathNode:
    for node in nodes:
        if node.position == position:
            return node
    return null

func _reconstruct_path(came_from: Dictionary, current: Vector2i) -> Array[Vector2i]:
    var path: Array[Vector2i] = [current]
    var current_pos = current
    
    while current_pos in came_from:
        current_pos = came_from[current_pos]
        path.push_front(current_pos)
    
    return path
```

#### **Path Validation**

```gdscript
func is_path_passable(path: Array[Vector2i], player_id: int) -> bool:
    for tile_pos in path:
        var tile_data = terrain_system.get_tile_data_at(tile_pos)
        
        # Check passable
        if not tile_data.is_passable:
            return false
        
        # Check for enemy units (block path)
        var unit = unit_manager.get_unit_at(tile_pos)
        if unit and unit.owner_id != player_id:
            return false
    
    return true

func _get_passable_neighbors(pos: Vector2i) -> Array[Vector2i]:
    var neighbors: Array[Vector2i] = []
    var adjacent = _get_adjacent_tiles(pos)
    
    for tile_pos in adjacent:
        var tile_data = terrain_system.get_tile_data_at(tile_pos)
        if tile_data and tile_data.is_passable:
            neighbors.push_back(tile_pos)
    
    return neighbors

class PathNode extends RefCounted:
    var position: Vector2i
    var g_score: int  # Cost from start
    var h_score: int  # Heuristic to target
    var f_score: int  # Total estimated cost
    
    func _init(p: Vector2i, g: int, h: int):
        position = p
        g_score = g
        h_score = h
        f_score = g + h
```

### **Integration Points**

**Inputs:**
- `tile_data` from TerrainSystem
- `unit_positions` from UnitManager

**Outputs:**
- `path` to MovementSystem (for movement order path)
- `is_path_clear` to BattleSystem (for battle participation)

---

## **9. UNIT MANAGER**

**File**: `scripts/game/unit_manager.gd`

### **Responsibilities**
- Create and destroy units
- Track all units on board
- Provide unit lookup by position and ID
- Handle unit state changes

### **Core Data Structures**

```gdscript
# Unit storage
var units: Dictionary = {}  # unit_id -> Unit
var units_by_position: Dictionary = {}  # grid_pos_str -> unit_ids[]
var units_by_player: Dictionary = {}  # player_id -> unit_ids[]

# Unit ID counter
var next_unit_id: int = 1

# Unit instances (scene instances)
var unit_instances: Dictionary = {}  # unit_id -> Unit scene instance
```

### **Key Algorithms**

#### **Unit Creation**

```gdscript
func create_unit(unit_type: String, owner_id: int, position: Vector2i) -> Unit:
    var unit_data = DataManager.get_unit_data(unit_type)
    
    # Create unit instance
    var unit_scene = load("res://scenes/units/unit.tscn")
    var unit_instance = unit_scene.instantiate()
    
    # Set unit data
    unit_instance.unit_id = next_unit_id
    unit_instance.unit_data = unit_data
    unit_instance.owner_id = owner_id
    unit_instance.grid_position = position
    
    # Set position
    unit_instance.global_position = _grid_to_world(position)
    
    # Add to game board
    game_board.units_container.add_child(unit_instance)
    
    # Store references
    units[next_unit_id] = unit_instance
    _add_to_position_index(position, next_unit_id)
    _add_to_player_index(owner_id, next_unit_id)
    unit_instances[next_unit_id] = unit_instance
    
    next_unit_id += 1
    
    return unit_instance
```

#### **Unit Lookup**

```gdscript
func get_unit(unit_id: int) -> Unit:
    return units.get(unit_id, null)

func get_unit_at(pos: Vector2i) -> Unit:
    var pos_key = str(pos.x) + "," + str(pos.y)
    var unit_ids = units_by_position.get(pos_key, [])
    
    # Return first unit (stacking not allowed)
    if unit_ids.is_empty():
        return null
    else:
        return get_unit(unit_ids[0])

func get_units_for_player(player_id: int) -> Array[Unit]:
    var unit_ids = units_by_player.get(player_id, [])
    var player_units: Array[Unit] = []
    
    for unit_id in unit_ids:
        var unit = get_unit(unit_id)
        if unit:
            player_units.push_back(unit)
    
    return player_units

func get_units_by_ability(ability: String, value: bool) -> Array[Unit]:
    var result: Array[Unit] = []
    
    for unit_id in units:
        var unit = units[unit_id]
        if unit and unit.data.get(ability) == value:
            result.push_back(unit)
    
    return result
```

#### **Unit Destruction**

```gdscript
func destroy_unit(unit_id: int) -> void:
    var unit = units.get(unit_id)
    if not unit:
        return
    
    # Remove from indices
    units.erase(unit_id)
    _remove_from_position_index(unit.grid_position, unit_id)
    _remove_from_player_index(unit.owner_id, unit_id)
    
    # Remove carried units
    if unit_id in carried_units:
        var carrier_id = carried_units[unit_id]
        var carried = carrier_capacity.get(carrier_id, [])
        
        for carried_id in carried:
            destroy_unit(carried_id)
        
        carried_units.erase(unit_id)
        carrier_capacity.erase(carrier_id)
    
    # Remove train composition
    if unit.data.is_train_engine and unit.unit_id in train_compositions:
        railway_system.detach_train_cars(unit)
    
    # Emit signal
    unit.destroyed.emit()
    
    # Free instance
    unit_instances.get(unit_id).queue_free()
    unit_instances.erase(unit_id)

func _add_to_position_index(pos: Vector2i, unit_id: int) -> void:
    var pos_key = str(pos.x) + "," + str(pos.y)
    if pos_key not in units_by_position:
        units_by_position[pos_key] = []
    units_by_position[pos_key].push_back(unit_id)

func _remove_from_position_index(pos: Vector2i, unit_id: int) -> void:
    var pos_key = str(pos.x) + "," + str(pos.y)
    if pos_key in units_by_position:
        units_by_position[pos_key].erase(unit_id)
        
        if units_by_position[pos_key].is_empty():
            units_by_position.erase(pos_key)

func _add_to_player_index(player_id: int, unit_id: int) -> void:
    if player_id not in units_by_player:
        units_by_player[player_id] = []
    units_by_player[player_id].push_back(unit_id)

func _remove_from_player_index(player_id: int, unit_id: int) -> void:
    if player_id in units_by_player:
        units_by_player[player_id].erase(unit_id)
        
        if units_by_player[player_id].is_empty():
            units_by_player.erase(player_id)
```

### **Integration Points**

**Inputs:**
- `unit_data` from DataManager
- `unit_position` from MovementSystem

**Outputs:**
- `unit_created(unit: Unit)` to NetworkSystem (recalculate)
- `unit_destroyed(unit_id: int)` to all systems
- `unit_positions_changed()` to BattleSystem
- `unit_active_state_changed(unit_id: int, active: bool)` to visual

---

## **10. TERRAIN SYSTEM**

**File**: `scripts/game/terrain_system.gd`

### **Responsibilities**
- Provide tile data access
- Manage tile state (destroyed infrastructure)
- Handle terrain destruction (sabotage)
- Support station ownership tracking

### **Core Data Structures**

```gdscript
# Tile grid
var tile_grid: Dictionary = {}  # grid_pos_str -> TileData

# Tile state (destructions, ownership)
var tile_states: Dictionary = {}  # grid_pos_str -> TileState

class TileState extends RefCounted:
    var position: Vector2i
    var is_destroyed: bool
    var owner_id: int = -1  # For neutral buildings

# Grid dimensions
var grid_width: int = 20
var grid_height: int = 25
```

### **Key Algorithms**

#### **Tile Data Loading**

```gdscript
func load_tiles_from_map(map_scene: Node) -> void:
    tile_grid.clear()
    tile_states.clear()
    
    var tilemap = map_scene.get_node("TileMap")
    grid_width = tilemap.get_used_rect().size.x
    grid_height = tilemap.get_used_rect().size.y
    
    # Iterate through all tiles
    for x in range(grid_width):
        for y in range(grid_height):
            var pos = Vector2i(x, y)
            var tile_data = tilemap.get_cell_tile_data(0, pos)
            
            if tile_data:
                var pos_key = str(x) + "," + str(y)
                tile_grid[pos_key] = tile_data
                
                # Initialize tile state
                tile_states[pos_key] = TileState.new()
                tile_states[pos_key].position = pos
```

#### **Tile State Management**

```gdscript
func destroy_tile(pos: Vector2i) -> void:
    var pos_key = str(pos.x) + "," + str(pos.y)
    
    if pos_key in tile_states:
        var state = tile_states[pos_key]
        state.is_destroyed = true
        
        # Revert to normal terrain (remove special effects)
        var original_tile = tile_grid.get(pos_key)
        
        # Create normal tile (e.g., grass)
        var normal_tile = _get_normal_tile_for_type(original_tile)
        
        # Update grid
        tile_grid[pos_key] = normal_tile
        
        # Update TileMap visually
        game_board.tilemap.set_cell(0, pos, normal_tile)
        
        # Emit signal
        tile_destroyed.emit(pos, original_tile)

func _get_normal_tile_for_type(original_tile: TileData) -> TileData:
    # Return appropriate base terrain
    match original_tile.display_name:
        "Rail (N-S)", "Rail (E-W)", "Tram (N-S)", "Tram (E-W)":
            return DataManager.get_tile_data("grass")
        "Station", "Telegraph Office":
            return DataManager.get_tile_data("grass")
        "Turntable":
            return DataManager.get_tile_data("rail_ns")  # Rail remains
        "Intersection":
            return DataManager.get_tile_data("rail_ns")  # Rail remains
        _:
            return original_tile  # No change

func restore_tile(pos: Vector2i) -> void:
    var pos_key = str(pos.x) + "," + str(pos.y)
    
    if pos_key in tile_states:
        var state = tile_states[pos_key]
        state.is_destroyed = false
        
        # Restore original tile (from map data)
        var original_tile = _get_original_tile_at(pos)
        
        if original_tile:
            tile_grid[pos_key] = original_tile
            game_board.tilemap.set_cell(0, pos, original_tile)

func _get_original_tile_at(pos: Vector2i) -> TileData:
    # This would require storing original map data
    # For now, return base terrain
    return DataManager.get_tile_data("grass")
```

#### **Station Ownership**

```gdscript
func set_station_owner(pos: Vector2i, owner_id: int) -> void:
    var pos_key = str(pos.x) + "," + str(pos.y)
    
    if pos_key in tile_states:
        tile_states[pos_key].owner_id = owner_id
        
        # Emit signal for Network System
        station_captured.emit(pos, owner_id)

func get_station_owner(pos: Vector2i) -> int:
    var pos_key = str(pos.x) + "," + str(pos.y)
    
    if pos_key in tile_states:
        return tile_states[pos_key].owner_id
    
    return -1
```

### **Integration Points**

**Inputs:**
- `tile_data` from DataManager
- `map_scene` from GameManager

**Outputs:**
- `tile_data` to Pathfinding (passable check)
- `tile_data` to MovementSystem (passable check)
- `tile_data` to NetworkSystem (line of effect check)
- `tile_destroyed(pos: Vector2i)` to NetworkSystem (recalculate)

---

## **11. SAVE SYSTEM**

**File**: `scripts/autoloads/save_system.gd`

### **Responsibilities**
- Save game state to JSON file
- Load game state from JSON file
- Manage save slots
- Handle version compatibility

### **Core Data Structures**

```gdscript
# Save directory
var save_directory: String = "user://saves/"

# Save slots (3 slots)
var slots: int = 3
```

### **Key Algorithms**

#### **Save Game**

```gdscript
func save_game(slot: int) -> Error:
    if slot < 0 or slot >= slots:
        return Error.ERR_INVALID_PARAMETER
    
    # Ensure save directory exists
    DirAccess.make_dir_absolute(save_directory)
    
    # Collect game state
    var save_data = _collect_game_state()
    
    # Convert to JSON
    var json_string = JSON.stringify(save_data)
    
    # Write to file
    var save_path = save_directory + "save_" + str(slot) + ".json"
    var file = FileAccess.open(save_path, FileAccess.WRITE)
    
    if file:
        file.store_string(json_string)
        file.close()
        return OK
    else:
        return Error.ERR_FILE_CANT_WRITE

func _collect_game_state() -> Dictionary:
    return {
        "version": "1.0",
        "map_id": game_board.current_map_id,
        "turn": turn_state_machine.turn_number,
        "current_player": turn_state_machine.current_player,
        "game_mode": turn_state_machine.game_mode,
        
        # Units
        "units": _collect_units_data(),
        
        # Tile states
        "tile_states": _collect_tile_states(),
        
        # Current orders
        "current_orders": _collect_orders_data(),
        
        # Campaign data
        "campaign_data": _collect_campaign_data()
    }

func _collect_units_data() -> Array[Dictionary]:
    var units_data: Array[Dictionary] = []
    
    for unit_id in unit_manager.units:
        var unit = unit_manager.units[unit_id]
        
        units_data.push_back({
            "unit_type": unit.data.display_name,
            "owner_id": unit.owner_id,
            "position": [unit.grid_position.x, unit.grid_position.y],
            "is_deployed": unit.is_deployed,
            "health": unit.health,
            "carried_unit_id": carried_units.get(unit_id, 0)
        })
    
    return units_data
```

#### **Load Game**

```gdscript
func load_game(slot: int) -> Error:
    if slot < 0 or slot >= slots:
        return Error.ERR_INVALID_PARAMETER
    
    var save_path = save_directory + "save_" + str(slot) + ".json"
    var file = FileAccess.open(save_path, FileAccess.READ)
    
    if not file:
        return Error.ERR_FILE_NOT_FOUND
    
    var json_string = file.get_as_text()
    file.close()
    
    # Parse JSON
    var json = JSON.new()
    var parse_result = json.parse(json_string)
    
    if parse_result.error != OK:
        return Error.ERR_PARSE_ERROR
    
    var save_data = parse_result.data
    
    # Validate version
    if save_data.get("version", "1.0") != "1.0":
        return Error.ERR_VERSION_MISMATCH
    
    # Restore game state
    _restore_game_state(save_data)
    
    return OK

func _restore_game_state(save_data: Dictionary) -> void:
    # Load map
    var map_id = save_data.map_id
    game_manager.load_map(map_id)
    
    # Restore turn state
    turn_state_machine.turn_number = save_data.turn
    turn_state_machine.current_player = save_data.current_player
    turn_state_machine.game_mode = save_data.game_mode
    
    # Restore units
    _restore_units(save_data.units)
    
    # Restore tile states
    _restore_tile_states(save_data.tile_states)
    
    # Restore orders
    _restore_orders(save_data.current_orders)
    
    # Restore campaign data
    _restore_campaign_data(save_data.campaign_data)
```

### **Integration Points**

**Inputs:**
- Game state from all systems

**Outputs:**
- Save files to disk
- Load data from disk to all systems

---

## **PERFORMANCE CONSIDERATIONS**

### **1. Pathfinding Optimization**
- **Cache paths**: Store computed paths in path cache
- **Limit cache size**: 1000 entries max, LRU eviction
- **Early exit**: If target is same as start, return immediately
- **Bidirectional A***: Consider both directions (if movement is symmetrical)

### **2. Network System Optimization**
- **Lazy recalculation**: Only recalculate when movement occurs
- **Incremental updates**: Don't rebuild entire graph every turn
- **Cache BFS results**: Store active units set, update incrementally

### **3. Battle System Optimization**
- **Spatial hashing**: Use grid-based lookup for range checks
- **Pre-computed ranges**: Cache unit ranges to avoid recalculation
- **Batch operations**: Process all battles in single pass

### **4. Memory Management**
- **Object pooling**: Reuse RefCounted objects instead of creating new ones
- **Weak references**: Use weak refs for non-critical caches
- **Explicit cleanup**: Clear caches on turn reset

### **5. Rendering Optimization**
- **Cull off-screen units**: Don't render units outside viewport
- **Batch sprite rendering**: Group units by owner for batch draw
- **Dirty flag**: Only redraw changed tiles

---

## **ERROR HANDLING**

### **Validation Errors**
- Invalid movement order → Show UI error message
- Invalid attack order → Show UI error message
- No valid retreat path → Destroy unit with fallback
- Invalid map data → Show error dialog, return to menu

### **System Errors**
- Pathfinding failure → Show error, block movement
- Battle calculation error → Show error, skip battle
- Save system failure → Show error dialog
- Load system failure → Show error dialog

### **User Feedback**
- All errors shown via UI (not console)
- Clear error messages with suggested actions
- Retry mechanism for recoverable errors

---

## **TESTING STRATEGY**

### **Unit Tests**
- Pathfinding: Verify path validity for all terrain combinations
- Battle calculation: Test edge cases (adjacent cavalry, retreats with no valid tiles)
- Network system: Test relay chains, telegraph propagation
- Railway system: Test train movements, turntables, intersections

### **Integration Tests**
- Full turn cycle: Planning → Movement → Battle → Resolution
- Multi-turn: Test 10+ turns with various strategies
- Edge cases: All units destroyed, network isolation, train collisions

### **Performance Tests**
- 20x20 map with 40 units: Measure frame time
- Pathfinding benchmark: 1000 random path queries
- Network system: Test with maximum relay chain length

---

**Last Updated**: January 2, 2026
**Version**: 1.0
