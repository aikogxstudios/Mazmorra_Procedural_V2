# 00 — Estado actual confirmado

**Última actualización:** 2026-07-26  
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

## Estado actual

```text
✅ Fase F — reintentos controlados.
✅ Fase G — overlap contra todas las salas aceptadas.
✅ Fase H — colocación física de todas las habitaciones hijas.
✅ Key y Boss añadidas como salas especiales adicionales.
✅ Max Rooms cuenta solo habitaciones Normal.
✅ Regresión final con seeds 12345 y 12346.
✅ SpawnFirstChildRoom eliminado y validado.
✅ SpawnRoomsFromCells eliminado y validado.
✅ Set Array Elem antiguos de Key/Boss eliminados y validados.
⏳ Limpieza controlada de prints, variables y nodos restantes.
⏳ Compatibilidad de puertas físicas para habitaciones prebuilt.
🔮 Pasillo procedural adaptable entre DoorPoints.
```

## Resultado confirmado

Configuración de prueba:

```text
Max Rooms = 10
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

Resultado:

```text
1 Start
10 Normal
1 Key
1 Boss
= 13 habitaciones físicas
```

Regresión:

```text
Seed 12345 → South → 13 habitaciones sin fallos
Seed 12346 → East  → 13 habitaciones sin fallos
```

En ambas:

```text
KEY SPECIAL ADDED
BOSS SPECIAL ADDED
sin SPECIAL FAILED
sin Max Placement Attempts reached
sin solapamientos visibles
```

## Flujo activo de GenerateDungeon

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

## Invariante principal

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

Reglas:

```text
No reordenar SpawnedRooms.
No insertar actores decorativos en SpawnedRooms.
No eliminar elementos intermedios.
No continuar silenciosamente si una habitación no puede colocarse.
```

## Estado lógico estable

```text
DungeonCells.Num == DungeonCellLinks.Num
DungeonCellLinks[0].ParentCellIndex = -1
DungeonCellLinks[0].bHasParent = false
Hijas: bHasParent = true
Hijas: ParentCellIndex válido
Hijas: ParentCellIndex < ChildIndex
Start: exactamente una salida lógica
```

Validación histórica del layout:

```text
10, 15, 20, 50 y 150 celdas
```

## Significado oficial de Max Rooms

```text
Max Rooms = cantidad de habitaciones Normal
```

`BuildDungeonLayout` termina cuando alcanza:

```text
Max Rooms + 1
```

El `+1` representa Start. Las habitaciones especiales se añaden después y no consumen ese límite.

Ejemplo actual:

```text
DungeonCells[0]     = Start
DungeonCells[1..10] = Normal
DungeonCells[11]    = Key
DungeonCells[12]    = Boss
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
→ resolver Parent Cell Index
→ resolver Parent Room Actor desde SpawnedRooms
→ seleccionar clase según Room Type
→ SpawnActor una sola vez
→ Init Room from Cell una sola vez
→ alinear DoorPoints padre-hija
→ hacer reintentos
→ comprobar overlap global
→ aceptar o destruir la candidata
```

Clases actuales:

```text
Normal → habitación procedural común
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

`GetDirectionVector`:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

## Placement y reintentos

```text
DesiredChildDoor =
ParentDoorLocation
+ GetDirectionVector(Parent Direction) * Corridor Length
```

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
```

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

Reglas:

```text
No repetir SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover siempre la misma Child Room Actor.
Volver a consultar Child Door Location en cada intento.
```

## DoesRoomOverlapPlacedRooms

Firma:

```text
Input : Candidate Room Actor
Output: Overlaps Placed Rooms
```

La función recorre `SpawnedRooms`, ignora actores inválidos y la propia candidata, consulta bounds y aplica AABB.

```text
OverlapX = Abs(CandidateCenterX - PlacedCenterX)
           <= CandidateExtentX + PlacedExtentX
OverlapY = Abs(CandidateCenterY - PlacedCenterY)
           <= CandidateExtentY + PlacedExtentY
OverlapZ = Abs(CandidateCenterZ - PlacedCenterZ)
           <= CandidateExtentZ + PlacedExtentZ
Overlap = X AND Y AND Z
```

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

El índice de éxito sale del `Return Value` de `DungeonCells.Add`.

## ChooseKeyAndBossCells

Los índices actuales representan padres normales seleccionados:

```text
Key Cell Index  = padre seleccionado para Key
Boss Cell Index = padre seleccionado para Boss
```

Flujo vigente:

```text
Sequence.Then 0 → TryAddSpecialCellFromParent(Key Cell Index, Key)
Sequence.Then 1 → TryAddSpecialCellFromParent(Boss Cell Index, Boss)
```

Los antiguos `Set Array Elem` que convertían habitaciones Normal en Key/Boss fueron eliminados. Ya no existe una ruta que sustituya habitaciones normales.

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

Bounds:

```text
RoomBounds
→ Get Component Bounds
   Origin     → Bounds Center
   Box Extent → Bounds Extent
```

Clases hijas confirmadas:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

## Limpieza confirmada

Eliminado y probado:

```text
SpawnFirstChildRoom
SpawnRoomsFromCells
Set Array Elem antiguos de Key/Boss
```

Después de estas eliminaciones:

```text
Blueprint compila
la generación completa sigue funcionando
13 habitaciones continúan apareciendo
Start, Key y Boss siguen incluidas
10 habitaciones Normal se conservan
```

Implementación activa:

```text
SpawnStartRoom
+
PlaceChildRoomFromParent
+
TryAddSpecialCellFromParent
```

## Limpieza pendiente

```text
Print String temporales
variables locales o miembro sin referencias
nodos desconectados restantes
```

Proceso obligatorio:

```text
Find References
→ compilar
→ prueba controlada
→ borrar
→ regresión
```

## Próxima fase — puertas físicas prebuilt

Una habitación prefabricada no debe inventar puertas.

Patrones previstos:

```text
Dead End → 1 puerta
Straight → 2 opuestas
Corner/L → 2 contiguas
T        → 3
Cross    → 4
```

Se deberá validar:

```text
clase compatible
+
rotación compatible 0/90/180/270
```

Las habitaciones procedurales seguirán soportando 1–4 conexiones, abriendo solo las necesarias y tapando las paredes no usadas.

## Pasillo procedural adaptable — plan V1

Después de colocar todas las habitaciones:

```text
Parent DoorPoint
→ distancia real
→ Child DoorPoint
→ pasillo recto adaptable
```

Primera versión prevista:

```text
suelo modular
2 paredes
techo opcional
longitud automática
sin spline obligatorio
```

Las futuras salas conectoras de parkour, trampas, puentes o mini combates se tratarán aparte.

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

## Documentos actuales

```text
docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md
sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md
sessions/2026-07-26_PHASE_H_REGRESSION_COMPLETE.md
sessions/2026-07-26_SPAWN_FIRST_CHILD_REMOVED.md
sessions/2026-07-26_SPAWN_ROOMS_FROM_CELLS_REMOVED.md
sessions/2026-07-26_LEGACY_KEY_BOSS_SET_ARRAY_REMOVED.md
knowledge/cleanup_status.yaml
```
