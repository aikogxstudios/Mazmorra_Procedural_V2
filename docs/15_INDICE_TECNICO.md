# 15 — Índice técnico rápido

Este archivo sirve para localizar nombres sin leer toda la documentación.

# Assets principales

```text
BP_DungeonGenerator_V2
ST_DungeonCell
ST_DungeonCellLink
E_DungeonDirection
E_DungeonRoomType
BPI_DungeonRoomV2
BP_Room_Debug_Start
BP_Room_Debug_Normal
BP_Room_Debug_Key
BP_Room_Debug_Boss
BP_RoomMaster
BP_RoomMaster_Dungeon
BP_FloorActor
BP_WallActor
BP_CeilingActor
BP_Corridor_Debug
BP_BossDoor_Lock
Pickup de llave
Mini Inventory Component
UI mini inventario
```

# BP_DungeonGenerator_V2

## Funciones

```text
GenerateDungeon
ResetDungeon
InitRandomStream
CreateStartCell
IsCellOccupied
GetNeighborCoord
TryAddRandomCell
BuildDungeonLayout
SpawnRoomsFromCells
GetOppositeDirection
SetConnectionOnCell
DebugPrintLayout
DebugDrawConnections
DebugDrawDoorPoints
FindCellIndexByCoord
DebugDrawDoorToDoorConnections
SpawnCorridorsFromConnections
ChooseKeyAndBossCells
SpawnBossRoomDoors
SpawnDoorAtRoomDirection
```

## Futuras funciones

```text
SpawnStartRoom
SpawnRemainingRoomsFromParents
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
GetDirectionVector
ValidateDungeonData
```

## Variables miembro

### Pasillos

```text
CorridorDebugClass
SpawnedCorridors
```

### Habitaciones

```text
CellSize
DungeonSeed
MaxGenerationAttempts
StartDebugRoomClass
SpawnedRooms
DungeonCells
RandomStream
MaxRooms
bUseRandomSeed
SelectedRoomClass
```

### Debug

```text
DebugRoomClass
bGenerateonBeginPlay
bDebugDungeon
```

### Boss

```text
BossCellIndex
BossGridX
BossGridY
BossDebugRoomClass
BestBossDistance
```

### Key

```text
KeyCellIndex
KeyGridX
KeyGridY
KeyDebugRoomClass
BestKeyDistance
```

### Puertas y links

```text
BossDoorClass
SpawnedDungeonDoors
DungeonCellLinks
```

## Locales de TryAddRandomCell

```text
Random Cell Index
Selected Cell
Random Direction Index
Selected Direction
Neighbor X
Neighbor Y
New Room Seed
Updated Selected Cell
New Base Cell
Opposite Direction
New Connected Cell
```

## Locales de ChooseKeyAndBossCells

```text
ConnectionCount
DistanceFromStart
DistanceFromBoss
KeyScore
```

## Locales de Boss Door

```text
BossRoomActor
DoorWorldLocation
DoorRotation
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

## Interfaz

```text
Get Room Bounds Data
Get Door World Location
Init Room from Cell
```

## Componentes

```text
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
RoomContentRoot
Scene
```

## Variables Room

```text
RoomWidth
RoomDepth
WallHeightTiles
TileSize
EffectiveTileSize
WallTileHeight
bGenerateCeiling
bRandomizeRoomSize
MinRoomWidth
MaxRoomWidth
MinRoomDepth
MaxRoomDepth
```

## Variables Opening

```text
OpeningWidthTiles
OpeningHeightTiles
bNorthOpening
bEastOpening
bWestOpening
bSouthOpening
bRandomizeExits
bUseDungeonConnections
```

## Variables Dungeon/Bounds

```text
CurrentCellData
BoundsPadding
BoundsPaddingZ
BoundsHalfHeight
```

## Arrays heredados

```text
ActiveExitArrows
ActiveExitDirections
```

# BP_RoomMaster original

Mismos sistemas base de generación, sin:

```text
bSouthOpening
bUseDungeonConnections
CurrentCellData
RoomContentRoot
RoomBounds
CenterRoomContentOnActorOrigin
UpdateRoomBounds
BPI_DungeonRoomV2
```

# BP_FloorActor

## Componentes

```text
SceneRoot
HISM_Floor
```

## Variables

```text
WidthTiles
DepthTiles
TileSize
bAutoTileSize
FloorMesh
FloorMaterial
bOverrideFloorMaterial
ThemeName
CurrentX
```

## Funciones

```text
ClearFloor
ApplyFloorVisuals
UpdateTileSize
GenerateFloor
```

# BP_WallActor

## Componentes

```text
SceneRoot
HISM_Wall
HISM_Column
HISM_EdgeColumn
```

## Variables de pared

```text
LengthTiles
HeightTiles
TileSize
WallTileHeight
bAutoTileSize
bAutoWallHeight
CurrentX
MeshMinZ
WallMesh
WallMaterial
bOverrideWallMaterial
ThemeName
```

## Variables de columna

```text
bGenerateColumns
ColumnMesh
ColumnMaterial
bOverrideColumnMaterial
ColumnSpacing
ColumnForwardOffset
ColumnMinZ
CurrentColumnIndex
```

## Variables de apertura

```text
HasOpening
OpeningStartIndex
OpeningWidthTiles
OpeningHeightTiles
```

## Variables edge

```text
bGenerateEdgeColumns
EdgeColumnMesh
EdgeColumnMaterial
```

## Funciones conocidas

```text
ClearWall
ApplyWallVisuals
ApplyColumnVisuals
ApplyEdgeColumnVisuals
UpdateWallSize
UpdateColumnSize
IsOpeningCell
GenerateWall
GenerateColumns
GenerateEdgeColumns
```

# BP_CeilingActor

```text
WidthTiles
DepthTiles
TileSize
CeilingMesh/FloorMesh
CeilingMaterial/FloorMaterial
GenerateCeiling/GenerateFloor
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

## Boss Door yaw

```text
N = 0
E = -90
S = 180
W = 90
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
SpawnCorridorsFromConnections procesa solo North/East
```

# Estado actual

```text
Links validados: 10, 15, 20, 50, 150
Start una salida: validado
Siguiente paso: SpawnStartRoom
```
