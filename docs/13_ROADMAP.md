# 13 — Roadmap de implementación

## Estado de fases

### Fase A — DungeonCellLinks

```text
✅ ST_DungeonCellLink creado.
✅ DungeonCellLinks creado.
✅ CreateStartCell añade link 0.
✅ TryAddRandomCell añade link por cada hija.
✅ ParentCellIndex = Random Cell Index.
✅ DirectionFromParent = Selected Direction.
✅ bHasParent=true para hijas.
✅ Cantidades validadas hasta 150 salas.
✅ Contenido básico de links validado.
```

### Fase B — Start con una sola salida

```text
✅ Implementada en TryAddRandomCell.
✅ Comprueba las cuatro conexiones.
✅ Devuelve Added=false si Start ya tiene una salida.
✅ Validada con 10, 15, 20, 50 y 150 salas.
```

### Fase C — SpawnStartRoom

```text
⏳ Siguiente fase inmediata.
```

Objetivo:

```text
Spawn solo de Start
→ índice 0 correcto
→ origen correcto
→ InitRoomFromCell
→ SpawnedRooms[0]
```

Antes de implementarla:

```text
Revisar capturas completas de SpawnRoomsFromCells.
```

### Fase D — Una sola hija

```text
⏳ Pendiente.
```

Objetivo:

```text
ChildIndex = 1
→ leer link
→ obtener padre
→ spawnear hija
→ alinear DoorPoint a DoorPoint
```

### Fase E — Bounds

```text
⏳ Pendiente.
```

Objetivo:

```text
GetRoomBoundsData
→ comparar hija con salas aceptadas
```

### Fase F — Reintentos

```text
⏳ Pendiente.
```

Objetivo:

```text
si hay choque
→ CorridorLength += PlacementRetryStep
→ mover misma hija
→ repetir
```

### Fase G — Todas las hijas

```text
⏳ Pendiente.
```

Objetivo:

```text
SpawnRemainingRoomsFromParents
```

### Fase H — Pasillos variables

```text
⏳ Pendiente.
```

Objetivo:

```text
pasillo corto/medio/largo
```

### Fase I — Regresión completa

```text
⏳ Pendiente.
```

Objetivo:

- Key/Boss;
- Boss Door;
- pickup;
- inventario;
- regeneración;
- layouts grandes.

## Próximo paso exacto

### 1. Recibir capturas

Necesarias de `SpawnRoomsFromCells`:

```text
ForEach DungeonCells
Break Cell
GridX/GridY * CellSize
Switch RoomType
SelectedRoomClass
SpawnActor
InitRoomFromCell
Add SpawnedRooms
```

### 2. Diseñar SpawnStartRoom

No inventar nodos que no existan en el montaje actual.

### 3. Prueba aislada

Temporalmente, `GenerateDungeon` debe poder ejecutar solo:

```text
layout lógico
→ ChooseKeyAndBossCells
→ SpawnStartRoom
```

sin lanzar todavía todas las salas si hace falta una prueba controlada.

### 4. Validación

```text
SpawnedRooms.Num == 1
SpawnedRooms[0] válido
DungeonCells[0].RoomType == Start
DungeonCellLinks[0].bHasParent == false
```

## Funciones futuras previstas

```text
SpawnStartRoom
SpawnRemainingRoomsFromParents
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
GetDirectionVector
```

Posible futura:

```text
ValidateDungeonData
```

No crear todas juntas.

## Variables futuras previstas

```text
MinCorridorLength
MaxInitialCorridorLength
PlacementRetryStep
MaxPlacementAttempts
RoomPlacementPadding
```

Los nombres y valores se confirman al implementarlas.

## Después de la colocación

Solo cuando el sistema físico esté estable:

```text
contenido de salas
→ nuevas RoomTypes
→ eventos
→ decoración
→ pasillos visuales finales
→ optimización medida
```

## Futuro reservado

### Habitaciones opcionales

```text
Treasure
Safe
Shop
Challenge
Secret
Elite
Event
Storage
Trade
Puzzle
```

### Puertas avanzadas

```text
DoorType
RequiredKeyID
DoorGroupID
bConsumesKey
bOpensWholeGroup
bOptional
```

### Seeds completas

```text
Una misma seed debe reproducir:
layout
tipos
tamaños
longitudes iniciales
contenido procedural
```

### Sistema híbrido

```text
Procedural común → BP_RoomMaster_Dungeon + HISM
Prebuilt especial → Blueprint Actor o Level Instance si se justifica
```

## Fuera de alcance actual

```text
❌ Conexión pared con pared.
❌ Migración total a Level Instances.
❌ Refactor grande de Child Actors.
❌ Decoración final.
❌ Inventario definitivo.
❌ Nuevos tipos antes de estabilizar placement.
```

## Regla de avance

```text
Una fase
→ compilar
→ prueba pequeña
→ prueba media
→ prueba grande
→ varias seeds
→ documentar
→ siguiente fase
```
