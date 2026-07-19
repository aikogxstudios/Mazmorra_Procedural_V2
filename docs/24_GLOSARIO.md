# 24 — Glosario

## Celda lógica

Entrada de `DungeonCells` que representa una habitación en el grafo, no necesariamente su posición física final.

## Habitación física

Actor spawneado y guardado en `SpawnedRooms`.

## Padre

Celda existente elegida por `Random Cell Index` desde la que nace una nueva celda.

## Hija

Nueva celda creada en `TryAddRandomCell`.

## ChildIndex

Índice de la hija en:

```text
DungeonCells
DungeonCellLinks
SpawnedRooms
```

## ParentCellIndex

Índice del padre guardado en `ST_DungeonCellLink`.

No es el ReturnValue de `Add DungeonCells`.

## DirectionFromParent

Dirección de salida utilizada desde el padre.

Ejemplo:

```text
Parent 3 → East → Child 7
```

La hija entra por `West`, pero el link guarda `East`.

## Opposite Direction

Dirección opuesta usada para conectar la hija de vuelta hacia el padre.

## Layout lógico

Grafo construido en `DungeonCells` mediante coordenadas y conexiones.

## Placement físico

Posición world de los actores de sala.

## DoorPoint

Punto world que representa el centro físico de una conexión de sala.

En salas procedurales se implementa con Arrow Components.

## RoomBounds

Box Collision manual que representa el volumen oficial de una sala.

## BoundsCenter

Centro world devuelto por `GetRoomBoundsData`.

## BoundsExtent

Semitamaño X/Y/Z devuelto por `GetRoomBoundsData`.

## CellSize

Separación física fija usada por el spawn actual basado en grid.

No representa el tamaño de un tile.

## TileSize

Tamaño base/manual de un tile.

## EffectiveTileSize

Tamaño real final de un tile después de generar el suelo.

## RoomWidth

Número de tiles de la habitación en X.

## RoomDepth

Número de tiles de la habitación en Y.

## WallHeightTiles

Número de tiles verticales de pared.

## HISM

`Hierarchical Instanced Static Mesh`. Permite representar muchos tiles como instancias en un componente en vez de actores separados.

## Opening

Hueco de una pared.

## OpeningStartIndex

Índice horizontal donde empieza el hueco.

## OpeningWidthTiles

Ancho del hueco en tiles.

## OpeningHeightTiles

Alto del hueco en tiles.

## Edge Column

Remate en los extremos de una pared para ocultar cortes.

## Start

Primera sala de la run. Prebuilt y con una sola salida.

## Normal

Sala común procedural o prebuilt.

## Key

Sala que contiene la llave principal de la run.

## Boss

Sala terminal prebuilt con una entrada.

## Boss Door

Actor separado que bloquea la entrada a Boss.

## Dead-end

Celda con `ConnectionCount == 1`.

## Distancia Manhattan

```text
Abs(DeltaX) + Abs(DeltaY)
```

## RandomStream

Fuente de random determinista inicializada desde una seed.

## RoomSeed

Seed propia guardada en `ST_DungeonCell`.

## Regresión

Prueba repetida después de un cambio para confirmar que sistemas estables siguen funcionando.

## Parte protegida

Elemento marcado `🛑 no tocar` que requiere bug demostrado antes de modificarse.

## Prebuilt

Habitación construida manualmente como actor o futuro Level Instance.

## Procedural

Habitación construida dinámicamente por `BP_RoomMaster_Dungeon` y actores HISM.

## Pasillo variable

Corredor cuya longitud depende de longitud inicial y reintentos por solapamiento.

## AABB

Caja alineada a ejes usada conceptualmente para comprobar solapamientos con centro y extents.
