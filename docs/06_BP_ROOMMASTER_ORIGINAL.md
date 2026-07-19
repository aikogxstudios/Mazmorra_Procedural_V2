# 06 — BP_RoomMaster original

## Estado

```text
BP_RoomMaster
→ sistema procedural original
→ no modificado
→ referencia independiente
```

La variante integrada en la mazmorra es `BP_RoomMaster_Dungeon`.

## Objetivo

`BP_RoomMaster` coordina una habitación modular procedural. No coloca cada mesh directamente; configura actores hijos especializados.

## Jerarquía original

```text
SceneRoot
├─ ChildActor_Floor       → BP_FloorActor
├─ ChildActor_Wall_South  → BP_WallActor
├─ ChildActor_Wall_North  → BP_WallActor
├─ ChildActor_Wall_East   → BP_WallActor
├─ ChildActor_Wall_West   → BP_WallActor
├─ ChildActor_Ceiling     → BP_CeilingActor
├─ Arrow_Entrance_South
├─ Arrow_Exit_North
├─ Arrow_Exit_East
└─ Arrow_Exit_West
```

## Filosofía original

```text
Primero funcional.
Luego procedural.
Luego bonito.
Luego optimizado.
```

Prioridades:

- Blueprints claros;
- sin PCG;
- sin plugins procedurales;
- HISM para rendimiento;
- fórmulas simples;
- sistema ampliable.

## Orientación original

```text
        North / salida opcional
              ↑

West ←      ROOM      → East
salida                 salida
opcional               opcional

              ↑
        South / entrada obligatoria
```

Reglas originales:

```text
South = entrada obligatoria siempre abierta
North = salida opcional
East = salida opcional
West = salida opcional
```

La variante Dungeon cambia esta regla y controla las cuatro aperturas.

## Variables principales

```text
RoomWidth
RoomDepth
WallHeightTiles
TileSize
EffectiveTileSize
WallTileHeight
bGenerateCeiling
OpeningWidthTiles
OpeningHeightTiles
bNorthOpening
bEastOpening
bWestOpening
bRandomizeExits
bRandomizeRoomSize
MinRoomWidth
MaxRoomWidth
MinRoomDepth
MaxRoomDepth
ActiveExitArrows
ActiveExitDirections
```

## Variables de tamaño

### RoomWidth

Número de tiles en X.

Afecta a:

- suelo;
- techo;
- South/North;
- bordes East;
- centros de aperturas South/North.

### RoomDepth

Número de tiles en Y.

Afecta a:

- suelo;
- techo;
- East/West;
- borde North;
- centros de aperturas East/West.

### TileSize

Tamaño base/manual de un tile.

### EffectiveTileSize

Tamaño real final de un tile después de generar el suelo.

```text
RoomSizeX = RoomWidth * EffectiveTileSize
RoomSizeY = RoomDepth * EffectiveTileSize
```

No es el tamaño total de la sala.

### WallHeightTiles

Número de filas verticales.

### WallTileHeight

Altura real de cada tile de pared.

```text
RoomHeight = WallHeightTiles * WallTileHeight
```

## Variables de aperturas

### OpeningWidthTiles

Ancho del hueco en tiles.

### OpeningHeightTiles

Alto del hueco en tiles.

Estado original:

```text
OpeningHeightTiles = 1
```

## ActiveExitArrows y ActiveExitDirections

`ActiveExitArrows` guarda flechas de salida activas.

`ActiveExitDirections` guarda el nombre de dirección en el mismo índice.

Ejemplo:

```text
ActiveExitArrows[0] = Arrow_Exit_North
ActiveExitDirections[0] = "North"
```

Regla:

```text
Ambos arrays deben mantener el mismo índice.
```

South no se guardaba porque era entrada obligatoria.

## GenerateRoom original

```text
RandomizeRoomSize
→ DecideRoomExits
→ SetupFloor
→ SetupWallSouth
→ SetupWallNorth
→ SetupWallEast
→ SetupWallWest
→ SetupCeiling
→ UpdateConnectionMarkers
```

`SetupFloor` debe ejecutarse antes de paredes y flechas porque calcula `EffectiveTileSize`.

## RandomizeRoomSize

```text
Branch bRandomizeRoomSize
True:
  RoomWidth = Random Integer(MinRoomWidth, MaxRoomWidth)
  RoomDepth = Random Integer(MinRoomDepth, MaxRoomDepth)
False:
  conservar valores manuales
```

No es obligatorio usar tamaños impares.

## DecideRoomExits

Primero:

```text
bNorthOpening = false
bEastOpening = false
bWestOpening = false
```

Luego:

```text
Random Integer 0→6
```

Casos:

```text
0 → North
1 → East
2 → West
3 → North + East
4 → North + West
5 → East + West
6 → North + East + West
```

Resultado original:

```text
South + entre 1 y 3 salidas opcionales
```

En modo Dungeon, `DecideRoomExits` se salta.

## SetupFloor

```text
ChildActor_Floor.GetChildActor
→ Cast BP_FloorActor
→ WidthTiles = RoomWidth
→ DepthTiles = RoomDepth
→ TileSize = TileSize
→ GenerateFloor
→ EffectiveTileSize = FloorActor.TileSize
```

## Setup general de paredes

Cada función:

```text
calcular ubicación
→ Set Relative Location
→ Set Relative Rotation
→ Get Child Actor
→ Cast BP_WallActor
→ pasar variables
→ GenerateWall
```

Variables enviadas:

```text
LengthTiles
HeightTiles
TileSize
WallTileHeight
HasOpening
OpeningStartIndex
OpeningWidthTiles
OpeningHeightTiles
```

Dimensiones:

```text
South/North → RoomWidth
East/West → RoomDepth
```

Centro:

```text
OpeningStartIndex = (LengthTiles - OpeningWidthTiles) / 2
```

## SetupWallSouth original

```text
Location X = 0
Location Y = EffectiveTileSize * -0.5
Location Z = 0
Rotation Z = 0
LengthTiles = RoomWidth
HasOpening = true
```

## SetupWallNorth original

```text
NorthY = EffectiveTileSize * -0.5 + RoomDepth * EffectiveTileSize
LengthTiles = RoomWidth
HasOpening = bNorthOpening
```

## SetupWallEast original

```text
EastX = EffectiveTileSize * -0.5 + RoomWidth * EffectiveTileSize
LengthTiles = RoomDepth
HasOpening = bEastOpening
```

## SetupWallWest original

```text
WestX = EffectiveTileSize * -0.5
LengthTiles = RoomDepth
HasOpening = bWestOpening
```

## SetupCeiling

```text
Branch bGenerateCeiling
True:
  CeilingZ = WallHeightTiles * WallTileHeight
  Set Relative Location Z = CeilingZ
  Cast BP_CeilingActor
  WidthTiles = RoomWidth
  DepthTiles = RoomDepth
  TileSize = EffectiveTileSize
  GenerateCeiling/GenerateFloor
```

## UpdateConnectionMarkers

Bordes:

```text
SouthY = EffectiveTileSize * -0.5
NorthY = SouthY + RoomDepth * EffectiveTileSize
WestX = EffectiveTileSize * -0.5
EastX = WestX + RoomWidth * EffectiveTileSize
```

Centro South/North:

```text
OpeningCenterX = ((RoomWidth - OpeningWidthTiles) / 2) * EffectiveTileSize
```

Centro East/West:

```text
OpeningCenterY = ((RoomDepth - OpeningWidthTiles) / 2) * EffectiveTileSize
```

### Arrow_Entrance_South

```text
X = OpeningCenterX
Y = SouthY
Z = 0
Yaw = -90
Visible = true
```

### Arrow_Exit_North

```text
X = OpeningCenterX
Y = NorthY
Z = 0
Yaw = 90
Visible = bNorthOpening
```

### Arrow_Exit_East

```text
X = EastX
Y = OpeningCenterY
Z = 0
Yaw = 0
Visible = bEastOpening
```

### Arrow_Exit_West

```text
X = WestX
Y = OpeningCenterY
Z = 0
Yaw = 180
Visible = bWestOpening
```

Después:

```text
Clear ActiveExitArrows
Clear ActiveExitDirections
→ añadir solo salidas activas
```

## Debug recomendado

Para depurar una sala:

```text
bRandomizeRoomSize = false
bRandomizeExits = false
```

Probar manualmente:

```text
RoomWidth
RoomDepth
bNorthOpening
bEastOpening
bWestOpening
```

## Diferencias frente a BP_RoomMaster_Dungeon

`BP_RoomMaster_Dungeon` añade:

```text
bSouthOpening
bUseDungeonConnections
CurrentCellData
RoomContentRoot
RoomBounds
BoundsPadding
BoundsPaddingZ
BoundsHalfHeight
CenterRoomContentOnActorOrigin
UpdateRoomBounds
BPI_DungeonRoomV2
```

Además:

- salta `DecideRoomExits` en modo Dungeon;
- controla cuatro aperturas desde `ST_DungeonCell`;
- usa mappings heredados y validados;
- centra geometría;
- devuelve bounds.

## Regla de mantenimiento

No modificar `BP_RoomMaster` original para solucionar problemas de la variante Dungeon. Los cambios de integración pertenecen a `BP_RoomMaster_Dungeon`.
