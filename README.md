# Stealth Game AI - Unreal Engine 5

## Project Description
A stealth game where the player controls a young boy who is outside during curfew and must avoid patrolling drones. The boy must reach the lit up door as he sneaks through the town. Be careful, if one drone spots you they will all be alerted and come for you!
Learn patrol routes, scale buildings or take a lift from the speeding driver to help the boy reach his home.

## Behaviour Tree Structure

### BT_Guard Overview
The Behaviour Tree controls guard AI with three main states:

**Root → Selector**
- **Branch 1: Chase Player** (highest priority)
  - Condition: `CanSeePlayer == true`
  - Behavior: MoveTo → TargetActor (follows player directly)
  - This makes it so that if the guard sees the player it will chase the player and drop everything else.
  
- **Branch 2: Investigate Noise** (medium priority)
  - Condition: `HeardNoise == true`
  - Behavior: MoveTo → NoiseLocation, then BTTask_LookAround
  - BTTask_LookAround: Moves to 4 random points around noise location, pauses at each to search
  - This makes it so that if the player makes noise the guard will go to the spot of the noise and look for the player.
  
- **Branch 3: Patrol** (default/lowest priority)
  - Condition: No active threats detected
  - Behavior: BTTask_FindPatrolPoint → MoveTo patrol point → loop
  - This makes it so that the guard follows the patrol points set out for it

The Selector evaluates from top to bottom, executing the first branch whose conditions are met.

## Blackboard Keys

| Key Name | Type | Purpose |
|----------|------|---------|
| `TargetActor` | Object | Stores reference to detected player actor |
| `CanSeePlayer` | Bool | True when guard has visual contact with player |
| `HeardNoise` | Bool | True when guard detects noise from player |
| `NoiseLocation` | Vector | World position where noise was heard |
| `PatrolPoint` | Vector/Object | Current patrol destination | 

## AI Perception System

### Sight Detection
**Trigger:** Guard's line of sight intersects with player character

**Response:**
1. Sets `CanSeePlayer = true`
2. Stores player reference in `TargetActor`
3. Alerts all other guards in level (team alertness)
4. All guards switch to chase/investigate state
5. Guard directly chases player using MoveTo

**Return to Patrol:** When player breaks line of sight, `CanSeePlayer` is cleared and guard returns to patrol after investigation

### Hearing Detection
**Trigger:** Player presses N key, calls Report Noise Event at player location

**Response:**
1. Sets `HeardNoise = true`
2. Stores player's exact location in `NoiseLocation`
3. Alerts all other guards in level (team alertness)
4. All guards move to noise location
5. Guards execute BTTask_LookAround - moves to 4 random points in radius around noise, pausing to search

**Return to Patrol:** After completing look-around sequence, `HeardNoise` and `NoiseLocation` are cleared, guard resumes patrol

## Team Alertness System
When any guard detects the player (sight or hearing):
- **Get All Actors Of Class** finds all AI guards in level
- **For Each Loop** iterates through guards
- Calls custom **Alert to Player** event on each guard's AI Controller
- All guards receive same target location and switch from patrol to investigate/chase
- No distance limitation - all guards respond regardless of proximity

## Technical Implementation
- **AI Controller:** BP_PatrolAIController with AIPerception component
- **NPC Character:** BP_AI_Guard with patrol behavior
- **Player Character:** BP_ThirdPersonCharacter with noise generation (N key)
- **Catch Mechanic:** Sphere collision on guards triggers level restart when touching player
- **Win Condition:** BP_Ending actor triggers victory when player reaches goal

## Additional Features

### Moving Vehicle System
**Implementation:**
- Blueprint actor with Spline Component
- Vehicle follows spline path around the level at constant speed
- Continuously loops through spline route

**Purpose:** Adds dynamic element to stealth gameplay - vehicle can transport player quickly through the town or serve as moving obstacle for guard patrols

## Setup Requirements
- NavMesh covers walkable area
- Target Point actors placed for patrol routes
- BP_Ending actor placed at goal location
- Multiple BP_AI_Guard instances in level
- Spline path configured for vehicle route
