# **GUY DEBORD’S GAME OF WAR \- RUSSIAN CIVIL WAR EDITION**

In which I recreate Guy Debord’s game and add a few new unit and tile types to it to represent a conflict in the early 20th century and also update it for simultaneous turns to eliminate first turn advantage.

## **I. WIN CONDITIONS**

1. Destroy every unit in your opponent’s army, OR  
2. Leave your opponent with no units that are “active”.

## **II. THE TURN**

1. **Planning Phase:** Each player selects up to 5 of their units to move, and declares 1 attack. These plans are kept secret.  
2. **Resolution Phase:** Plans are revealed simultaneously.  
    a. All movement is resolved (see **Movement & Collision**).  
    b. All battles are calculated (see **Battle Rules**).  
3. **End Phase:** The round ends with both players removing or retreating units according to the battle results. If destroyed, units are removed from the board. If retreating, units move 1 tile away from the battle.

## **III. MOVEMENT & COLLISION**

1. **Movement:** Players can only move units that are “active” (see **Communications**).  
2. **Simultaneous Collision:** Moves are revealed at the same time.  
    a. If two units attempt to enter the same tile, or pass through each other on the same vector, **collision** occurs.  
    b. Both units stop their movement on the last tile they occupied before the collision point.  
    c. If this causes another movement collision with a different unit, the second unit is also stopped one tile early, and so on.  
3. **Railway Exception:** Armored Trains follow unique collision rules found in **Railway Operations**.

## **IV. COMMUNICATIONS**

1. **Active State:** A unit is “active” under the following conditions:  
    a. **Source Connection:** You can draw a direct line of effect to it from a **Source** unit (Railway Station or Arsenal).  
    b. **Chain of Command:** It is adjacent to another unit that is “active”.  
    c. **Relay Connection:** You can draw a direct line of effect from a **Relay** unit, which is itself “active”.  
2. **Lines of Effect:**  
    a. Lines are orthogonal or diagonal.  
    b. Lines are blocked by impassable terrain (Mountains, City Blocks) and enemy units.  
    c. **Telegraph Propagation:** An “active” telegraph station will also turn any other connected telegraph stations that you control “active”.  
3. **Circular Logic:** A “Source” command unit must always be the starting point of the communication path.

## **V. BATTLE RULES**

1. **Declaring Attacks:** Battles are declared against any enemy unit or tile within range of any of your units.  
    a. **Targeting:** You may target any specific unit or tile without restriction.  
    b. **Moving Targets:** If a unit you targeted moves, the battle is resolved against its new location.  
    c. **Swapped Targets:** If a tile you targeted has a different unit move onto it, you resolve the battle against the new unit.  
    d. **Empty Tiles:** If a tile you targeted has no unit on it, no battle occurs.  
2. **Participation:** Every unit from both sides that has the battle in range, and whose path is not blocked by enemy units or impassable terrain, takes part.  
3. **Resolution:** If both players declare battles involving the same units, they are evaluated separately before applying results simultaneously.  
4. **Calculating Outcome:**  
    a. Add up the **Attack** stats of all attacking units vs. the **Defense** stats of all defending units (plus terrain modifiers).  
    b. **Defense Wins or Tie:** The attack fails.  
    c. **Attack Wins by 1:** The unit on the battle tile is **Forced to Retreat**.  
    \* It must move 1 tile away from the battle that is not towards an enemy unit.  
    \* If no such tile exists, the unit is destroyed.  
    d. **Attack Wins by \>1:** The unit on the battle tile is **Destroyed**.  
5. **Simultaneity:** All battle effects are evaluated before any are carried out. A unit destroyed this turn may still participate in another battle this turn.

## **VI. TERRAIN**

1. **General Terrain:** Most terrain is aesthetic and applies no bonuses or penalties.  
2. **Impassable Terrain:** Mountains and City Blocks cannot be moved onto and block lines of effect.  
3. **Defensible Terrain:** Fortresses and City Plazas add **4 Defense** to the unit occupying them.  
4. **Roads:** Increase the movement of certain units (see Unit List).  
5. **Infrastructure:**  
    a. **Train/Tram Tracks:** Can be moved over by Trains and Trams.  
    b. **Turnarounds/Turntables:** Allow trains/trams to change direction.  
    c. **Intersections:** Allow trains/trams to turn (but not turn around).  
    d. **Telegraph Lines:** Connect telegraph stations.

## **VII. SPECIAL ACTIONS**

Some actions can be taken in lieu of movement. This still costs one of your 5 movements per round.

1. **Sabotage:**  
    a. Any unit can destroy the terrain feature of the tile it currently occupies (telegraph poles, train stations, tracks, roads, plazas, turntables).  
    b. When destroyed, the tile reverts to a “normal” tile with no special effects.  
2. **Packing and Deploying:**  
    a. Heavy units (Maxim Machine Gun and Signal Balloon) must be **Deployed** to be used and **Packed** to move.  
    b. A unit that Packs cannot move on the same turn. A unit that Deploys can participate in battles on the same turn.  
3. **Transportation:**  
    a. **Embarking:** A unit moves onto the tile of a unit that can "carry" it. This can happen before or after the carrier moves, at the player’s discretion.  
    b. **Restrictions:** While "embarked," a carried unit cannot participate in battles. If the carrier is destroyed, the carried unit is also destroyed. A single unit cannot embark and disembark on the same turn.  
    c. **Disembarking:** A carried unit can exit the carrier on any turn simply by moving itself again.  
    d. **Stacking:** Units that "carry" cannot be carried themselves. Armored Trains cannot be carried.

## **VIII. RAILWAY OPERATIONS**

This section covers specific rules for units locked to tracks.

1. **Track Movement:** Trains and Trams are locked to their specific lines. They can only continue in one direction until reaching a turnaround, turntable, or intersection.  
2. **Armored Train Composition:**  
    a. An Armored Train consists of an **Engine** plus attached **Gun** or **Troop** cars.  
    b. **Synced Movement:** When the Engine moves, attached cars automatically move in the same direction, remaining adjacent to it. This does not cost extra movement.  
3. **Armored Train Collision Exception:**  
   * Armored Trains are massive; they do not stop for standard collisions.  
      a. **vs Units:** If an Armored Train would collide with a unit, the Train continues moving as expected. The colliding unit is stopped one tile early.  
      b. **The Squish Rule:** If the colliding unit is still in the path of the Train after stopping, they must make a forced 1-tile movement (following the same logic as a **Forced Retreat** from battle).  
      c. **vs Trains:** If two Armored Trains collide, the two colliding cars (usually the Engines) are immediately destroyed.  If it is ambiguous as to which part of a multi-part train would be the one destroyed (ie multiple parts of one train all cross the path of the other), then the opposing player decides which train part is the one that is destroyed.

## **IX. NEUTRAL BUILDINGS**

1. **Control:** Buildings do not inherently belong to a player. They are controlled by the last player that moved an infantry unit onto their tile.  
2. **Railway Stations:**  
    a. A captured station automatically captures every other connected station (cascade effect), unless the connection is blocked by an enemy unit.  
    b. A railway station always acts as a **Source** for the player controlling it.  
3. **Telegraph Stations:**  
    a. These act as **Relays**, not Sources. They must be linked to a Source and controlled to become active.

## **X. THE UNITS**

1. **INFANTRY**  
   * Move: 1 | Attack: 4 | Defense: 6 | Range: 2  
2. **CAVALRY**  
   * Move: 2 | Attack: 4 | Defense: 5 | Range: 2  
   * *Cavalry Charge:* \+3 Attack if adjacent to target.  
3. **ARTILLERY (Field Gun)**  
   * Move: 1 | Attack: 5 | Defense: 8 | Range: 3  
4. **SWIFT ARTILLERY (Tachanka)**  
   * Move: 2 | Attack: 5 | Defense: 8 | Range: 3  
5. **RELAY**  
   * Move: 1 | Attack: 0 | Defense: 1 | Range: 0  
   * *Acts as a “relay”.*  
6. **SWIFT RELAY**  
   * Move: 2 | Attack: 0 | Defense: 1 | Range: 0  
   * *Acts as a “relay”.*  
7. **MACHINE GUN NEST (Maxim)**  
   * Move: 1 | Attack: 5 | Defense: 10 | Range: 2  
   * *Heavy:* Must be Deployed to act.  
8. **OBSERVATION BALLOON**  
   * Move: 1 | Attack: 0 | Defense: 0 | Range: 0  
   * *Heavy:* Must be Deployed to act.  
   * *Acts as a “relay” that can transmit over impassable terrain.*  
9. **ARMORED TRAM**  
   * Move: 4 | Attack: 4 | Defense: 6 | Range: 2  
   * Must move along tram line. Can carry 2 infantry or 1 other unit.  
10. **ARMORED TRAIN ENGINE**  
    * Move: 4 | Attack: 0 | Defense: 0 | Range: 0  
    * Must move along train tracks.  
11. **ARMORED TRAIN GUN CAR**  
    * Move: 0 | Attack: 6 | Defense: 8 | Range: 3  
    * Automatically moves with attached Engine.  
12. **ARMORED TRAIN TROOP CAR**  
    * Move: 0 | Attack: 0 | Defense: 0 | Range: 0  
    * Can carry 2 infantry or 1 other unit. Automatically moves with attached Engine.  
13. **CAR**  
    * Move: 2 (3 on roads) | Attack: 4 | Defense: 3 | Range: 2  
    * *Always considered “active”.*  
14. **ARMORED CAR**  
    * Move: 2 (3 on roads) | Attack: 6 | Defense: 5 | Range: 2  
    * *Always considered “active”.*

