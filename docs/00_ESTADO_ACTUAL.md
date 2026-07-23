# 00 — Estado actual confirmado

**Última actualización:** 2026-07-23  
**Motor:** Unreal Engine 5.4  
**Sistema:** Aguja del Caos / Mazmorra Procedural V2  
**Implementación:** Blueprints

Este archivo es la referencia operativa principal del repositorio. Cuando exista una contradicción, mandan las capturas y pruebas actuales de Unreal Engine.

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
✅ Fase D.1 — separación inicial de pasillo para ChildIndex 1.
✅ Fase E — RoomBounds y detección AABB padre-hija.
⏳ Próxima fase: Fase F — reintentos controlados.
```

Se continúa todavía únicamente con:

```text
ChildIndex = 1
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
```

Antes de empezar mañana, confirmar que el valor temporal del nodo `Vector * Float` volvió de `-500` a `1000`.

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
→ SpawnFirstChildRoom
```

El flujo antiguo permanece desconectado temporalmente, no eliminado:

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

No reordenar `SpawnedRooms`, no insertar actores decorativos en ese array y no eliminar elementos intermedios.

## Estado lógico estable

Confirmado:

```text
DungeonCells.Num == DungeonCellLinks.Num
```

Validado con:

```text
10, 15, 20, 50 y 150 habitaciones
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

## SpawnStartRoom

Estado:

```text
✅ creada, compilada y probada
✅ valida DungeonCells[0]
✅ comprueba RoomType == Start
✅ usa GetActorLocation del generador
✅ Make Transform con Rotation 0 y Scale 1,1,1
✅ valida SpawnActor Return Value
✅ InitRoomFromCell(DungeonCells[0]) una sola vez
✅ añade el actor como SpawnedRooms[0]
```

Durante la sesión se sustituyó temporalmente la clase Start debug por `BP_Room_PreBuilt_Base` para validar el nuevo contrato de habitaciones preconstruidas.

## SpawnFirstChildRoom

Trabaja únicamente con:

```text
ChildIndex = 1
```

Validaciones confirmadas:

```text
DungeonCells.IsValidIndex(1)
DungeonCellLinks.IsValidIndex(1)
bHasParent == true
SpawnedRooms.IsValidIndex(ParentCellIndex)
Parent Room Actor válido
Child Room Actor válido
```

Variables locales confirmadas:

```text
Parent Room Actor    : Actor Object Reference
Child Room Actor     : Actor Object Reference
Child Entry Direction: E_DungeonDirection
Parent Direction     : E_DungeonDirection
Parent Door Location : Vector
Child Door Location  : Vector
Parent Bounds Center : Vector
Parent Bounds Extent : Vector
Child Bounds Center  : Vector
Child Bounds Extent  : Vector
bBoundsOverlap       : Boolean
```

Ninguna de las referencias de actor anteriores es un array.

## Dirección padre-hija

Origen del dato:

```text
DungeonCellLinks[1]
→ Break ST_DungeonCellLink
→ Direction From Parent
→ Set Parent Direction
```

Entrada de la hija:

```text
Direction From Parent
→ GetOppositeDirection
→ Child Entry Direction
```

`GetOppositeDirection` es Pure y mantiene:

```text
North → South
East  → West
South → North
West  → East
```

## GetDirectionVector

Función Pure confirmada:

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

## Separación inicial de pasillo

Montaje validado:

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

Importante: `1000` sigue siendo un valor temporal escrito en el nodo. Una variable formal `Corridor Length` aún debe crearse o confirmarse durante la Fase F.

Pruebas reproducibles:

```text
Use Random Seed = false
Seed 12345 → North → Distance = 1000
Seed 12346 → East  → Distance = 1000
SpawnedRooms.Num = 2 en ambas
```

La hija se genera e inicializa una sola vez; después solo se mueve.

## BP_Room_PreBuilt_Base

Se creó/adaptó `BP_Room_PreBuilt_Base` como controlador común para habitaciones construidas a mano.

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

La base todavía contiene componentes visuales/debug heredados del prototipo. No limpiarlos hasta asegurar que no participan en `Init Room from Cell` ni en el cierre de paredes.

Arquitectura aprobada:

```text
Habitación procedural común
→ BP_RoomMaster_Dungeon
→ geometría HISM
→ RoomBounds dinámico

Habitación especial preconstruida
→ BP_Room_PreBuilt_Base o Blueprint hijo
→ RoomBounds y DoorPoints en el actor controlador
→ contenido visual futuro mediante Level Instance o Packed Level Blueprint
```

Cada Blueprint hijo prebuilt deberá ajustar manualmente:

```text
RoomBounds.BoxExtent
RoomBounds.RelativeLocation
DoorPoints
```

## Get Room Bounds Data

Firma real confirmada:

```text
Outputs:
Bounds Center : Vector
Bounds Extent : Vector
```

Implementación en `BP_RoomMaster_Dungeon` y `BP_Room_PreBuilt_Base`:

```text
RoomBounds
→ Get Component Bounds
   Origin     → Bounds Center
   Box Extent → Bounds Extent
```

Valores validados en ejecución:

```text
Start prebuilt:
Bounds Extent = X 980, Y 980, Z 400

Primera hija procedural:
Bounds Extent = X 995, Y 995, Z 420
```

`RoomBounds.RelativeLocation` de la Start debe verificarse visualmente en la siguiente revisión; el `Box Extent` sí está confirmado por Print en runtime.

## Detección AABB padre-hija

Después de mover la hija se obtienen los bounds de ambos actores y se guardan en variables locales.

Fórmula validada:

```text
OverlapX = Abs(ParentCenterX - ChildCenterX)
           <= ParentExtentX + ChildExtentX

OverlapY = Abs(ParentCenterY - ChildCenterY)
           <= ParentExtentY + ChildExtentY

OverlapZ = Abs(ParentCenterZ - ChildCenterZ)
           <= ParentExtentZ + ChildExtentZ

bBoundsOverlap = OverlapX AND OverlapY AND OverlapZ
```

Error corregido durante el montaje:

```text
INCORRECTO:
Abs(ParentCenter) - ChildCenter

CORRECTO:
Abs(ParentCenter - ChildCenter)
```

Pruebas confirmadas:

```text
Valor 1000  → bBoundsOverlap = False
Valor -500  → bBoundsOverlap = True
```

Después de la prueba forzada, restaurar el valor a `1000`.

## Próxima fase exacta — Fase F

Objetivo: reintentos controlados para la misma primera hija.

Flujo previsto:

```text
mover Child Room Actor
→ obtener Parent Bounds y Child Bounds
→ calcular bBoundsOverlap
```

Si hay solapamiento:

```text
Corridor Length += Placement Retry Step
→ recalcular Desired Child Door
→ mover la misma Child Room Actor
→ volver a consultar Child Bounds
→ repetir
```

Si no hay solapamiento:

```text
aceptar posición
→ añadir/confirmar Child Room Actor en SpawnedRooms[1]
→ terminar
```

Variables previstas; nombres y tipos deben confirmarse al crearlas:

```text
Corridor Length       : Float
Placement Retry Step  : Float
Placement Attempt     : Integer
Max Placement Attempts: Integer
```

Reglas obligatorias:

```text
No volver a SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover siempre la misma Child Room Actor.
Usar un límite de intentos.
Seguir trabajando solo con ChildIndex = 1.
```

## Partes protegidas

```text
🛑 GetNeighborCoord
🛑 IsCellOccupied
🛑 FindCellIndexByCoord
🛑 SetConnectionOnCell
🛑 BuildDungeonLayout base
🛑 SpawnCorridorsFromConnections
🛑 selección Boss/Key
🛑 SpawnBossRoomDoors
🛑 CenterRoomContentOnActorOrigin
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

## Regla de cantidad V1

Aprobada conceptualmente, todavía no implementada:

```text
15 habitaciones normales/procedurales
+ 1 Start
+ 1 Key
+ 1 Boss
= 18 habitaciones físicas
```

## Documentos de esta fase

```text
docs/28_ROOM_BOUNDS_FIRST_CHILD.md
sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md
```
