# 03 — BP_DungeonGenerator_V2

**Última actualización:** 2026-07-26

## Responsabilidad general

`BP_DungeonGenerator_V2` controla:

```text
layout lógico
DungeonCellLinks
selección de padres para Key/Boss
adición de salas especiales
spawn físico
colocación padre-hija
reintentos y overlap global
pasillos y puertas futuros
limpieza y regeneración
```

Principio:

```text
DATOS PRIMERO → SPAWN DESPUÉS
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

### Layout

```text
Dungeon Seed             : Integer
Use Random Seed          : Boolean
Max Rooms                : Integer
Max Generation Attempts  : Integer
Dungeon Cells            : Array<ST_DungeonCell>
Dungeon Cell Links       : Array<ST_DungeonCellLink>
Spawned Rooms            : Array<Actor Reference>
Random Stream            : Random Stream
Selected Room Class      : Actor Class Reference
```

### Clases

```text
Start Room Class
Room Class / DebugRoomClass
Key Room Class
Boss Room Class
```

Clases especiales confirmadas:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

### Otros

```text
Corridor Debug Class
Spawned Corridors
Boss Door Class
Spawned Dungeon Doors
Generate on Begin Play
Debug Dungeon
```

## Funciones activas confirmadas

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
GetOppositeDirection
GetDirectionVector
SetConnectionOnCell
ChooseKeyAndBossCells
SpawnStartRoom
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
SpawnRoomsFromCells
SpawnCorridorsFromConnections
SpawnBossRoomDoors
SpawnDoorAtRoomDirection
DebugPrintLayout
DebugDrawConnections
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
FindCellIndexByCoord
```

`SpawnFirstChildRoom` ya no existe. Fue eliminado después de comprobar referencias y validar que el sistema completo seguía funcionando.

## GenerateDungeon

Flujo actual:

```text
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ For Loop with Break
   First Index = 1
   Last Index = DungeonCells.Length - 1
   Loop Body → PlaceChildRoomFromParent(Index)
   Room Placed = False → Break
```

Flujo histórico desconectado:

```text
SpawnRoomsFromCells
SpawnCorridorsFromConnections
SpawnBossRoomDoors
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

No borrar cada función sin `Find References` y prueba posterior.

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

## CreateStartCell

```text
DungeonCells[0]
GridX = 0
GridY = 0
RoomType = Start
RoomID = 0
```

```text
DungeonCellLinks[0]
ParentCellIndex = -1
DirectionFromParent = North
bHasParent = false
```

North es relleno para Start y no se usa porque `bHasParent=false`.

## BuildDungeonLayout

`Max Rooms` cuenta solo habitaciones Normal.

```text
DungeonCells.Length >= Max Rooms + 1
```

El `+1` representa Start.

Ejemplo:

```text
Max Rooms = 10
→ BuildDungeonLayout termina con 11 celdas
→ 1 Start + 10 Normal
```

Key y Boss se añaden después.

## TryAddRandomCell

Añade una hija normal y su link.

```text
ParentCellIndex = Random Cell Index
DirectionFromParent = Selected Direction
bHasParent = true
```

Regla Start:

```text
Random Cell Index == 0
AND Start ya tiene cualquier conexión
→ Added = false
```

Validado históricamente con 10, 15, 20, 50 y 150.

## TryAddSpecialCellFromParent

### Firma

```text
Inputs:
Parent Cell Index : Integer
Special Room Type : E_DungeonRoomType

Outputs:
bAdded : Boolean
New Cell Index : Integer
```

### Validaciones

```text
DungeonCells.Length > 0
DungeonCells.IsValidIndex(Parent Cell Index)
Special Room Type != Start
Special Room Type != Normal
```

Fallos:

```text
bAdded = false
New Cell Index = -1
```

### Selección de dirección

```text
(Direction Start Index + For Loop.Index) % 4
```

```text
0 North
1 East
2 South
3 West
```

`For Loop`:

```text
First Index = 0
Last Index = 3
```

Si la coordenada está ocupada, continúa la siguiente iteración.

Si está libre:

```text
crear New Room Seed
actualizar conexión del padre
crear ST_DungeonCell con Special Room Type
crear conexión opuesta
Add DungeonCells
Add DungeonCellLinks
Return true
```

`New Cell Index` sale del `Return Value` de `DungeonCells.Add`.

Si las cuatro direcciones están ocupadas:

```text
Completed → Return false, -1
```

Error corregido:

```text
INCORRECTO: (Start + Index) / 4
CORRECTO:   (Start + Index) % 4
```

## ChooseKeyAndBossCells

La búsqueda de candidatos mantiene sus cálculos históricos.

Boss padre:

```text
RoomType != Start
ConnectionCount == 1
→ candidato dead-end lejano
```

Key padre:

```text
no Start
diferente del candidato Boss
distancia mínima desde Start
→ mejor score de distancia
```

Significado actual:

```text
Boss Cell Index = padre normal para la Boss adicional
Key Cell Index  = padre normal para la Key adicional
```

Los antiguos `Set Array Elem` están desconectados y pendientes de limpieza.

```text
Sequence.Then 0
→ TryAddSpecialCellFromParent(Key Cell Index, Key)

Sequence.Then 1
→ TryAddSpecialCellFromParent(Boss Cell Index, Boss)
```

Key se añade primero. Boss comprueba ocupación después.

## SpawnStartRoom

```text
validar DungeonCells[0]
→ RoomType == Start
→ SpawnActor desde GetActorLocation del generador
→ validar actor
→ Init Room from Cell una sola vez
→ Add SpawnedRooms
```

Resultado:

```text
SpawnedRooms[0] = Start
```

## PlaceChildRoomFromParent

### Firma

```text
Input : Child Index : Integer
Output: Room Placed : Boolean
```

### Validaciones

```text
DungeonCells.IsValidIndex(Child Index)
DungeonCellLinks.IsValidIndex(Child Index)
bHasParent == true
SpawnedRooms.IsValidIndex(Parent Cell Index)
Parent Room Actor válido
Child Room Actor válido
```

### Selección de clase

```text
DungeonCells[Child Index].RoomType
→ Switch on E_DungeonRoomType
```

```text
Normal → procedural
Key    → prebuilt Key
Boss   → prebuilt Boss
Start  → error
```

### Spawn

```text
SpawnActor en ubicación temporal del generador
Collision Handling = Always Spawn, Ignore Collisions
→ validar
→ Init Room from Cell una sola vez
```

### Dirección

```text
Parent Direction = DungeonCellLinks[Child Index].DirectionFromParent
Child Entry Direction = GetOppositeDirection(Parent Direction)
```

### DoorPoints

```text
Parent Door = Get Door World Location(Parent Room Actor, Parent Direction)
Child Door  = Get Door World Location(Child Room Actor, Child Entry Direction)
```

### Posición

```text
DesiredChildDoor =
ParentDoorLocation
+ GetDirectionVector(Parent Direction) * Corridor Length
```

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma Child Room Actor
```

En cada intento se vuelve a consultar `Child Door Location`.

### Variables locales confirmadas

```text
Parent Room Actor      : Actor Object Reference
Child Room Actor       : Actor Object Reference
Child Entry Direction  : E_DungeonDirection
Parent Direction       : E_DungeonDirection
Parent Door Location   : Vector
Child Door Location    : Vector
Corridor Length        : Float
Placement Attempt      : Integer
Placement Retry Step   : Float
Max Placement Attempts : Integer
bPlacement Succeeded   : Boolean
Bounds Overlap         : Boolean
```

Valores probados:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

### Reintentos

```text
For Loop with Break
First Index = 0
Last Index = Max Placement Attempts - 1
```

Por intento:

```text
Placement Attempt = Index
Get Door World Location de la hija actual
Set Child Door Location
Set Actor Location
DoesRoomOverlapPlacedRooms
```

Overlap `true`:

```text
Corridor Length += Placement Retry Step
```

Overlap `false`:

```text
bPlacementSucceeded = true
→ Break
```

Completed con éxito:

```text
Add Child Room Actor a SpawnedRooms
→ Room Placed = true
```

Fallo:

```text
Print diagnóstico
→ Destroy Actor
→ Room Placed = false
```

## DoesRoomOverlapPlacedRooms

### Firma

```text
Input : Candidate Room Actor
Output: Overlaps Placed Rooms
```

### Flujo

```text
validar candidata
→ Get Room Bounds Data de candidata
→ Found Overlap = false
→ For Each Loop with Break sobre SpawnedRooms
```

Por actor colocado:

```text
validar actor
actor != Candidate Room Actor
→ Get Room Bounds Data
→ AABB X/Y/Z
```

Si solapa:

```text
Found Overlap = true
→ Break
```

Completed:

```text
Return Found Overlap
```

### AABB

```text
Abs(CandidateCenterX - PlacedCenterX)
<= CandidateExtentX + PlacedExtentX
```

Igual para Y y Z. Resultado final con AND.

## Error Key/Boss corregido

La clase debug Key devolvía bounds cero y provocaba falsos overlaps.

Solución:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

Ambas heredan `BPI_DungeonRoomV2`, `RoomBounds` y DoorPoints.

## SpawnFirstChildRoom — limpieza completada

Estado:

```text
✅ eliminado
```

Era el prototipo histórico de `ChildIndex = 1` para desarrollar:

```text
alineación
separación
AABB
reintentos
overlap global
```

Verificación:

```text
Find References
→ solo definición visible
→ borrar
→ compilar
→ generación completa funcionando
```

Reemplazo activo:

```text
PlaceChildRoomFromParent
```

Documento:

```text
sessions/2026-07-26_SPAWN_FIRST_CHILD_REMOVED.md
```

## Regresión final de Fase H

```text
Seed 12345 → South → 13 habitaciones sin fallos
Seed 12346 → East  → 13 habitaciones sin fallos
```

En ambas:

```text
1 Start
10 Normal
1 Key
1 Boss
```

## SpawnRoomsFromCells — sistema antiguo

```text
✅ referencia histórica
⏳ reemplazado funcionalmente por loop padre-hija
```

No borrar hasta revisar referencias.

## Limpieza pendiente

```text
SpawnRoomsFromCells
Set Array Elem antiguos de Key/Boss
Print String temporales
variables sin referencias
nodos desconectados
```

Proceso:

```text
Find References
→ compilar
→ prueba controlada
→ borrar
→ regresión
```

## Próxima fase

Compatibilidad entre conexiones lógicas y puertas físicas de habitaciones prefabricadas:

```text
Dead End
Straight
Corner/L
T
Cross
```

Debe seleccionar clase + rotación antes de `SpawnActor` y nunca inventar puertas.

Después se preparará la primera versión del pasillo procedural adaptable entre DoorPoints reales.

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

## Referencias

```text
docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md
sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md
sessions/2026-07-26_PHASE_H_REGRESSION_COMPLETE.md
sessions/2026-07-26_SPAWN_FIRST_CHILD_REMOVED.md
```
