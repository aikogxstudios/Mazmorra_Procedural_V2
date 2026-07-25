# 23 — Seeds y reproducibilidad

**Última actualización:** 2026-07-25

## Objetivo

La misma seed debe permitir reproducir una run para depuración, compartir layouts, repetir errores y preparar pruebas automáticas.

## Variables conocidas

```text
DungeonSeed : Integer
bUseRandomSeed : Boolean
RandomStream : Random Stream
ST_DungeonCell.RoomSeed : Integer
New Room Seed : Integer
```

Variables de placement:

```text
Corridor Length
Placement Retry Step
Max Placement Attempts
```

## InitRandomStream

```text
resolver DungeonSeed
→ inicializar RandomStream
→ usarlo antes de BuildDungeonLayout
```

El gráfico exacto requiere captura antes de modificaciones.

## TryAddRandomCell

Usa `RandomStream` para:

```text
seleccionar padre
seleccionar dirección
crear RoomSeed
```

Determinismo lógico depende de:

```text
misma DungeonSeed
mismo Max Rooms
mismo Max Generation Attempts
mismo orden de llamadas al stream
```

## TryAddSpecialCellFromParent

Usa `RandomStream` para elegir el índice inicial de dirección:

```text
Random Integer in Range from Stream
Min = 0
Max = 3
```

Después prueba las cuatro direcciones de forma determinista:

```text
(Direction Start Index + Loop Index) % 4
```

Esto significa que, con el mismo estado del stream y los mismos datos previos, el orden de direcciones será reproducible.

## Placement

El placement actual es determinista cuando permanecen iguales:

```text
orden de DungeonCells
tamaños de habitaciones
DoorPoints
RoomBounds
Corridor Length
Placement Retry Step
Max Placement Attempts
orden de SpawnedRooms
```

Los reintentos no consumen random adicional: aumentan la longitud con un paso fijo.

## Seeds confirmadas

Corrección actual:

```text
Seed 12345 → South
Seed 12346 → East
```

No documentar 12345 como North sin una nueva prueba visual.

## Resultado actual con Max Rooms 10

```text
1 Start
10 Normal
1 Key
1 Boss
= 13
```

Key y Boss se añaden mediante `TryAddSpecialCellFromParent`, por lo que cambian el consumo del RandomStream respecto al sistema antiguo que convertía habitaciones existentes.

Esto es esperado: una modificación del orden de llamadas puede cambiar resultados posteriores aun usando la misma DungeonSeed.

## RoomSeed

Cada nueva celda guarda:

```text
RoomSeed = New Room Seed
```

Pendiente confirmar que todas las decisiones internas de `BP_RoomMaster_Dungeon` usan un stream derivado de ese `RoomSeed` en lugar de random global.

## Riesgo de random global

Si el tamaño o contenido interno usa nodos random globales:

```text
misma DungeonSeed
≠ necesariamente misma geometría
```

La reproducibilidad completa requiere streams explícitos o sub-seeds.

## Sub-seeds futuras

```text
DungeonSeed
├ LayoutSeed
├ RoomSeed por celda
├ CorridorSeed por conexión
└ ContentSeed por sala
```

Estado:

```text
🔮 diseño futuro
```

## Registros recomendados para bugs

```text
DungeonSeed
MaxRooms
DungeonCells.Num
DungeonCellLinks.Num
SpawnedRooms.Num
ChildIndex
ParentCellIndex
DirectionFromParent
RoomType
RoomSeed
CorridorLength
PlacementAttempt
CandidateBounds
PlacedRoomIndex
```

Para especiales añadir:

```text
SpecialRoomType
Parent Cell Index
Direction Start Index
New Cell Index
```

## Pruebas pendientes

```text
[ ] Ejecutar seed 12345 varias veces y comparar layout completo.
[ ] Ejecutar seed 12346 varias veces y comparar layout completo.
[ ] Confirmar mismos índices Key/Boss.
[ ] Confirmar mismas posiciones físicas.
[ ] Confirmar mismos tamaños procedurales.
[ ] Confirmar misma selección futura de prebuilt/rotación.
```

## Estado actual

```text
✅ RandomStream usado para layout.
✅ RoomSeed guardado en ST_DungeonCell.
✅ Dirección inicial especial usa RandomStream.
✅ Reintentos físicos usan pasos fijos.
🟡 reproducibilidad lógica probable.
🟡 reproducibilidad de placement probable con bounds/tamaños iguales.
⚪ reproducibilidad interna de geometría procedural no confirmada.
```
