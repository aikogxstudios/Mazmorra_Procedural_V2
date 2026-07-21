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

Fase cerrada el 2026-07-21.

### Fase D — Una sola hija alineada

```text
✅ SpawnFirstChildRoom creada.
✅ ChildIndex = 1.
✅ DungeonCells[1] validada.
✅ DungeonCellLinks[1] validada.
✅ bHasParent=true validado.
✅ ParentCellIndex resuelto.
✅ Parent Room Actor obtenido desde SpawnedRooms.
✅ Hija spawneada e inicializada una sola vez.
✅ ChildEntryDirection opuesta calculada.
✅ ParentDoor y ChildDoor obtenidos por interfaz.
✅ Misma hija movida sin regenerar HISM.
✅ Añadida como SpawnedRooms[1].
✅ SpawnedRooms.Num == 2.
✅ Error de alineación final = 0.0.
```

La prueba base sin separación de pasillo queda validada.

### Fase D.1 — Separación inicial para pasillo

```text
⏳ Próxima fase inmediata.
```

Ya creado:

```text
✅ GetDirectionVector.
✅ North → (0,1,0).
✅ East → (1,0,0).
✅ South → (0,-1,0).
✅ West → (-1,0,0).
```

Pendiente:

```text
Confirmar/activar Pure en GetDirectionVector.
Definir CorridorLength de prueba.
DesiredChildDoor = ParentDoor + DirectionVector * CorridorLength.
Mover la misma hija.
Comprobar DoorDistance == CorridorLength.
Probar al menos dos direcciones.
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

No basta con alinear puertas: el sistema solo aceptará una sala cuando su `RoomBounds` no invada ninguna habitación colocada anteriormente.

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

Reglas:

```text
No regenerar la habitación.
No repetir InitRoomFromCell.
Límite de intentos obligatorio.
```

### Fase G — Todas las hijas

```text
⏳ Pendiente.
```

Objetivo:

```text
SpawnRemainingRoomsFromParents
```

Debe conservar:

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

### Fase H — Pasillos variables

```text
⏳ Pendiente.
```

Objetivo:

```text
pasillo corto/medio/largo
```

El sistema de colocación decidirá primero la distancia física válida. Después `SpawnCorridorsFromConnections` o su adaptación visual conectará los DoorPoints reales.

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
- layouts grandes;
- rendimiento;
- varias seeds.

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

### 1. Confirmar GetDirectionVector

En la captura final el mapping es correcto, pero todavía aparecen pines blancos de ejecución.

```text
Abrir GetDirectionVector
→ confirmar o activar Pure
→ compilar
```

### 2. Elegir longitud de prueba

No fijar todavía el valor definitivo de producción. Usar un valor temporal coherente con el tamaño actual de las salas y baldosas.

Nombre pendiente de confirmar al crearlo en Blueprint:

```text
CorridorLength
```

### 3. Calcular puerta deseada

```text
DirectionVector = GetDirectionVector(DirectionFromParent)
Offset = DirectionVector * CorridorLength
DesiredChildDoor = ParentDoorLocation + Offset
```

### 4. Mover la hija

Sustituir solo la resta actual:

```text
Antes:
MoveDelta = ParentDoorLocation - ChildDoorLocation

Después:
MoveDelta = DesiredChildDoor - ChildDoorLocation
```

Mantener:

```text
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma referencia
```

### 5. Validación

```text
SpawnedRooms.Num == 2
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
Distance(ParentDoor, ChildDoorAfterMove) == CorridorLength
```

Probar al menos dos `DirectionFromParent` distintos.

## Funciones creadas durante el refactor

```text
SpawnStartRoom
SpawnFirstChildRoom
GetDirectionVector
```

## Funciones futuras previstas

```text
PlaceChildRoomFromParent
SpawnRemainingRoomsFromParents
DoesRoomOverlapPlacedRooms
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
