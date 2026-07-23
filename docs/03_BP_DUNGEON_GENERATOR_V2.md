# 03 — BP_DungeonGenerator_V2

**Última actualización:** 2026-07-23

## Responsabilidad general

`BP_DungeonGenerator_V2` controla:

```text
layout lógico
DungeonCellLinks
selección Start/Key/Boss
generación física
colocación padre-hija
pasillos y puertas
limpieza y regeneración
```

El refactor actual trabaja con datos primero y spawn después.

## Componente confirmado

```text
Scene
```

## Invariante crítica

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

No reordenar `SpawnedRooms` ni usarlo para decoración.

## Variables miembro conocidas

### Layout y habitaciones

```text
Dungeon Seed          : Integer
Use Random Seed       : Boolean
Max Rooms             : Integer
Max Generation Attempts : Integer
Dungeon Cells         : Array<ST_DungeonCell>
Dungeon Cell Links    : Array<ST_DungeonCellLink>
Spawned Rooms         : Array<Actor Reference>
Random Stream         : Random Stream
Selected Room Class   : Actor Class Reference
```

### Clases de habitación

```text
Start Room Class / StartDebugRoomClass : Actor Class Reference
Room Class / DebugRoomClass            : Actor Class Reference
Key Debug Room Class                   : Actor Class Reference
Boss Debug Room Class                  : Actor Class Reference
```

Durante la prueba actual:

```text
Start Room Class = BP_Room_PreBuilt_Base
```

### Pasillos

```text
Corridor Debug Class : Actor Class Reference
Spawned Corridors    : Array<Actor Reference>
```

### Puertas

```text
Boss Door Class       : Actor Class Reference
Spawned Dungeon Doors : Array<Actor Reference>
```

### Debug

```text
Generate on Begin Play : Boolean
Debug Dungeon          : Boolean
```

## Funciones confirmadas

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
GetDirectionVector
SetConnectionOnCell
ChooseKeyAndBossCells
SpawnStartRoom
SpawnFirstChildRoom
SpawnCorridorsFromConnections
SpawnBossRoomDoors
SpawnDoorAtRoomDirection
DebugPrintLayout
DebugDrawConnections
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
FindCellIndexByCoord
```

## GenerateDungeon

### Flujo temporal actual

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

El flujo anterior está desconectado temporalmente, no eliminado.

## ResetDungeon

Estado:

```text
🛑 protegido
```

Responsabilidad:

```text
destruir puertas
→ destruir pasillos
→ destruir salas
→ limpiar arrays físicos
→ limpiar DungeonCells y DungeonCellLinks
→ resetear índices Boss/Key
```

Solo modificar cuando se añada un array o actor persistente nuevo.

## CreateStartCell

Crea:

```text
DungeonCells[0]
GridX = 0
GridY = 0
RoomType = Start
RoomID = 0
```

Y:

```text
DungeonCellLinks[0]
ParentCellIndex = -1
DirectionFromParent = North
bHasParent = false
```

North es valor de relleno para Start; no se usa porque `bHasParent=false`.

## TryAddRandomCell

Añade una hija lógica y su link padre.

Link confirmado:

```text
ParentCellIndex = Random Cell Index
DirectionFromParent = Selected Direction
bHasParent = true
```

Regla Start:

```text
Random Cell Index == 0
AND
Selected Cell tiene cualquier conexión activa
→ Return Added=false
```

Pruebas:

```text
10, 15, 20, 50 y 150 habitaciones
```

## Funciones protegidas de layout

```text
🛑 GetNeighborCoord
🛑 IsCellOccupied
🛑 FindCellIndexByCoord
🛑 SetConnectionOnCell
🛑 BuildDungeonLayout
```

Mappings de dirección lógica:

```text
North → X, Y+1
East  → X+1, Y
South → X, Y-1
West  → X-1, Y
```

## GetOppositeDirection

Pure confirmada:

```text
North → South
East  → West
South → North
West  → East
```

## GetDirectionVector

Pure confirmada.

Firma:

```text
Input : Direction        : E_DungeonDirection
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
```

## ChooseKeyAndBossCells

Estado:

```text
🛑 protegido
```

Boss:

```text
RoomType != Start
ConnectionCount == 1
→ elegir dead-end más lejano
```

Key:

```text
no Start
no Boss
DistanceFromStart >= 3
→ mejor score DistanceFromBoss * DistanceFromStart
```

## SpawnRoomsFromCells — sistema antiguo

Estado:

```text
✅ referencia funcional
⏳ reemplazo progresivo por placement padre-hija
```

Limitación:

```text
GridX/GridY * CellSize
→ no es seguro para habitaciones de tamaños distintos
```

No borrarlo hasta cerrar el nuevo sistema.

## SpawnStartRoom

Estado:

```text
✅ creada
✅ compilada
✅ prueba aislada superada
```

Flujo:

```text
validar DungeonCells[0]
→ comprobar RoomType == Start
→ Make Transform desde GetActorLocation del generador
→ SpawnActor con Start Room Class
→ validar Return Value
→ Init Room from Cell(DungeonCells[0])
→ Add SpawnedRooms
```

Resultado:

```text
SpawnedRooms[0] = Start
```

Reglas:

```text
Scale = 1,1,1
Init Room from Cell una sola vez
validar actor antes de usarlo
```

## SpawnFirstChildRoom

Estado:

```text
✅ ChildIndex 1 spawneado
✅ separación 1000 validada
✅ bounds padre-hija validados
✅ AABB validada
```

### Validaciones

```text
DungeonCells.IsValidIndex(1)
DungeonCellLinks.IsValidIndex(1)
bHasParent == true
SpawnedRooms.IsValidIndex(ParentCellIndex)
Parent Room Actor válido
Child Room Actor válido
```

### Variables locales confirmadas

```text
Parent Room Actor     : Actor Object Reference
Child Room Actor      : Actor Object Reference
Child Entry Direction : E_DungeonDirection
Parent Direction      : E_DungeonDirection
Parent Door Location  : Vector
Child Door Location   : Vector
Parent Bounds Center  : Vector
Parent Bounds Extent  : Vector
Child Bounds Center   : Vector
Child Bounds Extent   : Vector
bBoundsOverlap        : Boolean
```

### Resolución del padre

```text
DungeonCellLinks[1].ParentCellIndex
→ SpawnedRooms[ParentCellIndex]
→ Parent Room Actor
```

### Dirección

```text
DungeonCellLinks[1].DirectionFromParent
→ Parent Direction
```

```text
Direction From Parent
→ GetOppositeDirection
→ Child Entry Direction
```

### Selección de clase hija

```text
DungeonCells[1].RoomType
→ Switch on E_DungeonRoomType
```

Ramas actuales relevantes:

```text
Normal
Key
Boss
Start → error y Return
```

### Spawn de hija

```text
SpawnActor en GetActorLocation del generador
Collision Handling = Always Spawn, Ignore Collisions
→ validar Return Value
→ Set Child Room Actor
→ Init Room from Cell(DungeonCells[1]) una sola vez
```

La hija nace temporalmente superpuesta y luego se mueve.

### DoorPoints

```text
Parent Door =
Get Door World Location(
  Parent Room Actor,
  Parent Direction
)
```

```text
Child Door =
Get Door World Location(
  Child Room Actor,
  Child Entry Direction
)
```

### Alineación base histórica

```text
MoveDelta = ParentDoorLocation - ChildDoorLocation
```

Resultado validado:

```text
Distance = 0.0
```

### Separación actual

Variable local:

```text
Parent Direction : E_DungeonDirection
```

Montaje:

```text
Parent Direction
→ GetDirectionVector
→ Vector * Float (1000 temporal)
```

```text
DesiredChildDoor =
ParentDoorLocation
+ DirectionVector * 1000
```

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma Child Room Actor
```

Importante:

```text
1000 es un literal temporal.
No existe todavía una variable formal confirmada Corridor Length.
```

Pruebas:

```text
Seed 12345 → North → Distance 1000
Seed 12346 → East  → Distance 1000
SpawnedRooms.Num = 2
```

### Bounds del padre

```text
Parent Room Actor
→ Get Room Bounds Data
→ Parent Bounds Center
→ Parent Bounds Extent
```

### Bounds de la hija

```text
Child Room Actor
→ Get Room Bounds Data
→ Child Bounds Center
→ Child Bounds Extent
```

### Comparación AABB

```text
OverlapX = Abs(ParentCenterX - ChildCenterX)
           <= ParentExtentX + ChildExtentX

OverlapY = Abs(ParentCenterY - ChildCenterY)
           <= ParentExtentY + ChildExtentY

OverlapZ = Abs(ParentCenterZ - ChildCenterZ)
           <= ParentExtentZ + ChildExtentZ

bBoundsOverlap = OverlapX AND OverlapY AND OverlapZ
```

Error corregido:

```text
INCORRECTO: Abs(ParentCenter) - ChildCenter
CORRECTO:   Abs(ParentCenter - ChildCenter)
```

Pruebas:

```text
1000 → False
-500 → True
```

Antes de continuar:

```text
confirmar que el literal volvió a 1000
```

### Resultado del array

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
```

## Fase F — reintentos controlados

Próxima implementación, todavía solo dentro de `SpawnFirstChildRoom` y `ChildIndex=1`.

Variables previstas:

```text
Corridor Length        : Float
Placement Retry Step   : Float
Placement Attempt      : Integer
Max Placement Attempts : Integer
```

Los nombres y valores deben confirmarse al crearlos.

Flujo:

```text
calcular posición candidata
→ mover misma Child Room Actor
→ consultar bounds
→ calcular bBoundsOverlap
```

Si colisiona:

```text
Corridor Length += Placement Retry Step
Placement Attempt += 1
→ repetir
```

Si no colisiona:

```text
aceptar posición
→ conservar SpawnedRooms[1]
```

Si alcanza máximo:

```text
Print String de error
→ Return
```

Reglas:

```text
No volver a SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover la misma referencia.
No comparar todavía contra todas las salas.
```

## SpawnCorridorsFromConnections

Estado:

```text
🛑 protegido
```

Procesa solo:

```text
North
East
```

para evitar duplicados.

## Mappings protegidos

### InitRoomFromCell

```text
North → SouthOpening
East  → WestOpening
South → NorthOpening
West  → EastOpening
```

### GetDoorWorldLocation

```text
North → Arrow_Entrance_South
East  → Arrow_Exit_East
South → Arrow_Exit_North
West  → Arrow_Exit_West
```

## Funciones futuras previstas

```text
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
SpawnRemainingRoomsFromParents
ValidateDungeonData (opcional)
```

No crear todas juntas.

## Referencias

```text
docs/26_SPAWN_START_ROOM.md
docs/27_SPAWN_FIRST_CHILD_ROOM.md
docs/28_ROOM_BOUNDS_FIRST_CHILD.md
docs/29_BP_ROOM_PREBUILT_BASE.md
sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md
```
