Phase 1: The "Tabletop" (Hot Seat Multiplayer) 

Goal: Implement the full ruleset for two human players on one screen. 
1.1 The Board & TileMap 

     Action: Create a 20x25 TileMap.
     TileSet Definition: Create data structures for:
         Base: Snow/Grass.
         Obstacles: Mountains (Blocks LoC), City Blocks (Blocks LoC, Impassable).
         Infrastructure: Rail, Tram, Road, Telegraph Lines.
         Interactables: Stations, Telegraph Offices, Plazas, Turntables.
         
     Visuals: Use simple colored squares initially, but differentiate "Tracks" (lines) from "Open Ground".
     

1.2 The Logic Core (Script: GameRules.gd) 

     The "Network" Engine: Write the recursive raycasting function to determine is_active(unit).
         Input: Unit position, Source positions.
         Output: Boolean.
         Special Case: Handle the Balloon (ignores mountains) and Telegraph Chain (node-to-node connectivity).
         
     The "Railway" Engine: Write the graph traversal for Cascading Captures.
         When a Station changes owner, perform a Breadth-First Search (BFS) along connected Rail tiles to flip ownership of connected stations.
         
     

1.3 The Simultaneous Turn State Machine 

This is critical. You cannot rely on real-time movement. 

     State 1: Planning Phase (Player A)
         UI is active.
         Player clicks units -> Adds to a List<MovementOrder>.
         Player clicks enemies -> Adds to a List<AttackOrder>.
         Visual: Draw a "Ghost" of where the unit will be.
         
     State 2: The Curtain
         UI is blocked. Screen shows "Pass Device to Player B".
         
     State 3: Planning Phase (Player B)
         Same as above.
         
     State 4: Resolution (Movement)
         Collision Logic: Process the movement list. Check for overlaps.
             If Overlap -> Stop both units one tile back.
             If Train Overlap -> Stop unit one tile back, Train proceeds.
             
         Apply the moves. Update unit positions.
         
     State 5: Resolution (Combat)
         Loop through AttackOrders.
         Sum Attack/Defense based on new positions.
         Apply Damage/Retreat/Death.
         
     State 6: End Turn
         Reset lists. Go back to State 1.
         
     

1.4 The Unit Implementation 

     Unit Scene (Unit.tscn): Contains an Area2D for selection.
     Stats: Load from the Unit List (Infantry, Cavalry, etc.).
     Transport: Implement the "Embark/Disembark" logic. If a unit enters a Transport tile, it sets parent = transport.
     

1.5 The Battle View 

     Trigger: When State 5 (Combat) detects a battle with total damage > 0.
     Scene: Instantiate BattleScene.tscn.
     Logic:
         Identify Attackers (Red) and Defenders (Blue).
         Spawn the "Advance Wars" style sprites (profile view).
         Play animation (Fire/Charge).
         Emit Signal battle_finished.
         Resume GameRules.gd.
         
     

Phase 2: The "General" (AI Opponent) 

Goal: Create a CPU opponent that respects the simultaneous rules. 
2.1 The AI Architecture 

     AI Agent (DebordAI.gd):
         Perception: The AI cheats (Perfect Information) to calculate the "True" board state.
         Strategy: Score based on "Disconnecting" the enemy (Destroying Relays/Stations) > Killing Units > Capturing Land.
         
     Simultaneous Logic: The AI must predict where the player might move.
         Simplification: The AI assumes the player will stay still or move forward one tile. It calculates the "Worst Case" (player blocks its move) and "Best Case" (move succeeds).
         
     

2.2 AI Behaviors 

     The Harvester: Prioritize capturing neutral Railway Stations to expand the "Active" zone.
     The Saboteur: If a powerful enemy unit is nearby, move an Infantry onto a Rail/Telegraph tile and order "Sabotage" to cut the enemy's supply line next turn.
     The Train Driver: Utilize the Armored Train to run over enemy infantry blocking a line, knowing the Collision Rule favors the train.
     

Phase 3: The "Front" (Campaign Framework) 

Goal: 3 Maps, Narrative Structure, Victory/Defeat conditions. 
3.1 The Narrative Manager 

     System: A simple state machine that tracks current_level_index.
     Between Levels: Show a text screen describing the historical context (e.g., "The Whites are retreating to the mountains...").
     

3.2 The Three Maps 

     Map 1: "The Iron Horse" (Flatlands & Rails)
         Focus: Armored Trains, Sabotage, long range Artillery.
         Terrain: Mostly open. Large rail network.
         Objective: Capture the Central Rail Hub (Source).
         
     Map 2: "Red Petrograd" (Urban Warfare)
         Focus: Maxim Nests, City Blocks (Mountains), Street Fighting.
         Terrain: Dense city grid. Plazas act as mini-fortresses.
         Objective: Clear the city of enemy units (Annihilation).
         
     Map 3: "The Woods of Bryansk" (Guerrilla War)
         Focus: Cavalry, Scouts, Observation Balloons (seeing over trees).
         Terrain: Forests (Impassable or LoC blockers?). Sparse telegraph lines.
         Objective: Destroy the enemy Balloon (Cut their eyes) and survive.
         
     