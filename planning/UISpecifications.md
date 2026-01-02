# **UI/UX SPECIFICATIONS**

**Godot 4.x | Mobile/PC | Industrial Revolution Theme | Russian Civil War Era**

---

## **DESIGN PHILOSOPHY**

### **1. Two-Phase Development**

**Phase 1: Functional Prototype** (Current Goal)
- Basic, readable, accessible UI
- Clean information display
- Responsive to input
- Placeholder graphics (colored shapes, simple text)

**Phase 2: Thematic Expression** (Post-Prototype)
- Rich Russian Revolution aesthetic
- Historical imagery and photography
- Industrial/mechanical theme
- Animated elements and transitions
- Period-appropriate sound design

### **2. Core Principles**
- **Accessibility First**: Dyslexia-friendly fonts, scalable vector assets, high contrast
- **Undo-Everything**: Players can experiment before confirming
- **Clear Feedback**: Every action has visual and audio feedback
- **Minimal Click Depth**: 1-2 clicks to perform any action
- **Touch-Optimized**: Large touch targets, intuitive gestures

---

## **INPUT SYSTEM**

### **Primary Actions**

```gdscript
# Left Click / Tap
- Select unit
- Select tile (for movement or attack)
- Click button
- Dismiss menu
- Confirm action

# Right Click / Long Tap (Hold > 0.5s)
- Special action menu (Sabotage, Pack/Deploy, Embark/Disembark)
- Unit info detail view
- Tile info detail view
- Context menu
```

### **Gestures**

```gdscript
# Mouse
- Drag: Pan camera
- Scroll wheel: Zoom in/out
- Right-click drag: Quick pan

# Touch
- One-finger drag: Pan camera
- Two-finger pinch: Zoom in/out
- Two-finger drag: Pan camera
- Long tap: Context menu (special actions)
- Swipe (optional): Scroll through lists
```

### **Input Feedback**

```gdscript
# Visual
- Hover: Highlight unit/tile with outline
- Selected: Bright highlight with glow
- Invalid action: Red outline or shake animation
- Valid action: Green outline or subtle pulse

# Audio
- Select: Subtle click
- Confirm: Satisfying "thud" or mechanical click
- Invalid: Low "buzzer" or dull thud
- Error: Soft "ding" with negative tone
```

---

## **PHASE 1 UI: FUNCTIONAL PROTOTYPE**

### **Main Menu Scene**

**Layout:**
```
+----------------------------------+
|                                  |
|       GAME OF WAR: RUSSIAN        |
|       REVOLUTION EDITION            |
|                                  |
|    [Start Campaign]                |
|    [Quick Battle]                  |
|    [Level Select]                  |
|    [Options]                      |
|    [Exit]                         |
|                                  |
+----------------------------------+
```

**Components:**
- **Title Label**: Large, centered, serif font
- **Buttons**:
  - Start Campaign (Start new campaign)
  - Quick Battle (Single map skirmish)
  - Level Select (Choose specific map)
  - Options (Settings)
  - Exit (Quit game)

**Behavior:**
- Simple button layout, centered vertically
- Hover effects: Scale up slightly, color shift
- Click effects: Button press animation (scale down then up)
- Navigation: Up/down arrows or click to select

**Accessibility:**
- Dyslexia-friendly serif font (e.g., Open Dyslexic, Readex Pro)
- High contrast (dark background, light text)
- Large touch targets (minimum 44x44 pixels on mobile)
- Keyboard navigation support

---

### **Level Select Scene**

**Layout:**
```
+----------------------------------+
|    Select Map                   |
|                                  |
|  [Map 1: The Iron Horse]       |
|    Flatlands & Rail              |
|                                  |
|  [Map 2: Red Petrograd]        |
|    Urban Warfare                 |
|                                  |
|  [Map 3: Woods of Bryansk]     |
|    Guerrilla War                 |
|                                  |
|       [Back]    [Start]         |
+----------------------------------+
```

**Components:**
- **Map List**: Scrollable list of campaign maps
- **Map Cards**: Each card shows:
  - Map name (bold, large)
  - Brief description (smaller)
  - Optional: Thumbnail (placeholder for Phase 1)
- **Navigation**: Back button, Start button

**Behavior:**
- Click map card to select
- Selected card: Highlighted border, scale up slightly
- Scroll if more than 3 maps
- Start button enabled only when map selected

**Accessibility:**
- Same font as main menu
- Clear selection indicator
- Keyboard: Arrow keys to navigate, Enter to select

---

### **Options Scene** (Phase 1 - Basic)

**Layout:**
```
+----------------------------------+
|    Options                        |
|                                  |
|  [x] Show Battle Animations       |
|  [ ] Individual Battle Skip        |
|  [ ] Auto-Skip All Battles       |
|                                  |
|  Battle Animation Speed            |
|  [Slow] [Normal] [Fast]         |
|                                  |
|  Music Volume: [=====-]          |
|  Sound Volume: [=======-]         |
|                                  |
|  [ ] Turn Timer (Hot Seat)       |
|  Timer Duration: [10] seconds     |
|                                  |
|         [Back]                    |
+----------------------------------+
```

**Components:**
- **Toggle Options**: Checkboxes for boolean settings
- **Radio Buttons**: For mutually exclusive options (animation speed)
- **Sliders**: For volume controls
- **Number Input**: For timer duration
- **Back Button**: Return to main menu

**Behavior:**
- Toggle: Click to toggle on/off with animation
- Radio: Click to select, others deselect
- Sliders: Drag or click to set value
- Settings save immediately to player preferences

**Accessibility:**
- Clear labels above each option
- Large checkboxes and radio buttons
- Sliders with clear value indicators

---

### **Narrative Screen** (Phase 1 - Basic)

**Layout:**
```
+----------------------------------+
|                                  |
|     [Portrait Placeholder]          |
|      (Optional for Phase 1)        |
|                                  |
|  +------------------------------+ |
|  | Historical Context:          | |
|  |                             | |
|  | In early 1919, the Red    | |
|  | Army launched an...        | |
|  +------------------------------+ |
|                                  |
|           [Continue]               |
+----------------------------------+
```

**Components:**
- **Portrait Area**: Placeholder rectangle (Phase 1), actual portrait (Phase 2)
- **Text Box**: Scrollable text area showing narrative
- **Continue Button**: Proceed to game

**Behavior (Phase 1):**
- Text appears instantly (no typewriter effect yet)
- No animations or cutaways yet
- Simple text with basic formatting

**Behavior (Phase 2):**
- Typewriter effect: Text appears character-by-character
- Click to skip typewriter
- Portrait animations: Breathing, blinking, subtle movements
- Cutaways: Occasional images/video appear in portrait area
- Dispatch effect: Text appears as typed telegram

**Accessibility:**
- Readable serif font
- High contrast text box (dark background, light text)
- Text size adjustable in options
- Click anywhere to continue (not just button)

---

### **Game Board UI** (Phase 1 - Basic)

**Layout:**
```
+----------------------------------+
| [Turn 5 | Red Army's Turn]     |  <- Top HUD
+----------------------------------+
|                                  |
|                                  |
|       [Game Board]                |
|       (TileMap with units)         |
|                                  |
|                                  |
+----------------------------------+
| Movements: 3/5  Attacks: 0/1   |  <- Bottom HUD
| [Undo] [Special Actions] [Ready] |
+----------------------------------+
```

**Components:**

#### **Top HUD**
- **Turn Counter**: "Turn X" display
- **Current Player**: "Red Army's Turn" or "White Army's Turn"
- **Phase Indicator**: "Planning Phase" / "Resolution Phase"

#### **Game Board**
- **TileMap**: Main gameplay area
- **Units**: Unit markers on tiles
- **Camera Controls** (if needed):
  - Zoom in/out buttons
  - Pan buttons (or drag-to-pan)

#### **Bottom HUD**
- **Movement Counter**: "Movements: X/5"
- **Attack Counter**: "Attacks: X/1"
- **Undo Button**: Remove last order
- **Special Actions Button**: Open special action menu
- **Ready Button**: Submit orders and end planning phase

**Behavior:**

**Unit Selection:**
- Click unit to select
- Show movement range (highlighted tiles)
- Show attack range (red outline on valid targets)
- Display unit info in side panel (if screen allows)

**Issuing Orders:**
- Click valid tile to move unit
- Click valid target to attack
- Each movement consumes 1 from counter
- Attack consumes 1 from counter
- Ghost units show planned movement

**Undo System:**
- Click "Undo" to remove last order
- Undo removes movement OR attack (most recent)
- Can undo all orders back to start of turn
- Orders stored in stack for easy reversal

**Special Actions Menu:**
- Opens on click (or right-click unit)
- Shows available actions:
  - Sabotage (if on destroyable terrain)
  - Pack (if heavy unit and deployed)
  - Deploy (if heavy unit and packed)
  - Embark (if on carrier)
  - Disembark (if carried)
- Click action to execute
- Some actions consume movement (check rules)

**Ready Button:**
- Enabled when orders are valid
- Disabled if:
  - Movement orders > 5
  - Attack orders > 1
  - Invalid orders present
- Click to submit and end planning phase

**Accessibility:**
- Large, readable fonts
- High contrast indicators
- Clear color coding (green = valid, red = invalid)
- Keyboard shortcuts (optional):
  - U: Undo
  - S: Special actions
  - Enter: Ready

---

### **Iron Curtain Scene** (Hot Seat Only)

**Layout:**
```
+----------------------------------+
|                                  |
|                                  |
|      PASS DEVICE TO                |
|      PLAYER WHITE                  |
|                                  |
|                                  |
|                                  |
|                                  |
|                                  |
|                                  |
|     [I'm Ready]                   |
|                                  |
+----------------------------------+
```

**Components:**
- **Message Label**: "Pass device to Player X"
- **Subtitle**: "Player X click 'I'm Ready' when ready"
- **Ready Button**: Large, centered button

**Behavior:**
- Covers entire screen (full overlay)
- Hides all game UI
- Simple, clear message
- No timer (manual "I'm Ready" only)

**Accessibility:**
- Very large text
- High contrast
- Large touch target (entire bottom half of screen)

---

### **Reveal Orders Scene** (Hot Seat Only)

**Layout:**
```
+----------------------------------+
| Turn 5: Review Orders             |  <- Top
+----------------------------------+
| Red Army Orders    White Army Orders |  <- Side Panels
| [Infantry move to A] [Cavalry attack B]|
| [Artillery move to C] [Relay move to D]  |
|                                  |
+----------------------------------+
| [Battles this turn: 2]           |  <- Center
|  Infantry vs Cavalry (Tile A)     |
|  Artillery vs Relay (Tile C)      |
|                                  |
+----------------------------------+
|      [Begin Turn]                 |  <- Bottom
+----------------------------------+
```

**Components:**
- **Turn Counter**: "Turn X: Review Orders"
- **Order Lists**: Side-by-side panels showing each player's orders
- **Battle Preview**: Center panel showing battles this turn
- **Begin Turn Button**: Start resolution

**Visual Feedback:**
- Ghost units on board (semi-transparent)
- Attack lines (red arrows from attacker to target)
- Collision warnings (yellow on overlapping movements)
- Order count: "Movements: X/5 | Attacks: X/1"

**Behavior:**
- Both players review orders
- Ghost units show planned positions
- Attack lines show planned attacks
- Click "Begin Turn" to start resolution

**Accessibility:**
- Clear visual separation between players
- Color coding (Red Army = red, White Army = white/blue)
- Battle preview with clear attacker/defender distinction

---

### **Battle Scene** (Phase 1 - Basic)

**Layout:**
```
+----------------------------------+
| Attack: 8  Defense: 6           |  <- Top
+----------------------------------+
|                                  |
|   [Attacker Sprites]   [Defender  |  <- Center
|    (Left side)          Sprites   |
|                          (Right side)|
|                                  |
+----------------------------------+
|      UNIT DESTROYED               |  <- Bottom
|   [Continue]  [Skip]            |
+----------------------------------+
```

**Components:**
- **Stats Panel**: "Attack: X | Defense: Y"
- **Sprite Container**: Left = attackers, Right = defenders
- **Outcome Text**: Battle result (large, centered)
- **Action Buttons**: Continue (next battle), Skip (skip all)

**Behavior:**
- Show attacker units on left
- Show defender units on right
- Play attack animation
- Show outcome text
- Wait for player to click "Continue"

**Phase 2 Enhancement:**
- "Binoculars" shadow vignette around screen edges
- Silhouette sprites of units (NATO-style markers)
- Dynamic camera shake on impact
- Explosion/destruction effects

**Accessibility:**
- Clear outcome text (large font)
- High contrast colors
- Skip button always visible
- Keyboard: Space = Continue, S = Skip

---

### **Victory/Defeat Screen**

**Layout:**
```
+----------------------------------+
|                                  |
|       VICTORY! (or DEFEAT!)     |
|                                  |
|      [Portrait/Scene Placeholder]    |
|                                  |
|  [Victory/Defeat Message]         |
|                                  |
|  [Back to Menu]  [Retry Level]    |
|                                  |
+----------------------------------+
```

**Components:**
- **Result Label**: "VICTORY!" or "DEFEAT!"
- **Portrait/Scene**: Placeholder (Phase 1), thematic image (Phase 2)
- **Message Text**: Description of outcome
- **Action Buttons**: Back to menu, Retry level

**Behavior:**
- Show immediately after win/loss condition met
- Play victory/defeat music
- Buttons return to appropriate screens

**Phase 2 Enhancement:**
- Thematic portrait (general celebrating victory, or defeated)
- Battle map with arrows showing final positions
- Historical context for outcome

**Accessibility:**
- Clear victory/defeat distinction (color: green/red)
- Readable text
- Large touch targets

---

## **PHASE 2 UI: THEMATIC ENHANCEMENT**

### **Main Menu - Full Theme**

**Visual Style:**
- **Industrial Revolution aesthetic**: Gears, steam pipes, factory background
- **Period Typography**: Early 20th century serif fonts
- **Russian Revolution imagery**: Soviet posters, revolutionary art
- **Animated Elements**: Rotating gears, steam effects

**Dynamic Content:**
- **Variants**: Each menu visit shows different imagery
- **Cutaways**: Period photographs, battle maps, famous figures
- **Dossiers**: Pop-up info about historical figures
- **Battle Maps**: Historical maps with troop movements (arrows)

**Animation:**
- Background: Slowly rotating gears, subtle steam
- Buttons: Mechanical press-down effect
- Transitions: Fade with mechanical wipe effect

**Example Variants:**
1. **Variant A**: Poster of Lenin, Red Army propaganda
2. **Variant B**: Trotsky giving speech, crowd scene
3. **Variant C**: Map of Eastern Front with troop movements
4. **Variant D**: Factory workers, industrial scene
5. **Variant E**: Battle scene from civil war

---

### **In-Game UI - Industrial/Mechanical Theme**

**Visual Style:**
- **Mechanical Computer Panels**: Brass/copper textures, dials, gauges
- **Slide-Rule Displays**: Unit stats shown on slide-rule style readouts
- **Gears and Cogs**: Animated decorations around UI panels
- **Industrial Color Palette**: Brass, copper, dark iron, oil-stained wood

**Unit Info Panel:**
```
+----------------------------+
| MECHANICAL COMPUTER PANEL   |
+----------------------------+
| Unit: INFANTRY            |
|                            |
| ATTACK  =========+ 4      | <- Slide-rule style
| DEFENSE  ==========+ 6      |
| MOVE     ====+ 1          |
| RANGE    =====+ 2          |
|                            |
| STATUS: ACTIVE  [LED ON]   |
+----------------------------+
```

**Animation:**
- Dials rotate when values change
- Gauges move smoothly
- LEDs blink for status
- Mechanical click sounds on interactions

**Expression Opportunities:**
- Background: Factory interior with steam
- Decorative elements: Pipes, valves, gauges
- Panel wear: Rust, oil stains, scratches
- CRT monitor effect: Scanlines, subtle flicker

---

### **Options Menu - Train Board Theme**

**Visual Style:**
- **Train Schedule Board**: Large board with flip-letter display
- **Flip Letters**: Animated as user scrolls through options
- **Period Aesthetic**: Wooden board, brass frame, chalk/marker writing

**Layout:**
```
+----------------------------------+
|    OPTIONS                      |
|  +============================+  |
|  |  SHOW BATTLE ANIMATIONS  |  | <- Flip letters
|  +============================+  |
|       [x] (flip animation)    |
|                                  |
|  +============================+  |
|  |   BATTLE ANIMATION SPEED |  | <- Flip letters
|  +============================+  |
|    [SLOW]  [NORMAL]  [FAST]  | <- Flip letters
|                                  |
|          [BACK]                   |
+----------------------------------+
```

**Animation:**
- **Flip Effect**: Letters flip like train schedule boards
- Each letter/card flips independently with delay
- Sound: Mechanical card-flip or slot-machine sound
- Timing: 0.1-0.2s per letter

**Variants:**
- Different train station backgrounds
- Various period typography styles
- Rust/wear variations for authenticity

---

### **Battle Scene - Binoculars Effect**

**Visual Style:**
- **Vignette**: Dark circular shadow around screen edges
- **Binoculars Frame**: Rounded corners, slight curvature distortion
- **Horseback Perspective**: Camera slightly bobbing, as if on horse

**Layout Enhancement:**
```
+------------------------+
| \                    / |  <- Vignette edges
|  \      BATTLE     /  |
|   \   BINOCULARS   /   |
|    \               /     |
+------------------------+
| Attack: 8 | Defense: 6     |
+------------------------+
| [Attackers]    [Defenders]   |
+------------------------+
|   UNIT DESTROYED            |
|   [Continue]    [Skip]     |
+------------------------+
| /                    \ |  <- Vignette edges
|  /      BATTLE     \   |
|   /   BINOCULARS   \    |
|  /               \      |
+------------------------+
```

**Animation:**
- **Camera Shake**: Slight movement on impact
- **Flash**: Bright flash on attack
- **Smoke/Dust**: Particle effects on explosions
- **Blood/Injury**: Subtle, period-appropriate effects

**Distance-Based Visuals:**
- **Foreground**: Clear, detailed sprites
- **Midground**: Slightly desaturated
- **Background**: More desaturated, lower detail
- **Far**: Heavily desaturated, silhouettes

---

### **Narrative Screen - Visual Novel Style**

**Visual Style:**
- **Fire Emblem Style**: 2D character portraits, text box at bottom
- **Russian Revolution Portraits**: Period dress, uniforms, expressions
- **Dispatch Effect**: Typewriter with carrier sound
- **Typewriter VFX**: Text appears as typed, with blinking cursor

**Layout:**
```
+----------------------------------+
|                                  |
|     [Character Portrait]            |
|      (2D, animated)              |
|                                  |
|  (Cutaway images appear here       |
|   occasionally)                   |
|                                  |
+----------------------------------+
| [DISPATCH FROM FRONT LINE]        | <- Header
| Comrade Commander,                |
| The Red Army has...              |
| (text appears as typed) _         | <- Typing cursor
|                                  |
|                    [Continue]     | <- Button
+----------------------------------+
```

**Character Portraits:**
- **Leon Trotsky**: Party commander, stern expression
- **General Brusilov**: White Army general, weathered face
- **Propaganda Worker**: Enthusiastic, holding poster
- **Peasant Soldier**: Young, determined, dirtied uniform

**Animations:**
- **Breathing**: Subtle chest movement
- **Blinking**: Periodic eye blinks
- **Expression Changes**: React to story beats
- **Lip Sync**: Optional (during voiceover)

**Cutaways:**
- **Historical Photos**: Actual period photographs
- **Battle Maps**: Maps with troop movements
- **Propaganda Posters**: Soviet revolutionary art
- **Factory Scenes**: Industrial locations
- **War Scenery**: Trenches, ruins, cities

**Typewriter Effect:**
- **Character-by-Character**: Text appears slowly
- **Typing Sound**: Satisfying mechanical click
- **Carrier Sound**: Mechanical typewriter carrier return
- **Blinking Cursor**: Underscore or block cursor
- **Skip on Click**: Full text appears instantly on click

**Accessibility:**
- **Dyslexia-Friendly Font**: Readable serif with good letter spacing
- **Text Speed**: Adjustable in options (Slow, Normal, Fast)
- **Auto-Advance**: Option to auto-advance after delay
- **Text Size**: Scalable in options

---

## **AUDIO DESIGN**

### **Music System**

**Dynamic Layering Approach:**
- **Planning Track**: Subtle, calm period music
- **Action Track**: Same melody with added instruments
- **Crossfade**: Smooth transition between layers
- **Battle Crescendo**: Build up during battle scene without track change

**Track Composition:**
- **Planning Layer**:
  - Solo instruments (piano, balalaika)
  - Slow tempo
  - Lower volume
  - Melancholy but hopeful

- **Action Layer** (added during battles):
  - Orchestral instruments (strings, brass, percussion)
  - Faster tempo
  - Higher energy
  - Maintains planning melody

**Crossfade Logic:**
```gdscript
# In audio_controller.gd

var planning_volume: float = 0.8
var action_volume: float = 0.0

func enter_battle_scene():
    # Fade in action layer
    tween_action_volume(0.0, 0.6, 2.0)  # 2 seconds
    # Slightly reduce planning volume
    tween_planning_volume(0.8, 0.5, 2.0)

func exit_battle_scene():
    # Fade out action layer
    tween_action_volume(0.6, 0.0, 2.0)
    # Restore planning volume
    tween_planning_volume(0.5, 0.8, 2.0)
```

**Music Tracks (Public Domain):**
1. **"Varshavianka"** (Russian folk, planning)
2. **"Polyushko-Pole"** (Russian military song, planning)
3. **"Dubinushka"** (Russian folk, planning)
4. **"The Internationale"** (Revolutionary, action layer added)
5. **"Svyaschennaya Voyna"** (Sacred War, action layer)

---

### **Sound Effect Categories**

#### **Menu Sounds** (Bassy, Clunky)
- **Button Click**: Mechanical switch sound (bassy, low-mid range)
- **Flip Letter**: Card flipping, slot-machine reel (bassy, rhythmic)
- **Menu Transition**: Heavy mechanical slide (low rumble)
- **Error**: Dull thud, rejected (low, unsatisfying)
- **Confirm**: Satisfying mechanical click (bassy, clear)

**Characteristics:**
- Frequency: 100-300 Hz (bass-heavy)
- No high-pitched sounds
- Mechanical, industrial feel
- Subtle, not fatiguing

#### **Typewriter Sounds** (Satisfying, Low)
- **Key Press**: Mechanical typewriter key (clear, satisfying)
- **Carrier Return**: Typewriter carrier slide (smooth, mechanical)
- **Bell**: Omitted (too fatiguing over time)
- **Spacebar**: Slightly louder key press

**Characteristics:**
- Frequency: 200-500 Hz
- Duration: 0.05-0.1s per keypress
- Satisfying mechanical clack
- No high-pitched bells or dings

#### **Planning/Gameplay Sounds** (Subtle, Practical)
- **Unit Select**: Subtle click (high-mid, soft)
- **Movement Order**: Soft swoosh or shuffle (low-mid)
- **Attack Order**: Subtle confirm (mid range)
- **Order Cancel**: Gentle undo sound (low-mid)
- **Special Action**: Mechanical whir (bassy)
- **Terrain Hover**: Subtle texture sound (low, quiet)

**Characteristics:**
- Frequency: 200-800 Hz
- Low volume (planning phase)
- Clear but not intrusive
- Practical, not cinematic

#### **Battle Sounds** (Bombastic, Immersive)
- **Gunfire (Various)**:
  - Rifle shots (sharp, crack)
  - Machine gun (rhythmic, loud)
  - Artillery (boom, echo)
  - Pistol shots (crack, shorter)
  
- **Explosions**:
  - Small grenade (pop, debris)
  - Large artillery (boom, rumble, echo)
  - Train collision (massive boom, metal crunch)

- **Human Sounds**:
  - Yelling/screaming (battle chaos)
  - Running (footsteps on terrain)
  - Horse hooves (rhythmic, varied)
  - Vehicle sounds (engine rumble, tracks clanking)

- **Impact/Destruction**:
  - Metal hit (clang, ring)
  - Flesh hit (thud, squish)
  - Building collapse (crash, rumble)
  - Train destruction (massive crunch, explosion)

**Distance-Based Filtering:**
```gdscript
# Calculate distance from "camera" to sound source

func play_unit_sound(unit: Unit, sound_type: String):
    var distance: float = calculate_distance_to_camera(unit)
    var volume: float = 1.0 - (distance / max_distance)
    var frequency_shift: float = 1.0 - (distance * 0.1)  # 0.8-1.0x
    
    audio_controller.play_sound(
        sound_type,
        volume,
        frequency_shift,
        spatial_position = unit.global_position
    )

# Examples:
# Foreground unit (0m): Volume 1.0, Pitch 1.0x
# Midground unit (10m): Volume 0.8, Pitch 0.9x
# Background unit (20m): Volume 0.6, Pitch 0.8x
# Far unit (30m): Volume 0.4, Pitch 0.7x
```

**Characteristics:**
- **Frequency**: 50-2000 Hz (full range)
- **Volume**: Loud (0.8-1.0) for battle immersion
- **Spatial Audio**: 3D positioning, distance filtering
- **Variety**: Multiple variations of each sound type

---

### **Accessibility Features**

#### **Visual Accessibility**
- **Dyslexia-Friendly Fonts**:
  - Open Dyslexic (free, designed for dyslexia)
  - Readex Pro (free, clear letterforms)
  - Lexend (free, optimized for readability)

- **High Contrast**:
  - Dark backgrounds (RGB: 20-30)
  - Light text (RGB: 220-240)
  - Avoid red/green combinations (colorblind issues)

- **Vector Assets**:
  - SVG or Godot vector graphics
  - Scale to any resolution
  - Maintain clarity at any size

- **Text Sizing**:
  - Base size: 16-18pt for body text
  - Scalable in options: 0.75x, 1.0x, 1.25x, 1.5x
  - Minimum readable size: 14pt on mobile

#### **Audio Accessibility**
- **Volume Controls**:
  - Master volume (0-100%)
  - Music volume (0-100%)
  - Sound effects volume (0-100%)

- **Audio Cues**:
  - Visual indicator for important sounds (if sound disabled)
  - Subtitles for narrative (optional)
  - Sound visualization (optional)

- **Non-Fatiguing Design**:
  - Avoid repetitive high-pitched sounds
  - No bells or dings for frequent actions
  - Low-frequency menu sounds
  - Satisfying but subtle typewriter sounds

#### **Input Accessibility**
- **Touch Targets**:
  - Minimum 44x44 pixels on mobile
  - Larger for important buttons (60x60)
  - Spacing: Minimum 8px between targets

- **Multiple Input Methods**:
  - Mouse: Left-click, right-click, scroll
  - Touch: Tap, long-press, pinch-zoom
  - Keyboard: Arrow keys, Enter, Space, Esc
  - Gamepad: D-pad, A/B/X/Y buttons (optional)

- **Undo System**:
  - Unlimited undo during planning
  - Clear visual indicator of undo available
  - Keyboard shortcut: U key

---

## **RESPONSIVE DESIGN**

### **Screen Size Handling**

```gdscript
# Dynamic UI scaling based on screen size

func _ready():
    var screen_size: Vector2 = get_viewport().size
    
    if screen_size.x < 768:  # Small mobile
        scale_ui(0.75)
        use_compact_layout()
    elif screen_size.x < 1024:  # Tablet
        scale_ui(0.9)
        use_tablet_layout()
    elif screen_size.x < 1920:  # Desktop
        scale_ui(1.0)
        use_standard_layout()
    else:  # Large desktop
        scale_ui(1.25)
        use_large_layout()

func scale_ui(factor: float):
    # Scale all UI elements
    for node in get_ui_nodes():
        if node is Control:
            node.scale = Vector2(factor, factor)
```

**Layout Variants:**

**Compact (Mobile < 768px)**:
- Single column for order lists
- Smaller fonts (14pt base)
- Collapsible panels
- Bottom-aligned controls

**Tablet (768-1024px)**:
- Side-by-side order lists
- Medium fonts (16pt base)
- Panel overlays
- Bottom-aligned controls

**Standard (1024-1920px)**:
- Side-by-side order lists
- Standard fonts (18pt base)
- Floating panels
- Bottom-aligned controls

**Large (> 1920px)**:
- Side-by-side order lists
- Large fonts (20pt base)
- Expanded panels
- Side-aligned controls (optional)

---

## **UI ANIMATION SYSTEM**

### **Animation Types**

#### **Transitions**
- **Fade In/Out**: 0.3-0.5s
- **Slide**: 0.4-0.6s
- **Wipe**: 0.5-0.8s (mechanical, for train board)
- **Zoom In/Out**: 0.3s (for map zoom)

#### **Button Interactions**
- **Hover**: Scale 1.05x (0.1s)
- **Click**: Scale 0.95x (0.1s), then bounce to 1.0x (0.2s)
- **Disable**: Gray out, scale 0.9x (0.2s)

#### **Unit Interactions**
- **Selection**: Flash (0.1s), then glow (0.3s)
- **Movement Order**: Ghost appears, slide to target (0.3s)
- **Attack Order**: Red arrow draws (0.2s)
- **Error**: Shake (0.2s, 3 shakes), red flash (0.1s)

#### **Scene Effects**
- **Steam Rising**: Particles rising from bottom (continuous)
- **Gear Rotation**: Smooth rotation (5-10 RPM)
- **Typewriter**: Character-by-character (0.05s per char)
- **Flip Letter**: Card flip (0.2s per letter)
- **Vignette**: Fade in/out (0.5s)

---

## **IMPLEMENTATION PRIORITY**

### **Phase 1: Functional Prototype**
- [ ] Basic main menu with buttons
- [ ] Level select with map cards
- [ ] Simple options menu (toggles, sliders)
- [ ] Basic narrative screen (instant text)
- [ ] Game board UI (HUD, orders, undo)
- [ ] Iron curtain scene
- [ ] Reveal orders scene
- [ ] Basic battle scene
- [ ] Victory/defeat screens

### **Phase 2: Thematic Enhancement**
- [ ] Industrial theme for in-game UI
- [ ] Train board options menu with flip letters
- [ ] Binoculars effect for battle scenes
- [ ] Visual novel narrative system
- [ ] Historical portraits and cutaways
- [ ] Dynamic music layering
- [ ] Comprehensive sound design
- [ ] Menu variants with period imagery
- [ ] Mechanical computer displays
- [ ] Advanced animations and effects

---

## **TOOL INTEGRATION**

### **Godot UI Nodes**
- **Control**: Base for all UI elements
- **Button**: Clickable buttons
- **Label**: Text display
- **TextureRect**: Image/texture display
- **Panel**: Container with border
- **HBoxContainer**: Horizontal layout
- **VBoxContainer**: Vertical layout
- **GridContainer**: Grid layout
- **ScrollContainer**: Scrollable content
- **NinePatchRect**: Scalable UI backgrounds

### **Tweening System**
```gdscript
# Use Godot's TweenNode for animations

func animate_button_hover(button: Button):
    var tween = create_tween()
    tween.tween_property(button, "scale", Vector2(1.05, 1.05), 0.1)
    tween.set_ease(Tween.EASE_OUT_QUAD)

func animate_flip_letter(label: Label, from_char: String, to_char: String):
    var tween = create_tween()
    # Scale down (flip)
    tween.tween_property(label, "scale:x", 0.0, 0.1)
    # Change character mid-flip
    tween.tween_callback(func(): label.text = to_char)
    # Scale up (reveal)
    tween.tween_property(label, "scale:x", 1.0, 0.1)
    tween.set_ease(Tween.EASE_IN_OUT_QUAD)
```

---

**Last Updated**: January 2, 2026
**Version**: 1.0
