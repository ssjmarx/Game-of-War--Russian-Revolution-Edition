# **SCENE HIERARCHY**

**Godot 4.x | Node-by-Node Breakdown | Signal Connections**

---

## **DESIGN PHILOSOPHY**

### **1. Scene-Script Parity**
Each `.tscn` has a corresponding `.gd` with the same name
- Easy navigation and understanding
- Clear responsibility boundaries

### **2. Minimal Depth**
Keep scene trees shallow (3-5 levels max) when possible
- Deep hierarchies make debugging difficult
- Shallow trees are easier to reason about

### **3. Node Naming**
Use descriptive, camelCase node names:
- `TopPanel` (not `HBoxContainer`)
- `ReadyButton` (not `Button`)
- Makes scene tree readable in editor

### **4. Signal-Based Communication**
Never directly access sibling or parent nodes
- Use signals to communicate
- Maintain loose coupling

---

## **MAIN MENU SCENE**

**File**: `scenes/ui/main_menu.tscn`
**Script**: `scripts/ui/main_menu.gd`

### **Node Hierarchy**

```
MainMenu (Control)
├── Background (TextureRect)
├── ContentContainer (CenterContainer)
│   ├── VBoxContainer
│   │   ├── GameTitle (Label)
│   │   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   │   ├── ButtonContainer (VBoxContainer)
│   │   │   ├── StartCampaignButton (Button)
│   │   │   ├── QuickBattleButton (Button)
│   │   │   ├── LevelSelectButton (Button)
│   │   │   ├── OptionsButton (Button)
│   │   │   └── ExitButton (Button)
│   │   └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── VersionLabel (Label)
└── AudioManager (Node)
    └── MenuMusicPlayer (AudioStreamPlayer)
```

### **Node Descriptions**

#### **MainMenu (Control - Root)**
- **Purpose**: Root container for entire scene
- **Type**: Control (UI scene)
- **Script**: `MainMenu.gd`
- **Signals**:
  - `start_campaign_clicked()`
  - `quick_battle_clicked()`
  - `level_select_clicked()`
  - `options_clicked()`
  - `exit_clicked()`

#### **Background (TextureRect)**
- **Purpose**: Display background image (Phase 2)
- **Type**: TextureRect
- **Phase 1**: Solid color or simple pattern
- **Phase 2**: Industrial revolution imagery, rotating gears

#### **ContentContainer (CenterContainer)**
- **Purpose**: Center all content vertically and horizontally
- **Type**: CenterContainer
- **Anchors**: All corners anchored (full screen)

#### **GameTitle (Label)**
- **Purpose**: Display game title
- **Type**: Label
- **Text**: "GAME OF WAR: RUSSIAN REVOLUTION EDITION"
- **Font**: Large serif (Phase 2: period font)

#### **ButtonContainer (VBoxContainer)**
- **Purpose**: Vertically stack menu buttons
- **Type**: VBoxContainer
- **Separation**: 8px between buttons

#### **StartCampaignButton (Button)**
- **Purpose**: Navigate to campaign start
- **Type**: Button
- **Signal**: `pressed()` → `MainMenu.start_campaign_clicked()`

#### **QuickBattleButton (Button)**
- **Purpose**: Navigate to quick battle setup
- **Type**: Button
- **Signal**: `pressed()` → `MainMenu.quick_battle_clicked()`

#### **LevelSelectButton (Button)**
- **Purpose**: Navigate to level selection
- **Type**: Button
- **Signal**: `pressed()` → `MainMenu.level_select_clicked()`

#### **OptionsButton (Button)**
- **Purpose**: Navigate to options menu
- **Type**: Button
- **Signal**: `pressed()` → `MainMenu.options_clicked()`

#### **ExitButton (Button)**
- **Purpose**: Exit game
- **Type**: Button
- **Signal**: `pressed()` → `MainMenu.exit_clicked()`

#### **VersionLabel (Label)**
- **Purpose**: Display version number
- **Type**: Label
- **Text**: "v1.0.0"

#### **MenuMusicPlayer (AudioStreamPlayer)**
- **Purpose**: Play menu background music
- **Type**: AudioStreamPlayer
- **Stream**: Menu music track
- **Autoplay**: True

---

## **LEVEL SELECT SCENE**

**File**: `scenes/ui/level_select.tscn`
**Script**: `scripts/ui/level_select.gd`

### **Node Hierarchy**

```
LevelSelect (Control)
├── Background (TextureRect)
├── ContentContainer (MarginContainer)
│   ├── TopBar (HBoxContainer)
│   │   ├── BackButton (Button)
│   │   └── TitleLabel (Label)
│   └── Spacer (Control)
│   └── ScrollContainer
│       └── MapList (VBoxContainer)
│           ├── MapCard1 (Panel)
│           │   ├── MapName (Label)
│           │   ├── MapDescription (Label)
│           │   └── MapThumbnail (TextureRect) - Phase 2 only
│           ├── MapCard2 (Panel)
│           │   └── ... (same structure)
│           └── MapCard3 (Panel)
│               └── ... (same structure)
├── BottomBar (HBoxContainer)
│   ├── Spacer (Control)
│   ├── StartButton (Button)
│   └── Spacer (Control)
└── AudioManager (Node)
    └── MenuMusicPlayer (AudioStreamPlayer)
```

### **Node Descriptions**

#### **LevelSelect (Control - Root)**
- **Purpose**: Root container for level selection
- **Type**: Control
- **Script**: `LevelSelect.gd`
- **Signals**:
  - `map_selected(map_id: String)`
  - `start_clicked()`
  - `back_clicked()`

#### **MapCard (Panel)**
- **Purpose**: Display single map information
- **Type**: Panel (Control with border)
- **Purpose**: Clickable container for map selection
- **Signal**: `gui_input()` → Detect click on entire card

#### **MapName (Label)**
- **Purpose**: Display map name
- **Type**: Label
- **Font**: Bold, large

#### **MapDescription (Label)**
- **Purpose**: Display brief map description
- **Type**: Label
- **Font**: Normal, smaller
- **Autowrap Mode**: Word Smart

#### **MapThumbnail (TextureRect)**
- **Purpose**: Display map preview (Phase 2)
- **Type**: TextureRect
- **Phase 1**: Empty or placeholder
- **Phase 2**: Actual map screenshot

#### **BackButton (Button)**
- **Purpose**: Return to main menu
- **Type**: Button
- **Signal**: `pressed()` → `LevelSelect.back_clicked()`

#### **StartButton (Button)**
- **Purpose**: Start selected map
- **Type**: Button
- **Enabled**: Only when map selected
- **Signal**: `pressed()` → `LevelSelect.start_clicked()`

---

## **OPTIONS SCENE**

**File**: `scenes/ui/options.tscn`
**Script**: `scripts/ui/options.gd`

### **Node Hierarchy (Phase 1 - Basic)**

```
Options (Control)
├── Background (TextureRect)
├── ContentContainer (MarginContainer)
│   ├── TitleLabel (Label)
│   ├── ScrollContainer
│   │   └── OptionsList (VBoxContainer)
│   │       ├── OptionItem1 (HBoxContainer)
│   │       │   ├── OptionLabel (Label)
│   │       │   └── OptionControl (Button/CheckBox/Slider)
│   │       ├── OptionItem2 (HBoxContainer)
│   │       │   └── ... (same structure)
│   │       └── OptionItem3 (HBoxContainer)
│   │           └── ... (same structure)
│   └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── BackButton (Button)
└── AudioManager (Node)
    └── MenuMusicPlayer (AudioStreamPlayer)
```

### **Node Hierarchy (Phase 2 - Train Board Theme)**

```
Options (Control)
├── Background (TextureRect) - Train board texture
├── ContentContainer (MarginContainer)
│   ├── TitleLabel (Label) - Flip-letter style
│   ├── FlipLetterContainer (HBoxContainer)
│   │   └── OptionTitle (Label) - Animated flip letters
│   ├── ScrollContainer
│   │   └── OptionsList (VBoxContainer)
│   │       ├── OptionItem1 (HBoxContainer)
│   │       │   ├── OptionLabel (Label)
│   │       │   └── OptionControl (CheckBox/Radio/Slider)
│   │       │       └── FlipAnimation (Control) - Phase 2 only
│   │       └── ...
│   └── Spacer (Control)
│   └── BackButton (Button) - With flip letter animation
└── AudioManager (Node)
    └── MenuMusicPlayer (AudioStreamPlayer)
```

### **Node Descriptions**

#### **Options (Control - Root)**
- **Purpose**: Root container for options menu
- **Type**: Control
- **Script**: `Options.gd`
- **Signals**:
  - `option_changed(option_name: String, value: Variant)`
  - `back_clicked()`

#### **OptionItem (HBoxContainer)**
- **Purpose**: Container for single option
- **Type**: HBoxContainer
- **Layout**: Label on left, control on right
- **Separation**: 16px between label and control

#### **OptionControl (Button/CheckBox/Slider)**
- **Purpose**: Control for setting value
- **Types**:
  - **CheckBox**: Boolean options (show animations, turn timer)
  - **RadioButton**: Mutually exclusive options (animation speed)
  - **HSlider**: Volume controls, timer duration
- **Signal**: 
  - CheckBox: `toggled(button_pressed)`
  - RadioButton: `pressed()`
  - HSlider: `value_changed(value)`

#### **FlipAnimation (Control)**
- **Purpose**: Animate letter flip effect (Phase 2)
- **Type**: Control
- **Content**: Label with flip animation
- **Tween**: Scale X from 1.0 → 0.0 → 1.0 (0.2s total)
- **Sound**: Card flip sound on flip point

---

## **NARRATIVE SCREEN SCENE**

**File**: `scenes/ui/narrative_screen.tscn`
**Script**: `scripts/ui/narrative_screen.gd`

### **Node Hierarchy (Phase 1 - Basic)**

```
NarrativeScreen (Control)
├── Background (TextureRect)
├── ContentContainer (MarginContainer)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── PortraitContainer (CenterContainer)
│   │   └── PortraitPlaceholder (TextureRect)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── TextBox (Panel)
│       ├── TextScroll (ScrollContainer)
│       │   └── NarrativeText (RichTextLabel)
│       └── ContinueButton (Button)
└── AudioManager (Node)
    ├── TypewriterSFX (AudioStreamPlayer)
    └── NarrativeMusicPlayer (AudioStreamPlayer)
```

### **Node Hierarchy (Phase 2 - Visual Novel)**

```
NarrativeScreen (Control)
├── Background (TextureRect)
├── ContentContainer (MarginContainer)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── PortraitContainer (CenterContainer)
│   │   ├── PortraitSprite (Sprite2D) - 2D character portrait
│   │   ├── PortraitAnimator (AnimationPlayer) - Breathing, blinking
│   │   └── CutawayOverlay (TextureRect) - Shows cutaway images
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── TextBox (Panel)
│       ├── DispatchHeader (Label) - "DISPATCH FROM FRONT LINE"
│       ├── TextScroll (ScrollContainer)
│       │   └── NarrativeText (RichTextLabel)
│       │       └── TypingCursor (Label) - Blinking cursor
│       └── ContinueButton (Button)
└── AudioManager (Node)
    ├── TypewriterSFX (AudioStreamPlayer) - Key press, carrier return
    ├── NarrativeMusicPlayer (AudioStreamPlayer) - Background music
    └── VoiceoverPlayer (AudioStreamPlayer) - Optional voice
```

### **Node Descriptions**

#### **NarrativeScreen (Control - Root)**
- **Purpose**: Root container for narrative scene
- **Type**: Control
- **Script**: `NarrativeScreen.gd`
- **Signals**:
  - `continue_clicked()`
  - `text_finished_typing()`

#### **PortraitSprite (Sprite2D)**
- **Purpose**: Display 2D character portrait (Phase 2)
- **Type**: Sprite2D
- **Texture**: Character portrait image
- **Animation**: Breathing, blinking via AnimationPlayer

#### **PortraitAnimator (AnimationPlayer)**
- **Purpose**: Animate portrait (Phase 2)
- **Type**: AnimationPlayer
- **Animations**:
  - "breathing" (slow scale Y: 1.0 ↔ 1.05)
  - "blinking" (subtle texture change or overlay)

#### **CutawayOverlay (TextureRect)**
- **Purpose**: Display cutaway images (Phase 2)
- **Type**: TextureRect
- **Content**: Historical photos, battle maps, propaganda
- **Visibility**: Fades in/out based on narrative beat

#### **DispatchHeader (Label)**
- **Purpose**: Display dispatch header (Phase 2)
- **Type**: Label
- **Text**: "DISPATCH FROM FRONT LINE"
- **Font**: Typewriter/mono font

#### **NarrativeText (RichTextLabel)**
- **Purpose**: Display narrative text
- **Type**: RichTextLabel
- **Phase 1**: Text appears instantly
- **Phase 2**: Typewriter effect (character-by-character)
- **Autowrap Mode**: Word Smart
- **Fit Content**: True

#### **TypingCursor (Label)**
- **Purpose**: Blinking cursor during typewriter effect (Phase 2)
- **Type**: Label
- **Text**: "_"
- **Animation**: Blink (0.5s on, 0.5s off)

#### **TypewriterSFX (AudioStreamPlayer)**
- **Purpose**: Play typewriter sound effects (Phase 2)
- **Type**: AudioStreamPlayer
- **Streams**:
  - Key press: Short mechanical click
  - Carrier return: Mechanical slide sound
- **Playback**: Triggered by typing animation

---

## **GAME BOARD SCENE**

**File**: `scenes/game/game_board.tscn`
**Script**: `scripts/game/game_board.gd`

### **Node Hierarchy**

```
GameBoard (Control)
├── Background (TextureRect)
├── CameraContainer (Control)
│   └── Camera2D
│       └── CameraGizmo (Node2D) - Debug camera controls
├── MapContainer (Control)
│   ├── TileMap (TileMapLayer)
│   │   ├── BaseTerrainLayer (TileMapLayer)
│   │   ├── ObstacleLayer (TileMapLayer)
│   │   ├── InfrastructureLayer (TileMapLayer)
│   │   ├── BuildingLayer (TileMapLayer)
│   │   └── SpecialLayer (TileMapLayer)
│   └── UnitsContainer (Node2D)
│       ├── RedArmyUnits (Node2D)
│       │   ├── Unit1 (Unit scene instance)
│       │   ├── Unit2 (Unit scene instance)
│       │   └── ...
│       └── WhiteArmyUnits (Node2D)
│           ├── Unit1 (Unit scene instance)
│           ├── Unit2 (Unit scene instance)
│           └── ...
├── UIContainer (Control)
│   ├── TopHUD (HBoxContainer)
│   │   ├── TurnLabel (Label)
│   │   ├── Separator (Label - "|")
│   │   ├── PlayerLabel (Label)
│   │   ├── Separator (Label - "|")
│   │   └── PhaseLabel (Label)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── BottomHUD (HBoxContainer)
│       ├── MovementCounter (Panel)
│       │   ├── MovementLabel (Label)
│       │   └── MovementValue (Label)
│       ├── AttackCounter (Panel)
│       │   ├── AttackLabel (Label)
│       │   └── AttackValue (Label)
│       ├── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
│       ├── UndoButton (Button)
│       ├── SpecialActionsButton (Button)
│       └── ReadyButton (Button)
├── SpecialActionsMenu (Panel) - Hidden by default
│   ├── MenuTitle (Label)
│   ├── ActionList (VBoxContainer)
│   │   ├── SabotageAction (Button)
│   │   ├── PackAction (Button)
│   │   ├── DeployAction (Button)
│   │   ├── EmbarkAction (Button)
│   │   └── DisembarkAction (Button)
│   └── CloseButton (Button)
├── UnitInfoPanel (Panel) - Hidden by default
│   ├── PanelTitle (Label)
│   ├── UnitName (Label)
│   ├── StatsContainer (VBoxContainer)
│   │   ├── MoveStat (HBoxContainer)
│   │   │   ├── MoveLabel (Label - "MV:")
│   │   │   └── MoveValue (Label)
│   │   ├── AttackStat (HBoxContainer)
│   │   │   ├── AttackLabel (Label - "ATK:")
│   │   │   └── AttackValue (Label)
│   │   ├── DefenseStat (HBoxContainer)
│   │   │   ├── DefenseLabel (Label - "DEF:")
│   │   │   └── DefenseValue (Label)
│   │   └── RangeStat (HBoxContainer)
│   │       ├── RangeLabel (Label - "RNG:")
│   │       └── RangeValue (Label)
│   └── CloseButton (Button)
├── IronCurtain (ColorRect) - Hidden by default
│   ├── CurtainBackground (ColorRect)
│   │   ├── CurtainLabel (Label) - "PASS DEVICE TO PLAYER X"
│   │   ├── CurtainSubtitle (Label) - "Player X click 'I'm Ready' when ready"
│   │   └── CurtainReadyButton (Button) - "I'm Ready"
├── RevealOrdersOverlay (ColorRect) - Hidden by default
│   └── RevealContainer (HBoxContainer)
│       ├── LeftPanel (Panel) - Red Army orders
│       │   ├── PanelTitle (Label - "Red Army Orders")
│       │   └── OrderList (VBoxContainer)
│       │       ├── OrderItem1 (Label)
│       │       ├── OrderItem2 (Label)
│       │       └── ...
│       ├── CenterPanel (Panel) - Battle preview
│       │   ├── PanelTitle (Label - "Battles This Turn")
│       │   └── BattleList (VBoxContainer)
│       │       ├── BattleItem1 (Label)
│       │       ├── BattleItem2 (Label)
│       │       └── ...
│       └── RightPanel (Panel) - White Army orders
│           ├── PanelTitle (Label - "White Army Orders")
│           └── OrderList (VBoxContainer)
│               ├── OrderItem1 (Label)
│               ├── OrderItem2 (Label)
│               └── ...
│   └── BeginTurnButton (Button) - "Begin Turn"
├── SystemsContainer (Node)
│   ├── TurnStateMachine (Node)
│   │   ├── script: scripts/game/turn_state_machine.gd
│   │   └── No children (pure logic)
│   ├── OrderManager (Node)
│   │   ├── script: scripts/game/order_manager.gd
│   │   └── No children
│   ├── CollisionSystem (Node)
│   │   ├── script: scripts/game/collision_system.gd
│   │   └── No children
│   ├── BattleSystem (Node)
│   │   ├── script: scripts/game/battle_system.gd
│   │   └── No children
│   ├── NetworkSystem (Node)
│   │   ├── script: scripts/game/network_system.gd
│   │   └── No children
│   ├── RailwaySystem (Node)
│   │   ├── script: scripts/game/railway_system.gd
│   │   └── No children
│   └── Pathfinding (Node)
│       ├── script: scripts/utils/pathfinding.gd
│       └── No children
├── BattleSceneContainer (Control) - Hidden by default
│   └── BattleScene (Battle scene instance)
└── AudioManager (Node)
    ├── PlanningMusic (AudioStreamPlayer)
    ├── ActionMusic (AudioStreamPlayer)
    ├── SFXPlayer (AudioStreamPlayer) - Gameplay sounds
    └── BattleSFXPlayer (AudioStreamPlayer) - Battle sounds
```

### **Node Descriptions**

#### **GameBoard (Control - Root)**
- **Purpose**: Root container for entire game
- **Type**: Control
- **Script**: `GameBoard.gd`
- **Signals**:
  - `unit_selected(unit: Unit)`
  - `unit_deselected(unit: Unit)`
  - `move_order_issued(unit: Unit, target: Vector2i)`
  - `attack_order_issued(attacker: Unit, target: Vector2i)`
  - `special_action_requested(action: String, unit: Unit)`
  - `undo_clicked()`
  - `ready_clicked()`

#### **Camera2D**
- **Purpose**: Viewport camera for map
- **Type**: Camera2D
- **Position**: Centered on map
- **Zoom**: Adjustable (default 1.0)
- **Drag Enabled**: True (mouse drag to pan)
- **Zoom Enabled**: True (scroll wheel to zoom)

#### **TileMap (TileMapLayer)**
- **Purpose**: Main tile grid
- **Type**: TileMapLayer
- **TileSet**: `resources/tilesets/game_tileset.tres`
- **Cell Size**: 32x32 (default)
- **Grid Size**: 20x25 (maps can vary)
- **Quadrant 0**: BaseTerrainLayer (grass, snow)
- **Quadrant 1**: ObstacleLayer (mountains, city blocks)
- **Quadrant 2**: InfrastructureLayer (rails, roads, telegraph)
- **Quadrant 3**: BuildingLayer (stations, plazas)

#### **UnitsContainer (Node2D)**
- **Purpose**: Container for all unit instances
- **Type**: Node2D
- **Position**: (0, 0) (relative to map)
- **Children**:
  - **RedArmyUnits**: Container for Player 0 units
  - **WhiteArmyUnits**: Container for Player 1 units

#### **Unit Scene Instance** (Unit.tscn)
- **Purpose**: Individual unit on board
- **Type**: Scene instance (from `scenes/units/unit.tscn`)
- **Position**: Grid position converted to world position
- **Script**: `scripts/units/unit.gd`
- **Signals**:
  - `selected()`
  - `deselected()`
  - `moved(from: Vector2i, to: Vector2i)`
  - `attacked(target: Unit)`
  - `destroyed()`

#### **TopHUD (HBoxContainer)**
- **Purpose**: Display turn information
- **Type**: HBoxContainer
- **Layout**: Turn | Player | Phase
- **Anchors**: Top-left and top-right corners

#### **TurnLabel (Label)**
- **Purpose**: Display current turn number
- **Type**: Label
- **Text**: "Turn X"

#### **PlayerLabel (Label)**
- **Purpose**: Display current player
- **Type**: Label
- **Text**: "Red Army's Turn" or "White Army's Turn"

#### **PhaseLabel (Label)**
- **Purpose**: Display current phase
- **Type**: Label
- **Text**: "Planning Phase" / "Resolution Phase"

#### **BottomHUD (HBoxContainer)**
- **Purpose**: Display player actions and counters
- **Type**: HBoxContainer
- **Layout**: Movement | Attack | Undo | Special | Ready
- **Anchors**: Bottom-left and bottom-right corners

#### **MovementCounter (Panel)**
- **Purpose**: Display movement order count
- **Type**: Panel
- **Content**: "Movements: X/5"

#### **AttackCounter (Panel)**
- **Purpose**: Display attack order count
- **Type**: Panel
- **Content**: "Attacks: X/1"

#### **UndoButton (Button)**
- **Purpose**: Remove last order
- **Type**: Button
- **Signal**: `pressed()` → `GameBoard.undo_clicked()`
- **Enabled**: When orders exist

#### **SpecialActionsButton (Button)**
- **Purpose**: Open special actions menu
- **Type**: Button
- **Signal**: `pressed()` → Show SpecialActionsMenu
- **Context**: Opens menu for selected unit

#### **SpecialActionsMenu (Panel)**
- **Purpose**: Display available special actions
- **Type**: Panel
- **Visibility**: Hidden by default, shown when button clicked
- **Position**: Below button or centered on screen

#### **ReadyButton (Button)**
- **Purpose**: Submit orders and end planning phase
- **Type**: Button
- **Signal**: `pressed()` → `GameBoard.ready_clicked()`
- **Enabled**: When orders are valid

#### **IronCurtain (ColorRect)**
- **Purpose**: Overlay to hide Player A's orders from Player B
- **Type**: ColorRect
- **Color**: Semi-transparent black (Alpha: 0.8)
- **Visibility**: Only during IRON_CURTAIN phase
- **Anchors**: All corners (full screen)

#### **RevealOrdersOverlay (ColorRect)**
- **Purpose**: Show both players' orders before resolution
- **Type**: ColorRect
- **Color**: Semi-transparent dark (Alpha: 0.6)
- **Visibility**: Only during REVEAL_ORDERS phase
- **Anchors**: All corners (full screen)

#### **SystemsContainer (Node)**
- **Purpose**: Container for all game systems
- **Type**: Node
- **Children**: All pure logic systems (no visual children)

#### **BattleSceneContainer (Control)**
- **Purpose**: Container for battle scene overlay
- **Type**: Control
- **Visibility**: Only during PLAY_BATTLE_ANIMS phase
- **Anchors**: All corners (full screen)

#### **BattleScene (Battle scene instance)**
- **Purpose**: Display individual battle animation
- **Type**: Scene instance (from `scenes/game/battle_scene.tscn`)
- **Script**: `scripts/game/battle_scene.gd`
- **Signals**:
  - `battle_finished()`

---

## **BATTLE SCENE**

**File**: `scenes/game/battle_scene.tscn`
**Script**: `scripts/game/battle_scene.gd`

### **Node Hierarchy (Phase 1 - Basic)**

```
BattleScene (Control)
├── Background (ColorRect)
├── ContentContainer (HBoxContainer)
│   ├── LeftSide (VBoxContainer)
│   │   ├── AttackerLabel (Label)
│   │   ├── AttackerContainer (VBoxContainer)
│   │   │   ├── AttackerSprite1 (TextureRect)
│   │   │   ├── AttackerSprite2 (TextureRect)
│   │   │   └── ...
│   │   └── Spacer (Control)
│   ├── CenterPanel (VBoxContainer)
│   │   ├── StatsPanel (HBoxContainer)
│   │   │   ├── AttackLabel (Label - "Attack:")
│   │   │   ├── AttackValue (Label)
│   │   │   ├── Separator (Label - "|")
│   │   │   ├── DefenseLabel (Label - "Defense:")
│   │   │   └── DefenseValue (Label)
│   │   └── OutcomeLabel (Label)
│   │       └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   │   └── AnimationPlayer (AnimationPlayer)
│   └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── RightSide (VBoxContainer)
│       ├── DefenderLabel (Label)
│       ├── DefenderContainer (VBoxContainer)
│       │   ├── DefenderSprite1 (TextureRect)
│       │   ├── DefenderSprite2 (TextureRect)
│       │   └── ...
│       └── Spacer (Control)
├── BottomBar (HBoxContainer)
│   ├── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
│   ├── ContinueButton (Button)
│   ├── SkipButton (Button)
│   └── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
└── AudioManager (Node)
    ├── BattleMusic (AudioStreamPlayer)
    └── BattleSFX (AudioStreamPlayer)
```

### **Node Hierarchy (Phase 2 - Binoculars Effect)**

```
BattleScene (Control)
├── BinocularsVignette (TextureRect) - Dark circular shadow
├── CameraShake (Camera2D) - Subtle bobbing
├── Background (ColorRect)
├── ContentContainer (HBoxContainer)
│   ├── LeftSide (VBoxContainer)
│   │   ├── AttackerLabel (Label)
│   │   ├── AttackerContainer (VBoxContainer)
│   │   │   ├── AttackerSprite1 (TextureRect) - Silhouette
│   │   │   ├── AttackerSprite2 (TextureRect) - Silhouette
│   │   │   └── ...
│   │   └── Spacer (Control)
│   ├── CenterPanel (VBoxContainer)
│   │   ├── StatsPanel (HBoxContainer) - Mechanical computer style
│   │   │   ├── AttackDial (TextureRect) - Dial showing attack value
│   │   │   ├── DefenseDial (TextureRect) - Dial showing defense value
│   │   └── OutcomeLabel (Label)
│   │       └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   │       └── AnimationPlayer (AnimationPlayer)
│   └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── RightSide (VBoxContainer)
│       ├── DefenderLabel (Label)
│       ├── DefenderContainer (VBoxContainer)
│       │   ├── DefenderSprite1 (TextureRect) - Silhouette
│       │   ├── DefenderSprite2 (TextureRect) - Silhouette
│       │   └── ...
│       └── Spacer (Control)
├── ParticleContainer (Node2D) - Explosion/dust effects
│   ├── ExplosionParticles (GPUParticles2D)
│   ├── SmokeParticles (GPUParticles2D)
│   └── DebrisParticles (GPUParticles2D)
├── FlashOverlay (ColorRect) - Bright flash on impact
├── BottomBar (HBoxContainer)
│   ├── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
│   ├── ContinueButton (Button)
│   ├── SkipButton (Button)
│   └── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
└── AudioManager (Node)
    ├── BattleMusic (AudioStreamPlayer)
    └── BattleSFX (AudioStreamPlayer)
```

### **Node Descriptions**

#### **BattleScene (Control - Root)**
- **Purpose**: Root container for battle visualization
- **Type**: Control
- **Script**: `BattleScene.gd`
- **Signals**:
  - `battle_finished()`
  - `skip_battle()`
  - `skip_all_battles()`

#### **BinocularsVignette (TextureRect)**
- **Purpose**: Create binoculars viewing effect (Phase 2)
- **Type**: TextureRect
- **Texture**: Vignette texture (dark edges, center transparent)
- **Modulate**: Multiply with scene

#### **CameraShake (Camera2D)**
- **Purpose**: Subtle camera movement simulating horseback (Phase 2)
- **Type**: Camera2D
- **Animation**: Bobbing up/down (±2 pixels, 1.5s period)

#### **AttackerSprite (TextureRect)**
- **Purpose**: Display attacker unit
- **Type**: TextureRect
- **Texture**: Unit silhouette (NATO-style marker)
- **Phase 2**: Silhouette, distance-based desaturation

#### **DefenderSprite (TextureRect)**
- **Purpose**: Display defender unit
- **Type**: TextureRect
- **Texture**: Unit silhouette (NATO-style marker)
- **Phase 2**: Silhouette, distance-based desaturation

#### **AttackDial / DefenseDial (TextureRect)**
- **Purpose**: Display attack/defense values as mechanical dials (Phase 2)
- **Type**: TextureRect
- **Texture**: Dial graphic with needle
- **Animation**: Needle rotates to value position (0.3s)

#### **AnimationPlayer (AnimationPlayer)**
- **Purpose**: Play attack animation
- **Type**: AnimationPlayer
- **Animations**:
  - "attack" (projectile, flash)
  - "explosion" (destruction)
  - "retreat" (unit moving away)

#### **ParticleContainer (Node2D)**
- **Purpose**: Container for particle effects (Phase 2)
- **Type**: Node2D
- **Children**:
  - ExplosionParticles: Fire, sparks
  - SmokeParticles: Rising smoke
  - DebrisParticles: Falling debris

#### **FlashOverlay (ColorRect)**
- **Purpose**: Bright flash on impact (Phase 2)
- **Type**: ColorRect
- **Color**: White (Alpha: 1.0 → 0.0 in 0.1s)
- **Modulate**: Additive blend mode

#### **ContinueButton (Button)**
- **Purpose**: Proceed to next battle
- **Type**: Button
- **Signal**: `pressed()` → `BattleScene.battle_finished()`

#### **SkipButton (Button)**
- **Purpose**: Skip current or all battles
- **Type**: Button
- **Signal**: 
  - Short press: Skip current battle
  - Long press: Skip all battles (if enabled)

---

## **VICTORY/DEFEAT SCREEN SCENE**

**File**: `scenes/menus/victory_screen.tscn` (or `defeat_screen.tscn`)
**Script**: `scripts/menus/victory_screen.gd` (or `defeat_screen.gd`)

### **Node Hierarchy (Phase 1 - Basic)**

```
VictoryScreen (Control)
├── Background (ColorRect)
├── ContentContainer (VBoxContainer)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── ResultLabel (Label) - "VICTORY!" or "DEFEAT!"
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── PortraitPlaceholder (TextureRect)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── MessageLabel (RichTextLabel)
│   └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
└── BottomBar (HBoxContainer)
    ├── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
    ├── BackToMenuButton (Button)
    ├── RetryButton (Button)
    └── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
└── AudioManager (Node)
    └── VictoryMusic (AudioStreamPlayer)
```

### **Node Hierarchy (Phase 2 - Full Theme)**

```
VictoryScreen (Control)
├── Background (TextureRect) - Thematic background
├── ContentContainer (VBoxContainer)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── ResultLabel (Label) - "VICTORY!" or "DEFEAT!"
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── PortraitContainer (CenterContainer)
│   │   ├── PortraitSprite (Sprite2D) - Thematic portrait
│   │   └── PortraitAnimator (AnimationPlayer)
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   ├── CutawayContainer (TextureRect) - Historical map with arrows
│   ├── Spacer (Control, size_flags_vertical = EXPAND_FILL)
│   └── MessageLabel (RichTextLabel) - Historical context
│   └── Spacer (Control, size_flags_vertical = EXPAND_FILL)
└── BottomBar (HBoxContainer)
    ├── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
    ├── BackToMenuButton (Button)
    ├── RetryButton (Button)
    └── Spacer (Control, size_flags_horizontal = EXPAND_FILL)
└── AudioManager (Node)
    └── VictoryMusic (AudioStreamPlayer) - Victory/defeat theme
```

### **Node Descriptions**

#### **VictoryScreen (Control - Root)**
- **Purpose**: Root container for victory/defeat screen
- **Type**: Control
- **Script**: `VictoryScreen.gd` (or `DefeatScreen.gd`)
- **Signals**:
  - `back_to_menu_clicked()`
  - `retry_clicked()`

#### **ResultLabel (Label)**
- **Purpose**: Display victory or defeat
- **Type**: Label
- **Text**: "VICTORY!" (green) or "DEFEAT!" (red)
- **Font**: Very large, bold

#### **PortraitSprite (Sprite2D)**
- **Purpose**: Display thematic portrait (Phase 2)
- **Type**: Sprite2D
- **Texture**: General celebrating victory or defeated
- **Animation**: Subtle breathing, expression changes

#### **CutawayContainer (TextureRect)**
- **Purpose**: Display historical map with troop movements (Phase 2)
- **Type**: TextureRect
- **Texture**: Battle map with arrow overlays

#### **MessageLabel (RichTextLabel)**
- **Purpose**: Display historical context or victory message
- **Type**: RichTextLabel
- **Autowrap Mode**: Word Smart
- **Fit Content**: True

#### **BackToMenuButton (Button)**
- **Purpose**: Return to main menu
- **Type**: Button
- **Signal**: `pressed()` → `VictoryScreen.back_to_menu_clicked()`

#### **RetryButton (Button)**
- **Purpose**: Restart current map
- **Type**: Button
- **Signal**: `pressed()` → `VictoryScreen.retry_clicked()`

---

## **UNIT SCENE**

**File**: `scenes/units/unit.tscn`
**Script**: `scripts/units/unit.gd`

### **Node Hierarchy**

```
Unit (Area2D)
├── CollisionShape (CollisionShape2D)
├── VisualContainer (Control)
│   ├── UnitMarker (ColorRect) - Main marker shape
│   ├── DirectionIndicator (Polygon2D) - Small triangle
│   ├── SelectionGlow (ColorRect) - Hidden by default
│   ├── ActiveIndicator (ColorRect) - Hidden by default
│   └── StatusIcon (TextureRect) - Deployed/packed/relay icons
├── NetworkLineContainer (Node2D) - For debug/visualization
│   ├── LineOfEffectLines (Line2D) - Lines to sources
│   └── TelegraphLines (Line2D) - Telegraph connections
├── BehaviorContainer (Node)
│   └── BehaviorScript (Script) - Unit-specific behavior
└── AudioManager (Node)
    ├── SelectSFX (AudioStreamPlayer)
    ├── MoveSFX (AudioStreamPlayer)
    └── AttackSFX (AudioStreamPlayer)
```

### **Node Descriptions**

#### **Unit (Area2D - Root)**
- **Purpose**: Root container for unit
- **Type**: Area2D
- **Script**: `Unit.gd`
- **Collision Layer**: "units" layer
- **Collision Mask**: "units" layer
- **Signals**:
  - `selected()`
  - `deselected()`
  - `moved(from: Vector2i, to: Vector2i)`
  - `attacked(target: Unit)`
  - `destroyed()`

#### **CollisionShape (CollisionShape2D)**
- **Purpose**: Click detection and unit collision
- **Type**: CollisionShape2D
- **Shape**: Rectangle (matches tile size)
- **Size**: 32x32 (or based on unit.size)
- **Collision Layer**: "units"
- **Collision Mask**: "units"

#### **UnitMarker (ColorRect)**
- **Purpose**: Display unit visual marker
- **Type**: ColorRect
- **Size**: Based on unit_data.marker_size
- **Shape**: Based on unit_data.marker_shape
- **Color**: Based on owner_id (Red/White team colors)

#### **DirectionIndicator (Polygon2D)**
- **Purpose**: Show unit facing direction
- **Type**: Polygon2D
- **Shape**: Triangle pointing forward
- **Rotation**: Based on unit facing
- **Color**: Same as unit marker

#### **SelectionGlow (ColorRect)**
- **Purpose**: Highlight selected unit
- **Type**: ColorRect
- **Color**: White (Alpha: 0.3)
- **Visibility**: Hidden by default, shown when selected
- **Animation**: Pulse (scale 1.0 ↔ 1.2, 1s period)

#### **ActiveIndicator (ColorRect)**
- **Purpose**: Show if unit is active (connected to source)
- **Type**: ColorRect
- **Color**: Green (Alpha: 0.5) for active, Gray for inactive
- **Visibility**: Always visible (alpha changes based on active state)

#### **StatusIcon (TextureRect)**
- **Purpose**: Display unit status (deployed, packed, relay)
- **Type**: TextureRect
- **Texture**: 
  - Gear icon: Heavy unit, deployed
  - Box icon: Heavy unit, packed
  - Relay icon: Relay unit
- **Visibility**: Based on unit state

#### **NetworkLineContainer (Node2D)**
- **Purpose**: Debug visualization of network connections
- **Type**: Node2D
- **Visibility**: Only in debug mode
- **Children**:
  - LineOfEffectLines: Lines to sources
  - TelegraphLines: Lines to telegraph stations

#### **BehaviorScript (Script)**
- **Purpose**: Unit-specific behavior (optional)
- **Type**: Script (attached dynamically)
- **Examples**:
  - `CavalryBehavior.gd`: Cavalry charge bonus
  - `TrainBehavior.gd`: Track movement, car coordination
  - `HeavyUnitBehavior.gd`: Pack/deploy mechanics
  - `RelayBehavior.gd`: Network relay functionality

---

## **AUTOLOAD SINGLETONS**

These are registered in `project.godot` and are globally accessible.

### **GameManager**

**File**: `scripts/autoloads/game_manager.gd`

**Node Structure**: No children (pure logic node)

**Responsibilities**:
- Scene transitions
- Turn management coordination
- Global game state
- Event emission for major game events

**Signals**:
- `scene_changed(new_scene: String)`
- `turn_started(player_id: int)`
- `turn_ended(player_id: int)`
- `game_ended(winner_id: int, victory_type: String)`

---

### **SaveSystem**

**File**: `scripts/autoloads/save_system.gd`

**Node Structure**: No children (pure logic node)

**Responsibilities**:
- Save game state to JSON
- Load game state from JSON
- Manage save slots
- Handle version compatibility

**Methods**:
- `save_game(slot: int) -> Error`
- `load_game(slot: int) -> Error`
- `delete_save(slot: int) -> Error`
- `get_save_info(slot: int) -> Dictionary`

---

### **DataManager**

**File**: `scripts/autoloads/data_manager.gd`

**Node Structure**: No children (pure logic node)

**Responsibilities**:
- Load unit definitions from .tres files
- Load tile data and map configurations
- Provide data access methods
- Cache frequently accessed data

**Methods**:
- `get_unit_data(unit_type: String) -> UnitData`
- `get_tile_data(tile_type: String) -> TileData`
- `load_map(map_id: String) -> MapData`

---

### **AudioController**

**File**: `scripts/autoloads/audio_controller.gd`

**Node Structure**:
```
AudioController (Node)
├── MusicBus (AudioBus) - "Music" bus
├── SFXBus (AudioBus) - "SFX" bus
├── PlanningMusicPlayer (AudioStreamPlayer)
├── ActionMusicPlayer (AudioStreamPlayer)
└── SFXPlayers (Node)
    ├── MenuSFX (AudioStreamPlayer)
    ├── TypewriterSFX (AudioStreamPlayer)
    ├── GameplaySFX (AudioStreamPlayer)
    └── BattleSFX (AudioStreamPlayer)
```

**Responsibilities**:
- Play sound effects (moves, attacks, explosions)
- Play background music
- Handle volume controls
- Manage audio buses (SFX, Music)
- Dynamic music layering (planning + action tracks)

**Methods**:
- `play_sound(sound_name: String)`
- `play_music(track_name: String)`
- `set_sfx_volume(volume: float)`
- `set_music_volume(volume: float)`
- `tween_action_volume(from: float, to: float, duration: float)`
- `tween_planning_volume(from: float, to: float, duration: float)`

---

## **SCENE LOADING PATTERNS**

### **Load Scene**

```gdscript
# In GameManager.gd

func load_scene(scene_path: String) -> void:
    var scene = load(scene_path)
    var instance = scene.instantiate()
    get_tree().change_scene_to_packed(instance)
```

### **Add Scene as Child**

```gdscript
# In GameBoard.gd - for overlays

func show_battle_scene(battle_data: BattleData) -> void:
    var scene = load("res://scenes/game/battle_scene.tscn")
    var instance = scene.instantiate()
    instance.set_battle_data(battle_data)
    add_child(instance)
    
    instance.battle_finished.connect(_on_battle_finished)

func _on_battle_finished() -> void:
    # Remove battle scene
    var battle_scene = $BattleSceneContainer
    battle_scene.queue_free()
```

### **Scene Transition Animation**

```gdscript
# In GameManager.gd

func transition_to_scene(scene_path: String, fade_duration: float = 0.5) -> void:
    var tween = create_tween()
    
    # Fade out current scene
    var current_scene = get_tree().current_scene
    var canvas_layer = CanvasLayer.new()
    current_scene.add_child(canvas_layer)
    var fade_rect = ColorRect.new()
    fade_rect.set_fullscreen_rect()
    canvas_layer.add_child(fade_rect)
    
    tween.tween_property(fade_rect, "color:a", 1.0, fade_duration / 2.0)
    tween.tween_callback(func():
        # Load new scene at fade midpoint
        load_scene(scene_path)
    )
    tween.parallel()
    tween.tween_property(fade_rect, "color:a", 0.0, fade_duration / 2.0)
    tween.tween_callback(func():
        # Cleanup fade rect
        fade_rect.queue_free()
        canvas_layer.queue_free()
    )
```

---

## **SIGNAL CONNECTION PATTERNS**

### **Pattern 1: Child → Parent**

```gdscript
# Unit emits signal, GameBoard receives

# In unit.gd
signal selected()

# In game_board.gd
func _ready():
    for unit in get_all_units():
        unit.selected.connect(_on_unit_selected)

func _on_unit_selected(unit: Unit) -> void:
    # Handle unit selection
    update_unit_info_panel(unit)
```

### **Pattern 2: Parent → Sibling**

```gdscript
# GameBoard receives signal, emits to UI components

# In game_board.gd
signal unit_selected(unit: Unit)

# In unit_info_panel.gd
func _ready():
    game_board.unit_selected.connect(_on_unit_selected)

func _on_unit_selected(unit: Unit) -> void:
    show_unit_info(unit)
```

### **Pattern 3: System → AutoLoad**

```gdscript
# GameBoard emits signal, GameManager receives

# In game_board.gd
signal game_ended(winner_id: int)

# In game_manager.gd
func _ready():
    # GameBoard is not an autoload, so connect differently
    pass  # AutoLoad signals are globally accessible

# In game_board.gd
func _on_victory():
    GameManager.game_ended.emit(winner_id, "annihilation")
```

---

## **DEBUG VISUALIZATION**

### **Debug Overlay**

```gdscript
# In debug_visualizer.gd

extends Node

func _draw() -> void:
    # Draw network lines
    draw_network_lines()
    
    # Draw collision boxes
    draw_collision_boxes()
    
    # Draw movement paths
    draw_movement_paths()
```

### **Development Tools**

Add to scene tree only in debug mode:
- `StateLabel`: Show current phase
- `FPSLabel`: Show performance
- `MousePositionLabel`: Show tile coordinates
- `NetworkVisualizer`: Draw lines of effect

---

**Last Updated**: January 2, 2026
**Version**: 1.0
