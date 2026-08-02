# Fix: empty inherited override returned zero RoomBounds

Date: 2026-08-02

## Confirmed symptom

The physical placement loop stopped before spawning all 13 rooms.

The failing case was:

```text
PLACE FAIL | Index=11 | Type=Key | Class=BP_Room_PreBuilt_Base_Child_Key_C
```

During every placement retry, the Key actor moved correctly, but its bounds were invalid:

```text
Actor Location changed
Bounds Center = 0,0,0
Bounds Extent = 0,0,0
```

This made `DoesRoomOverlapPlacedRooms` report overlap on every retry, so `PlaceChildRoomFromParent` returned `false`. The general placement loop then stopped before reaching Boss.

## Root cause

`BP_Room_PreBuilt_Base_Child_Key` accidentally created its own empty implementation/override of:

```text
Get Room Bounds Data
```

The correct inherited implementation already existed in `BP_Room_PreBuilt_Base`:

```text
RoomBounds
→ Get Component Bounds
→ Origin → Bounds Center
→ Box Extent → Bounds Extent
```

Because the child override was empty, it replaced the inherited result and returned the default zero vectors.

## Fix applied

- Removed/replaced the broken Key child.
- Recreated `BP_Room_PreBuilt_Base_Child_Key` directly from `BP_Room_PreBuilt_Base`.
- Kept inherited interface functions untouched.
- Recompiled and tested the dungeon.

## Validation

The generator returned to the expected physical result:

```text
1 Start
10 Normal
1 Key
1 Boss
= 13 rooms
```

## Important rule for future prebuilt children

Children of `BP_Room_PreBuilt_Base` inherit these functions automatically:

```text
Get Room Bounds Data
Get Door World Location
Init Room from Cell
```

Do not create an implementation/override inside a child unless the child genuinely needs different behavior. An empty override silently replaces the correct parent implementation and returns default output values.

## Debugging lesson

The old placement failure message was ambiguous. A failed placement after overlap retries is not the same as a failed `SpawnActor` call. Keep separate messages during debugging:

```text
DUNGEON ERROR SPAWN — SpawnActor returned invalid
DUNGEON ERROR PLACE — No free position after retries
```
