# Eliminación de los Set Array Elem antiguos de Key/Boss

**Fecha:** 2026-07-26  
**Blueprint:** `BP_DungeonGenerator_V2`  
**Función:** `ChooseKeyAndBossCells`

## Elementos eliminados

Se eliminaron los bloques antiguos desconectados de `Set Array Elem` que cambiaban el `Room Type` de habitaciones `Normal` existentes a:

```text
Key
Boss
```

## Motivo

Ese comportamiento ya fue reemplazado por el sistema aditivo:

```text
TryAddSpecialCellFromParent(Key Cell Index, Key)
TryAddSpecialCellFromParent(Boss Cell Index, Boss)
```

La arquitectura vigente conserva todas las habitaciones `Normal` y añade las especiales al final de los arrays.

## Resultado confirmado

Con `Max Rooms = 10` se mantiene:

```text
1 Start
10 Normal
1 Key
1 Boss
= 13 habitaciones
```

El usuario confirmó que, tras borrar los bloques antiguos, no afectan a la generación y el sistema continúa funcionando.

## Regla vigente

```text
Max Rooms cuenta solo habitaciones Normal.
Key y Boss se añaden como nuevas celdas especiales.
```

Se conserva la invariante:

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```
