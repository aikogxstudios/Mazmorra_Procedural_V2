# 00 — Estado actual confirmado

**Última actualización:** 2026-07-21  
**Motor:** Unreal Engine 5.4  
**Sistema:** Aguja del Caos / Mazmorra Procedural V2

Este archivo es la referencia operativa más importante del repositorio.

## Leyenda

```text
✅ confirmado
🟡 confirmado conceptualmente
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 no tocar
```

## Qué funciona actualmente

```text
✅ Layout lógico en DungeonCells.
✅ Conexiones simétricas North/East/South/West.
✅ Selección de Start, Key y Boss.
✅ Boss preferida como dead-end.
✅ Spawn de salas debug mediante el sistema antiguo.
✅ SpawnStartRoom creada, compilada y probada aisladamente.
✅ SpawnFirstChildRoom creada y probada con ChildIndex 1.
✅ Primera hija alineada matemáticamente con su padre: error 0.0.
✅ SpawnedRooms.Num == 2 en la prueba controlada.
✅ GetDirectionVector creada con mapping N/E/S/W correcto.
✅ BP_RoomMaster_Dungeon como sala Normal procedural de prueba.
✅ RoomWidth y RoomDepth variables.
✅ WallHeightTiles variable.
✅ Suelo, paredes, columnas y techo por HISM.
✅ Contenido procedural centrado en el origen del actor.
✅ Aperturas sincronizadas con DungeonCells.
✅ DoorWorldLocation sincronizado con pasillos.
✅ Pasillos verdes correctos por los cuatro lados en el sistema anterior.
✅ RoomBounds adaptado al tamaño de la sala.
✅ GetRoomBoundsData disponible mediante BPI_DungeonRoomV2.
✅ Boss Door procedural en la entrada de la Boss Room.
✅ Pickup temporal de llave.
✅ Mini inventario experimental.
✅ Boss Door abre solo cuando el inventario contiene la llave requerida.
```

## DungeonCellLinks

Existen:

```text
ST_DungeonCellLink
DungeonCellLinks : Array<ST_DungeonCellLink>
```

Campos:

```text
ParentCellIndex : Integer, default -1
DirectionFromParent : E_DungeonDirection
bHasParent : Boolean, default false
```

La Start añade:

```text
ParentCellIndex = -1
DirectionFromParent = North
bHasParent = false
```

La dirección North de la Start es un valor de relleno; no se usa porque `bHasParent=false`.

Cada hija añade:

```text
ParentCellIndex = Random Cell Index
DirectionFromParent = Selected Direction
bHasParent = true
```

No se usa para `ParentCellIndex`:

```text
Return Value del Add de DungeonCells
Random Direction Index
Opposite Direction
```

## Validaciones completadas

Se comprobó:

```text
DungeonCells.Num == DungeonCellLinks.Num
```

Con:

```text
✅ 10 habitaciones
✅ 15 habitaciones
✅ 20 habitaciones
✅ 50 habitaciones
✅ 150 habitaciones
```

También se revisó:

```text
✅ Índice 0: bHasParent=false y ParentCellIndex=-1.
✅ Índices > 0: bHasParent=true.
✅ ParentCellIndex válido.
✅ ParentCellIndex < ChildIndex.
✅ DirectionFromParent contiene la dirección seleccionada desde el padre.
```

El debug temporal usado para imprimir cantidades y links queda desactivado, pero no eliminado.

## Regla Start con una sola salida

La Start ya está limitada a una sola conexión.

Dentro de `TryAddRandomCell`, después de seleccionar `Random Cell Index` y `Selected Cell`, se comprueba:

```text
Random Cell Index == 0
AND
SelectedCell.bNorth OR SelectedCell.bEast OR SelectedCell.bSouth OR SelectedCell.bWest
```

Si el resultado es `true`:

```text
Return Added = false
```

Pruebas superadas:

```text
✅ 10 habitaciones
✅ 15 habitaciones
✅ 20 habitaciones
✅ 50 habitaciones
✅ 150 habitaciones
```

En todas, la Start mantuvo una sola salida y el layout siguió alcanzando el número objetivo.

## SpawnStartRoom

### Estado

```text
✅ Función creada.
✅ Compila.
✅ Prueba aislada superada.
✅ Solo genera la habitación Start.
✅ Usa Start Room Class / StartDebugRoomClass.
✅ La posición nace desde GetActorLocation del BP_DungeonGenerator_V2.
✅ Transform con escala 1,1,1.
✅ Valida DungeonCells[0].
✅ Comprueba RoomType == Start.
✅ Valida el Return Value de SpawnActor.
✅ Ejecuta InitRoomFromCell con DungeonCells[0].
✅ Añade el actor válido a SpawnedRooms como índice 0.
```

Flujo confirmado:

```text
SpawnStartRoom
→ DungeonCells.IsValidIndex(0)
→ DungeonCells[0]
→ comprobar RoomType == Start
→ MakeTransform(GetActorLocation, Rotation 0, Scale 1)
→ SpawnActor(Start Room Class)
→ IsValid(Return Value)
→ InitRoomFromCell(DungeonCells[0])
→ Add SpawnedRooms
```

Resultado:

```text
✅ Solo aparece Start.
✅ SpawnedRooms.Num == 1.
✅ SpawnedRooms[0] es válido.
✅ Start conserva una única apertura.
```

Documento completo: `docs/26_SPAWN_START_ROOM.md`.

## SpawnFirstChildRoom

### Estado

```text
✅ Función creada y compilada.
✅ Se ejecuta después de SpawnStartRoom en la prueba controlada.
✅ Trabaja únicamente con ChildIndex = 1.
✅ Valida DungeonCells[1].
✅ Valida DungeonCellLinks[1].
✅ Comprueba bHasParent=true.
✅ Obtiene ParentCellIndex y DirectionFromParent.
✅ Obtiene el actor padre desde SpawnedRooms[ParentCellIndex].
✅ Selecciona clase Normal, Key o Boss según DungeonCells[1].RoomType.
✅ Spawnea la hija una sola vez.
✅ InitRoomFromCell(DungeonCells[1]) se ejecuta una sola vez.
✅ La hija se mueve; no se regenera.
✅ Se añade como SpawnedRooms[1].
✅ SpawnedRooms.Num == 2 confirmado.
```

Variables locales confirmadas:

```text
Parent Room Actor : Actor Object Reference
Child Room Actor : Actor Object Reference
Child Entry Direction : E_DungeonDirection
Parent Door Location : Vector
Child Door Location : Vector
```

Ninguna es un array.

### Dirección

```text
DirectionFromParent
→ GetOppositeDirection
→ ChildEntryDirection
```

`GetOppositeDirection` se cambió a función Pure.

### Puertas

```text
ParentDoor = ParentRoomActor.GetDoorWorldLocation(DirectionFromParent)
ChildDoor = ChildRoomActor.GetDoorWorldLocation(ChildEntryDirection)
```

### Movimiento confirmado

Primera prueba sin separación de pasillo:

```text
MoveDelta = ParentDoorLocation - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation(ChildRoomActor, NewChildLocation)
```

Después del movimiento se volvió a consultar la puerta hija y se midió:

```text
Distance(ChildDoorAfterMove, ParentDoorLocation)
```

Resultado confirmado:

```text
0.0
```

Esto demuestra matemáticamente que ambos DoorPoints coinciden exactamente.

### Error de medición resuelto

La primera medición mostró `2950` porque se comparaba accidentalmente la puerta de la hija contra la ubicación del actor/centro de la habitación. No era un fallo de colocación.

La medición correcta usa:

```text
V1 = ChildDoorWorldLocation después de mover
V2 = ParentDoorLocation
```

Documento completo: `docs/27_SPAWN_FIRST_CHILD_ROOM.md`.

## GetDirectionVector

Función creada con:

```text
Input: Direction : E_DungeonDirection
Output: Direction Vector : Vector
```

Mapping confirmado por captura:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

Estado de la propiedad Pure:

```text
🟡 Pendiente de confirmar.
```

La captura final todavía muestra pines blancos de ejecución. El mapping y el resultado están correctos; en la próxima sesión se confirmará o activará `Pure` para simplificar su uso.

## Flujo temporal actual de GenerateDungeon

Durante el refactor físico controlado:

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

El sistema antiguo permanece desconectado temporalmente, no eliminado:

```text
SpawnRoomsFromCells
SpawnCorridorsFromConnections
SpawnBossRoomDoors
Debugs físicos posteriores
```

## Flujo estable anterior de GenerateDungeon

Antes del refactor físico:

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

## Blueprint procedural correcto

```text
BP_RoomMaster
→ original
→ no modificado
```

```text
BP_RoomMaster_Dungeon
→ duplicado integrado con la mazmorra
→ contiene RoomContentRoot
→ contiene RoomBounds
→ implementa BPI_DungeonRoomV2
→ contiene CenterRoomContentOnActorOrigin
→ contiene UpdateRoomBounds
```

## Regla de cantidad V1 aprobada conceptualmente

Objetivo jugable previsto:

```text
15 habitaciones normales/procedurales
+ 1 Start
+ 1 Key
+ 1 Boss
= 18 habitaciones físicas totales
```

Start, Key y Boss no consumen el contador de 15 normales, pero todas permanecen dentro de los arrays para conservar índices. Esta regla aún no está implementada en `MaxRooms`; se aplicará después de estabilizar la colocación padre-hija.

## Siguiente paso exacto

### Separación inicial para el pasillo de la primera hija

Primero confirmar o activar `Pure` en `GetDirectionVector`.

Después modificar únicamente la posición deseada de la puerta hija:

```text
DesiredChildDoor =
ParentDoorLocation
+ GetDirectionVector(DirectionFromParent) * CorridorLength
```

Luego:

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
```

Prueba mínima:

```text
SpawnedRooms.Num == 2
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
Distance(ParentDoor, ChildDoorAfterMove) == CorridorLength
```

Probar al menos dos direcciones antes de continuar.

## No avanzar todavía a

```text
❌ Loop de todas las hijas.
❌ Comparación global de RoomBounds.
❌ Reintentos por solapamiento.
❌ Pasillos variables completos corto/medio/largo.
❌ Regeneración repetida de HISM.
❌ Conexión pared con pared.
❌ Nuevas RoomTypes.
```
