# Session — Corner prebuilt integration paused

**Date:** 2026-08-02  
**System:** Aguja del Caos / Mazmorra Procedural V2  
**Engine:** Unreal Engine 5.4  
**Implementation:** Blueprints

## Decision

The fixed prebuilt Corner integration is paused before adding more rotation or local-door conversion logic.

Reason:

```text
The base dungeon is stable again.
The experimental Corner selection works partially.
The remaining orientation problem is not yet clear enough.
Adding more functions now increases complexity and regression risk.
```

No new function should be added for this problem until the approach is reviewed and simplified.

## Stable confirmed state

With:

```text
Max Rooms = 10
Seed = 12345
```

the generator again produces:

```text
1 Start
10 Normal
1 Key
1 Boss
= 13 physical rooms
```

The currently stable actor-Yaw table in `PlaceChildRoomFromParent` is:

```text
Rot0   = 180
Rot90  = 270
Rot180 = 0
Rot270 = 90
```

Changing the table to:

```text
Rot0   = 180
Rot90  = 90
Rot180 = 0
Rot270 = 270
```

caused a Corner placement failure after all overlap retries. Therefore the alternate table is rejected for the current implementation.

## Confirmed functions created

### GetPreBuiltPatternDoors

Owner:

```text
BP_DungeonGenerator_V2
```

Status:

```text
Compiled and isolated runtime tested.
```

Confirmed test:

```text
Corner + Rot180
→ North = false
→ East  = false
→ South = true
→ West  = true
```

### DoesPreBuiltPatternMatchCell

Owner:

```text
BP_DungeonGenerator_V2
```

Status:

```text
Compiled and isolated runtime tested.
```

Confirmed tests:

```text
Corner Rot180 vs South+West → true
Corner Rot180 vs South only → false
```

### FindMatchingPreBuiltDoorRotation

Owner:

```text
BP_DungeonGenerator_V2
```

Status:

```text
Created and compiled.
Used by the experimental Normal-room Corner selection path.
Not fully validated because final visual orientation remains unresolved.
```

### WorldDirectionToLocalDoorDirection

Owner:

```text
BP_Room_PreBuilt_Base
```

Status:

```text
Created and connected before Get Door World Location's direction switch.
Current dungeon can produce 13 rooms with the stable Yaw table.
The function's final correctness for every Corner orientation is not yet visually validated.
```

Current intended mapping uses direction indexes:

```text
North = 0
East  = 1
South = 2
West  = 3
```

and quarter-turn actor rotation derived from Yaw.

## Corner test asset

```text
BP_Room_PreBuilt_Test_Corner
Parent: BP_Room_PreBuilt_Base
DoorPattern: Corner
DoorPatternBaseRotation: Rot180
Authored physical openings: South + West
```

The generator can select and spawn this class for compatible Normal cells.

Current unresolved problem:

```text
At least one spawned L-room does not visually line up as expected.
The exact failing layer is not yet proven:
- actor Yaw conversion,
- requested world direction to local DoorPoint mapping,
- or local wall/opening initialization after rotation.
```

Do not assume the remaining problem is `Init Room from Cell` until it is proven with a focused test.

## Critical bug fixed during this session

The Key and Corner children accidentally contained empty overrides of:

```text
Get Room Bounds Data
```

An empty child override replaced the correct inherited parent implementation and returned:

```text
Bounds Center = 0,0,0
Bounds Extent = 0,0,0
```

This produced permanent overlap results, exhausted placement retries and stopped the physical generation loop.

Affected assets:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Test_Corner
```

After removing/recreating the broken child implementations, both assets returned valid bounds and the full 13-room generation was restored.

Permanent rule:

```text
Children of BP_Room_PreBuilt_Base inherit:
- Get Room Bounds Data
- Get Door World Location
- Init Room from Cell

Do not create empty child overrides.
```

## Debug conclusions

The old error text was ambiguous. Two different failures had used the same message.

Keep these separate while debugging:

```text
DUNGEON ERROR SPAWN — SpawnActor returned invalid
DUNGEON ERROR PLACE — No free position after retries
```

Confirmed in the failed cases:

```text
SpawnActor succeeded.
Set Actor Location moved the candidate correctly.
The real early failures came from zero RoomBounds caused by empty overrides.
```

## Temporary debug still present

The following diagnostics may still exist in `PlaceChildRoomFromParent`:

```text
BOUNDS TEST
MOVE TEST
PLACE FAIL | Index / Type / Class
MVP: PREBUILT CORNER SELECTED
Global Overlap
```

Do not remove them until the direction/rotation experiment is either completed or reverted to a clean stable baseline.

## Not implemented

The proposed function:

```text
GetLocalDoorFlagsFromCell
```

was not implemented and is not part of the current system.

## Next decision point

Before continuing, choose one of these routes:

```text
A. Keep the current grid-first generator and integrate fixed prebuilt rooms with a much smaller isolated placement test.

B. Temporarily remove fixed Corner selection from the beta path and continue toward corridors and a playable 0.0.5.9 build.

C. Redesign future layout growth around available/open doors, but only after the first playable beta.
```

Recommended immediate priority for speed:

```text
Protect the stable 13-room generator.
Do not add another rotation helper yet.
Decide whether the Corner is required for the first private beta.
```
