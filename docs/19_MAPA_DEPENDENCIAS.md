# 19 — Mapa de dependencias

## Dependencias de alto nivel

```text
BP_DungeonGenerator_V2
├─ ST_DungeonCell
├─ ST_DungeonCellLink
├─ E_DungeonDirection
├─ E_DungeonRoomType
├─ BPI_DungeonRoomV2
├─ StartDebugRoomClass
├─ DebugRoomClass / BP_RoomMaster_Dungeon
├─ KeyDebugRoomClass
├─ BossDebugRoomClass
├─ CorridorDebugClass / BP_Corridor_Debug
└─ BossDoorClass / BP_BossDoor_Lock
```

```text
BP_RoomMaster_Dungeon
├─ ST_DungeonCell
├─ BPI_DungeonRoomV2
├─ BP_FloorActor
├─ BP_WallActor ×4
└─ BP_CeilingActor
```

```text
BP_BossDoor_Lock
└─ Mini Inventory Component / item requerido
```

```text
BP_Room_Debug_Key
└─ Pickup de llave
   └─ Mini Inventory Component
```

## Flujo de datos lógico

```text
InitRandomStream
→ RandomStream
→ BuildDungeonLayout
→ TryAddRandomCell
→ DungeonCells
→ DungeonCellLinks
```

```text
DungeonCells
→ ChooseKeyAndBossCells
→ DungeonCells con RoomType Start/Normal/Key/Boss
```

```text
DungeonCells
→ SpawnRoomsFromCells actual
→ SpawnedRooms
```

```text
DungeonCells + SpawnedRooms
→ SpawnCorridorsFromConnections
→ SpawnedCorridors
```

```text
BossCellIndex + DungeonCells + SpawnedRooms
→ SpawnBossRoomDoors
→ SpawnedDungeonDoors
```

## Dependencias por función del generador

| Función | Lee | Escribe/produce | Llama |
|---|---|---|---|
| `GenerateDungeon` | autoridad/configuración | generación completa | todas las fases principales |
| `ResetDungeon` | arrays físicos/lógicos | estado vacío | `DestroyActor`, `Clear` |
| `InitRandomStream` | seed/config | `RandomStream` | inicialización stream |
| `CreateStartCell` | seed | índice 0 de cells/links | `Make`, `Add` |
| `BuildDungeonLayout` | `MaxRooms`, intentos | layout lógico | `TryAddRandomCell` |
| `TryAddRandomCell` | cells, stream | padre actualizado, hija, link | neighbor, occupied, opposite, set connection |
| `SetConnectionOnCell` | cell + direction | cell modificada | Make/Break struct |
| `GetNeighborCoord` | coord + direction | coord vecina | switch direction |
| `GetOppositeDirection` | direction | opposite | switch/select |
| `IsCellOccupied` | cells + coord | bool | ForEach |
| `FindCellIndexByCoord` | cells + coord | index | ForEach |
| `ChooseKeyAndBossCells` | cells | tipos/índices Key/Boss | distancias |
| `SpawnRoomsFromCells` | cells/classes | `SpawnedRooms` | SpawnActor, interface |
| `SpawnCorridorsFromConnections` | cells/rooms | `SpawnedCorridors` | neighbor, index, interface |
| `SpawnBossRoomDoors` | Boss index/cell/room | puertas | `SpawnDoorAtRoomDirection` |
| `SpawnDoorAtRoomDirection` | room/direction/class | puerta | interface, SpawnActor |

## Dependencias de TryAddRandomCell

```text
RandomStream
→ Random Cell Index
→ Selected Cell
```

```text
RandomStream
→ Random Direction Index
→ Selected Direction
```

```text
Selected Cell + Selected Direction
→ GetNeighborCoord
→ Neighbor X/Y
```

```text
Neighbor X/Y
→ IsCellOccupied
```

```text
Selected Cell + Selected Direction
→ SetConnectionOnCell
→ Updated Selected Cell
→ Set Array Elem DungeonCells[Random Cell Index]
```

```text
Neighbor X/Y + New Room Seed + DungeonCells.Length
→ New Base Cell
```

```text
Selected Direction
→ GetOppositeDirection
→ SetConnectionOnCell(New Base Cell)
→ New Connected Cell
→ Add DungeonCells
```

```text
Random Cell Index + Selected Direction
→ ST_DungeonCellLink
→ Add DungeonCellLinks
```

## Dependencias de BP_RoomMaster_Dungeon

### GenerateRoom

```text
RandomizeRoomSize
→ RoomWidth/RoomDepth
```

```text
SetupFloor
→ EffectiveTileSize
```

```text
EffectiveTileSize
→ paredes
→ techo
→ DoorPoints
→ centrado
→ bounds
```

### InitRoomFromCell

```text
ST_DungeonCell
→ CurrentCellData
→ booleans de apertura
→ GenerateRoom
```

### SetupFloor

```text
RoomWidth/RoomDepth/TileSize
→ BP_FloorActor
→ EffectiveTileSize
```

### SetupWall*

```text
RoomWidth/RoomDepth
WallHeightTiles
EffectiveTileSize
WallTileHeight
Opening*
→ BP_WallActor
```

### SetupCeiling

```text
RoomWidth/RoomDepth
EffectiveTileSize
WallHeightTiles*WallTileHeight
→ BP_CeilingActor
```

### CenterRoomContentOnActorOrigin

```text
RoomWidth/RoomDepth/EffectiveTileSize
→ RoomContentRoot.RelativeLocation
```

### UpdateRoomBounds

```text
RoomWidth/RoomDepth/WallHeightTiles
EffectiveTileSize/WallTileHeight
BoundsPadding/BoundsPaddingZ
→ RoomBounds extent/location
```

## Dependencias de colocación futura

```text
DungeonCellLinks[ChildIndex]
→ ParentCellIndex
→ SpawnedRooms[ParentCellIndex]
```

```text
DirectionFromParent
→ ParentDoor
→ DirectionVector
→ ChildEntryDirection
```

```text
Child actor inicializado
→ ChildDoor
→ MoveDelta
→ SetActorLocation
```

```text
ChildRoom.GetRoomBoundsData
+ bounds de SpawnedRooms aceptadas
→ overlap
→ aceptar o reintentar
```

## Orden obligatorio de disponibilidad

Para cada hija:

```text
DungeonCells completo
DungeonCellLinks completo
ParentRoom ya spawneada
ChildRoom spawneada
ChildRoom inicializada
DoorPoints actualizados
RoomBounds actualizado
→ placement
```

No consultar DoorPoints ni bounds antes de `InitRoomFromCell`/`GenerateRoom`.

## Dependencias de limpieza

```text
SpawnedDungeonDoors
SpawnedCorridors
SpawnedRooms
```

deben destruirse antes de limpiar:

```text
DungeonCells
DungeonCellLinks
```

## Dependencias protegidas

Cambiar una de estas piezas exige revisar sus consumidores:

### E_DungeonDirection

Afecta:

```text
GetNeighborCoord
GetOppositeDirection
SetConnectionOnCell
InitRoomFromCell
GetDoorWorldLocation
SpawnCorridorsFromConnections
SpawnBossRoomDoors
SpawnDoorAtRoomDirection
GetDirectionVector futuro
```

### ST_DungeonCell

Afecta todos los `Make`, `Break`, selección Key/Boss e interfaz.

### ST_DungeonCellLink

Afecta `CreateStartCell`, `TryAddRandomCell` y toda la colocación futura.

### BPI_DungeonRoomV2

Afecta todas las clases de habitación y el generador.
