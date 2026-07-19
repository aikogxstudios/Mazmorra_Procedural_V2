# 20 — Fórmulas y valores de referencia

## Coordenadas lógicas

```text
North → (X, Y+1)
East  → (X+1, Y)
South → (X, Y-1)
West  → (X-1, Y)
```

## Opuestos

```text
North ↔ South
East  ↔ West
```

## Vectores físicos futuros

```text
North = (0, +1, 0)
East  = (+1, 0, 0)
South = (0, -1, 0)
West  = (-1, 0, 0)
```

## Grid físico actual

```text
WorldX = GridX * CellSize
WorldY = GridY * CellSize
```

Estado:

```text
✅ funcional para tamaños similares
⏳ será sustituido para tamaños variables
```

## Tamaño de habitación

```text
RoomSizeX = RoomWidth * EffectiveTileSize
RoomSizeY = RoomDepth * EffectiveTileSize
RoomSizeZ = WallHeightTiles * WallTileHeight
```

## Centrado de contenido

```text
OffsetX = (RoomWidth - 1) * EffectiveTileSize * -0.5
OffsetY = (RoomDepth - 1) * EffectiveTileSize * -0.5
OffsetZ = 0
```

## Bounds

```text
ExtentX = RoomSizeX * 0.5 + BoundsPadding
ExtentY = RoomSizeY * 0.5 + BoundsPadding
ExtentZ = RoomSizeZ * 0.5 + BoundsPaddingZ
```

```text
RoomBounds.RelativeLocation = (0, 0, RoomSizeZ * 0.5)
```

Valores actuales:

```text
BoundsPadding = 5
BoundsPaddingZ = 20
BoundsHalfHeight = 300 (existente; verificar uso)
```

## Posiciones de paredes

### South

```text
X = 0
Y = EffectiveTileSize * -0.5
Z = 0
Yaw = 0
LengthTiles = RoomWidth
```

### North

```text
X = 0
Y = RoomDepth * EffectiveTileSize + EffectiveTileSize * -0.5
Z = 0
Yaw = 0
LengthTiles = RoomWidth
```

### West

```text
X = EffectiveTileSize * -0.5
Y = 0
Z = 0
Yaw = 90
LengthTiles = RoomDepth
```

### East

```text
X = RoomWidth * EffectiveTileSize + EffectiveTileSize * -0.5
Y = 0
Z = 0
Yaw = 90
LengthTiles = RoomDepth
```

## Centro de apertura

```text
OpeningStartIndex = (LengthTiles - OpeningWidthTiles) / 2
```

Original RoomMaster, centro visual South/North:

```text
OpeningCenterX = ((RoomWidth - OpeningWidthTiles) / 2) * EffectiveTileSize
```

Centro East/West:

```text
OpeningCenterY = ((RoomDepth - OpeningWidthTiles) / 2) * EffectiveTileSize
```

Valores actuales:

```text
OpeningWidthTiles = normalmente 1
OpeningHeightTiles = 1
```

## Door marker borders originales

```text
SouthY = EffectiveTileSize * -0.5
NorthY = SouthY + RoomDepth * EffectiveTileSize
WestX = EffectiveTileSize * -0.5
EastX = WestX + RoomWidth * EffectiveTileSize
```

## Tiles de suelo

```text
LocationX = IndexX * TileSize
LocationY = IndexY * TileSize
LocationZ = 0
```

Loops:

```text
X = 0 → WidthTiles-1
Y = 0 → DepthTiles-1
```

## Tiles de pared

```text
LocationX = IndexX * TileSize
LocationY = 0
LocationZ = IndexZ * WallTileHeight - MeshMinZ
```

Loops:

```text
X = 0 → LengthTiles-1
Z = 0 → HeightTiles-1
```

## IsOpeningCell

```text
HasOpening
AND CellX >= OpeningStartIndex
AND CellX < OpeningStartIndex + OpeningWidthTiles
AND CellZ < OpeningHeightTiles
```

## Columnas

```text
ColumnIndex % ColumnSpacing == 0
```

```text
X = CurrentColumnIndex * TileSize
Y = ColumnForwardOffset
Z = IndexZ * WallTileHeight - ColumnMinZ
```

## Edge columns

Inicio:

```text
X = TileSize * -0.5
```

Final:

```text
X = ((LengthTiles - 1) * TileSize) + (TileSize * 0.5)
```

No usar:

```text
(((LengthTiles - 1) * TileSize) + TileSize) * 0.5
```

## Techo

```text
CeilingZ = WallHeightTiles * WallTileHeight
```

## Distancia Manhattan desde Start

```text
DistanceFromStart = Abs(GridX) + Abs(GridY)
```

## Distancia a Boss

```text
DistanceFromBoss =
Abs(GridX - BossGridX)
+ Abs(GridY - BossGridY)
```

## Key Score

```text
KeyScore = DistanceFromBoss * DistanceFromStart
```

## ConnectionCount

```text
ConnectionCount =
(bNorth ? 1 : 0)
+ (bEast ? 1 : 0)
+ (bSouth ? 1 : 0)
+ (bWest ? 1 : 0)
```

En Blueprint, cada `Select Int`:

```text
True / Pick A = 1
False / B = 0
```

## Seeds de hija

Rango actual en `TryAddRandomCell`:

```text
1 → 999999
```

## Dirección aleatoria

```text
Random Direction Index = 0 → 3
0 North
1 East
2 South
3 West
```

## Boss Door yaw

```text
North = 0
East = -90
South = 180
West = 90
```

## Alineación padre-hija futura

```text
DesiredChildDoorLocation =
ParentDoorLocation
+ DirectionVector * CorridorLength
```

```text
MoveDelta = DesiredChildDoorLocation - ChildDoorLocation
```

```text
NewChildActorLocation = CurrentChildActorLocation + MoveDelta
```

## Overlap AABB conceptual

```text
OverlapX = Abs(CenterA.X-CenterB.X) <= ExtentA.X+ExtentB.X+Padding
OverlapY = Abs(CenterA.Y-CenterB.Y) <= ExtentA.Y+ExtentB.Y+Padding
OverlapZ = Abs(CenterA.Z-CenterB.Z) <= ExtentA.Z+ExtentB.Z+PaddingZ

Overlap = OverlapX AND OverlapY AND OverlapZ
```

## Reintento futuro

```text
CorridorLength += PlacementRetryStep
```

## Valores de prueba recomendados

```text
RoomWidth: 5, 7, 9, 11
RoomDepth: 5, 7, 9, 11
WallHeightTiles: 3 o 4
OpeningWidthTiles: 1
OpeningHeightTiles: 1
```

Pruebas rectangulares importantes:

```text
5x5
8x5
5x10
10x10
```

## Escalas de prueba del layout

```text
10
15
20
50
150 habitaciones
```
