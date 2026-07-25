# 15 — Índice técnico rápido

**Última actualización:** 2026-07-25

Este archivo sirve para localizar nombres sin leer toda la documentación.

# Assets principales

```text
BP_DungeonGenerator_V2
ST_DungeonCell
ST_DungeonCellLink
E_DungeonDirection
E_DungeonRoomType
BPI_DungeonRoomV2
BP_RoomMaster
BP_RoomMaster_Dungeon
BP_Room_PreBuilt_Base
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
BP_FloorActor
BP_WallActor
BP_CeilingActor
BP_Corridor_Debug
BP_BossDoor_Lock
```

# BP_DungeonGenerator_V2

## Funciones confirmadas

```text
GenerateDungeon
ResetDungeon
InitRandomStream
CreateStartCell
IsCellOccupied
GetNeighborCoord
TryAddRandomCell
TryAddSpecialCellFromParent
BuildDungeonLayout
SpawnRoomsFromCells
SpawnStartRoom
SpawnFirstChildRoom
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
GetOppositeDirection
GetDirectionVector
SetConnectionOnCell
ChooseKeyAndBossCells
DebugPrintLayout
DebugDrawConnections
DebugDrawDoorPoints
FindCellIndexByCoord
DebugDrawDoorToDoorConnections
SpawnCorridorsFromConnections
SpawnBossRoomDoors
SpawnDoorAtRoomDirection
```

## Flujo actual

```text
CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ For Loop with Break
→ PlaceChildRoomFromParent(Index)
```

## Variables miembro clave

```text
DungeonSeed
MaxGenerationAttempts
SpawnedRooms
DungeonCells
DungeonCellLinks
RandomStream
MaxRooms
bUseRandomSeed
SelectedRoomClass
BossCellIndex
BossGridX
BossGridY
BossDebugRoomClass
BestBossDistance
KeyCellIndex
KeyGridX
KeyGridY
KeyDebugRoomClass
BestKeyDistance
BossDoorClass
SpawnedDungeonDoors
```

## Semántica actual

```text
MaxRooms = cantidad de Normal
BossCellIndex = padre normal seleccionado para Boss
KeyCellIndex = padre normal seleccionado para Key
```

## PlaceChildRoomFromParent

### Input/Output

```text
Child Index : Integer
Room Placed : Boolean
```

### Locales

```text
Parent Room Actor
Child Room Actor
Child Entry Direction
Parent Direction
Parent Door Location
Child Door Location
Corridor Length
Placement Attempt
Placement Retry Step
Max Placement Attempts
bPlacement Succeeded
Bounds Overlap
```

### Fórmula

```text
DesiredChildDoor = ParentDoorLocation + DirectionVector * CorridorLength
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewLocation = ChildActorLocation + MoveDelta
```

## DoesRoomOverlapPlacedRooms

### Input/Output

```text
Candidate Room Actor
Overlaps Placed Rooms
```

### Locales

```text
Candidate Bounds Center
Candidate Bounds Extent
Placed Room Actor
Placed Bounds Center
Placed Bounds Extent
Found Overlap
```

### Fórmula

```text
Abs(CandidateCenter - PlacedCenter)
<= CandidateExtent + PlacedExtent
```

por X/Y/Z y AND final.

## TryAddSpecialCellFromParent

### Input/Output

```text
Parent Cell Index
Special Room Type
bAdded
New Cell Index
```

### Locales

```text
Direction Start Index
Random Cell Index
Random Direction Index
Selected Cell
Selected Direction
Neighbor X
Neighbor Y
New Room Seed
Updated Selected Cell
New Base Cell
Opposite Direction
New Connected Cell
```

### Dirección

```text
(Direction Start Index + For Loop.Index) % 4
```

```text
0 North
1 East
2 South
3 West
```

### Retornos

```text
Fallo: bAdded=false, New Cell Index=-1
Éxito: New Cell Index = DungeonCells.Add Return Value
```

# ST_DungeonCell

```text
GridX
GridY
RoomType
bNorth
bEast
bSouth
bWest
RoomSeed
RoomID
```

# ST_DungeonCellLink

```text
ParentCellIndex
DirectionFromParent
bHasParent
```

# E_DungeonDirection

```text
North
East
South
West
```

# E_DungeonRoomType

```text
Start
Normal
Combat
Treasure
Puzzle
Safe
Shop
Rest
Key
Boss Door
Boss
Portal
Secret
Event
Storage
Trade
Corridor
Stairs Up
Stairs Down
```

# BPI_DungeonRoomV2

```text
InitRoomFromCell
GetDoorWorldLocation
GetRoomBoundsData
```

# BP_RoomMaster_Dungeon

## Funciones propias

```text
GenerateRoom
SetupFloor
SetupWall South
SetupWall East
SetupWall North
SetupWall West
SetupCeiling
UpdateConnectionMarkers
DecideRoomExits
RandomizeRoomSize
CenterRoomContentOnActorOrigin
UpdateRoomBounds
```

## Componentes técnicos

```text
RoomContentRoot
RoomBounds
Arrow_Entrance_South
Arrow_Exit_West
Arrow_Exit_East
Arrow_Exit_North
ChildActor_Wall_North
ChildActor_Ceiling
ChildActor_Floor
ChildActor_Wall_West
ChildActor_Wall_East
ChildActor_Wall_South
```

# BP_Room_PreBuilt_Base

```text
BPI_DungeonRoomV2
RoomBounds
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
```

Hijos confirmados:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

Próximos datos previstos:

```text
patrón de puertas
rotaciones permitidas
clase/shape de habitación
```

# Mappings críticos

## InitRoomFromCell

```text
N → SouthOpening
E → WestOpening
S → NorthOpening
W → EastOpening
```

## GetDoorWorldLocation

```text
N → Arrow_Entrance_South
E → Arrow_Exit_East
S → Arrow_Exit_North
W → Arrow_Exit_West
```

## Direction Vector

```text
N → (0,1,0)
E → (1,0,0)
S → (0,-1,0)
W → (-1,0,0)
```

# Invariantes rápidas

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

```text
ParentCellIndex < ChildIndex
```

```text
Start ConnectionCount == 1
```

```text
La candidata se genera una vez y después se mueve.
No regenerar HISM durante reintentos.
```

# Cantidad actual

```text
Max Rooms = Normal
10 Normal + Start + Key + Boss = 13
```

# Seeds actuales

```text
12345 → South
12346 → East
```

# Próxima fase

```text
Compatibilidad prebuilt:
Dead End
Straight
Corner/L
T
Cross
rotaciones 0/90/180/270
```
