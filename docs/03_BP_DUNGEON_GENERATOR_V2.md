# 03 — BP_DungeonGenerator_V2

## Responsabilidad general

`BP_DungeonGenerator_V2` controla el grafo lógico, la selección de salas especiales y la generación física de la mazmorra.

## Componentes

Confirmado por captura:

```text
Scene
```

No se ha documentado una jerarquía especial adicional. Si aparece un nuevo componente, actualizar este archivo.

## Funciones reales confirmadas

El panel `My Blueprint` muestra 20 funciones propias, más `ConstructionScript`:

```text
1. GenerateDungeon
2. ResetDungeon
3. InitRandomStream
4. CreateStartCell
5. IsCellOccupied
6. GetNeighborCoord
7. TryAddRandomCell
8. BuildDungeonLayout
9. SpawnRoomsFromCells
10. GetOppositeDirection
11. SetConnectionOnCell
12. DebugPrintLayout
13. DebugDrawConnections
14. DebugDrawDoorPoints
15. FindCellIndexByCoord
16. DebugDrawDoorToDoorConnections
17. SpawnCorridorsFromConnections
18. ChooseKeyAndBossCells
19. SpawnBossRoomDoors
20. SpawnDoorAtRoomDirection
```

Además:

```text
ConstructionScript
```

El contador de “overridable” de Unreal no equivale al número de funciones propias.

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
```

### Key Room

```text
KeyCellIndex : Integer
KeyGridX : Integer
KeyGridY : Integer
KeyDebugRoomClass : Actor Class Reference
```

### Puertas y links

```text
BossDoorClass : Actor Class Reference
SpawnedDungeonDoors : Array Actor Reference
DungeonCellLinks : Array ST_DungeonCellLink
```

### Variables documentadas pero no visibles en la captura principal de variables

```text
BestBossDistance : Integer
BestKeyDistance : Integer
```

`BestKeyDistance` funciona realmente como mejor `KeyScore`. Un renombrado futuro puede mejorar claridad, pero no es necesario ahora.

## GenerateDungeon

### Estado

✅ Flujo real confirmado por captura.

### Flujo

```text
GenerateDungeon
→ Switch Has Authority
   Authority → continuar
   Remote → no generar
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

La autoridad evita que clientes remotos ejecuten una segunda generación.

### Futuro

`SpawnRoomsFromCells` será dividido o sustituido por:

```text
SpawnStartRoom
SpawnRemainingRoomsFromParents
```

No cambiar el flujo completo hasta probar primero `SpawnStartRoom` de forma aislada.

## ResetDungeon

### Estado

✅ Estable.  
🛑 Modificar solo para limpiar nuevos arrays o actores.

### Responsabilidad

```text
Destruir puertas
→ destruir pasillos
→ destruir salas
→ limpiar arrays físicos
→ limpiar arrays lógicos
→ resetear índices y scores
```

### Orden recomendado

```text
ForEach SpawnedDungeonDoors
→ IsValid
→ DestroyActor
→ Clear SpawnedDungeonDoors

ForEach SpawnedCorridors
→ IsValid
→ DestroyActor
→ Clear SpawnedCorridors

ForEach SpawnedRooms
→ IsValid
→ DestroyActor
→ Clear SpawnedRooms

Clear DungeonCells
Clear DungeonCellLinks

BossCellIndex = -1
KeyCellIndex = -1
BestBossDistance = -1
BestKeyDistance = -1
```

Riesgo histórico:

```text
No limpiar SpawnedDungeonDoors
→ puertas antiguas o duplicadas al regenerar.
```

## InitRandomStream

### Estado

✅ Responsabilidad confirmada.  
🟡 Nodos exactos no documentados completamente.  
🛑 No tocar ahora.

### Responsabilidad

```text
Elegir DungeonSeed o una seed aleatoria según bUseRandomSeed
→ Initialize Random Stream
→ guardar RandomStream
```

Objetivo: reproducibilidad.

## CreateStartCell

### Estado

✅ Completo.

### Flujo lógico

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
RoomSeed = valor actual del sistema
→ Add DungeonCells

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

La regla de una sola salida de Start no se implementa aquí; se aplica al seleccionar padres en `TryAddRandomCell`.

## BuildDungeonLayout

### Estado

✅ Estable.

### Responsabilidad

```text
Llamar repetidamente a TryAddRandomCell
hasta alcanzar MaxRooms
o MaxGenerationAttempts
```

No debe spawnear actores.

### Invariantes

```text
DungeonCells.Num <= MaxRooms
Intentos limitados
Sin loops infinitos
```

Las pruebas actuales alcanzaron correctamente 10, 15, 20, 50 y 150 salas.

## TryAddRandomCell

### Estado

✅ Núcleo funcional.  
✅ DungeonCellLinks terminado.  
✅ Start limitada a una salida.  
✅ Validado hasta 150 salas.

### Entrada/salida

```text
Exec In
→ Added : Boolean
```

### Variables locales reales

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

### Flujo completo confirmado por capturas

#### 1. Array vacío

```text
DungeonCells Length <= 0
→ Branch
   True → Return Added=false
   False → continuar
```

#### 2. Elegir padre

```text
Random Integer in Range from Stream
Min = 0
Max = DungeonCells.Length - 1
→ Set Random Cell Index

DungeonCells[Random Cell Index]
→ Set Selected Cell
```

#### 3. Evitar segunda salida de Start

Después de `Set Selected Cell`:

```text
Random Cell Index == 0
AND
(
 SelectedCell.bNorth
 OR SelectedCell.bEast
 OR SelectedCell.bSouth
 OR SelectedCell.bWest
)
→ Branch
```

```text
True → Return Added=false
False → continuar
```

Importante:

```text
El Return de la rama True debe tener Added desmarcado/false.
El OR debe incluir las cuatro direcciones.
```

#### 4. Elegir dirección

```text
Random Integer in Range from Stream
Min = 0
Max = 3
→ Set Random Direction Index

Select E_DungeonDirection
0 = North
1 = East
2 = South
3 = West
→ Set Selected Direction
```

#### 5. Calcular coordenada vecina

```text
Break Selected Cell
→ GetNeighborCoord
   In Grid X = SelectedCell.GridX
   In Grid Y = SelectedCell.GridY
   Direction = Selected Direction
→ Set Neighbor X
→ Set Neighbor Y
```

#### 6. Comprobar ocupación

```text
IsCellOccupied(Neighbor X, Neighbor Y)
→ Branch
   True → Return Added=false
   False → continuar
```

#### 7. Generar seed

```text
Random Integer in Range from Stream
Min = 1
Max = 999999
→ Set New Room Seed
```

#### 8. Conectar el padre

```text
SetConnectionOnCell
In Cell = Selected Cell
Direction = Selected Direction
→ Out Cell
→ Set Updated Selected Cell

Set Array Elem
Target Array = DungeonCells
Index = Random Cell Index
Item = Updated Selected Cell
```

#### 9. Crear hija base

```text
Make ST_DungeonCell
GridX = Neighbor X
GridY = Neighbor Y
RoomType = Normal
bNorth = false
bEast = false
bSouth = false
bWest = false
RoomSeed = New Room Seed
RoomID = DungeonCells.Length
→ Set New Base Cell
```

#### 10. Conectar hija hacia padre

```text
GetOppositeDirection(Selected Direction)
→ Set Opposite Direction

SetConnectionOnCell
In Cell = New Base Cell
Direction = Opposite Direction
→ Out Cell
→ Set New Connected Cell
```

#### 11. Añadir celda y link

```text
DungeonCells.Add(New Connected Cell)
→ Print "Dungeon V2 | Added random cell"
→ DungeonCellLinks.Add(
     Make ST_DungeonCellLink
     ParentCellIndex = Random Cell Index
     DirectionFromParent = Selected Direction
     bHasParent = true
   )
→ Return Added=true
```

### Invariantes

```text
Éxito:
DungeonCells +1
DungeonCellLinks +1

Fallo:
DungeonCells +0
DungeonCellLinks +0
```

## SetConnectionOnCell

### Estado

✅ Firma real confirmada.  
🛑 No cambiar sin actualizar todas las llamadas.

### Firma

```text
In Cell : ST_DungeonCell
Direction : E_DungeonDirection
Out Cell : ST_DungeonCell
```

### Funcionamiento

```text
Break In Cell
→ Switch Direction
   North → bNorth=true
   East → bEast=true
   South → bSouth=true
   West → bWest=true
→ Make ST_DungeonCell conservando todos los demás campos
→ Out Cell
```

No modifica `DungeonCells` directamente.

El caller decide:

```text
Padre → Set Array Elem
Hija → guardar Out Cell en New Connected Cell
```

## GetNeighborCoord

### Estado

✅ Confirmado.  
🛑 No tocar.

### Firma

```text
Inputs: In Grid X, In Grid Y, Direction
Outputs: Out Grid X, Out Grid Y
```

### Mapping

```text
North → X, Y+1
East → X+1, Y
South → X, Y-1
West → X-1, Y
```

## GetOppositeDirection

### Estado

✅ Confirmado.  
🛑 No tocar.

```text
North → South
East → West
South → North
West → East
```

## IsCellOccupied

### Estado

✅ Confirmado.  
🛑 No tocar.

### Lógica

```text
ForEach DungeonCells
→ Break Cell
→ Cell.GridX == InGridX
AND Cell.GridY == InGridY
→ ocupado
```

Debe comparar ambas coordenadas simultáneamente.

## FindCellIndexByCoord

### Estado

✅ Confirmado.  
🛑 No tocar.

### Lógica

```text
Resultado inicial = -1
→ recorrer DungeonCells
→ comparar GridX y GridY
→ devolver ArrayIndex al encontrar
```

## ChooseKeyAndBossCells

### Estado

✅ Funcional y probado.

### Inicialización

```text
BossCellIndex = -1
KeyCellIndex = -1
BestBossDistance = -1
BestKeyDistance = -1
```

### Boss

Contar conexiones:

```text
ConnectionCount =
SelectInt(bNorth, 1, 0)
+ SelectInt(bEast, 1, 0)
+ SelectInt(bSouth, 1, 0)
+ SelectInt(bWest, 1, 0)
```

Configuración correcta:

```text
True / Pick A = 1
False / B = 0
```

Candidato Boss:

```text
RoomType != Start
AND ConnectionCount == 1
```

Distancia:

```text
DistanceFromStart = Abs(GridX) + Abs(GridY)
```

Se guarda el dead-end más lejano:

```text
BestBossDistance
BossCellIndex
BossGridX
BossGridY
```

Después se reconstruye la celda preservando todos los campos y cambiando solo:

```text
RoomType = Boss
```

### Key

Condiciones:

```text
RoomType != Start
ArrayIndex != BossCellIndex
DistanceFromStart >= 3
```

Cálculos:

```text
DistanceFromStart = Abs(GridX) + Abs(GridY)
DistanceFromBoss = Abs(GridX-BossGridX) + Abs(GridY-BossGridY)
KeyScore = DistanceFromBoss * DistanceFromStart
```

La mejor puntuación se guarda en `BestKeyDistance` y su índice en `KeyCellIndex`.

Después se cambia solo:

```text
RoomType = Key
```

## SpawnRoomsFromCells

### Estado

✅ Funciona con grid físico fijo.  
⏳ Será sustituida o dividida para tamaños variables.  
⚪ Montaje exacto pendiente de capturas completas antes de tocar.

### Flujo conceptual confirmado

```text
ForEach DungeonCells
→ Break Cell
→ WorldX = GridX * CellSize
→ WorldY = GridY * CellSize
→ seleccionar clase por RoomType
→ SpawnActor
→ InitRoomFromCell Message
→ Add SpawnedRooms
```

Clases:

```text
Start → StartDebugRoomClass
Normal → DebugRoomClass (actualmente puede ser BP_RoomMaster_Dungeon)
Key → KeyDebugRoomClass
Boss → BossDebugRoomClass
```

Limitación:

```text
Salas mayores que el tamaño previsto pueden solaparse.
```

Siguiente refactor aprobado:

```text
SpawnStartRoom
SpawnRemainingRoomsFromParents
```

## SpawnCorridorsFromConnections

### Estado

✅ Validado visualmente.  
🛑 No tocar.

Para evitar duplicados procesa solo:

```text
North
East
```

### Rama North

```text
CurrentCell.bNorth
→ GetNeighborCoord North
→ FindCellIndexByCoord
→ CurrentRoom = SpawnedRooms[CurrentIndex]
→ NeighborRoom = SpawnedRooms[NeighborIndex]
→ Start = CurrentRoom.GetDoorWorldLocation(North)
→ End = NeighborRoom.GetDoorWorldLocation(South)
→ Spawn BP_Corridor_Debug
→ Add SpawnedCorridors
```

### Rama East

```text
CurrentCell.bEast
→ GetNeighborCoord East
→ FindCellIndexByCoord
→ Start = CurrentRoom.GetDoorWorldLocation(East)
→ End = NeighborRoom.GetDoorWorldLocation(West)
→ Spawn BP_Corridor_Debug
→ Add SpawnedCorridors
```

No procesar South y West porque duplicaría conexiones.

## SpawnBossRoomDoors

### Estado

✅ Funcional.

### Validación

```text
BossCellIndex >= 0
DungeonCells.IsValidIndex(BossCellIndex)
SpawnedRooms.IsValidIndex(BossCellIndex)
```

### Flujo

```text
Get Boss Cell
Get Boss Room Actor
→ comprobar bNorth/bEast/bSouth/bWest
→ por cada true llamar SpawnDoorAtRoomDirection
```

Como Boss suele ser dead-end, normalmente solo se crea una puerta.

## SpawnDoorAtRoomDirection

### Estado

✅ Funcional.

### Firma

```text
RoomActor : Actor Reference
Direction : E_DungeonDirection
```

### Flujo

```text
IsValid RoomActor
→ IsValidClass BossDoorClass
→ RoomActor.GetDoorWorldLocation(Direction)
→ elegir DoorRotation
→ SpawnActor BossDoorClass
   Collision Handling = Always Spawn, Ignore Collisions
→ Add SpawnedDungeonDoors
```

Rotaciones aprobadas:

```text
North = Yaw 0
East = Yaw -90
South = Yaw 180
West = Yaw 90
```

## Funciones Debug

Documentadas en `docs/12_FUNCIONES_DEBUG.md`:

```text
DebugPrintLayout
DebugDrawConnections
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

## Siguiente función a crear

```text
SpawnStartRoom
```

No construirla sin revisar antes capturas completas de `SpawnRoomsFromCells`.
