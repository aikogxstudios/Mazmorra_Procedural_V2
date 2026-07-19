# 23 — Seeds y reproducibilidad

## Objetivo

La misma seed debería permitir reproducir una run para:

- depuración;
- compartir layouts;
- repetir errores;
- pruebas automáticas futuras;
- contenido determinista.

## Variables conocidas

```text
DungeonSeed : Integer
bUseRandomSeed : Boolean
RandomStream : Random Stream
ST_DungeonCell.RoomSeed : Integer
New Room Seed : Integer local de TryAddRandomCell
```

## InitRandomStream

Responsabilidad conocida:

```text
resolver DungeonSeed
→ inicializar RandomStream
→ usarlo antes de BuildDungeonLayout
```

El gráfico exacto aún necesita captura si se modifica.

## Uso del stream en TryAddRandomCell

Confirmado:

### Selección de padre

```text
Random Integer in Range from Stream
Min = 0
Max = DungeonCells.Length - 1
```

### Selección de dirección

```text
Random Integer in Range from Stream
Min = 0
Max = 3
```

### Seed de hija

```text
Random Integer in Range from Stream
Min = 1
Max = 999999
```

Esto hace determinista el crecimiento lógico siempre que:

- el stream se inicialice igual;
- el orden de llamadas no cambie;
- los mismos reintentos ocurran;
- `MaxRooms` y `MaxGenerationAttempts` sean iguales.

## RoomSeed

Cada hija guarda:

```text
RoomSeed = New Room Seed
```

Objetivo futuro:

```text
BP_RoomMaster_Dungeon
→ usar RoomSeed para tamaño y contenido interno
```

Estado actual exacto:

```text
⚪ No está confirmado si RandomizeRoomSize usa esa seed.
```

## Riesgo de random global

Si `BP_RoomMaster_Dungeon.RandomizeRoomSize` usa nodos random globales en lugar de un stream basado en `RoomSeed`:

```text
misma DungeonSeed
≠
misma geometría de salas
```

Esto debe verificarse antes de declarar la run completamente reproducible.

## Placement futuro y seeds

La longitud inicial de pasillo debería usar `RandomStream` o un stream derivado.

```text
CorridorLength inicial
→ corto/medio/largo
```

No debe depender de random global si se quiere reproducibilidad.

Los reintentos por colisión son deterministas si:

```text
PlacementRetryStep fijo
MaxPlacementAttempts fijo
orden de salas fijo
bounds iguales
```

## Contenido futuro

La misma seed puede derivar sub-seeds:

```text
DungeonSeed
├─ LayoutSeed
├─ RoomSeed por celda
├─ CorridorSeed por conexión
└─ ContentSeed por sala
```

Estado:

```text
🔮 diseño futuro
```

No introducir múltiples streams durante `SpawnStartRoom`.

## Reglas de prueba

### Layout

```text
Misma DungeonSeed
Mismo MaxRooms
Mismo MaxGenerationAttempts
→ mismos GridX/GridY y links
```

### Habitación

```text
Mismo RoomSeed
→ mismo RoomWidth/RoomDepth/altura/contenido
```

Aún pendiente de validar.

### Placement

```text
Mismos tamaños y mismas longitudes iniciales
→ mismas posiciones físicas
```

Pendiente.

## Registro recomendado

Cuando ocurra un bug, imprimir o guardar:

```text
DungeonSeed
MaxRooms
ChildIndex
ParentCellIndex
DirectionFromParent
RoomSeed
CorridorLength
PlacementAttempt
```

## Estado actual

```text
✅ RandomStream usado para layout.
✅ RoomSeed guardado en ST_DungeonCell.
🟡 reproducibilidad lógica probable.
⚪ reproducibilidad de tamaño de sala no confirmada.
⏳ reproducibilidad de placement pendiente.
```
