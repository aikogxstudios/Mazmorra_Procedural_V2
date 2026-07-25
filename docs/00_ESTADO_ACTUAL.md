# 00 — Estado actual confirmado

**Última actualización:** 2026-07-25  
**Motor:** Unreal Engine 5.4  
**Sistema:** Aguja del Caos / Mazmorra Procedural V2  
**Implementación:** Blueprints

Este archivo es la referencia operativa principal. Cuando exista una contradicción, mandan las capturas y pruebas actuales de Unreal Engine.

## Leyenda

```text
✅ confirmado
🟡 aprobado conceptualmente
⚪ no visible o no comprobado
⏳ pendiente
🔮 futuro
🛑 estable / no tocar sin bug demostrado
```

## Punto exacto al cerrar la sesión

```text
✅ Fase F — reintentos controlados.
✅ Fase G — overlap contra todas las salas aceptadas.
✅ Fase H — colocación física de todas las hijas.
✅ Key y Boss añadidas como salas especiales adicionales.
✅ Max Rooms cuenta solo habitaciones Normal.
⏳ Próximo: regresión final y compatibilidad de puertas para habitaciones prebuilt.
```

Resultado final confirmado:

```text
Max Rooms = 10
→ 10 Normal
+ 1 Start
+ 1 Key
+ 1 Boss
= 13 habitaciones físicas
```

Mensajes confirmados:

```text
KEY SPECIAL ADDED
BOSS SPECIAL ADDED
Total = 13 habitaciones
```

## Flujo temporal actual de GenerateDungeon

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

El flujo antiguo permanece desconectado, no eliminado:

```text
SpawnRoomsFromCells
SpawnCorridorsFromConnections
SpawnBossRoomDoors
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

## Invariante principal

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

No reordenar `SpawnedRooms`, no insertar actores decorativos y no eliminar elementos intermedios.

## Estado lógico estable

Confirmado:

```text
DungeonCells.Num == DungeonCellLinks.Num
```

Validado históricamente con:

```text
10, 15, 20, 50 y 150 celdas
```

También:

```text
DungeonCellLinks[0].ParentCellIndex = -1
DungeonCellLinks[0].bHasParent = false
Hijas: bHasParent = true
Hijas: ParentCellIndex válido
Hijas: ParentCellIndex < ChildIndex
Start: exactamente una salida lógica
```

## Nuevo significado de Max Rooms

```text
Max Rooms = cantidad de habitaciones Normal
```

`BuildDungeonLayout` compara contra:

```text
Max Rooms + 1
```

El `+1` representa únicamente Start.

Las especiales se añaden después y no consumen el límite:

```text
Start
Key
Boss
futuros Room Types especiales
```

Ejemplo actual:

```text
DungeonCells[0]    = Start
DungeonCells[1..10]= Normal
DungeonCells[11]   = Key
DungeonCells[12]   = Boss
```

## SpawnStartRoom

```text
✅ valida DungeonCells[0]
✅ comprueba RoomType == Start
✅ usa GetActorLocation del generador
✅ Make Transform con Rotation 0 y Scale 1,1,1
✅ valida SpawnActor Return Value
✅ InitRoomFromCell una sola vez
✅ añade como SpawnedRooms[0]
```

## PlaceChildRoomFromParent

Firma:

```text
Input : Child Index : Integer
Output: Room Placed : Boolean
```

Responsabilidad:

```text
validar celda y link
resolver Parent Cell Index
resolver Parent Room Actor desde SpawnedRooms
seleccionar clase según Room Type
SpawnActor una sola vez
Init Room from Cell una sola vez
alinear DoorPoints padre-hija
hacer reintentos
comprobar overlap global
aceptar o destruir la candidata
```

Clase por tipo:

```text
Normal → procedural común
Key    → BP_Room_PreBuilt_Base_Child_Key
Boss   → BP_Room_PreBuilt_Base_Child_Boss
Start  → error como hija
```

## Dirección padre-hija

```text
Parent Direction = DungeonCellLinks[Child Index].DirectionFromParent
Child Entry Direction = GetOppositeDirection(Parent Direction)
```

`GetOppositeDirection`:

```text
North → South
East  → West
South → North
West  → East
```

`GetDirectionVector` Pure:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

## Placement y pasillo

```text
DesiredChildDoor =
ParentDoorLocation
+ GetDirectionVector(Parent Direction) * Corridor Length
```

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
```

En cada intento se vuelve a consultar `Child Door Location` después de mover el actor.

Variables locales confirmadas:

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

Valores de prueba:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

## Reintentos controlados

Se usa `For Loop with Break`:

```text
First Index = 0
Last Index = Max Placement Attempts - 1
```

Si solapa:

```text
Corridor Length += Placement Retry Step
→ siguiente intento
```

Si no solapa:

```text
bPlacementSucceeded = true
→ Break
→ añadir a SpawnedRooms
```

Si agota intentos:

```text
Print diagnóstico
→ Destroy Actor
→ Room Placed = false
```

Reglas:

```text
No repetir SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover siempre la misma Child Room Actor.
```

## DoesRoomOverlapPlacedRooms

Firma:

```text
Input : Candidate Room Actor
Output: Overlaps Placed Rooms
```

Recorre `SpawnedRooms`, ignora actores inválidos y la propia candidata, consulta bounds y aplica AABB.

Fórmula:

```text
OverlapX = Abs(CandidateCenterX - PlacedCenterX)
           <= CandidateExtentX + PlacedExtentX
OverlapY = Abs(CandidateCenterY - PlacedCenterY)
           <= CandidateExtentY + PlacedExtentY
OverlapZ = Abs(CandidateCenterZ - PlacedCenterZ)
           <= CandidateExtentZ + PlacedExtentZ
Overlap = X AND Y AND Z
```

`<=` considera contacto exacto como overlap.

## BP_Room_PreBuilt_Base

Contrato confirmado:

```text
BPI_DungeonRoomV2
Init Room from Cell
Get Door World Location
Get Room Bounds Data
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
RoomBounds
```

Implementación de bounds:

```text
RoomBounds
→ Get Component Bounds
   Origin      → Bounds Center
   Box Extent  → Bounds Extent
```

Valores validados:

```text
Start prebuilt:        980,980,400
Procedural de prueba:  995,995,420
```

## Error Key/Boss corregido

La antigua `BP_Room_Debug_Key_C` devolvía:

```text
Candidate Center = 0,0,0
Candidate Extent = 0,0,0
```

Se sustituyó por hijos de la base prebuilt:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

Ahora heredan interfaz, bounds y DoorPoints válidos.

## TryAddSpecialCellFromParent

Firma:

```text
Inputs:
Parent Cell Index : Integer
Special Room Type : E_DungeonRoomType

Outputs:
bAdded : Boolean
New Cell Index : Integer
```

Validaciones:

```text
DungeonCells no vacío
Parent Cell Index válido
Special Room Type != Start
Special Room Type != Normal
```

Todos los fallos devuelven:

```text
bAdded = false
New Cell Index = -1
```

El índice de éxito se obtiene del `Return Value` de `DungeonCells → Add`.

La función prueba las cuatro direcciones:

```text
(Direction Start Index + For Loop.Index) % 4
```

```text
0 North
1 East
2 South
3 West
```

Error corregido:

```text
INCORRECTO: división entre 4
CORRECTO: módulo % 4
```

## ChooseKeyAndBossCells

La búsqueda de candidatos se conserva, pero sus índices ahora representan padres normales:

```text
Key Cell Index  = padre seleccionado para Key
Boss Cell Index = padre seleccionado para Boss
```

Los antiguos `Set Array Elem` que convertían normales en Key/Boss quedaron desconectados.

Al terminar:

```text
Sequence.Then 0 → añadir Key
Sequence.Then 1 → añadir Boss
```

Boss comprueba ocupación después de añadir Key.

## Seeds conocidas

Corrección confirmada:

```text
Seed 12345 → South
Seed 12346 → East
```

## Arquitectura híbrida

```text
Habitaciones normales/procedurales
→ BP_RoomMaster_Dungeon
→ HISM
→ 1–4 conexiones
→ abre solo las necesarias
→ tapa paredes no usadas
```

```text
Habitaciones especiales prebuilt
→ BP_Room_PreBuilt_Base y Blueprints hijos
→ RoomBounds manual
→ DoorPoints manuales
→ Level Instance/Packed Level Blueprint futuro
```

## Próxima fase — compatibilidad de puertas prebuilt

Una habitación prebuilt no debe inventar puertas.

Patrones previstos:

```text
Dead End → 1
Straight → 2 opuestas
Corner/L → 2 contiguas
T        → 3
Cross    → 4
```

Se deberá validar clase + rotación 0/90/180/270 antes del spawn.

## Partes protegidas

```text
🛑 GetNeighborCoord
🛑 IsCellOccupied
🛑 FindCellIndexByCoord
🛑 SetConnectionOnCell
🛑 mappings de dirección
🛑 UpdateRoomBounds de BP_RoomMaster_Dungeon
```

Mappings protegidos:

```text
InitRoomFromCell:
North → SouthOpening
East  → WestOpening
South → NorthOpening
West  → EastOpening

GetDoorWorldLocation:
North → Arrow_Entrance_South
East  → Arrow_Exit_East
South → Arrow_Exit_North
West  → Arrow_Exit_West
```

## Limpieza pendiente

No borrar hasta superar regresión:

```text
SpawnFirstChildRoom
SpawnRoomsFromCells
Set Array Elem antiguos Key/Boss
Print String temporales
variables sin referencias
```

Usar siempre:

```text
Find References
→ compilar
→ prueba controlada
→ borrar
→ regresión
```

## Documentos actuales

```text
docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md
sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md
```
