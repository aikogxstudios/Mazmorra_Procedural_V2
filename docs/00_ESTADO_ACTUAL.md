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

Flujo confirmado por capturas:

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

Errores controlados mediante prints:

```text
DungeonCells[0] inválida
DungeonCells[0] no es Start
fallo al spawnear Start Room
```

Resultado de la prueba:

```text
✅ Solo aparece Start.
✅ SpawnedRooms.Num == 1.
✅ SpawnedRooms[0] es válido.
✅ Start conserva una única apertura.
✅ No se generaron hijas durante la prueba aislada.
```

## Flujo estable anterior de GenerateDungeon

Confirmado por captura antes del refactor físico:

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

`SpawnStartRoom` se probó aisladamente sustituyendo temporalmente el bloque físico posterior. La integración final con todas las hijas todavía no está terminada.

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

### Fase D — Una sola hija

No generar todavía toda la mazmorra.

Objetivo inmediato:

```text
ChildIndex = 1
→ leer DungeonCellLinks[1]
→ validar bHasParent
→ obtener ParentCellIndex
→ obtener SpawnedRooms[ParentCellIndex]
→ spawnear una sola habitación hija
→ InitRoomFromCell(DungeonCells[1])
→ obtener ParentDoor y ChildDoor
→ mover la hija para alinear DoorPoint con DoorPoint
→ añadirla como SpawnedRooms[1]
```

Primera prueba esperada:

```text
SpawnedRooms.Num == 2
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
las puertas elegidas quedan alineadas
```

Todavía no añadir en esta prueba:

```text
❌ Loop de todas las hijas.
❌ Comparación de bounds contra todo el layout.
❌ Reintentos por solapamiento.
❌ Pasillos variables.
❌ Regeneración repetida de HISM.
❌ Conexión pared con pared.
```

## Próxima información visual necesaria

Antes de cablear la primera hija, confirmar mediante capturas el montaje real que se vaya creando y revisar cualquier función nueva paso a paso. No inventar nombres, pins ni estructura visual.