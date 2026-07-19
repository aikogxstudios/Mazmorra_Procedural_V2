# 07 — BP_FloorActor, BP_WallActor y BP_CeilingActor

# BP_FloorActor

## Responsabilidad

Generar el suelo como una grilla de instancias HISM.

No crea un actor por tile.

## Componentes

```text
SceneRoot
HISM_Floor
```

## Variables

```text
WidthTiles : Integer
DepthTiles : Integer
TileSize : Float
bAutoTileSize : Boolean
FloorMesh : Static Mesh Reference
FloorMaterial : Material Interface Reference
bOverrideFloorMaterial : Boolean
ThemeName : Name
CurrentX : Integer
```

## Funciones

```text
ClearFloor
ApplyFloorVisuals
UpdateTileSize
GenerateFloor
```

## ClearFloor

```text
HISM_Floor.ClearInstances
```

Debe ejecutarse antes de regenerar.

## ApplyFloorVisuals

```text
Si FloorMesh es válido:
  HISM_Floor.SetStaticMesh(FloorMesh)

Si bOverrideFloorMaterial=true y FloorMaterial es válido:
  HISM_Floor.SetMaterial(0, FloorMaterial)
```

Debe ejecutarse antes de `UpdateTileSize` porque cambiar el mesh puede cambiar sus bounds.

## UpdateTileSize

```text
Branch bAutoTileSize
True:
  HISM_Floor.GetLocalBounds
  Size = Max - Min
  TileSize = Abs(Size.X)
False:
  conservar TileSize manual
```

## GenerateFloor

```text
ClearFloor
→ ApplyFloorVisuals
→ UpdateTileSize
→ ForLoop X = 0 → WidthTiles-1
   → Set CurrentX
   → ForLoop Y = 0 → DepthTiles-1
      Location.X = CurrentX * TileSize
      Location.Y = Y * TileSize
      Location.Z = 0
      MakeTransform
      HISM_Floor.AddInstance
```

La grilla crece desde el origen hacia `+X/+Y`.

## Regla clave

```text
EffectiveTileSize representa el tamaño de un tile.
No representa el tamaño total de la habitación.
```

Tamaño total:

```text
RoomSizeX = RoomWidth * EffectiveTileSize
RoomSizeY = RoomDepth * EffectiveTileSize
```

# BP_WallActor

## Responsabilidad

Generar una pared completa mediante HISM:

```text
Tiles de pared
Columnas normales
Edge columns / remates
Huecos / aperturas
```

## Componentes

```text
SceneRoot
HISM_Wall
HISM_Column
HISM_EdgeColumn
```

## Variables de pared

```text
LengthTiles : Integer
HeightTiles : Integer
TileSize : Float
WallTileHeight : Float
bAutoTileSize : Boolean
bAutoWallHeight : Boolean
CurrentX : Integer
MeshMinZ : Float
WallMesh : Static Mesh Reference
WallMaterial : Material Interface Reference
bOverrideWallMaterial : Boolean
ThemeName : Name
```

## Variables de columnas

```text
bGenerateColumns : Boolean
ColumnMesh : Static Mesh Reference
ColumnMaterial : Material Interface Reference
bOverrideColumnMaterial : Boolean
ColumnSpacing : Integer
ColumnForwardOffset : Float
ColumnMinZ : Float
CurrentColumnIndex : Integer
```

## Variables de apertura

```text
HasOpening : Boolean
OpeningStartIndex : Integer
OpeningWidthTiles : Integer
OpeningHeightTiles : Integer
```

Estado actual recomendado:

```text
OpeningHeightTiles = 1
```

## Variables de edge columns

```text
bGenerateEdgeColumns : Boolean
EdgeColumnMesh : Static Mesh Reference
EdgeColumnMaterial : Material Interface Reference
```

## ClearWall

```text
HISM_Wall.ClearInstances
HISM_Column.ClearInstances
HISM_EdgeColumn.ClearInstances
```

## ApplyWallVisuals

```text
Si WallMesh válido:
  HISM_Wall.SetStaticMesh(WallMesh)

Si bOverrideWallMaterial=true y WallMaterial válido:
  HISM_Wall.SetMaterial(0, WallMaterial)
```

## ApplyColumnVisuals

```text
Si ColumnMesh válido:
  HISM_Column.SetStaticMesh(ColumnMesh)

Si bOverrideColumnMaterial=true y ColumnMaterial válido:
  HISM_Column.SetMaterial(0, ColumnMaterial)
```

## ApplyEdgeColumnVisuals

```text
Si EdgeColumnMesh válido:
  HISM_EdgeColumn.SetStaticMesh(EdgeColumnMesh)

Si EdgeColumnMaterial válido:
  HISM_EdgeColumn.SetMaterial(0, EdgeColumnMaterial)
```

## UpdateWallSize

```text
HISM_Wall.GetLocalBounds
Size = Max - Min

Si bAutoTileSize=true:
  TileSize = Abs(Size.X)

Si bAutoWallHeight=true:
  WallTileHeight = Abs(Size.Z)

MeshMinZ = Min.Z
```

Corrección de pivote:

```text
Z = IndexZ * WallTileHeight - MeshMinZ
```

## UpdateColumnSize

```text
HISM_Column.GetLocalBounds
ColumnMinZ = Min.Z
```

Corrección:

```text
Z = IndexZ * WallTileHeight - ColumnMinZ
```

## IsOpeningCell

### Firma

```text
Inputs:
CellX : Integer
CellZ : Integer

Output:
ReturnValue : Boolean
```

### Lógica

```text
ReturnValue =
HasOpening
AND CellX >= OpeningStartIndex
AND CellX < OpeningStartIndex + OpeningWidthTiles
AND CellZ < OpeningHeightTiles
```

Interpretación:

```text
True → pertenece al hueco; no crear instancia.
False → crear pared/columna.
```

Regla crítica:

```text
Branch True → vacío
Branch False → Add Instance
```

## GenerateWall

Orden:

```text
ClearWall
→ ApplyWallVisuals
→ ApplyColumnVisuals
→ ApplyEdgeColumnVisuals
→ UpdateWallSize
→ UpdateColumnSize
→ generar tiles
→ GenerateColumns
→ GenerateEdgeColumns
```

Loop:

```text
ForLoop X = 0 → LengthTiles-1
→ Set CurrentX
→ ForLoop Z = 0 → HeightTiles-1
   → IsOpeningCell(CurrentX, Z)
   → Branch
      True → no hacer nada
      False:
        X = CurrentX * TileSize
        Y = 0
        Z = IndexZ * WallTileHeight - MeshMinZ
        HISM_Wall.AddInstance
```

## GenerateColumns

```text
Branch bGenerateColumns
True:
  ForLoop ColumnIndex = 0 → LengthTiles-1
  → ColumnIndex % ColumnSpacing == 0
  → si true:
     Set CurrentColumnIndex
     ForLoop Z = 0 → HeightTiles-1
     → IsOpeningCell(CurrentColumnIndex, Z)
     → True: no crear
     → False:
        X = CurrentColumnIndex * TileSize
        Y = ColumnForwardOffset
        Z = IndexZ * WallTileHeight - ColumnMinZ
        HISM_Column.AddInstance
```

Regla:

```text
Last Index = LengthTiles - 1
```

No usar `LengthTiles`, porque crea una columna extra.

## GenerateEdgeColumns

### Borde inicial

```text
X = TileSize * -0.5
Y = ColumnForwardOffset
Z = IndexZ * WallTileHeight - ColumnMinZ
```

### Borde final

```text
X = ((LengthTiles - 1) * TileSize) + (TileSize * 0.5)
Y = ColumnForwardOffset
Z = IndexZ * WallTileHeight - ColumnMinZ
```

Fórmula correcta:

```text
((LengthTiles - 1) * TileSize) + (TileSize * 0.5)
```

Fórmula incorrecta:

```text
(((LengthTiles - 1) * TileSize) + TileSize) * 0.5
```

La incorrecta desplaza el remate hacia el centro.

# BP_CeilingActor

## Responsabilidad

Generar el techo como una grilla similar al suelo.

Puede ser un duplicado/adaptación de `BP_FloorActor`.

## Variables equivalentes

```text
WidthTiles
DepthTiles
TileSize
CeilingMesh o FloorMesh según implementación
CeilingMaterial o FloorMaterial según implementación
```

## SetupCeiling desde RoomMaster

```text
Branch bGenerateCeiling
True:
  CeilingZ = WallHeightTiles * WallTileHeight

  ChildActor_Ceiling.SetRelativeLocation(
    X=0,
    Y=0,
    Z=CeilingZ
  )

  GetChildActor
  → Cast BP_CeilingActor
  → WidthTiles = RoomWidth
  → DepthTiles = RoomDepth
  → TileSize = EffectiveTileSize
  → GenerateCeiling o GenerateFloor
```

Si no se ve desde dentro:

- revisar material `Two Sided`;
- revisar normales;
- revisar rotación.

# Reglas de dimensiones

```text
South/North:
LengthTiles = RoomWidth
OpeningStartIndex usa RoomWidth

East/West:
LengthTiles = RoomDepth
OpeningStartIndex usa RoomDepth
```

# Errores comunes

## Índice fuera de rango

```text
Attempted to access index 1 from array of length 1
```

Regla:

```text
Length 1 → índice 0
Length 2 → índices 0,1
```

No usar índices fijos en arrays variables.

## Columna extra

Causa:

```text
Last Index = LengthTiles
```

Solución:

```text
Last Index = LengthTiles - 1
```

## Remate en el centro

Causa: fórmula incorrecta del borde final.

## Pared flotando o hundida

Causa: pivote vertical del mesh.

Solución: `MeshMinZ`.

## Columna atravesando puerta

Causa: `GenerateColumns` no consulta `IsOpeningCell`.

## Hueco relleno

Causa: conectar `True` de `IsOpeningCell` a `AddInstance`.

# Checklist de depuración HISM

```text
1. ¿Se limpian instancias antes de regenerar?
2. ¿Se aplican meshes antes de medir bounds?
3. ¿EffectiveTileSize tiene valor correcto?
4. ¿SetupFloor va antes que paredes?
5. ¿South/North usan RoomWidth?
6. ¿East/West usan RoomDepth?
7. ¿OpeningStartIndex usa la dimensión correcta?
8. ¿OpeningHeightTiles está en 1?
9. ¿IsOpeningCell devuelve true solo en el hueco?
10. ¿Branch True no crea instancia?
11. ¿GenerateColumns consulta IsOpeningCell?
12. ¿Loops terminan en LengthTiles-1?
13. ¿Edge columns usan fórmulas correctas?
14. ¿MeshMinZ y ColumnMinZ corrigen pivotes?
```
