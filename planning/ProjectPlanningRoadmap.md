# **PROJECT PLANNING ROADMAP**

**Godot 4.x | Mobile/PC | Code-Generated Assets**

---

## **CURRENT STATUS**

### **Completed Planning Documents**
- ✅ **rules.md**: Complete game rules and mechanics
- ✅ **Implementation.md**: High-level implementation phases (1-3)

### **Identified Gaps**
The existing documents provide an excellent foundation, but to create a functional prototype in Godot, we need detailed specifications for:
1. Project structure and organization
2. Data formats and schemas
3. UI/UX systems (critical for simultaneous turns)
4. Scene hierarchies and node architecture
5. State management implementation details
6. Technical specifications for core systems

---

## **REQUIRED PLANNING DOCUMENTS**

### **1. ProjectStructure.md**
**Purpose**: Define the complete Godot project folder structure and organization.

**Contents**:
- Root directory structure (scenes/, scripts/, resources/, etc.)
- Scene file naming conventions
- Script organization (autoloads, state machines, utilities)
- Asset folder structure (sounds, fonts, shaders, etc.)
- AutoLoad singleton definitions and their responsibilities

**Key Considerations**:
- Godot 4.x specific features (TileMapLayer, new signals syntax)
- Mobile/PC compatibility (input handling, touch controls)
- Code-generated asset organization

**Complexity**: Medium
**Dependencies**: None
**Priority**: HIGH - Needed before any coding begins

---

### **2. DataFormats.md**
**Purpose**: Define schemas for all game data that needs to be loaded from files.

**Contents**:
- **Unit Definitions**: JSON or .tres resource format for all 14 unit types
  - Stats (move, attack, defense, range)
  - Special abilities (Cavalry Charge, Heavy, Relay, etc.)
  - Visual properties (color, size, shape)
  
- **Tile Definitions**: Terrain tile properties
  - Type (Base, Obstacle, Infrastructure, Interactable)
  - Properties (passable, blocks LoC, defensible bonus, etc.)
  - Infrastructure connections (rail type, telegraph lines, roads)
  
- **Map Data Format**: Structure for the 3 campaign maps
  - Grid dimensions (default 20x25)
  - Tile layout
  - Unit starting positions
  - Neutral building placements
  - Victory conditions per map
  
- **Save Game Format**: What data to serialize
  - Turn state
  - Unit positions and states
  - Building ownership
  - Campaign progress

**Key Considerations**:
- JSON vs. Godot Resources (.tres) - recommend .tres for editor integration
- Validation and error handling
- Version compatibility for save files

**Complexity**: Medium
**Dependencies**: None
**Priority**: HIGH - Needed before creating units/tilesets

---

### **3. StateManagement.md**
**Purpose**: Detailed implementation of the turn state machine and order processing system.

**Contents**:
- **Turn State Machine**: Complete state transition diagram
  - Planning Phase A → Curtain → Planning Phase B → Resolution → End
  - State entry/exit actions
  - Valid actions per state
  - Transition conditions
  
- **Order Data Structures**:
  - MovementOrder: source_pos, target_pos, unit_ref
  - AttackOrder: attacker_ref, target_pos, target_unit_ref
  - Validation rules per order type
  
- **Movement Resolution Algorithm**:
  - Order processing sequence
  - Collision detection logic
  - Train collision exception handling
  - Path validation
  
- **Battle Resolution Algorithm**:
  - Battle participant identification
  - Attack/Defense summing
  - Simultaneous effect evaluation
  - Retreat/Death application
  
- **State Persistence**: How to save/restore mid-game state

**Key Considerations**:
- Simultaneous turn complexity - both players plan before resolution
- Order validation during planning phase
- Performance (network caching, collision optimization)
- Debug visualization for state transitions

**Complexity**: HIGH
**Dependencies**: DataFormats.md
**Priority**: HIGH - Core of the game loop

---

### **4. UISpecifications.md**
**Purpose**: Complete UI flow and interaction design for all game screens.

**Contents**:
- **Main Menu Scene**:
  - Start Campaign
  - Level Select
  - Options
  - Touch/Mouse control toggle
  
- **Board Game UI**:
  - Turn phase indicator
  - Movement orders remaining counter
  - Attack orders remaining counter
  - End Turn button
  - Unit selection panel (shows selected unit info)
  - Valid move indicators (ghost units)
  - Attack range indicators (red highlighting)
  - Active/inactive unit visualization
  - Camera controls (pan/zoom buttons, or touch/mouse drag)
  
- **Planning Phase UI**:
  - Unit selection feedback
  - Order confirmation dialogs
  - Special action buttons (Sabotage, Pack/Deploy, Embark/Disembark)
  - Order list display (current turn's planned actions)
  - Cancel order functionality
  
- **The "Curtain" Scene**:
  - "Pass Device to Player [X]" message
  - Transition animation
  - Security (no way to peek at opponent's plans)
  
- **Battle Scene**:
  - Attacker vs. Defender display
  - Attack/Defense totals
  - Animation playback area
  - Result display (Victory/Defeat/Retreat)
  - Continue button
  
- **Campaign UI**:
  - Narrative context screens (historical text)
  - Level progression map
  - Victory/Defeat screens
  - Campaign completion summary
  
- **Input Handling**:
  - Mouse click/tap logic
  - Right-click/tap-and-hold for info
  - Keyboard shortcuts (optional)
  - Touch gestures (pinch-zoom, two-finger pan)

**Key Considerations**:
- Mobile responsiveness (small screens, touch targets)
- Clear visual feedback for simultaneous turn planning
- Preventing information leakage during curtain phase
- Accessibility (colorblind-friendly unit markers)

**Complexity**: HIGH
**Dependencies**: StateManagement.md
**Priority**: HIGH - Critical for playable prototype

---

### **5. SceneHierarchy.md**
**Purpose**: Node-by-node breakdown of each major scene and their relationships.

**Contents**:
- **Main Scene (GameBoard.tscn)**:
  - Root node (Control or Node2D)
  - TileMapLayer structure
  - Camera2D setup
  - UI overlay nodes
  - Unit container
  - GameRules script attachment
  
- **Unit Scene (Unit.tscn)**:
  - Root (Area2D or Node2D)
  - Visual node (ColorRect or Sprite2D)
  - Selection indicator
  - Connection line visuals (for network display)
  - Stats/properties
  
- **Battle Scene (BattleScene.tscn)**:
  - Background
  - Attacker sprite container
  - Defender sprite container
  - UI panels (stats, result)
  - Animation player
  
- **Curtain Scene (CurtainScene.tscn)**:
  - Background (maybe simple animation)
  - Text label
  - Transition timer
  
- **Menu Scenes**:
  - MainMenu.tscn
  - LevelSelect.tscn
  - Options.tscn
  - NarrativeScreen.tscn
  
- **Autoload Singletons**:
  - GameManager.gd: Turn control, scene transitions
  - SaveSystem.gd: Save/load functionality
  - AudioController.gd: Sound management
  - DataManager.gd: Load unit/tile data

**Key Considerations**:
- Godot 4.x node types (TileMapLayer vs. TileMap)
- Signal connections between nodes
- Reference passing (who needs access to what)
- Code-generated graphics (ColorRect, Line2D, etc.)

**Complexity**: Medium
**Dependencies**: ProjectStructure.md
**Priority**: HIGH - Blueprint for scene creation

---

### **6. TechnicalSpecifications.md**
**Purpose**: Implementation details for complex game systems.

**Contents**:
- **Network (Communication) System**:
  - Recursive raycasting algorithm
  - Line-of-effect calculation
  - Telegraph chain propagation
  - Balloon special case
  - Performance optimization (caching active states)
  
- **Railway System**:
  - Graph construction from tile data
  - BFS cascade algorithm for station capture
  - Train movement constraints (track following)
  - Turnaround/intersection logic
  
- **Collision Detection System**:
  - Movement order preprocessing
  - Overlap detection algorithm
  - Train collision exception
  - "Squish rule" implementation
  - Cascading collision resolution
  
- **Battle Participation Logic**:
  - Range checking per unit type
  - Pathfinding to target
  - Obstacle/enemy blocking detection
  - Simultaneous battle coordination
  
- **Transport System**:
  - Embark/disembark state changes
  - Capacity checking
  - Carrier movement coordination
  - Destruction propagation

**Key Considerations**:
- Algorithm efficiency (no lag during resolution)
- Edge cases (corner cases in rules)
- Debug visualization tools
- Unit testing approach

**Complexity**: VERY HIGH
**Dependencies**: StateManagement.md, DataFormats.md
**Priority**: MEDIUM - Can be implemented incrementally

---

### **7. AssetGeneration.md**
**Purpose**: Specifications for code-generated placeholder graphics.

**Contents**:
- **Unit Markers**:
  - Shape system (circles, squares, triangles for different unit types)
  - Color coding (Red Army vs. White Army)
  - Size variations (infantry small, artillery large)
  - Direction indicator (small triangle pointing forward)
  - Selection highlight
  
- **Terrain Tiles**:
  - Base tiles: Green (grass), White (snow)
  - Obstacles: Gray with mountain icon (mountains), Dark gray blocks (city blocks)
  - Defensible: Blue tint (fortresses), Light blue (plazas)
  - Roads: Brown lines
  - Rail: Black parallel lines
  - Tram: Black single line
  - Telegraph: Thin black dots along path
  
- **Infrastructure Icons**:
  - Stations: Building shape icon
  - Telegraph offices: Telegraph pole icon
  - Turntables: Circular arrow icon
  
- **Battle Scene Assets**:
  - Simple silhouettes for each unit type
  - Attack effect (projectile or flash)
  - Explosion effect
  - Retreat animation (moving sprite)

- **UI Assets**:
  - Buttons (colored rectangles with text)
  - Panels (semi-transparent backgrounds)
  - Icons (simple geometric shapes)

**Key Considerations**:
- All generated via code (no external image files)
- Scalable with screen size
- Mobile-friendly touch targets
- Colorblind-friendly alternatives (patterns, shapes)

**Complexity**: Low
**Dependencies**: SceneHierarchy.md
**Priority**: MEDIUM - Can be implemented alongside scenes

---

## **IMPLEMENTATION PRIORITY**

### **Phase 0: Planning Completion** (Do this first!)
1. ✅ Review existing documents
2. Create ProjectStructure.md
3. Create DataFormats.md
4. Create SceneHierarchy.md
5. Create StateManagement.md
6. Create UISpecifications.md
7. Create TechnicalSpecifications.md
8. Create AssetGeneration.md

### **Phase 1: Foundation Setup** (From Implementation.md)
- Set up project structure
- Create data resource files
- Implement basic scene scaffolding

### **Phase 2: Core Mechanics**
- Implement GameRules.gd with state machine
- Implement network/railway systems
- Create placeholder graphics system

### **Phase 3: UI Integration**
- Build all UI screens
- Implement input handling (mouse/touch)
- Connect UI to game state

### **Phase 4: Battle System**
- Create battle scene
- Implement animations
- Integrate with main game loop

### **Phase 5: Campaign**
- Create 3 campaign maps
- Implement narrative system
- Add save/load functionality

---

## **TECHNICAL REQUIREMENTS**

### **Engine Version**
- Godot 4.2 or later

### **Target Platforms**
- **Mobile**: Android/iOS
  - Touch controls required
  - Portrait and landscape support
  - Low video requirements
  
- **PC**: Windows/Linux/Mac
  - Mouse and keyboard controls
  - Adjustable window size
  - Low video requirements

### **Input Requirements**
- Primary: Touch (tap, tap-and-hold, two-finger pan, pinch-zoom)
- Secondary: Mouse (click, right-click, drag, scroll wheel)
- Optional: Keyboard shortcuts

### **Performance Targets**
- 60 FPS on mid-range mobile devices
- < 500MB RAM usage
- < 50MB total game size (code-generated assets)

---

## **DEPENDENCY GRAPH**

```
ProjectStructure.md ──────────────────────────┐
    │                                         │
    ├─→ SceneHierarchy.md ────────────────────┤
    │         │                               │
    │         └─→ AssetGeneration.md          │
    │                                         │
DataFormats.md ─────────────────────────────┤
    │         │                               │
    └─→ StateManagement.md ──────────────────┤
              │                               │
              └─→ UISpecifications.md ────────┤
                      │                       │
                      └─→ TechnicalSpecifications.md
```

---

## **NOTES & QUESTIONS**

### **Decisions Needed During Implementation**
1. **Data Storage**: JSON or .tres resources? (Recommend .tres)
2. **Camera Control**: Drag-to-pan or edge scrolling?
3. **Battle Scene**: Skippable or always show?
4. **AI Complexity**: Random moves vs. strategic evaluation?
5. **Save System**: Auto-save or manual save only?

### **Potential Risks**
- **Simultaneous Turn Complexity**: The order validation and collision resolution will be tricky
- **Performance**: Network calculations on every turn could be expensive
- **UI Complexity**: Many states and modes to manage
- **Mobile Testing**: Need to test on actual devices for touch feel

### **Success Criteria for Prototype**
- ✅ Two players can complete a full hot-seat game
- ✅ All rules are correctly implemented
- ✅ Visual feedback is clear and understandable
- ✅ Game runs smoothly on mobile device
- ✅ No crashes or soft-locks in normal play

---

**Last Updated**: January 2, 2026
**Status**: Planning Phase - Documents to be created
