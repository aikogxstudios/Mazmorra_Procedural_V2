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
✅ Función creada.
✅ DungeonCells[0] validada.
✅ RoomType Start validado.
✅ Usa la clase Start.
✅ Usa GetActorLocation del generador como origen.
✅ SpawnActor validado.
✅ InitRoomFromCell ejecutado.
✅ Actor añadido como SpawnedRooms[0].
✅ Prueba aislada: SpawnedRooms.Num == 1.
✅ Start conserva una sola apertura.
```

La fase queda cerrada el 2026-07-21.

### Fase D — Una sola hija

```text
⏳ Fase inmediata actual.
```

Objetivo:

```text
ChildIndex = 1
→ leer DungeonCellLinks[1]
→ validar bHasParent
→ obtener ParentCellIndex
→ obtener SpawnedRooms[ParentCellIndex]
→ spawnear una sola hija
→ InitRoomFromCell(DungeonCells[1])
→ calcular ChildEntryDirection opuesta
→ obtener ParentDoor y ChildDoor
→ mover la hija para alinear puertas
→ añadirla como SpawnedRooms[1]
```

Prueba de aceptación:

```text
SpawnedRooms.Num == 2
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
ParentDoor y ChildDoor quedan alineadas con la separación acordada
```

No incluir todavía bounds globales ni reintentos.

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

## Regla de cantidad V1

Aprobada conceptualmente, todavía no implementada:

```text
15 habitaciones normales/procedurales
+ 1 Start
+ 1 Key
+ 1 Boss
= 18 habitaciones físicas
```

Start, Key y Boss no consumen el contador de 15 normales, pero permanecen dentro de los arrays y conservan sus índices.

## Próximo paso exacto

### 1. Preparar prueba de una sola hija

No recorrer todavía todas las celdas.

```text
Start ya existe como SpawnedRooms[0]
→ trabajar exclusivamente con ChildIndex 1
```

### 2. Validar datos

```text
DungeonCells.IsValidIndex(1)
DungeonCellLinks.IsValidIndex(1)
DungeonCellLinks[1].bHasParent == true
ParentCellIndex válido
SpawnedRooms.IsValidIndex(ParentCellIndex)
```

### 3. Spawnear e inicializar una vez

```text
seleccionar clase según DungeonCells[1].RoomType
→ SpawnActor
→ IsValid
→ InitRoomFromCell(DungeonCells[1])
```

### 4. Alinear puertas

```text
ParentDirection = DirectionFromParent
ChildEntryDirection = GetOppositeDirection(ParentDirection)
ParentDoor = ParentRoom.GetDoorWorldLocation(ParentDirection)
ChildDoor = ChildRoom.GetDoorWorldLocation(ChildEntryDirection)
MoveDelta = DesiredChildDoor - ChildDoor
SetActorLocation(CurrentChildLocation + MoveDelta)
```

La separación inicial del pasillo se definirá al montar esta prueba. No inventar un nombre de variable definitivo hasta crearla y verla en Blueprint.

### 5. Validación

```text
SpawnedRooms.Num == 2
índices 0 y 1 correctos
una sola hija generada
una sola llamada de InitRoomFromCell para la hija
puertas visualmente alineadas
```

## Funciones previstas

Ya creada:

```text
SpawnStartRoom
```

Futuras, una por fase:

```text
PlaceChildRoomFromParent
SpawnRemainingRoomsFromParents
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