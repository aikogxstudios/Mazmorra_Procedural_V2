# 03 — BP_DungeonGenerator_V2

## Responsabilidad general

`BP_DungeonGenerator_V2` controla el grafo lógico, la selección de salas especiales y la generación física de la mazmorra.

Este documento conserva el estado funcional actual del generador. Los detalles específicos de las nuevas funciones están ampliados en:

```text
docs/26_SPAWN_START_ROOM.md
docs/27_SPAWN_FIRST_CHILD_ROOM.md
```

## Componentes

Confirmado por captura:

```text
Scene
```

## Funciones confirmadas

Funciones originales:

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

Funciones creadas durante el refactor padre-hija:

```text
SpawnStartRoom
SpawnFirstChildRoom
GetDirectionVector
```

Además existe:

```text
ConstructionScript
```

## Variables miembro confirmadas

### Pasillos

```text
CorridorDebugClass : Actor Class Reference
SpawnedCorridors : Array Actor Reference
```

### Habitaciones

```text
CellSize : Float
DungeonSeed : Integer
MaxGenerationAttempts : Integer
StartDebugRoomClass : Actor Class Reference
SpawnedRooms : Array Actor Reference
DungeonCells : Array ST_DungeonCell
RandomStream : Random Stream
MaxRooms : Integer
bUseRandomSeed : Boolean
SelectedRoomClass : Actor Class Reference
```

### Debug

```text
DebugRoomClass : Actor Class Reference
bGenerateonBeginPlay : Boolean
bDebugDungeon : Boolean
```

### Boss Room

```text
BossCellIndex : Integer
BossGridX : Integer
BossGridY : Integer
BossDebugRoomClass : Actor Class Reference
BestBossDistance : Integer
```

### Key Room

```text
KeyCellIndex : Integer
KeyGridX : Integer
KeyGridY : Integer
KeyDebugRoomClass : Actor Class Reference
BestKeyDistance : Integer
```

`BestKeyDistance` funciona realmente como mejor `KeyScore`.

### Puertas y links

```text
BossDoorClass : Actor Class Reference
SpawnedDungeonDoors : Array Actor Reference
DungeonCellLinks : Array ST_DungeonCellLink
```

## GenerateDungeon

### Flujo estable anterior

```text
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnRoomsFromCells
→ SpawnCorridorsFromConnections
→ SpawnBossRoomDoors
→ DebugDrawDoorPoints
→ DebugDrawDoorToDoorConnections
```

### Flujo temporal actual durante el refactor

```text
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ SpawnFirstChildRoom
```

El sistema antiguo físico permanece desconectado temporalmente, no eliminado.

## ResetDungeon

### Estado

```text
✅ Estable
🛑 Modificar solo para limpiar nuevos arrays o actores
```

Responsabilidad:

```text
Destruir puertas
→ destruir pasillos
→ destruir salas
→ limpiar arrays físicos
→ limpiar arrays lógicos
→ resetear índices y scores
```

Orden documentado:

```text
ForEach SpawnedDungeonDoors → DestroyActor → Clear
ForEach SpawnedCorridors → DestroyActor → Clear
ForEach SpawnedRooms → DestroyActor → Clear
Clear DungeonCells
Clear DungeonCellLinks
BossCellIndex = -1
KeyCellIndex = -1
BestBossDistance = -1
BestKeyDistance = -1
```

## InitRandomStream

### Estado

```text
✅ Responsabilidad confirmada
🟡 Nodos exactos no documentados completamente
🛑 No tocar ahora
```

```text
Elegir DungeonSeed o seed aleatoria según bUseRandomSeed
→ Initialize Random Stream
→ guardar RandomStream
```

## CreateStartCell

```text
Make ST_DungeonCell
GridX = 0
GridY = 0
RoomType = Start
bNorth = false
bEast = false
bSouth = false
bWest = false
RoomID = 0
RoomSeed = valor actual
→ Add DungeonCells
```

```text
Make ST_DungeonCellLink
ParentCellIndex = -1
DirectionFromParent = North
bHasParent = false
→ Add DungeonCellLinks
```

Resultado:

```text
DungeonCells.Num = 1
DungeonCellLinks.Num = 1
```

## BuildDungeonLayout

```text
Llamar repetidamente a TryAddRandomCell
hasta alcanzar MaxRooms
o MaxGenerationAttempts
```

Pruebas alcanzadas:

```text
10
15
20
50
150 habitaciones
```

## TryAddRandomCell

### Variables locales

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

### Flujo resumido confirmado

```text
validar DungeonCells no vacío
→ elegir Random Cell Index
→ obtener Selected Cell
→ impedir segunda salida de Start
→ elegir Selected Direction
→ GetNeighborCoord
→ IsCellOccupied
→ generar RoomSeed
→ SetConnectionOnCell del padre
→ actualizar padre en DungeonCells
→ crear New Base Cell Normal
→ GetOppositeDirection
→ conectar hija hacia padre
→ Add DungeonCells
→ Add DungeonCellLinks
→ Return Added=true
```

Regla Start:

```text
Random Cell Index == 0
AND
Selected Cell tiene cualquier conexión
→ Return Added=false
```

Link de hija:

```text
ParentCellIndex = Random Cell Index
DirectionFromParent = Selected Direction
bHasParent = true
```

Invariante:

```text
Éxito → DungeonCells +1 y DungeonCellLinks +1
Fallo → ambos +0
```

## SetConnectionOnCell

Firma:

```text
In Cell : ST_DungeonCell
Direction : E_DungeonDirection
Out Cell : ST_DungeonCell
```

Mapping:

```text
North → bNorth=true
East → bEast=true
South → bSouth=true
West → bWest=true
```

Conserva todos los demás campos y no modifica el array directamente.

## GetNeighborCoord

```text
North → X, Y+1
East → X+1, Y
South → X, Y-1
West → X-1, Y
```

## GetOppositeDirection

Estado actual:

```text
✅ Confirmada
✅ Convertida a Pure durante el refactor padre-hija
```

```text
North → South
East → West
South → North
West → East
```

## GetDirectionVector

Creada el 2026-07-21.

Firma:

```text
Input: Direction : E_DungeonDirection
Output: Direction Vector : Vector
```

Mapping:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

Implementación:

```text
Select indexado por Direction
→ Direction Vector
```

Estado de `Pure`:

```text
🟡 Pendiente de confirmar
```

La captura final todavía muestra pines de ejecución.

## IsCellOccupied

```text
ForEach DungeonCells
→ comparar GridX e GridY simultáneamente
→ ocupado
```

## FindCellIndexByCoord

```text
Resultado inicial = -1
→ recorrer DungeonCells
→ comparar GridX/GridY
→ devolver ArrayIndex al encontrar
```

## ChooseKeyAndBossCells

### Boss

```text
ConnectionCount = suma de N/E/S/W
candidato = RoomType != Start AND ConnectionCount == 1
DistanceFromStart = Abs(GridX) + Abs(GridY)
elegir dead-end más lejano
→ RoomType = Boss preservando todos los datos
```

### Key

```text
RoomType != Start
ArrayIndex != BossCellIndex
DistanceFromStart >= 3
DistanceFromBoss = Manhattan
KeyScore = DistanceFromBoss * DistanceFromStart
→ elegir mejor score
→ RoomType = Key preservando todos los datos
```

## SpawnRoomsFromCells — sistema antiguo

Estado:

```text
✅ Funciona con grid físico fijo
⏳ Sustitución progresiva para tamaños variables
```

Flujo confirmado por capturas:

```text
ForEach DungeonCells
→ Break ST_DungeonCell
→ GridX * CellSize
→ GridY * CellSize
→ Make Vector
→ Make Transform
→ Switch on E_DungeonRoomType
→ Set Selected Room Class
→ SpawnActor
→ InitRoomFromCell
→ Add SpawnedRooms
```

Clases:

```text
Start → StartDebugRoomClass
Normal → DebugRoomClass / clase procedural
Key → KeyDebugRoomClass
Boss → BossDebugRoomClass
```

Limitación:

```text
Salas de tamaños distintos pueden solaparse
```

## SpawnStartRoom

Estado:

```text
✅ Creada
✅ Compila
✅ Prueba aislada superada
✅ SpawnedRooms.Num == 1
✅ SpawnedRooms[0] = Start
```

Flujo:

```text
validar DungeonCells[0]
→ comprobar RoomType Start
→ SpawnActor Start Room Class en GetActorLocation del generador
→ validar actor
→ InitRoomFromCell(DungeonCells[0])
→ Add SpawnedRooms
```

Detalles completos: `docs/26_SPAWN_START_ROOM.md`.

## SpawnFirstChildRoom

Estado:

```text
✅ Creada
✅ Compila
✅ ChildIndex 1 validado
✅ SpawnedRooms.Num == 2
✅ Error final DoorPoint = 0.0
```

Validaciones:

```text
DungeonCells.IsValidIndex(1)
DungeonCellLinks.IsValidIndex(1)
bHasParent == true
SpawnedRooms.IsValidIndex(ParentCellIndex)
Parent actor válido
Child actor válido
```

Variables locales:

```text
Parent Room Actor : Actor Object Reference, no array
Child Room Actor : Actor Object Reference, no array
Child Entry Direction : E_DungeonDirection
Parent Door Location : Vector
Child Door Location : Vector
```

Selección de clase:

```text
Normal → clase Normal/procedural
Key → Key Debug Room Class
Boss → Boss Debug Room Class
Start → error y Return
```

Spawn temporal:

```text
Location = GetActorLocation del generador
Collision Handling = Always Spawn, Ignore Collisions
InitRoomFromCell(DungeonCells[1]) una vez
```

Direcciones:

```text
ParentDirection = DungeonCellLinks[1].DirectionFromParent
ChildEntryDirection = GetOppositeDirection(ParentDirection)
```

Puertas:

```text
ParentDoor = GetDoorWorldLocation(ParentRoomActor, ParentDirection)
ChildDoor = GetDoorWorldLocation(ChildRoomActor, ChildEntryDirection)
```

Alineación exacta validada:

```text
MoveDelta = ParentDoorLocation - ChildDoorLocation
NewLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma hija
```

Resultado:

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
Distance(ChildDoorAfterMove, ParentDoorLocation) = 0.0
```

Error de debug resuelto:

```text
2950
```

Era la distancia entre la puerta y el centro/ubicación del actor, no un error de alineación.

Detalles completos: `docs/27_SPAWN_FIRST_CHILD_ROOM.md`.

## SpawnCorridorsFromConnections

Estado:

```text
✅ Validado visualmente
🛑 No tocar todavía
```

Procesa solo:

```text
North
East
```

North:

```text
CurrentRoom Door North
→ NeighborRoom Door South
→ Spawn BP_Corridor_Debug
```

East:

```text
CurrentRoom Door East
→ NeighborRoom Door West
→ Spawn BP_Corridor_Debug
```

South y West no se procesan para evitar duplicados.

## SpawnBossRoomDoors

Validaciones:

```text
BossCellIndex >= 0
DungeonCells.IsValidIndex(BossCellIndex)
SpawnedRooms.IsValidIndex(BossCellIndex)
```

```text
Get Boss Cell
Get Boss Room Actor
→ comprobar conexiones
→ SpawnDoorAtRoomDirection
```

## SpawnDoorAtRoomDirection

Firma:

```text
RoomActor : Actor Reference
Direction : E_DungeonDirection
```

```text
IsValid RoomActor
→ IsValidClass BossDoorClass
→ GetDoorWorldLocation
→ elegir rotación
→ SpawnActor Always Spawn, Ignore Collisions
→ Add SpawnedDungeonDoors
```

Rotaciones:

```text
North = 0
East = -90
South = 180
West = 90
```

## Funciones Debug

```text
DebugPrintLayout
DebugDrawConnections
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

Documentadas en `docs/12_FUNCIONES_DEBUG.md`.

## Invariantes críticas

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
Una habitación se genera una vez y después se mueve.
No regenerar HISM durante intentos de colocación.
```

## Siguiente paso exacto

```text
confirmar Pure en GetDirectionVector
→ definir CorridorLength temporal
→ DesiredChildDoor = ParentDoorLocation + DirectionVector * CorridorLength
→ MoveDelta = DesiredChildDoor - ChildDoorLocation
→ mover misma hija
→ validar DoorDistance == CorridorLength
→ probar dos direcciones
```

Después se implementarán RoomBounds y reintentos por solapamiento.
