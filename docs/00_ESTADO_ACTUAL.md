# 00 — Estado actual confirmado

**Última actualización:** 2026-07-19  
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
✅ Spawn de salas debug.
✅ BP_RoomMaster_Dungeon como sala Normal procedural de prueba.
✅ RoomWidth y RoomDepth variables.
✅ WallHeightTiles variable.
✅ Suelo, paredes, columnas y techo por HISM.
✅ Contenido procedural centrado en el origen del actor.
✅ Aperturas sincronizadas con DungeonCells.
✅ DoorWorldLocation sincronizado con pasillos.
✅ Pasillos verdes correctos por los cuatro lados.
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

No se usa:

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

Si es `false`, `TryAddRandomCell` continúa normalmente.

Pruebas superadas:

```text
✅ 10 habitaciones
✅ 15 habitaciones
✅ 20 habitaciones
✅ 50 habitaciones
✅ 150 habitaciones
```

En todas, la Start mantuvo una sola salida y el layout siguió alcanzando el número objetivo de habitaciones.

## Flujo real actual de GenerateDungeon

Confirmado por captura:

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

`DebugPrintLayout` y `DebugDrawConnections` existen como funciones de diagnóstico; no necesariamente forman parte del cable principal mostrado.

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

## Siguiente paso exacto

Crear y probar únicamente:

```text
SpawnStartRoom
```

Objetivo:

```text
Validar DungeonCells[0]
→ usar StartDebugRoomClass
→ SpawnActor en el origen del generador o world origin acordado
→ InitRoomFromCell con DungeonCells[0]
→ Add a SpawnedRooms
→ garantizar que queda como SpawnedRooms[0]
```

Prueba mínima:

```text
DungeonCells[0]
=
DungeonCellLinks[0]
=
SpawnedRooms[0]
```

No se debe implementar todavía toda la colocación de hijas.

## No avanzar aún a

```text
❌ Colocar todas las habitaciones de una vez.
❌ Solapamiento completo de todo el layout.
❌ Regenerar HISM durante intentos de posición.
❌ Conexión pared con pared.
❌ Level Instances como arquitectura principal.
❌ Decoración, contenido avanzado o nuevas RoomTypes.
```

## Próxima información visual necesaria

Antes de diseñar `SpawnStartRoom`, revisar capturas completas de:

```text
SpawnRoomsFromCells
```

Debe verse:

```text
ForEach DungeonCells
GridX/GridY * CellSize
Switch o selección de RoomType
SelectedRoomClass
SpawnActor
InitRoomFromCell
Add SpawnedRooms
```

No inventar el montaje exacto sin esas capturas.
