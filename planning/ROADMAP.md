# **DEVELOPMENT ROADMAP**

**Godot 4.x | Systematic Development | Phased Approach**

---

## **OVERVIEW**

This roadmap provides a systematic approach to developing "Game of War: Russian Revolution Edition" with clear objectives, deliverables, and testing procedures for each phase. The strategy prioritizes getting a playable prototype as quickly as possible, then systematically building out features.

**Key Principles**:
1. **Playable First**: Get a working hot-seat multiplayer version before adding complexity
2. **Placeholder Assets**: Use Godot shapes and lines throughout development; create production assets after demo campaign
3. **Systematic Testing**: Each phase has clear testing procedures before moving forward
4. **Build Incrementally**: Each phase naturally builds on the previous one

---

## **PHASE 1: HOT SEAT MULTIPLAYER (Playable Prototype)**

**Objective**: Create a fully playable two-player hot seat version on a single map with placeholder assets.

### **Deliverables**

1. **Basic game board**
   - 20x25 grid using TileMapLayer
   - Camera controls (pan/zoom)
   - Grid display

2. **Unit system**
   - All 14 unit types with basic stats (from DataFormats.md)
   - Unit instantiation and placement
   - Unit selection/deselection
   - Unit movement visualization
   - Unit state tracking (active/inactive, deployed/packed)

3. **Terrain system**
   - Base terrain tiles (grass, snow)
   - Obstacle tiles (mountains, city blocks)
   - Infrastructure tiles (rails, roads, telegraph lines)
   - Building tiles (stations, telegraph offices, plazas)

4. **Turn state machine**
   - All turn phases implemented (QUEUE_ORDERS_A → IRON_CURTAIN → QUEUE_ORDERS_B → REVEAL_ORDERS → CALCULATE_MOVEMENT → CALCULATE_BATTLES → RESOLVE_BATTLES → CLEANUP → CHECK_WIN_CONDITION → RESET_TURN)
   - Phase transitions with validation
   - State change signals

5. **Order management**
   - Movement order system (max 5 per player)
   - Attack order system (max 1 per player)
   - Order validation during planning phase
   - Undo functionality
   - Clear orders

6. **"Iron Curtain" screen**
   - Hide Player A's plans
   - Display "Pass device to Player B" message
   - Player B "I'm Ready" button

7. **Reveal orders screen**
   - Show both players' planned movements (ghost units)
   - Show both players' planned attacks (red markers)
   - Side-by-side order lists
   - Battle preview

8. **Movement resolution**
   - Process all movement orders
   - Collision detection (same tile, crossing paths)
   - Apply early stops for collisions
   - Update unit positions

9. **Battle calculation**
   - Identify all battle participants
   - Sum attack/defense values
   - Calculate outcomes (fail/retreat/destroy)
   - Handle cavalry charge bonus
   - Handle terrain bonuses

10. **Battle resolution**
    - Apply destructions simultaneously
    - Apply forced retreats
    - Handle retreat path validation
    - Remove destroyed units

11. **Network system**
    - Calculate lines of effect from sources
    - Determine active/inactive unit states
    - Handle relay units
    - Handle station sources
    - Visual indicators for active/inactive

12. **Railway system**
    - Train movement (track-locked)
    - Station capture cascade (BFS)
    - Station ownership tracking
    - Train collision exceptions (squish rule)

13. **Basic UI**
    - Turn indicator (current player, turn number)
    - Movement counter (X/5)
    - Attack counter (0/1)
    - Selected unit info panel
    - Order list display
    - Ready button
    - Clear orders button

14. **Win condition checking**
    - Annihilation (opponent has 0 units)
    - Station capture (control victory target)
    - Victory/defeat screens

15. **Save/load system**
    - Save game state to JSON
    - Load game state from JSON
    - Manage save slots

### **Testing Procedures**

#### **Core Gameplay Tests**
- [ ] Complete 10-turn test game with another person
- [ ] Verify all 14 unit types can move correctly
- [ ] Verify all 14 unit types can attack correctly
- [ ] Verify movement counter decrements correctly (5 → 0)
- [ ] Verify attack counter works (0 → 1, blocks additional attacks)
- [ ] Test undo removes last order and restores counter
- [ ] Test clear orders removes all orders and resets counters

#### **Collision Detection Tests**
- [ ] Test two units targeting same tile (both stop 1 tile early)
- [ ] Test units crossing paths (both stop before collision)
- [ ] Test train vs unit collision (unit stops, train continues)
- [ ] Test train vs train collision (both destroyed)
- [ ] Test squish rule (unit forced to retreat if still in path)
- [ ] Test cascading collisions (A stops B, B stops C)

#### **Network System Tests**
- [ ] Test station source activates connected units
- [ ] Test relay units extend network
- [ ] Test units outside network are inactive (dimmed)
- [ ] Test units inside network are active (highlighted)
- [ ] Test enemy units block lines of effect
- [ ] Test mountains block lines of effect
- [ ] Test observation balloon ignores mountains

#### **Railway System Tests**
- [ ] Test train movement follows tracks only
- [ ] Test station capture activates connected stations (BFS)
- [ ] Test station ownership requires infantry on tile
- [ ] Test train vs unit collision (unit stops early, train continues)
- [ ] Test train vs train collision (both engines destroyed)
- [ ] Test telegraph stations activate other telegraphs

#### **Battle Calculation Tests**
- [ ] Test attack = defense → attack fails
- [ ] Test attack > defense by 1 → forced retreat
- [ ] Test attack > defense by >1 → destroy unit
- [ ] Test cavalry charge bonus (+3 attack when adjacent)
- [ ] Test terrain bonuses (plaza +4, station +2)
- [ ] Test multiple attackers sum attack values
- [ ] Test multiple defenders sum defense values
- [ ] Test simultaneous effect (destroyed unit participates in other battles)

#### **Win Condition Tests**
- [ ] Test annihilation (opponent has 0 units → victory)
- [ ] Test station capture (control victory target → victory)
- [ ] Test opponent annihilates player → defeat
- [ ] Test multiple win conditions per map

#### **UI/UX Tests**
- [ ] Test unit selection shows correct info
- [ ] Test movement range indicators display correctly
- [ ] Test attack range indicators display correctly
- [ ] Test ghost units show planned movements
- [ ] Test attack markers show planned attacks
- [ ] Test "Ready" button only enables with valid orders
- [ ] Test Iron Curtain screen hides all plans
- [ ] Test Reveal Orders screen shows both players' plans

#### **Save/Load Tests**
- [ ] Test save during planning phase
- [ ] Test save during resolution phase
- [ ] Test load restores all unit positions
- [ ] Test load restores all unit states
- [ ] Test load restores current turn and phase
- [ ] Test load restores all orders

#### **Edge Case Tests**
- [ ] Test all units destroyed → game ends
- [ ] Test no valid retreat path → unit destroyed
- [ ] Test network completely isolated → all units inactive
- [ ] Test sabotage on all infrastructure types
- [ ] Test multiple battles in same turn
- [ ] Test unit destroyed mid-turn (simultaneity)

### **Success Criteria**

✅ Two human players can complete a full game (20+ turns)
✅ All rules from rules.md are correctly implemented
✅ No crashes or soft-locks in normal gameplay
✅ Visual feedback is clear for all game states
✅ Save/load preserves complete game state
✅ Game runs smoothly on target hardware (60 FPS)

### **Placeholder Assets**

- **Units**: Colored circles/squares with directional triangles
  - Red Army: Red shapes
  - White Army: White/gray shapes
  - Size varies by unit type (infantry small, artillery large)
  - Direction indicator (small triangle pointing forward)

- **Terrain**: Colored rectangles
  - Grass: Green (0.3, 0.7, 0.3)
  - Snow: White (0.95, 0.95, 0.98)
  - Mountains: Gray (0.5, 0.5, 0.5) with triangle icon
  - City Blocks: Dark gray (0.3, 0.3, 0.3) with square icon

- **Infrastructure**: Simple line drawings
  - Rails: Black parallel lines (for trains/trams)
  - Roads: Brown lines
  - Telegraph lines: Thin black dots along path

- **Buildings**: Simple shapes
  - Station: Building silhouette icon
  - Telegraph Office: Telegraph pole icon
  - Plaza: Blue-tinted square with border

- **UI**: Basic colored rectangles and text labels
  - Buttons: Colored rectangles with text
  - Panels: Semi-transparent backgrounds
  - Counters: Simple text displays

- **Battle scene**: Simple sprite placeholders
  - Attacker/defender: Profile view rectangles
  - Attack effect: Flash or projectile line
  - Explosion effect: Expanding circle

---

## **PHASE 2: MECHANICS VERIFICATION & POLISH**

**Objective**: Verify all game mechanics work correctly and improve user experience.

### **Deliverables**

1. **Complete unit behaviors**
   - Cavalry charge (+3 attack when adjacent to target)
   - Heavy unit pack/deploy mechanics (Maxim, Balloon)
   - Relay unit network extension
   - Always active units (Cars, Armored Cars)
   - Transport capacity (Trams, Train Troop Cars)
   - Train car attachment (Engine + Gun/Troop cars)

2. **Transport system**
   - Embark mechanics (unit enters carrier)
   - Disembark mechanics (unit exits carrier)
   - Carrier movement with carried units
   - Carrier destruction (carried units destroyed)
   - Capacity checking
   - Stacking restrictions (carriers cannot be carried)

3. **Special actions**
   - Sabotage (destroy terrain on current tile)
   - Pack (heavy unit cannot move same turn)
   - Deploy (heavy unit can attack same turn)

4. **Battle scene improvements**
   - Battle scene with animations (placeholder)
   - Attacker/defender sprite display
   - Attack/defense totals display
   - Battle outcome display
   - "Continue" button
   - "Skip Battle" button (if enabled)
   - "Skip All" button (if enabled)

5. **Pathfinding improvements**
   - A* pathfinding implementation
   - Path caching for performance
   - Path validation (passable tiles, no enemy blocking)
   - Railway path validation (track connectivity)
   - Turntable path handling
   - Intersection path handling

6. **Network system edge cases**
   - Observation balloon ignores mountains
   - Telegraph chain propagation
   - Relay unit chains
   - Multiple sources (network merging)
   - Enemy units blocking lines of effect
   - Impassable terrain blocking lines of effect

7. **Railway system special tiles**
   - Turntables (train can reverse direction)
   - Intersections (train can turn, not reverse)
   - Rail crossing (train tracks crossing)
   - Train composition (engine + multiple cars)
   - Car synchronization (engine moves, cars follow)
   - Car detachment (engine destroyed)

8. **Train collision destruction**
   - Engine vs engine collision (both destroyed)
   - Ambiguous collision resolution (opponent chooses)
   - Car destruction (engine destroyed → cars destroyed or stranded)
   - Squish rule refinement

9. **Telegraph chain propagation**
   - Active telegraph activates connected telegraphs
   - Telegraph network separate from railway network
   - Telegraph office ownership tracking
   - Telegraph sabotage effects

10. **Save/load improvements**
    - Save all unit states (deployed/packed, carried/carrier)
    - Save train compositions
    - Save tile states (destroyed infrastructure)
    - Save station ownership
    - Save telegraph office states
    - Load validation

11. **Pause menu**
    - Pause game state
    - Resume game
    - Save game
    - Exit to main menu
    - Options access

12. **Improved visual feedback**
    - Attack range indicators (red highlight)
    - Movement path visualization (dotted line)
    - Valid move tiles (green highlight)
    - Invalid move tiles (red highlight)
    - Active unit highlighting (glow effect)
    - Inactive unit dimming
    - Selected unit border
    - Battle outcome colors (fail=red, retreat=yellow, destroy=red)

13. **Bug fixes from Phase 1 testing**
    - Fix all game-breaking bugs
    - Fix edge cases
    - Improve error handling
    - Add validation warnings

### **Testing Procedures**

#### **Unit Special Ability Tests**
- [ ] Test cavalry charge (+3 attack when adjacent)
- [ ] Test heavy unit pack (cannot move same turn)
- [ ] Test heavy unit deploy (can attack same turn)
- [ ] Test relay unit extends network (when active)
- [ ] Test always active units (no network required)
- [ ] Test observation balloon ignores mountains

#### **Transport System Tests**
- [ ] Test embark onto carrier
- [ ] Test disembark from carrier
- [ ] Test carrier movement with carried units
- [ ] Test carrier destruction (carried units destroyed)
- [ ] Test capacity limits (cannot exceed max)
- [ ] Test stacking restrictions (carrier cannot be carried)

#### **Special Action Tests**
- [ ] Test sabotage on rails
- [ ] Test sabotage on telegraph lines
- [ ] Test sabotage on stations
- [ ] Test sabotage on turntables
- [ ] Test sabotage on plazas
- [ ] Test pack heavy unit
- [ ] Test deploy heavy unit
- [ ] Test pack then move (blocked)
- [ ] Test move then deploy (allowed)

#### **Battle Scene Tests**
- [ ] Test battle scene displays correctly
- [ ] Test attacker/defender sprites show
- [ ] Test attack/defense totals display
- [ ] Test battle outcome displays (fail/retreat/destroy)
- [ ] Test continue button
- [ ] Test skip battle button
- [ ] Test skip all button
- [ ] Test multiple battles play sequentially

#### **Pathfinding Tests**
- [ ] Test A* finds shortest path
- [ ] Test path around obstacles
- [ ] Test path around enemy units
- [ ] Test railway path follows tracks
- [ ] Test turntable allows direction change
- [ ] Test intersection allows turns
- [ ] Test invalid movement (no path)

#### **Network System Edge Cases**
- [ ] Test observation balloon over mountains
- [ ] Test long relay chain (5+ units)
- [ ] Test multiple sources merging
- [ ] Test enemy unit blocks line of effect
- [ ] Test mountain blocks line of effect
- [ ] Test isolated network (no active units)

#### **Railway System Special Tests**
- [ ] Test turntable reverses train direction
- [ ] Test intersection allows turns
- [ ] Test train with multiple cars
- [ ] Test engine destroyed → cars destroyed
- [ ] Test engine destroyed → cars stranded (if specified)
- [ ] Test ambiguous collision resolution

#### **Telegraph System Tests**
- [ ] Test telegraph office activates connected offices
- [ ] Test telegraph sabotage stops propagation
- [ ] Test telegraph ownership tracking
- [ ] Test telegraph network separate from railway

#### **Save/Load State Tests**
- [ ] Test save with deployed heavy units
- [ ] Test save with carried units
- [ ] Test save with train compositions
- [ ] Test save with destroyed infrastructure
- [ ] Test save with station ownership
- [ ] Test load restores all states correctly

#### **UI/UX Improvement Tests**
- [ ] Test attack range indicators accurate
- [ ] Test movement path visualization correct
- [ ] Test valid move tiles highlight correctly
- [ ] Test invalid move tiles highlight correctly
- [ ] Test active units glow
- [ ] Test inactive units dim
- [ ] Test selected unit has border
- [ ] Test battle outcome colors correct

#### **Pause Menu Tests**
- [ ] Test pause during planning phase
- [ ] Test pause during resolution phase
- [ ] Test resume continues game
- [ ] Test save from pause menu
- [ ] Test exit to main menu
- [ ] Test options access

### **Success Criteria**

✅ All unit special abilities work as specified
✅ All edge cases handled without crashes
✅ Save/load preserves complete game state (including all special states)
✅ Visual feedback clearly communicates all game information
✅ Battle scene provides clear battle outcomes
✅ Pathfinding works correctly for all unit types
✅ Transport system works flawlessly
✅ All bugs from Phase 1 are fixed

### **Placeholder Assets**

- **Battle scene**: Simple animation (flash for attack, explosion effect)
- **Unit markers**: Shape variations (circles, squares, triangles, hexagons, rectangles)
- **Special abilities**: Visual indicators (deployed state = different color, carried state = hidden)
- **Railway**: Track direction indicators (arrows showing direction)

---

## **PHASE 3: AI OPPONENT**

**Objective**: Implement AI for single-player vs computer mode.

### **Deliverables**

1. **AI architecture**
   - Perfect information access (sees entire board)
   - Decision-making system
   - Unit evaluation scoring
   - Threat assessment
   - Order generation (movement + attack)

2. **AI strategy behaviors**
   - **Harvester**: Prioritize capturing neutral railway stations to expand "Active" zone
   - **Saboteur**: Move infantry onto rail/telegraph tiles, order "Sabotage" to cut enemy supply lines
   - **Train Driver**: Use armored trains aggressively, exploit collision rules
   - **Defender**: Protect key positions (stations, relays)
   - **Aggressor**: Press attacks on weak enemy units

3. **AI difficulty levels**
   - **Easy**: Random moves, low strategy depth
   - **Normal**: Balanced strategy, medium depth
   - **Hard**: Optimal strategy, high depth, considers counter-moves

4. **AI game mode handling**
   - Remove Iron Curtain phase (not needed)
   - Remove Reveal Orders phase (not needed)
   - AI generates orders immediately after player
   - Skip to CALCULATE_MOVEMENT directly

5. **AI thinking time control**
   - Instant mode (AI moves immediately)
   - Delayed mode (1-3 second delay for realism)
   - Adjustable in options

6. **Game mode selection**
   - Main menu: "Hot Seat" vs "vs AI"
   - Difficulty selection (for vs AI)
   - AI-specific UI (opponent is computer indicator)

7. **AI order generation**
   - Generate 5 movement orders
   - Generate 1 attack order
   - Validate orders against game rules
   - Handle edge cases (no valid moves, no valid attacks)

8. **AI debugging tools**
   - Visualize AI decision process (optional)
   - Show AI threat map (optional)
   - Show AI unit evaluations (optional)

### **Testing Procedures**

#### **Easy Difficulty Tests**
- [ ] Play 20 games vs easy AI
- [ ] Verify AI wins are fair (no obvious cheating)
- [ ] Verify AI uses all unit types
- [ ] Verify AI respects simultaneous turn rules
- [ ] Verify AI doesn't crash on edge cases

#### **Normal Difficulty Tests**
- [ ] Play 20 games vs normal AI
- [ ] Verify AI wins ~30-50% of games
- [ ] Verify AI strategy is coherent
- [ ] Verify AI adapts to player moves
- [ ] Verify AI uses special abilities (sabotage, pack/deploy)

#### **Hard Difficulty Tests**
- [ ] Play 20 games vs hard AI
- [ ] Verify AI wins ~50-70% of games
- [ ] Verify AI uses optimal strategies
- [ ] Verify AI considers counter-moves
- [ ] Verify AI exploits player mistakes

#### **AI Behavior Tests**
- [ ] Test Harvester behavior (captures stations)
- [ ] Test Saboteur behavior (cuts supply lines)
- [ ] Test Train Driver behavior (uses trains aggressively)
- [ ] Test Defender behavior (protects key positions)
- [ ] Test Aggressor behavior (presses attacks)

#### **AI Order Generation Tests**
- [ ] Test AI generates exactly 5 movement orders
- [ ] Test AI generates exactly 1 attack order
- [ ] Test AI orders are valid (respect rules)
- [ ] Test AI orders when few units remain
- [ ] Test AI orders when no valid attacks exist
- [ ] Test AI orders when network isolated

#### **AI Edge Case Tests**
- [ ] Test AI with all units destroyed
- [ ] Test AI with network completely isolated
- [ ] Test AI with only infantry
- [ ] Test AI with only trains
- [ ] Test AI on small map
- [ ] Test AI on large map

#### **AI Performance Tests**
- [ ] Test AI generates orders in <1 second (instant mode)
- [ ] Test AI thinking delay works correctly (1-3 seconds)
- [ ] Test AI doesn't lag game (60 FPS maintained)
- [ ] Test AI memory usage is reasonable

### **Success Criteria**

✅ AI provides challenging but fair gameplay
✅ AI wins at least 30% of games on normal difficulty
✅ AI wins at least 50% of games on hard difficulty
✅ AI handles all unit types correctly
✅ AI respects all game rules (simultaneous turns, network, etc.)
✅ AI doesn't crash or infinite loop
✅ AI games take <5 minutes per turn (including thinking time)
✅ AI mode is playable and enjoyable

### **Placeholder Assets**

- Same as Phase 2 (no new assets needed)
- AI thinking indicator: Simple "Computer is thinking..." text

---

## **PHASE 4: MAP EDITOR & CUSTOM MAPS**

**Objective**: Create system for building, saving, and loading custom maps.

### **Deliverables**

1. **Map editor scene**
   - Tile painting interface
   - Tile palette (all tile types accessible)
   - Grid display (20x25 default)
   - Paint tool (click to place tile)
   - Erase tool (remove tile, revert to base terrain)
   - Fill tool (fill area with tile type)

2. **Map metadata editor**
   - Map name input
   - Map description input
   - Victory condition selection (annihilation, capture station, destroy balloon)
   - Victory target selection (for capture station objective)
   - Historical context input (for narrative)

3. **Unit placement tool**
   - Unit palette (all 14 unit types)
   - Select player (Red/White)
   - Place unit on grid
   - Remove unit from grid
   - Unit count display per player

4. **Map validation**
   - Check for passable paths (units can reach objectives)
   - Check victory conditions set correctly
   - Check at least one unit per player
   - Check station ownership requirements
   - Display validation errors clearly

5. **Save custom maps**
   - Save to user directory (user://maps/)
   - Save map tile data
   - Save unit placements
   - Save metadata
   - Auto-generate unique filename

6. **Load custom maps**
   - Load from user directory
   - Load map tile data
   - Load unit placements
   - Load metadata
   - Validate map on load

7. **Map management UI**
   - List all saved custom maps
   - Rename custom maps
   - Delete custom maps
   - Preview maps (thumbnail or mini-view)
   - Sort by name, date, etc.

8. **Default map templates**
   - Blank 20x25 map (all grass)
   - Pre-configured terrain templates (urban, forest, plains)
   - Template selection on new map

9. **Export/import map sharing**
   - Export map to file (.json or .map)
   - Import map from file
   - Share maps between players
   - Import validation

10. **Integration with game modes**
    - Play custom maps in Hot Seat mode
    - Play custom maps in vs AI mode
    - Select custom maps from level select
    - Campaign mode integration (optional)

### **Testing Procedures**

#### **Map Editor Interface Tests**
- [ ] Test painting tiles works correctly
- [ ] Test erasing tiles reverts to base terrain
- [ ] Test fill tool works for areas
- [ ] Test all tile types accessible in palette
- [ ] Test grid displays correctly
- [ ] Test undo/redo (if implemented)

#### **Map Metadata Tests**
- [ ] Test map name saves correctly
- [ ] Test map description saves correctly
- [ ] Test victory condition selection works
- [ ] Test victory target selection works (for capture station)
- [ ] Test historical context saves correctly

#### **Unit Placement Tests**
- [ ] Test placing units works correctly
- [ ] Test removing units works correctly
- [ ] Test selecting player (Red/White) works
- [ ] Test unit count displays correctly
- [ ] Test all 14 unit types can be placed
- [ ] Test units can be placed on valid tiles only

#### **Map Validation Tests**
- [ ] Test validation catches impossible maps (no paths)
- [ ] Test validation catches missing victory conditions
- [ ] Test validation catches missing units
- [ ] Test validation allows valid maps
- [ ] Test validation errors are clear and helpful
- [ ] Test validation catches station ownership issues

#### **Save/Load Custom Maps Tests**
- [ ] Test saving custom map to user directory
- [ ] Test loading custom map from user directory
- [ ] Test save preserves all tile data
- [ ] Test save preserves all unit placements
- [ ] Test save preserves metadata
- [ ] Test load restores map correctly
- [ ] Test multiple maps can be saved

#### **Map Management Tests**
- [ ] Test map list displays all saved maps
- [ ] Test rename custom map works
- [ ] Test delete custom map works
- [ ] Test preview displays correctly (if implemented)
- [ ] Test sorting works (by name, date)

#### **Default Templates Tests**
- [ ] Test blank 20x25 template loads
- [ ] Test urban template loads with city blocks
- [ ] Test forest template loads with trees
- [ ] Test plains template loads with rails

#### **Export/Import Tests**
- [ ] Test export map to file works
- [ ] Test import map from file works
- [ ] Test imported map plays correctly
- [ ] Test import validation catches corrupted files
- [ ] Test export/import preserves all data

#### **Game Integration Tests**
- [ ] Test custom map plays in Hot Seat mode
- [ ] Test custom map plays in vs AI mode
- [ ] Test custom map selectable from level select
- [ ] Test custom map respects victory conditions
- [ ] Test AI plays on custom maps correctly

#### **Edge Case Tests**
- [ ] Test creating map with maximum units
- [ ] Test creating map with minimum units
- [ ] Test creating map with no terrain obstacles
- [ ] Test creating map with all terrain obstacles
- [ ] Test saving map with no units (should fail validation)
- [ ] Test loading corrupted map file (should fail gracefully)

### **Success Criteria**

✅ Player can create a playable custom map within 10 minutes
✅ All created maps validate correctly
✅ Custom maps work in all game modes (Hot Seat, vs AI)
✅ Map editor is intuitive and bug-free
✅ Save/load preserves complete map state
✅ Export/import allows map sharing between players
✅ Default templates provide good starting points

### **Placeholder Assets**

- **Map editor UI**: Simple painting tools with clear tile indicators
- **Tile palette**: Colored squares representing tile types
- **Unit palette**: Colored shapes representing unit types
- **Map preview**: Mini-view of map (simple grid display)

---

## **PHASE 5: NARRATIVE ENGINE & DEMO CAMPAIGN**

**Objective**: Implement campaign system and create 3 demo campaign maps.

### **Deliverables**

1. **Narrative manager system**
   - Story state tracking (current level, completed levels)
   - Level unlock system (must complete previous level)
   - Narrative data structure (historical context, objectives)
   - Campaign save system (separate from game saves)

2. **Narrative screen**
   - Historical context display (text)
   - Mission objectives display
   - Victory conditions display
   - "Start Mission" button
   - Skip narrative option

3. **Campaign progression**
   - Level 1 unlock (unlocked by default)
   - Level 2 unlock (complete Level 1)
   - Level 3 unlock (complete Level 2)
   - Level select screen (shows locked/unlocked status)
   - Campaign completion state

4. **Campaign save system**
   - Save campaign progress (completed levels)
   - Load campaign progress
   - Separate save slots from game saves
   - Auto-save campaign after level completion

5. **Victory/defeat screens**
   - Victory screen with campaign context
   - Defeat screen with retry option
   - "Next Level" button (if victory)
   - "Retry Level" button (if defeat)
   - "Return to Menu" button

6. **Campaign completion summary**
   - Total turns played
   - Units destroyed
   - Missions completed
   - "Congratulations" message

7. **Map 1: "The Iron Horse"**
   - **Setting**: Flatlands with extensive rail network
   - **Terrain**: Mostly open grass, large rail network, few obstacles
   - **Focus**: Armored trains, sabotage, long-range artillery
   - **Victory Condition**: Capture Central Rail Hub (Source)
   - **Historical Context**: "In early 1919, the Red Army launched an offensive to secure key railway junctions in western Russia. The White Army's armored trains dominated the flatlands, but the Reds utilized sabotage tactics to disrupt supply lines."
   - **Red Army Setup**: 3 Infantry, 1 Artillery, 1 Armored Train
   - **White Army Setup**: 3 Infantry, 1 Swift Artillery, 1 Armored Train

8. **Map 2: "Red Petrograd"**
   - **Setting**: Dense urban environment
   - **Terrain**: City grid, city blocks (mountains), plazas (defensible)
   - **Focus**: Maxim machine guns, street fighting, plazas as fortresses
   - **Victory Condition**: Annihilation (destroy all enemy units)
   - **Historical Context**: "The battle for Petrograd (St. Petersburg) was brutal urban warfare. City blocks created natural choke points, while plazas became makeshift fortresses. Maxim machine guns dominated the narrow streets, making any advance costly."
   - **Red Army Setup**: 4 Infantry, 2 Maxim Nests, 1 Machine Gun Nest
   - **White Army Setup**: 4 Infantry, 1 Machine Gun Nest, 1 Swift Relay

9. **Map 3: "The Woods of Bryansk"**
   - **Setting**: Forested guerrilla warfare zone
   - **Terrain**: Forests (impassable or LoC blockers), sparse telegraph lines
   - **Focus**: Cavalry, Scouts, Observation Balloons (seeing over trees)
   - **Victory Condition**: Destroy enemy Observation Balloon (cut their eyes) and survive
   - **Historical Context**: "In the Bryansk forests, White partisans used guerrilla tactics against Red columns. Observation balloons were crucial for detecting enemy movements through the dense forest cover. The race was on to blind the enemy before being blinded yourself."
   - **Red Army Setup**: 2 Cavalry, 1 Scout, 1 Observation Balloon, 1 Artillery
   - **White Army Setup**: 2 Cavalry, 1 Scout, 1 Observation Balloon, 1 Swift Artillery

10. **Campaign menu**
    - Level list with locked/unlocked status
    - Level thumbnails (map preview)
    - "Start Campaign" button (begins at unlocked level)
    - "Reset Campaign" button (start over)
    - Campaign progress display

### **Testing Procedures**

#### **Narrative System Tests**
- [ ] Test narrative screen displays correctly
- [ ] Test historical context displays accurately
- [ ] Test objectives display correctly
- [ ] Test "Start Mission" button works
- [ ] Test skip narrative option works

#### **Campaign Progression Tests**
- [ ] Test Level 1 unlocked by default
- [ ] Test completing Level 1 unlocks Level 2
- [ ] Test completing Level 2 unlocks Level 3
- [ ] Test level select shows locked/unlocked status correctly
- [ ] Test cannot play locked levels
- [ ] Test can replay completed levels

#### **Campaign Save/Load Tests**
- [ ] Test campaign progress saves after level completion
- [ ] Test campaign progress loads correctly
- [ ] Test campaign saves separate from game saves
- [ ] Test reset campaign works

#### **Victory/Defeat Screen Tests**
- [ ] Test victory screen displays after winning
- [ ] Test defeat screen displays after losing
- [ ] Test "Next Level" button appears after victory
- [ ] Test "Retry Level" button appears after defeat
- [ ] Test "Return to Menu" button works
- [ ] Test retrying level from defeat screen

#### **Map 1 Tests (The Iron Horse)**
- [ ] Play map 1 and win via station capture
- [ ] Verify victory condition works
- [ ] Verify balance (challenging but winnable)
- [ ] Test sabotage mechanics are useful
- [ ] Test train mechanics are central to gameplay
- [ ] Test historical context displays correctly
- [ ] Test level completion unlocks Level 2

#### **Map 2 Tests (Red Petrograd)**
- [ ] Play map 2 and win via annihilation
- [ ] Verify victory condition works
- [ ] Verify balance (challenging but winnable)
- [ ] Test Maxim machine guns dominate streets
- [ ] Test plazas provide defensive bonus
- [ ] Test city blocks block movement and LoC
- [ ] Test historical context displays correctly
- [ ] Test level completion unlocks Level 3

#### **Map 3 Tests (The Woods of Bryansk)**
- [ ] Play map 3 and win via balloon destruction
- [ ] Verify victory condition works
- [ ] Verify balance (challenging but winnable)
- [ ] Test cavalry useful in forests
- [ ] Test observation balloons see over obstacles
- [ ] Test sparse telegraph lines create strategic decisions
- [ ] Test historical context displays correctly
- [ ] Test campaign completes after victory

#### **Campaign Completion Tests**
- [ ] Test completing all 3 levels shows summary screen
- [ ] Test summary displays total turns
- [ ] Test summary displays units destroyed
- [ ] Test summary displays congratulations message
- [ ] Test can restart campaign after completion

#### **AI on Campaign Maps Tests**
- [ ] Test AI plays Map 1 correctly
- [ ] Test AI plays Map 2 correctly
- [ ] Test AI plays Map 3 correctly
- [ ] Verify AI respects victory conditions

#### **Hot Seat on Campaign Maps Tests**
- [ ] Test Hot Seat mode works on Map 1
- [ ] Test Hot Seat mode works on Map 2
- [ ] Test Hot Seat mode works on Map 3

### **Success Criteria**

✅ All 3 campaign maps are playable and balanced
✅ Each map provides ~30-45 minutes of gameplay
✅ Narrative system provides good historical context
✅ Campaign progression works correctly (unlock system)
✅ Victory/defeat screens work as expected
✅ Campaign provides 2-3 hours of total gameplay
✅ Both Hot Seat and vs AI modes work on campaign maps
✅ Campaign completion summary displays correctly
✅ Historical context is accurate and engaging

### **Placeholder Assets**

- **Narrative screens**: Simple text on colored background
- **Campaign menu**: Simple level list with map thumbnails (mini-grid display)
- **Victory/defeat screens**: Simple colored backgrounds with text
- **Level thumbnails**: Mini grid display of map layout
- **All game assets**: Still placeholders from previous phases

---

## **PHASE 6: PRODUCTION ASSETS (Post-Demo)**

**Objective**: Replace all placeholder assets with production-quality assets.

**Note**: This phase is outside the scope of getting a playable demo and should be scheduled after Phase 5 is complete and gameplay is solid.

### **Deliverables**

1. **Final unit sprites/artwork**
   - 14 unit types × 2 armies = 28 unique sprites
   - Front view (for battle scene)
   - Top-down view (for game board)
   - Multiple frames (if animating)
   - Size variations consistent with unit types

2. **Final terrain artwork**
   - All tile types with textures
   - Base terrain (grass, snow)
   - Obstacles (mountains, city blocks)
   - Infrastructure (rails, roads, telegraph lines)
   - Buildings (stations, telegraph offices, plazas)
   - Special tiles (turntables, intersections)

3. **Final UI artwork**
   - Main menu background
   - Level select screen
   - Buttons (all UI elements)
   - Panels (info panels, order lists, etc.)
   - Icons (unit types, terrain types, actions)
   - Victory/defeat screens
   - Narrative screens

4. **Sound effects**
   - Move sound (unit movement)
   - Attack sound (weapon fire)
   - Explosion sound (destruction)
   - Victory fanfare
   - Defeat sound
   - Button clicks
   - Order confirmed
   - Sabotage effect
   - Train movement sound

5. **Background music**
   - Main menu theme
   - Battle theme (battle scene)
   - Campaign theme (narrative screens)
   - Victory theme
   - Defeat theme

6. **Battle scene animations**
   - Unit sprites (profile view)
   - Attack animations (fire, charge)
   - Impact effects (explosion, damage)
   - Retreat animation
   - Destruction animation

7. **Font selection and typography**
   - Main font (UI text)
   - Header font (titles)
   - Historical text font (narrative)
   - Legible at small sizes (mobile)

8. **Icon artwork**
   - Unit type icons (14 types)
   - Terrain type icons (all tile types)
   - Action icons (sabotage, pack, deploy, embark, disembark)
   - UI icons (save, load, options, etc.)

9. **Visual polish**
   - Animations (unit movement, transitions)
   - Particle effects (explosions, smoke, sparks)
   - Screen shake (explosions, train collisions)
   - Lighting effects (day/night cycle, if desired)
   - Smooth transitions between scenes

10. **Mobile touch optimization**
    - Touch targets appropriately sized
    - Touch gestures (pinch-zoom, two-finger pan)
    - Responsive layout for small screens
    - Portrait and landscape support

### **Testing Procedures**

#### **Asset Quality Tests**
- [ ] Verify all unit sprites are visually distinct
- [ ] Verify terrain tiles are visually clear
- [ ] Verify UI artwork is cohesive and professional
- [ ] Verify all icons are recognizable

#### **Performance Tests**
- [ ] Verify 60 FPS with all assets loaded
- [ ] Verify memory usage is acceptable
- [ ] Verify load times are reasonable

#### **Audio Tests**
- [ ] Verify all sound effects play correctly
- [ ] Verify sound effects don't overlap excessively
- [ ] Verify background music loops smoothly
- [ ] Verify volume controls work

#### **Animation Tests**
- [ ] Verify battle animations are smooth
- [ ] Verify particle effects look good
- [ ] Verify transitions are polished
- [ ] Verify mobile gestures work smoothly

### **Success Criteria**

✅ All placeholder assets replaced with production-quality assets
✅ Visual style is cohesive and professional
✅ Audio enhances gameplay experience
✅ Performance remains 60 FPS on target hardware
✅ Game looks and feels like a complete product

---

## **DEVELOPMENT PRIORITY & TIMING**

### **Critical Path (Must Do In Order)**
1. **Phase 1** (Hot Seat Multiplayer) → **This is the foundation**
   - All core systems must work before proceeding
   - Do not skip testing

2. **Phase 2** (Mechanics Verification & Polish)
   - Verify Phase 1 works completely
   - Add all edge cases and special abilities
   - Ensure save/load works perfectly

3. **Phase 5** (Narrative Engine & Demo Campaign)
   - Creates actual playable content
   - Provides ~2-3 hours of gameplay
   - Demonstrates complete game

### **Parallel Development (Can Do Concurrently)**
- **Phase 3** (AI Opponent) can start once Phase 1 is complete
  - Doesn't depend on Phase 2 or Phase 5
  - Can develop independently

- **Phase 4** (Map Editor & Custom Maps) can start once Phase 1 is complete
  - Doesn't depend on Phase 2, Phase 3, or Phase 5
  - Can develop independently

### **Recommended Timeline (Solo Developer)**

| Phase | Duration | Dependencies | Priority |
|--------|-----------|---------------|-----------|
| Phase 1: Hot Seat | 4-6 weeks | None | CRITICAL |
| Phase 2: Mechanics | 2-3 weeks | Phase 1 | HIGH |
| Phase 3: AI | 3-4 weeks | Phase 1 | MEDIUM |
| Phase 4: Map Editor | 2-3 weeks | Phase 1 | MEDIUM |
| Phase 5: Campaign | 2-3 weeks | Phase 1, 2 | HIGH |
| **Total to Playable Demo** | **13-19 weeks** | | |
| Phase 6: Assets | 4-6 weeks | Phase 1-5 | LOW (post-demo) |

**Total Time to Complete Product**: 17-25 weeks (4-6 months)

---

## **RISK MITIGATION**

### **Phase 1 Risks**

**Risk**: Complex collision resolution causing bugs
- **Mitigation**: Implement basic collision first, add edge cases incrementally
- **Backup**: If collision is too complex, simplify to "first come, first served" initially

**Risk**: Network system performance issues
- **Mitigation**: Cache active states, recalculate only when necessary
- **Backup**: Reduce map size or unit count if performance is poor

**Risk**: Railway system complexity
- **Mitigation**: Start with basic movement, add special tiles (turntables, intersections) later
- **Backup**: Remove turntables initially if they're too complex

### **Phase 2 Risks**

**Risk**: Too many edge cases to handle
- **Mitigation**: Focus on common cases first, log edge cases for later polish
- **Backup**: Defer rare edge cases to post-demo

**Risk**: Save/load state complexity
- **Mitigation**: Keep save format simple, add validation
- **Backup**: If save/load is too complex, implement basic save only (no load)

**Risk**: Special abilities interaction bugs
- **Mitigation**: Test each ability in isolation before combining
- **Backup**: Disable complex abilities (embark/disembark) initially

### **Phase 3 Risks**

**Risk**: AI too weak or too strong
- **Mitigation**: Implement difficulty tuning as adjustable parameters
- **Backup**: If AI is too complex, start with random moves only

**Risk**: AI decision-making performance
- **Mitigation**: Limit AI search depth, use heuristics
- **Backup**: Pre-calculate AI decisions if real-time is too slow

**Risk**: AI doesn't handle edge cases
- **Mitigation**: AI validation ensures orders are always valid
- **Backup**: AI defaults to "pass" if no valid orders found

### **Phase 4 Risks**

**Risk**: Map editor UI complexity
- **Mitigation**: Start with basic painting, add features incrementally
- **Backup**: Remove fill tool if it's too complex

**Risk**: Invalid maps created by players
- **Mitigation**: Robust validation with clear error messages
- **Backup**: Auto-fix invalid maps (remove impossible units)

### **Phase 5 Risks**

**Risk**: Campaign maps unbalanced
- **Mitigation**: Playtest extensively, adjust unit counts and terrain
- **Backup**: Provide multiple difficulty variants of each map

**Risk**: Narrative system bugs
- **Mitigation**: Keep it simple (text screens, no branching)
- **Backup**: Remove narrative system if it's too buggy, focus on gameplay

---

## **SUCCESS METRICS**

### **Phase 1 Success**
- Can complete a full 2-player game without crashes
- All core rules implemented correctly
- Playable within 1 hour of learning game
- Average game length: 30-60 minutes
- No game-breaking bugs

### **Phase 2 Success**
- Zero game-breaking bugs
- All unit types functional
- Save/load works reliably
- All edge cases handled
- Visual feedback is clear

### **Phase 3 Success**
- AI wins 30-50% of games on normal difficulty
- AI wins 50-70% of games on hard difficulty
- AI games take <5 minutes per turn
- AI doesn't crash or infinite loop
- AI respects all game rules

### **Phase 4 Success**
- Can create a playable map in 10 minutes
- Map editor is intuitive
- Custom maps work correctly
- Map validation catches invalid maps
- Export/import allows sharing

### **Phase 5 Success**
- Campaign provides 2-3 hours of gameplay
- All 3 maps are balanced and fun
- Narrative adds historical context without being intrusive
- Campaign progression works correctly
- Both Hot Seat and AI modes work

---

## **NEXT STEPS**

1. **Start Phase 1**: Begin implementing hot seat multiplayer prototype
2. **Create initial project structure**: Set up directory structure from ProjectStructure.md
3. **Implement core systems in order**:
   - TurnStateMachine
   - OrderManager
   - MovementSystem
   - CollisionSystem
   - BattleSystem
   - NetworkSystem
   - RailwaySystem
4. **Test continuously**: Don't wait until end of Phase 1 to test
5. **Iterate quickly**: Use placeholder assets, focus on gameplay
6. **Document bugs**: Keep a bug tracker for Phase 2 fixes

---

**Last Updated**: January 7, 2026
**Version**: 1.0