# **STATE MANAGEMENT**

**Godot 4.x | Flexible Turn State Machine | All Game Modes**

---

## **DESIGN PHILOSOPHY**

### **1. Unified State Machine**
- **Single state machine** for all game modes (hot seat, AI, online)
- **Mode flags** control behavior instead of separate machines
- **Minimal branching** - core logic is identical across modes

### **2. Mode Flags**
```gdscript
enum GameMode {
    HOT_SEAT,
    VS_AI,
    ONLINE
}

var game_mode: GameMode = HOT_SEAT
var show_battle_animations: bool = true
var allow_battle_skip: bool = true
```

### **3. Clear State Transitions**
- Every state has clear entry/exit actions
- Valid transitions are enforced
- Signals broadcast state changes for UI updates

---

## **TWO-LEVEL STATE HIERARCHY**

### **Meta-State Machine (Game Flow)**

High-level game state controlling menus, setup, and game over.

```gdscript
enum MetaState {
    MAIN_MENU,
    LEVEL_SELECT,
    OPTIONS,
    NARRATIVE_SCREEN,
    GAME_SETUP,
    IN_GAME,
    PAUSE_MENU,
    REVEAL_ORDERS,      # Show planned orders before resolution
    BATTLE_VIEW,
    VICTORY_SCREEN,
    DEFEAT_SCREEN,
    GAME_OVER
}
```

### **Game-State Machine (Turn Loop)**

Core turn sequence for actual gameplay.

```gdscript
enum TurnPhase {
    QUEUE_ORDERS_A,
    QUEUE_ORDERS_B,
    IRON_CURTAIN,
    REVEAL_ORDERS,       # Both players see planned moves
    CALCULATE_MOVEMENT,
    CALCULATE_BATTLES,
    PLAY_BATTLE_ANIMS,
    RESOLVE_BATTLES,
    CLEANUP,
    CHECK_WIN_CONDITION,
    RESET_TURN
}
```

---

## **META-STATE TRANSITIONS**

```
MAIN_MENU
    ├─→ LEVEL_SELECT
    ├─→ OPTIONS
    └─→ GAME_SETUP

LEVEL_SELECT
    ├─→ NARRATIVE_SCREEN
    └─→ GAME_SETUP

NARRATIVE_SCREEN
    └─→ GAME_SETUP

GAME_SETUP
    └─→ IN_GAME

IN_GAME
    ├─→ PAUSE_MENU
    ├─→ REVEAL_ORDERS (hot seat only)
    ├─→ BATTLE_VIEW
    ├─→ VICTORY_SCREEN
    └─→ DEFEAT_SCREEN

REVEAL_ORDERS
    └─→ IN_GAME (begin calculations)

BATTLE_VIEW
    └─→ IN_GAME

PAUSE_MENU
    ├─→ IN_GAME
    ├─→ GAME_SETUP
    └─→ MAIN_MENU

VICTORY_SCREEN
    ├─→ MAIN_MENU
    └─→ GAME_SETUP

DEFEAT_SCREEN
    ├─→ MAIN_MENU
    └─→ GAME_SETUP

GAME_OVER
    └─→ MAIN_MENU
```

---

## **TURN PHASE TRANSITIONS**

### **Hot Seat Mode**

```
QUEUE_ORDERS_A
    └─→ IRON_CURTAIN (Player A clicks "Ready")

IRON_CURTAIN
    └─→ QUEUE_ORDERS_B (Player B clicks "I'm Ready")

QUEUE_ORDERS_B
    └─→ REVEAL_ORDERS (Player B clicks "Ready")

REVEAL_ORDERS
    └─→ CALCULATE_MOVEMENT (Both players click "Begin Turn")

CALCULATE_MOVEMENT
    └─→ CALCULATE_BATTLES

CALCULATE_BATTLES
    └─→ PLAY_BATTLE_ANIMS

PLAY_BATTLE_ANIMS
    └─→ RESOLVE_BATTLES

RESOLVE_BATTLES
    └─→ CLEANUP

CLEANUP
    └─→ CHECK_WIN_CONDITION

CHECK_WIN_CONDITION
    ├─→ RESET_TURN (continue playing)
    ├─→ VICTORY_SCREEN (player wins)
    └─→ DEFEAT_SCREEN (player loses)

RESET_TURN
    └─→ QUEUE_ORDERS_A (back to Player A)
```

### **AI Mode**

```
QUEUE_ORDERS_A
    └─→ CALCULATE_MOVEMENT (AI generates orders automatically)

CALCULATE_MOVEMENT
    └─→ CALCULATE_BATTLES
    ...
    (same as hot seat from here)
```

### **Online Mode**

```
QUEUE_ORDERS_A
    └─→ CALCULATE_MOVEMENT (both players signal "ready")

CALCULATE_MOVEMENT
    └─→ CALCULATE_BATTLES
    ...
    (same as hot seat from here)
```

---

## **DETAILED STATE DEFINITIONS**

### **1. QUEUE_ORDERS_A / QUEUE_ORDERS_B**

**Purpose**: Players plan their moves and attacks for the turn.

**Entry Actions**:
- Enable UI for current player only
- Clear previous orders (if any)
- Show movement range indicators for active units
- Show attack range indicators for selected units
- Reset movement counter to 5
- Reset attack counter to 1
- Display: "Player X's Turn - Plan Your Orders"

**Valid Actions**:
- **Select Unit**: Click on your unit to select it
- **Issue Movement Order**: Click valid destination tile (consumes 1 movement order)
- **Issue Attack Order**: Click valid target (consumes 1 attack order)
- **Special Actions**:
  - Sabotage: Destroy terrain on current tile
  - Pack/Deploy: Toggle heavy unit state
  - Embark/Disembark: Board or exit transport
- **Cancel Order**: Remove last order or specific order
- **Clear Orders**: Remove all planned orders
- **Ready**: Submit orders and end planning phase

**UI Elements**:
- Movement counter: "Movements: X/5"
- Attack counter: "Attacks: 0/1"
- Selected unit info panel
- Order list showing current plans
- Ready button (enabled when orders valid)
- Clear orders button

**Exit Actions**:
- Validate all orders:
  - Movement orders ≤ 5
  - Attack orders ≤ 1
  - All source units are active
  - All target tiles are valid
- Store orders in OrderManager
- Disable UI input
- Emit `player_ready` signal

**Exit Condition**:
- Player clicks "Ready" button
- All orders are valid

---

### **2. IRON_CURTAIN** (Hot Seat Only)

**Purpose**: Hide Player A's orders from Player B to prevent information leakage.

**Entry Actions**:
- Hide all ghost units (movement indicators)
- Hide all attack target markers
- Hide order list display
- Show curtain overlay (full screen)
- Display text: "Pass device to Player B"
- Display subtitle: "Player B click 'I'm Ready' when ready"

**Valid Actions**:
- **I'm Ready**: Player B (after receiving device) clicks button

**UI Elements**:
- Curtain overlay (semi-transparent or solid)
- "Pass device to Player B" label (large, centered)
- "I'm Ready" button (centered, large)
- Optional: Background animation (subtle)

**Exit Actions**:
- Hide curtain overlay
- Clear Player A's order display
- Enable UI for Player B
- Reset movement/attack counters for Player B
- Display: "Player B's Turn - Plan Your Orders"

**Exit Condition**:
- Player B clicks "I'm Ready" button

---

### **3. REVEAL_ORDERS** (Hot Seat Only)

**Purpose**: Both players see each other's planned moves and battles before calculations begin.

**Entry Actions**:
- Show all planned movement orders as ghost units (semi-transparent)
- Show all attack orders with target markers (red circles/lines)
- Display both players' order lists side-by-side:
  - Left side: Player A's orders
  - Right side: Player B's orders
- Show battle preview (if any attacks target each other's units)
- Display: "Review Orders - Both Players"
- Show "Begin Turn" button (initially disabled)

**Valid Actions**:
- **Begin Turn**: Both players review and click to start calculations

**UI Elements**:
- Ghost units showing planned movements
- Attack lines showing planned attacks
- Player A order list (left panel)
- Player B order list (right panel)
- Battle preview panel (center, showing attacker/defender pairs)
- "Begin Turn" button (large, centered, initially disabled)
- Turn counter: "Turn X"

**Visual Feedback**:
- Ghost units: Semi-transparent color of player
- Attack lines: Red arrows from attacker to target
- Overlapping movements: Yellow warning on collision tiles
- Battle preview: "X battles this turn"

**Exit Actions**:
- Remove ghost units
- Hide attack lines
- Hide order lists
- Disable UI input
- Emit `review_complete` signal

**Exit Condition**:
- Both players click "Begin Turn" button (or single click in AI/online mode)

---

### **4. CALCULATE_MOVEMENT**

**Purpose**: Process all movement orders and resolve collisions.

**Entry Actions**:
- Lock all input (no player interaction)
- Show "Processing Movement..." indicator (small, unobtrusive)
- Load all movement orders from OrderManager

**Process**:
1. **Order Validation**:
   - Check if source unit is still active
   - Check if source unit hasn't been destroyed
   - Validate path is passable
   - Validate target tile is not blocked

2. **Movement Queue Building**:
   - Create list of all movements
   - Sort by unit (to process all movements together)

3. **Collision Detection**:
   - Identify overlapping movements:
     - Two units targeting same tile
     - Units passing through each other's paths
   - Apply collision rules:
     - Standard: Both units stop one tile before collision
     - Train vs. Unit: Unit stops one tile early, train continues
     - Train vs. Train: Both colliding cars destroyed

4. **Squish Rule** (Train Collisions):
   - If unit is still in train path after stopping early
   - Force unit to retreat 1 tile away from train
   - If no valid retreat tile: Destroy unit

5. **Apply Movements**:
   - Update unit.grid_position for all units
   - Emit `unit.moved(from, to)` signals
   - Update embark/disembark states:
     - If unit enters carrier tile: Set `unit.carrier_unit`
     - If unit exits carrier: Clear `unit.carrier_unit`
   - Update train car positions (if train engine moves)

**Exit Actions**:
- Clear movement order lists
- Hide processing indicator
- Emit `movement_complete` signal

**Exit Condition**:
- All movements processed and applied
- All collisions resolved
- All squish rules applied

---

### **5. CALCULATE_BATTLES**

**Purpose**: Identify all battles and calculate outcomes.

**Entry Actions**:
- Show "Calculating Battles..." indicator

**Process**:
1. **Collect Attack Orders**:
   - Load all attack orders from OrderManager
   - Load all attack targets

2. **Battle Identification**:
   For each attack order:
   - Identify target tile position
   - Find unit on target tile (if any)
   - Find all units in range of target tile:
     - Check each unit's range stat
     - Check path to target (no obstacles/enemies blocking)
     - Mark as participant if valid

3. **Battle Grouping**:
   - If multiple attacks involve same units, group them
   - Each group is evaluated separately

4. **Outcome Calculation**:
   For each battle:
   - Sum attack values: All attacking units' attack stats
   - Sum defense values: All defending units' defense + terrain bonus
   - Apply terrain bonuses (stations, plazas)
   - Apply special abilities:
     - Cavalry charge: +3 attack if adjacent
     - Road bonus: Extra move (doesn't affect battle)
   - Determine outcome:
     - Defense ≥ Attack: Attack fails
     - Attack > Defense by 1: Forced retreat
     - Attack > Defense by >1: Destroy unit
   - Store battle data for animation phase:
     - Attackers (list of units)
     - Defenders (list of units)
     - Total attack value
     - Total defense value
     - Outcome (fail/retreat/destroy)

5. **Simultaneity Handling**:
   - All battles evaluated before any effects applied
   - Units destroyed this turn can still participate in other battles

**Exit Actions**:
- Prepare battle list for animation
- Hide processing indicator
- Emit `battle_calculation_complete` signal

**Exit Condition**:
- All attacks evaluated
- All outcomes determined
- Battle data stored for animation

---

### **6. PLAY_BATTLE_ANIMS**

**Purpose**: Visualize battles with optional individual skip.

**Entry Actions**:
- Check `show_battle_animations` setting
- If disabled: Skip to RESOLVE_BATTLES
- If enabled: Load first battle data

**Process**:
For each battle in battle list:
1. **Instantiate Battle Scene**:
   - Load `scenes/game/battle_scene.tscn`
   - Add as child of game board
   - Pass battle data

2. **Battle Scene Display**:
   - Left side: Attacker sprites (profile view)
   - Right side: Defender sprites (profile view)
   - Top panel: "Attack: X | Defense: Y"
   - Center: Battle outcome text
   - Bottom: "Continue" button / "Skip" button

3. **Animation Playback**:
   - If `allow_battle_skip` is true:
     - Show "Skip Battle" button
     - Player can skip to next battle
   - Play attack animation:
     - Flash attacker
     - Show projectile (if ranged)
     - Impact effect on defender
   - Show outcome:
     - "Attack Failed" (red)
     - "Forced Retreat" (yellow)
     - "Unit Destroyed" (red, explosion effect)

4. **User Interaction**:
   - Player clicks "Continue" → Show next battle
   - Player clicks "Skip" → Skip remaining battles, go to RESOLVE_BATTLES

5. **Cleanup**:
   - On `battle_finished` signal:
     - Remove battle scene
     - Load next battle OR finish phase

**Valid Actions**:
- **Continue**: Proceed to next battle
- **Skip Battle**: Skip current battle (if enabled)
- **Skip All**: Skip remaining battles (if enabled)

**UI Elements**:
- Battle scene overlay
- "Continue" button
- "Skip Battle" button (if `allow_battle_skip`)
- "Skip All" button (if `allow_battle_skip`)
- Battle counter: "Battle X of Y"

**Exit Actions**:
- Clear battle list
- Hide battle scene
- Emit `battle_animations_complete` signal

**Exit Condition**:
- All battles animated OR skipped
- Player clicked "Continue" on last battle
- Player clicked "Skip All"

---

### **7. RESOLVE_BATTLES**

**Purpose**: Apply battle effects (destruction, retreat, damage).

**Entry Actions**:
- Show "Resolving Battles..." indicator
- Load all battle outcomes

**Process**:
1. **Mark Units for Destruction/Retreat**:
   - Don't remove immediately (must be simultaneous)
   - Create list of units to destroy
   - Create list of units to retreat

2. **Apply Destructions**:
   For each unit to destroy:
   - Remove unit from board
   - Emit `unit.destroyed()` signal
   - If unit is carrying others:
     - Mark carried units for destruction too
   - If unit is train engine:
     - Detach all train cars
     - Cars become independent (destroyed or stranded)
   - If unit is train car:
     - Detach from engine

3. **Apply Retreats**:
   For each unit to retreat:
   - Find valid retreat tiles:
     - 1 tile away from battle
     - Not towards enemy units
     - Passable terrain
   - If valid tile found:
     - Move unit to retreat tile
     - Emit `unit.retreated()` signal
   - If no valid tile:
     - Destroy unit (fallback)
     - Emit `unit.destroyed()` signal

4. **Simultaneous Application**:
   - All destructions/retreats applied in batch
   - No unit removal mid-iteration
   - Ensure fairness

**Exit Actions**:
- Clear battle data
- Hide processing indicator
- Emit `battle_resolution_complete` signal

**Exit Condition**:
- All destructions applied
- All retreats applied
- All train detachments handled

---

### **8. CLEANUP**

**Purpose**: Handle map-specific effects and cascading systems.

**Entry Actions**:
- Show "Processing Cleanup..." indicator

**Process**:
1. **Railway System**:
   - Check for newly captured stations:
     - Find stations with changed ownership
   - Perform BFS cascade:
     - From captured station, traverse connected rail
     - Flip ownership of all connected stations
     - Stop at blocked connections (enemy units)
   - Update station ownership in tile data
   - Emit `station_captured(station_pos, owner_id)` signals

2. **Telegraph System**:
   - Identify newly controlled telegraph offices
   - Propagate telegraph chain:
     - Activate connected telegraph offices
     - Update network system
   - Recalculate active states for relay units

3. **Network System**:
   - Recalculate which units are "active":
     - Find all source units (stations, arsenals)
     - Trace lines of effect
     - Mark units as active/inactive
     - Cache results for performance
   - Update visual indicators:
     - Dim inactive units
     - Highlight active units

4. **Map-Specific Effects**:
   - Trigger scripted events (if any)
   - Check for objective progress
   - Apply temporary terrain effects

5. **Tile State Cleanup**:
   - Remove destroyed infrastructure:
     - Convert sabotaged rails to passable terrain
     - Remove telegraph lines
   - Reset temporary terrain effects

6. **Unit State Cleanup**:
   - Update embark/disemback states
   - Clear temporary status effects
   - Reset heavy unit deploy/pack flags (if moved)

**Exit Actions**:
- Hide processing indicator
- Emit `cleanup_complete` signal

**Exit Condition**:
- All cascading systems updated
- All map effects processed
- All network states recalculated

---

### **9. CHECK_WIN_CONDITION**

**Purpose**: Determine if game should end.

**Entry Actions**:
- Load victory condition from MapData
- Check current game state

**Process**:
1. **Count Units Per Player**:
   - Red Army: Count all owner_id == 0 units
   - White Army: Count all owner_id == 1 units

2. **Check Victory Types**:

   **Annihilation**:
   - Red Army wins: White Army has 0 units
   - White Army wins: Red Army has 0 units

   **Capture Station**:
   - Check victory_target station ownership
   - If Player X controls target → Player X wins

   **Destroy Balloon**:
   - Check if enemy observation balloon is destroyed
   - If destroyed → Attacker wins

3. **Check Defeat Conditions**:
   - Current player has no units that can move
   - Current player has no way to achieve victory
   - (Implementation decision: auto-defeat or continue?)

4. **Determine Outcome**:
   - If victory detected:
     - Emit `game.victory(winner_id, victory_type)`
     - Transition to VICTORY_SCREEN
   - If defeat detected:
     - Emit `game.defeat(loser_id, defeat_reason)`
     - Transition to DEFEAT_SCREEN
   - If no winner:
     - Continue to RESET_TURN

**Exit Actions**:
- If ending: Prepare victory/defeat data
- If continuing: No special actions

**Exit Condition**:
- Victory/defeat detected → End game
- No winner yet → Continue turn

---

### **10. RESET_TURN**

**Purpose**: Prepare for next turn.

**Entry Actions**:
- Show brief "Next Turn" animation (1 second)
- Increment turn counter

**Process**:
1. **Clear Orders**:
   - Clear all movement order lists
   - Clear all attack order lists
   - Reset order counters

2. **Switch Current Player**:
   - Player A (0) → Player B (1)
   - Player B (1) → Player A (0)
   - Emit `turn_changed(new_player_id)` signal

3. **Update UI**:
   - Display: "Player X's Turn"
   - Update turn counter display
   - Reset movement/attack counters to 5/1

4. **Reset Unit States** (if needed):
   - Clear temporary status effects
   - Reset cooldowns (if any)

**Exit Actions**:
- Hide "Next Turn" animation
- Emit `turn_reset_complete` signal
- Transition to QUEUE_ORDERS for new current player

**Exit Condition**:
- Turn counter incremented
- Current player updated
- All orders cleared
- Ready for next planning phase

---

## **TRANSITION VALIDATION**

```gdscript
# In turn_state_machine.gd

func can_transition(from: TurnPhase, to: TurnPhase) -> bool:
    var valid_transitions: Dictionary = {
        TurnPhase.QUEUE_ORDERS_A: [
            TurnPhase.IRON_CURTAIN,
            TurnPhase.CALCULATE_MOVEMENT  # AI/online
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
    
    var valid: Array = valid_transitions.get(from, [])
    if to in valid:
        return true
    
    push_error("Invalid transition from %s to %s" % [from, to])
    return false

func transition(to: TurnPhase) -> void:
    if not can_transition(current_phase, to):
        return
    
    var old_phase: TurnPhase = current_phase
    current_phase = to
    
    emit_signal("state_changed", old_phase, to)
    emit_signal("phase_started", to)
    
    _on_phase_enter(to)

func _on_phase_enter(phase: TurnPhase) -> void:
    match phase:
        TurnPhase.QUEUE_ORDERS_A:
            _enter_queue_orders_a()
        TurnPhase.IRON_CURTAIN:
            _enter_iron_curtain()
        TurnPhase.REVEAL_ORDERS:
            _enter_reveal_orders()
        # ... etc
```

---

## **GAME MODE HANDLING**

```gdscript
# In turn_state_machine.gd

var game_mode: GameMode = HOT_SEAT
var current_player: int = 0  # 0 = Red, 1 = White

func on_player_ready(player_id: int) -> void:
    match game_mode:
        HOT_SEAT:
            _handle_hot_seat_ready(player_id)
        VS_AI:
            _handle_ai_ready(player_id)
        ONLINE:
            _handle_online_ready(player_id)

func _handle_hot_seat_ready(player_id: int) -> void:
    if player_id == 0:
        # Player A ready
        transition(TurnPhase.IRON_CURTAIN)
    elif player_id == 1:
        # Player B ready
        transition(TurnPhase.REVEAL_ORDERS)

func _handle_ai_ready(player_id: int) -> void:
    # Player A (human) ready
    if player_id == 0:
        # Generate AI orders for Player B
        generate_ai_orders()
        # Skip IRON_CURTAIN and REVEAL_ORDERS
        transition(TurnPhase.CALCULATE_MOVEMENT)

func _handle_online_ready(player_id: int) -> void:
    # Track which players are ready
    player_ready_status[player_id] = true
    
    # When both ready, proceed
    if player_ready_status[0] and player_ready_status[1]:
        transition(TurnPhase.CALCULATE_MOVEMENT)

func generate_ai_orders() -> void:
    # Delegate to AI system
    ai_manager.generate_orders_for_player(1)
```

---

## **SIGNAL INTEGRATION**

### **State Machine Signals**

```gdscript
# Core state signals
signal state_changed(from: TurnPhase, to: TurnPhase)
signal phase_started(phase: TurnPhase)
signal phase_ended(phase: TurnPhase)

# Turn signals
signal turn_started(player_id: int, turn_number: int)
signal turn_changed(player_id: int)
signal turn_ended(player_id: int)

# Phase-specific signals
signal player_ready(player_id: int)
signal orders_revealed()
signal movement_complete()
signal battles_calculated()
signal battles_resolved()
signal cleanup_complete()
```

### **Connecting to Game Board**

```gdscript
# In game_board.gd

func _ready():
    turn_state_machine.state_changed.connect(_on_state_changed)
    turn_state_machine.turn_started.connect(_on_turn_started)
    turn_state_machine.turn_changed.connect(_on_turn_changed)
    
    turn_state_machine.player_ready.connect(_on_player_ready)
    turn_state_machine.orders_revealed.connect(_on_orders_revealed)

func _on_state_changed(from: TurnPhase, to: TurnPhase) -> void:
    match to:
        TurnPhase.QUEUE_ORDERS_A, TurnPhase.QUEUE_ORDERS_B:
            enable_player_ui(current_player)
        TurnPhase.IRON_CURTAIN:
            show_curtain()
        TurnPhase.REVEAL_ORDERS:
            show_reveal_orders()
        TurnPhase.CALCULATE_MOVEMENT, TurnPhase.CALCULATE_BATTLES:
            disable_all_ui()
        TurnPhase.PLAY_BATTLE_ANIMS:
            # Battle scene handles its own UI
            pass
        TurnPhase.RESOLVE_BATTLES, TurnPhase.CLEANUP:
            disable_all_ui()
        TurnPhase.RESET_TURN:
            show_next_turn_animation()

func _on_turn_started(player_id: int, turn_number: int) -> void:
    ui_manager.update_turn_display(player_id, turn_number)

func _on_turn_changed(player_id: int) -> void:
    ui_manager.update_current_player(player_id)
```

---

## **ORDER DATA STRUCTURES**

```gdscript
# In order_manager.gd

class MovementOrder extends RefCounted:
    var unit_id: int
    var source_pos: Vector2i
    var target_pos: Vector2i
    var path: Array[Vector2i]  # Calculated path for collision detection
    
    func _init(p_unit_id: int, p_source: Vector2i, p_target: Vector2i):
        unit_id = p_unit_id
        source_pos = p_source
        target_pos = p_target
        path = []

class AttackOrder extends RefCounted:
    var attacker_id: int
    var attacker_pos: Vector2i
    var target_pos: Vector2i
    var target_unit_id: int  # May be 0 if targeting empty tile
    
    func _init(p_attacker_id: int, p_attacker_pos: Vector2i, p_target: Vector2i, p_target_unit_id: int = 0):
        attacker_id = p_attacker_id
        attacker_pos = p_attacker_pos
        target_pos = p_target
        target_unit_id = p_target_unit_id

class BattleData extends RefCounted:
    var attackers: Array[int]  # Unit IDs
    var defenders: Array[int]  # Unit IDs
    var target_pos: Vector2i
    var total_attack: int
    var total_defense: int
    var outcome: String  # "fail", "retreat", "destroy"
    
    func _init():
        attackers = []
        defenders = []
        target_pos = Vector2i.ZERO
        total_attack = 0
        total_defense = 0
        outcome = "fail"
```

---

## **PLAYER SETTINGS**

```gdscript
# Stored in player preferences

var battle_settings: Dictionary = {
    "show_battle_animations": true,
    "allow_individual_battle_skip": true,
    "allow_skip_all_battles": true,
    "battle_animation_speed": 1.0,  # 0.5 = fast, 1.0 = normal, 2.0 = slow
    "show_processing_indicators": true,
    "curtain_manual_only": true  # No timer, just "I'm Ready"
}

var game_settings: Dictionary = {
    "default_game_mode": HOT_SEAT,
    "show_tutorial": true,
    "difficulty": "normal"  # "easy", "normal", "hard" (for AI)
}
```

---

## **DEBUGGING STATE MACHINE**

```gdscript
# In debug_visualizer.gd

func visualize_state_transitions() -> void:
    # Show current state in corner
    var state_label: Label = Label.new()
    state_label.text = "Phase: %s | Player: %d" % [
        TurnPhase.keys()[current_phase],
        current_player
    ]
    add_child(state_label)

func visualize_orders() -> void:
    # Show movement arrows
    # Show attack targets
    # Display order counts

func visualize_network() -> void:
    # Draw lines of effect
    # Highlight active units
    # Mark source units
```

---

## **STATE MACHINE IMPLEMENTATION PATTERN**

```gdscript
# scripts/game/turn_state_machine.gd
extends Node

signal state_changed(from: TurnPhase, to: TurnPhase)
signal turn_started(player_id: int, turn_number: int)
signal turn_changed(player_id: int)

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

var current_phase: TurnPhase = TurnPhase.QUEUE_ORDERS_A
var current_player: int = 0
var turn_number: int = 1
var game_mode: GameMode = HOT_SEAT

var order_manager: OrderManager
var collision_system: CollisionSystem
var battle_system: BattleSystem
var railway_system: RailwaySystem
var network_system: NetworkSystem

func _ready() -> void:
    # Get references to other systems
    order_manager = $OrderManager
    collision_system = $CollisionSystem
    battle_system = $BattleSystem
    railway_system = $RailwaySystem
    network_system = $NetworkSystem

func start_game(mode: GameMode) -> void:
    game_mode = mode
    current_player = 0
    turn_number = 1
    transition(TurnPhase.QUEUE_ORDERS_A)

func transition(to: TurnPhase) -> void:
    if not can_transition(current_phase, to):
        return
    
    var old_phase: TurnPhase = current_phase
    current_phase = to
    
    emit_signal("state_changed", old_phase, to)
    _on_phase_enter(to)

func _on_phase_enter(phase: TurnPhase) -> void:
    match phase:
        TurnPhase.QUEUE_ORDERS_A, TurnPhase.QUEUE_ORDERS_B:
            _enter_queue_orders()
        TurnPhase.IRON_CURTAIN:
            _enter_iron_curtain()
        TurnPhase.REVEAL_ORDERS:
            _enter_reveal_orders()
        TurnPhase.CALCULATE_MOVEMENT:
            _enter_calculate_movement()
        TurnPhase.CALCULATE_BATTLES:
            _enter_calculate_battles()
        TurnPhase.PLAY_BATTLE_ANIMS:
            _enter_play_battle_anims()
        TurnPhase.RESOLVE_BATTLES:
            _enter_resolve_battles()
        TurnPhase.CLEANUP:
            _enter_cleanup()
        TurnPhase.CHECK_WIN_CONDITION:
            _enter_check_win_condition()
        TurnPhase.RESET_TURN:
            _enter_reset_turn()

# ... Phase implementation methods ...

func on_player_ready() -> void:
    match current_phase:
        TurnPhase.QUEUE_ORDERS_A:
            if game_mode == HOT_SEAT:
                transition(TurnPhase.IRON_CURTAIN)
            else:
                # AI or online
                _handle_non_hot_seat_ready()
        TurnPhase.QUEUE_ORDERS_B:
            transition(TurnPhase.REVEAL_ORDERS)
        TurnPhase.REVEAL_ORDERS:
            transition(TurnPhase.CALCULATE_MOVEMENT)
```

---

**Last Updated**: January 2, 2026
**Version**: 1.0
