# 18 — Dudas reales y verificaciones pendientes

Este archivo evita que otro chat convierta una deducción en un hecho.

## SpawnRoomsFromCells

Estado:

```text
🟡 responsabilidad confirmada
⚪ montaje completo no capturado todavía
```

Se conoce conceptualmente:

```text
ForEach DungeonCells
→ GridX/GridY * CellSize
→ elegir clase por RoomType
→ SpawnActor
→ InitRoomFromCell
→ Add SpawnedRooms
```

Falta confirmar visualmente:

- orden exacto de selección de clase;
- uso real de `SelectedRoomClass`;
- nodo Switch/Select exacto;
- política de collision handling;
- manejo de SpawnActor fallido;
- orden exacto entre `InitRoomFromCell` y `Add`;
- si todos los tipos llaman la interfaz igual;
- posición Z y transform completo;
- si existe lógica adicional fuera de la captura resumida.

Acción:

```text
Pedir capturas completas antes de crear SpawnStartRoom.
```

## InitRandomStream

Estado:

```text
✅ responsabilidad conocida
⚪ nodos exactos no visibles
```

Falta confirmar:

- cómo usa `bUseRandomSeed`;
- si modifica `DungeonSeed`;
- nodo exacto de inicialización;
- comportamiento al regenerar;
- si la misma seed reproduce tamaños de RoomMaster.

No tocar durante placement.

## BuildDungeonLayout

Estado:

```text
✅ funcional
🟡 loop conceptual conocido
⚪ montaje exacto parcial/no consolidado
```

Falta registrar nodo por nodo:

- tipo de loop;
- contador de intentos;
- condición exacta de salida;
- actualización cuando `Added=false`;
- prints actuales de finalización.

La regla Start ya fue validada sin modificar esta función.

## DebugDrawConnections

Definición conocida:

```text
Dibuja líneas entre celdas lógicas conectadas.
```

Falta el gráfico exacto.

## DebugDrawDoorPoints

Definición conocida:

```text
Dibuja puntos en puertas activas de salas spawneadas.
```

Falta el gráfico exacto.

## DebugDrawDoorToDoorConnections

Definición conocida:

```text
Dibuja líneas entre DoorPoints físicos conectados.
```

Falta el gráfico exacto.

## DebugPrintLayout

El núcleo está visible, pero falta comprobar si existe lógica adicional fuera de las capturas.

## BoundsHalfHeight

Estado:

```text
✅ variable existente
✅ valor reportado: 300
⚪ uso exacto actual no confirmado
```

`UpdateRoomBounds` usa altura dinámica:

```text
RoomSizeZ = WallHeightTiles * WallTileHeight
ExtentZ = RoomSizeZ * 0.5 + BoundsPaddingZ
```

No borrar `BoundsHalfHeight` hasta usar `Find References` o revisar todos sus pins.

## Seeds de habitación

`ST_DungeonCell.RoomSeed` existe y cada hija recibe `New Room Seed`.

Falta confirmar:

- si `BP_RoomMaster_Dungeon.RandomizeRoomSize` consume `RoomSeed`;
- si usa random global o stream propio;
- si una misma DungeonSeed reproduce dimensiones;
- cómo se transferirá seed en `InitRoomFromCell`.

## Salas prebuilt y bounds

La interfaz requiere `GetRoomBoundsData`, pero falta documentar completamente cómo las salas Start, Key y Boss prebuilt implementan sus bounds.

Antes de placement completo confirmar:

- Box Component o bounds manual;
- DoorPoints de cuatro direcciones;
- tamaño real de Start;
- tamaño real de Boss;
- implementación de interfaz en Key/Boss.

## Start origin

El objetivo dice “en el origen”, pero falta acordar exactamente:

```text
World Origin (0,0,0)
```

o:

```text
GetActorLocation de BP_DungeonGenerator_V2
```

La implementación actual de `SpawnRoomsFromCells` debe servir de referencia.

No decidirlo sin revisar el gráfico y el uso previsto del actor generador en nivel.

## SpawnActor collision handling de salas

Falta confirmar configuración actual.

Durante placement puede ser necesario:

```text
Always Spawn, Ignore Collisions
```

pero no debe asumirse hasta ver el nodo existente.

## Z de pasillos y salas

La lógica se centra en X/Y. Falta confirmar:

- altura Z de las salas;
- altura Z de DoorPoints;
- si `BP_Corridor_Debug` usa Z promedio;
- comportamiento futuro de `Stairs Up/Down`.

V1 actual es plana.

## DoesRoomOverlapPlacedRooms

Diseño pendiente:

```text
AABB manual
vs
BoxOverlap
vs
comprobación basada en BoundsCenter/Extent
```

La documentación aprueba usar `RoomBounds` como fuente, pero no fija todavía el nodo exacto.

## Valores de pasillo

Pendiente decidir:

```text
MinCorridorLength
MaxInitialCorridorLength
PlacementRetryStep
MaxPlacementAttempts
RoomPlacementPadding
```

Los valores deben salir de pruebas con tamaños reales, no de intuición.

## Fallo de placement

Pendiente fijar comportamiento final después de `MaxPlacementAttempts`.

Opciones ordenadas:

1. seguir ampliando;
2. longitud de emergencia;
3. regenerar layout;
4. reportar fallo;
5. nunca loop infinito.

## Inventario temporal

Funciona, pero falta documentar nombres exactos de:

- componente;
- Data Asset de llave;
- clase de pickup;
- función de consulta usada por Boss Door;
- widget.

No es necesario para SpawnStartRoom.

## Política para resolver dudas

```text
1. Pedir captura concreta.
2. Confirmar nombre exacto.
3. Actualizar este archivo.
4. Mover la información al documento principal correspondiente.
5. Marcar la duda como resuelta en CHANGELOG.
```
