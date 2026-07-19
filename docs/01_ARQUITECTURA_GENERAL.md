# 01 — Arquitectura general

## Principio central

```text
DATOS PRIMERO
→ SPAWN DESPUÉS
```

`BP_DungeonGenerator_V2` construye primero el grafo lógico. Solo cuando el layout está completo se generan actores físicos.

Esto separa:

```text
Lógica de mazmorra
≠
Representación física
```

## Responsabilidades por sistema

### BP_DungeonGenerator_V2

Responsable de:

- limpiar la generación anterior;
- inicializar el `RandomStream`;
- crear el layout lógico;
- mantener conexiones simétricas;
- guardar el padre de cada celda;
- elegir Start, Key y Boss;
- seleccionar clases físicas;
- spawnear salas en el orden correcto;
- spawnear pasillos;
- spawnear Boss Doors;
- guardar referencias para limpiar la run;
- en la siguiente fase, colocar salas hijas desde sus padres.

No debe:

- generar tiles individuales;
- gestionar internamente una Boss Door;
- gestionar el inventario final;
- decidir decoración interior concreta;
- regenerar una habitación completa por cada intento de posición.

### BP_RoomMaster_Dungeon

Responsable de:

- recibir `ST_DungeonCell`;
- configurar las cuatro aperturas;
- decidir tamaño procedural si corresponde;
- generar suelo, paredes, columnas y techo;
- centrar el contenido alrededor del origen del actor;
- actualizar DoorPoints;
- calcular `RoomBounds`;
- devolver posiciones de puertas y bounds mediante interfaz.

### BP_RoomMaster

Sistema procedural original. Se conserva sin cambios como referencia independiente.

### BP_FloorActor

Genera la grilla de suelo con HISM y determina `EffectiveTileSize`.

### BP_WallActor

Genera tiles de pared, columnas normales, remates de borde y aperturas mediante HISM.

### BP_CeilingActor

Genera el techo como una grilla equivalente al suelo.

### BPI_DungeonRoomV2

Contrato común entre el generador y cualquier sala física:

```text
InitRoomFromCell
GetDoorWorldLocation
GetRoomBoundsData
```

## Flujo estable actual

```text
ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnRoomsFromCells
→ SpawnCorridorsFromConnections
→ SpawnBossRoomDoors
```

El gráfico real de `GenerateDungeon` incluye además:

```text
Switch Has Authority
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

## Flujo objetivo para tamaños variables

```text
ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ ValidateDungeonCellLinks
→ SpawnStartRoom
→ SpawnRemainingRoomsFromParents
→ SpawnCorridorsFromConnections
→ SpawnBossRoomDoors
```

## Invariante principal de índices

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

Los tres elementos representan la misma habitación.

Nunca:

- ordenar `SpawnedRooms`;
- insertar decoración en `SpawnedRooms`;
- eliminar una referencia intermedia;
- añadir una celda sin link;
- añadir un link sin celda;
- usar el índice de la hija como índice de padre;
- añadir una sala física fuera del orden de `DungeonCells`.

## Arquitectura lógica

Cada `ST_DungeonCell` contiene:

```text
coordenada lógica
RoomType
conexiones N/E/S/W
RoomSeed
RoomID
```

Las coordenadas lógicas siguen siendo útiles para:

- construir el grafo;
- evitar celdas duplicadas;
- elegir Key y Boss;
- depurar;
- buscar vecinos.

Pero la posición física dejará de depender únicamente de:

```text
WorldX = GridX * CellSize
WorldY = GridY * CellSize
```

## Arquitectura física futura

Cada hija se colocará desde la puerta de su padre:

```text
Link = DungeonCellLinks[ChildIndex]
ParentRoom = SpawnedRooms[Link.ParentCellIndex]
ParentDirection = Link.DirectionFromParent
ChildEntryDirection = GetOppositeDirection(ParentDirection)
```

Luego:

```text
ParentDoorLocation
ChildDoorLocation
DirectionVector
CorridorLength
RoomBounds
```

se usarán para alinear, mover y validar la hija.

## Filosofía de generación

```text
Spawn una vez.
GenerateRoom una vez.
Mover varias veces si hace falta.
No destruir/regenerar HISM en cada intento.
Sin Tick.
```

## Visión V1 aprobada

### Start

```text
Siempre preconstruida.
Cualquier tamaño.
Una sola salida.
No vuelve a recibir conexiones.
No puede ser Key ni Boss.
```

### Normal

```text
Procedural o preconstruida.
Tamaños variables.
Puede bifurcar.
```

### Key

```text
Procedural o preconstruida.
Contiene la llave principal de la run.
Idealmente en una rama alejada.
```

### Boss

```text
Siempre preconstruida.
Terminal.
Una entrada en V1.
```

### Pasillos

```text
Siempre presentes en V1.
Cortos, medios o largos.
Se alargan cuando una sala chocaría.
```

### Conexión pared con pared

```text
🔮 Pospuesta
```

Motivos:

- grosor de paredes;
- columnas duplicadas;
- marcos solapados;
- puertas de tamaños distintos;
- complejidad de colisión innecesaria para V1.

## Rendimiento

Decisiones actuales:

```text
✅ HISM para suelo.
✅ HISM para pared.
✅ HISM para columnas.
✅ HISM para techo.
✅ Sin Tick.
✅ Generación por función/evento.
✅ Rooms, pasillos y puertas destruidos al regenerar.
🟡 Child Actors aceptables hasta medir un cuello real.
🛑 No migrar todo a Level Instances sin mediciones.
```

Uso híbrido futuro posible:

```text
Sala procedural común → Actor + HISM
Sala especial prebuilt → Blueprint Actor o Level Instance si se justifica
```

## Regla de colaboración técnica

Si una función no está visible o documentada nodo por nodo:

```text
pedir captura
→ confirmar nombres y pins
→ proponer cambio
→ probar una sola fase
```

No asumir estructura visual, nombres o conexiones por intuición.
