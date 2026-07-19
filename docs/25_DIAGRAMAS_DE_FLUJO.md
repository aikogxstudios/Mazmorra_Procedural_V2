# 25 — Diagramas de flujo

## Generación completa actual

```text
GenerateDungeon
│
├─ Switch Has Authority
│  ├─ Remote ───────────────→ fin
│  └─ Authority
│      │
│      ├─ ResetDungeon
│      ├─ InitRandomStream
│      ├─ CreateStartCell
│      ├─ BuildDungeonLayout
│      ├─ ChooseKeyAndBossCells
│      ├─ SpawnRoomsFromCells
│      ├─ SpawnCorridorsFromConnections
│      ├─ SpawnBossRoomDoors
│      ├─ DebugDrawDoorPoints
│      └─ DebugDrawDoorToDoorConnections
```

## BuildDungeonLayout conceptual

```text
CreateStartCell ya ejecutado
│
└─ repetir
   │
   ├─ TryAddRandomCell
   │  ├─ Added=true  → aumenta número de celdas
   │  └─ Added=false → solo aumenta intentos
   │
   ├─ DungeonCells.Num >= MaxRooms? → terminar
   └─ Attempts >= MaxGenerationAttempts? → terminar
```

## TryAddRandomCell actual

```text
DungeonCells.Length <= 0?
├─ sí → Return Added=false
└─ no
   │
   ├─ Random Cell Index
   ├─ Selected Cell
   │
   ├─ ¿índice 0 y Start ya tiene conexión?
   │  ├─ sí → Return Added=false
   │  └─ no
   │     │
   │     ├─ Random Direction Index 0-3
   │     ├─ Selected Direction
   │     ├─ GetNeighborCoord
   │     ├─ Neighbor X/Y
   │     │
   │     ├─ IsCellOccupied?
   │     │  ├─ sí → Return Added=false
   │     │  └─ no
   │     │     │
   │     │     ├─ New Room Seed
   │     │     ├─ SetConnectionOnCell(parent)
   │     │     ├─ Set Array Elem parent
   │     │     ├─ Make New Base Cell
   │     │     ├─ GetOppositeDirection
   │     │     ├─ SetConnectionOnCell(child)
   │     │     ├─ Add DungeonCells
   │     │     ├─ Add DungeonCellLinks
   │     │     └─ Return Added=true
```

## Relación padre-hija

```text
Parent Cell
   │
   ├─ Selected Direction = East
   │
   ├──────────────→ Child Cell
   │                  │
   │                  └─ conexión de vuelta = West
   │
   └─ DungeonCellLink de Child:
      ParentCellIndex = índice Parent
      DirectionFromParent = East
      bHasParent = true
```

## Selección Boss

```text
ForEach DungeonCells
│
├─ ConnectionCount
├─ RoomType != Start?
├─ ConnectionCount == 1?
│
└─ DistanceFromStart
   └─ si supera BestBossDistance
      → guardar índice y coord

Después:
DungeonCells[BossCellIndex].RoomType = Boss
preservando el resto del struct
```

## Selección Key

```text
ForEach DungeonCells
│
├─ no Start
├─ no Boss
├─ DistanceFromStart >= 3
├─ DistanceFromBoss
├─ KeyScore = DistanceFromBoss * DistanceFromStart
│
└─ guardar mejor score

Después:
DungeonCells[KeyCellIndex].RoomType = Key
preservando el resto del struct
```

## GenerateRoom de BP_RoomMaster_Dungeon

```text
GenerateRoom
│
├─ RandomizeRoomSize
├─ bUseDungeonConnections?
│  ├─ true  → saltar DecideRoomExits
│  └─ false → DecideRoomExits
│
├─ SetupFloor
├─ SetupWallSouth
├─ SetupWallNorth
├─ SetupWallEast
├─ SetupWallWest
├─ SetupCeiling
├─ UpdateConnectionMarkers
├─ CenterRoomContentOnActorOrigin
└─ UpdateRoomBounds
```

## InitRoomFromCell

```text
CellData
│
├─ Set CurrentCellData
├─ Break ST_DungeonCell
├─ bUseDungeonConnections=true
├─ bRandomizeExits=false
├─ N → SouthOpening
├─ E → WestOpening
├─ S → NorthOpening
├─ W → EastOpening
└─ GenerateRoom
```

## Spawn de pasillos

```text
ForEach DungeonCells
│
├─ bNorth?
│  └─ Current North Door
│     + Neighbor South Door
│     → Spawn Corridor
│
└─ bEast?
   └─ Current East Door
      + Neighbor West Door
      → Spawn Corridor
```

South/West no se procesan para evitar duplicados.

## Boss Door

```text
BossCellIndex válido
│
├─ Boss Cell
├─ Boss Room Actor
│
└─ por cada conexión true
   ├─ GetDoorWorldLocation
   ├─ elegir yaw
   ├─ Spawn BossDoorClass
   └─ Add SpawnedDungeonDoors
```

## Colocación padre-hija futura

```text
SpawnStartRoom
│
└─ SpawnedRooms[0]

Para cada ChildIndex > 0
│
├─ Link = DungeonCellLinks[ChildIndex]
├─ ParentRoom = SpawnedRooms[ParentCellIndex]
├─ ParentDirection
├─ ChildEntryDirection = Opposite
├─ Spawn child una vez
├─ InitRoomFromCell
├─ ParentDoor
├─ ChildDoor
├─ DesiredChildDoor
├─ MoveDelta
├─ SetActorLocation
├─ GetRoomBoundsData
│
├─ ¿overlap?
│  ├─ no → aceptar y Add SpawnedRooms
│  └─ sí → aumentar CorridorLength y mover misma hija
│
└─ continuar
```

## Limpieza

```text
ResetDungeon
│
├─ Destroy SpawnedDungeonDoors
├─ Clear SpawnedDungeonDoors
├─ Destroy SpawnedCorridors
├─ Clear SpawnedCorridors
├─ Destroy SpawnedRooms
├─ Clear SpawnedRooms
├─ Clear DungeonCells
├─ Clear DungeonCellLinks
├─ Reset Boss
└─ Reset Key
```
