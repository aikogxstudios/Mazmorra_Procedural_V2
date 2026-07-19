# 05 — BP_RoomMaster_Dungeon

## Identidad correcta

```text
BP_RoomMaster
→ original
→ no modificado
```

```text
BP_RoomMaster_Dungeon
→ duplicado adaptado para la Mazmorra Procedural V2
```

La captura con `RoomContentRoot`, `RoomBounds` y las funciones de interfaz corresponde a `BP_RoomMaster_Dungeon`.

## Responsabilidad

`BP_RoomMaster_Dungeon` recibe un `ST_DungeonCell` y genera una habitación procedural compatible con el layout lógico de la mazmorra.

Hace:

- randomizar tamaño si corresponde;
- saltar salidas aleatorias cuando la controla la mazmorra;
- generar suelo;
- generar cuatro paredes;
- abrir huecos según `DungeonCells`;
- generar columnas y remates;
- generar techo;
- actualizar flechas de conexión;
- centrar el contenido;
- actualizar bounds;
- devolver DoorPoints y bounds mediante interfaz.

## Jerarquía de componentes

Confirmada:

```text
DefaultSceneRoot
├─ RoomContentRoot
│  ├─ ChildActor_Floor
│  ├─ ChildActor_Wall_South
│  ├─ ChildActor_Wall_North
│  ├─ ChildActor_Wall_East
│  ├─ ChildActor_Wall_West
│  ├─ ChildActor_Ceiling
│  ├─ Arrow_Entrance_South
│  ├─ Arrow_Exit_North
│  ├─ Arrow_Exit_East
│  └─ Arrow_Exit_West
└─ RoomBounds
```

Reglas:

```text
RoomContentRoot se desplaza para centrar geometría y DoorPoints.
RoomBounds permanece bajo DefaultSceneRoot.
```

## Funciones visibles

### Funciones propias

```text
GenerateRoom
SetupFloor
SetupWall South
SetupWall East
SetupWall North
SetupWall West
SetupCeiling
UpdateConnectionMarkers
DecideRoomExits
RandomizeRoomSize
CenterRoomContentOnActorOrigin
UpdateRoomBounds
```

### Funciones de interfaz

```text
Get Room Bounds Data
Get Door World Location
Init Room from Cell
```

### Adicional

```text
ConstructionScript
```

## Variables heredadas principales

```text
RoomWidth : Integer
RoomDepth : Integer
WallHeightTiles : Integer
TileSize : Float
EffectiveTileSize : Float
WallTileHeight : Float
bGenerateCeiling : Boolean
OpeningWidthTiles : Integer
OpeningHeightTiles : Integer
bNorthOpening : Boolean
bEastOpening : Boolean
bWestOpening : Boolean
bRandomizeExits : Boolean
bRandomizeRoomSize : Boolean
MinRoomWidth : Integer
MaxRoomWidth : Integer
MinRoomDepth : Integer
MaxRoomDepth : Integer
ActiveExitArrows : Array Arrow Component Reference
ActiveExitDirections : Array Name
```

## Variables añadidas para Dungeon

| Variable | Tipo | Estado | Uso |
|---|---|---:|---|
| `bSouthOpening` | Boolean | ✅ | Controlar pared South |
| `bUseDungeonConnections` | Boolean | ✅ | Saltar `DecideRoomExits` |
| `CurrentCellData` | `ST_DungeonCell` | ✅ | Copia de la celda recibida |
| `BoundsPadding` | Float | ✅ | Margen XY |
| `BoundsPaddingZ` | Float | ✅ | Margen Z |
| `BoundsHalfHeight` | Float | ✅ existente | Variable de referencia/posible legacy |

Valores visuales aprobados:

```text
BoundsPadding = 5
BoundsPaddingZ = 20
BoundsHalfHeight = 300
```

`BoundsHalfHeight` no debe borrarse hasta confirmar sus referencias reales. La función actual usa altura dinámica.

## GenerateRoom adaptado

Orden aprobado:

```text
RandomizeRoomSize
→ Branch bUseDungeonConnections
   True  → saltar DecideRoomExits
   False → DecideRoomExits
→ SetupFloor
→ SetupWallSouth
→ SetupWallNorth
→ SetupWallEast
→ SetupWallWest
→ SetupCeiling
→ UpdateConnectionMarkers
→ CenterRoomContentOnActorOrigin
→ UpdateRoomBounds
```

Reglas:

```text
SetupFloor debe ir antes que paredes, flechas y bounds.
Debe existir una sola llamada a GenerateRoom durante InitRoomFromCell.
```

Error antiguo eliminado:

```text
Event ActorBeginOverlap → GenerateRoom
```

No volver a generar la sala por overlap.

## InitRoomFromCell

### Flujo

```text
Set CurrentCellData
→ Break ST_DungeonCell
→ Set bUseDungeonConnections = true
→ Set bRandomizeExits = false
→ adaptar las cuatro aperturas
→ GenerateRoom
```

### Mapping final confirmado

```text
ST_DungeonCell.bNorth → bSouthOpening
ST_DungeonCell.bEast  → bWestOpening
ST_DungeonCell.bSouth → bNorthOpening
ST_DungeonCell.bWest  → bEastOpening
```

Resumen:

```text
Cell N → Room opening S
Cell E → Room opening W
Cell S → Room opening N
Cell W → Room opening E
```

🛑 No cambiar por intuición. Fue validado visualmente.

## GetDoorWorldLocation

### Mapping final confirmado

```text
Dungeon North → Arrow_Entrance_South.GetWorldLocation
Dungeon East  → Arrow_Exit_East.GetWorldLocation
Dungeon South → Arrow_Exit_North.GetWorldLocation
Dungeon West  → Arrow_Exit_West.GetWorldLocation
```

Resumen:

```text
North/South invertidos.
East/West nominales.
```

🛑 No intentar unificarlo con el mapping de `InitRoomFromCell`.

## CenterRoomContentOnActorOrigin

### Problema original

`BP_FloorActor` genera:

```text
X = 0 → +X
Y = 0 → +Y
```

El origen del actor quedaba en una esquina de la geometría.

### Solución

```text
OffsetX = (RoomWidth - 1) * EffectiveTileSize * -0.5
OffsetY = (RoomDepth - 1) * EffectiveTileSize * -0.5
OffsetZ = 0
```

Aplicación:

```text
RoomContentRoot.SetRelativeLocation(OffsetX, OffsetY, 0)
```

Resultado confirmado:

- actor origin centrado en el suelo;
- DoorPoints mantienen posición relativa;
- habitaciones rectangulares siguen centradas;
- pasillos apuntan a huecos reales.

🛑 No cambiar mientras el suelo siga generándose entre `0` y `N-1`.

## UpdateRoomBounds

### Tamaño total

```text
RoomSizeX = RoomWidth * EffectiveTileSize
RoomSizeY = RoomDepth * EffectiveTileSize
RoomSizeZ = WallHeightTiles * WallTileHeight
```

### Extents

```text
ExtentX = RoomSizeX * 0.5 + BoundsPadding
ExtentY = RoomSizeY * 0.5 + BoundsPadding
ExtentZ = RoomSizeZ * 0.5 + BoundsPaddingZ
```

No multiplicar `ExtentZ` por `0.5` otra vez.

### Aplicación

```text
RoomBounds.SetBoxExtent(ExtentX, ExtentY, ExtentZ)
RoomBounds.SetRelativeLocation(0, 0, RoomSizeZ * 0.5)
```

La posición Z coloca la caja desde el suelo hasta el techo.

## GetRoomBoundsData

```text
RoomBounds
→ Get Component Bounds
   Origin → BoundsCenter
   Box Extent → BoundsExtent
```

No usar `GetActorBounds`.

## Fórmulas físicas de paredes

Después de la corrección East/West:

### South

```text
X = 0
Y = EffectiveTileSize * -0.5
Rotation Z = 0
LengthTiles = RoomWidth
```

### North

```text
X = 0
Y = (RoomDepth * EffectiveTileSize) + (EffectiveTileSize * -0.5)
Rotation Z = 0
LengthTiles = RoomWidth
```

### West

```text
X = EffectiveTileSize * -0.5
Y = 0
Rotation Z = 90
LengthTiles = RoomDepth
```

### East

```text
X = (RoomWidth * EffectiveTileSize) + (EffectiveTileSize * -0.5)
Y = 0
Rotation Z = 90
LengthTiles = RoomDepth
```

🛑 No intercambiar East y West sin una prueba aislada.

## Aperturas

Centro del hueco:

```text
OpeningStartIndex = (LengthTiles - OpeningWidthTiles) / 2
```

Dimensiones:

```text
South/North → RoomWidth
East/West → RoomDepth
```

## UpdateConnectionMarkers

Responsabilidad:

- colocar Arrows en el centro de paredes;
- mostrar/ocultar según aperturas;
- limpiar arrays de salidas;
- añadir salidas activas.

Aunque los nombres internos vienen del sistema original, el generador debe usar la interfaz y no acceder directamente a esos Arrows.

## Pruebas confirmadas

```text
✅ 5x5 centrada
✅ Habitaciones rectangulares centradas
✅ RoomBounds se adapta a tamaño
✅ Altura cubierta por bounds
✅ Pasillos correctos en cuatro lados
✅ DoorWorldLocation validado
```

Casos de regresión que deben mantenerse:

```text
5x5
8x5
5x10
10x10
alturas diferentes
```

## Errores históricos relacionados

| Error | Causa | Solución |
|---|---|---|
| Actor origin en una esquina | Suelo crecía desde 0 | `RoomContentRoot` y offsets |
| Pasillo atravesaba sala | Door mapping incorrecto | N/S invertidos, E/W nominales |
| Huecos en paredes equivocadas | Adaptador incorrecto | Mapping N→S, E→W, S→N, W→E |
| Bounds Z demasiado pequeño | `0.5` aplicado dos veces | una mitad + PaddingZ |
| Bounds apoyado en suelo | Relative Z incorrecto | `RoomSizeZ * 0.5` |
| East/West físicos intercambiados | posición de Child Actors | fórmulas actuales protegidas |

## Política de estabilidad

🛑 No tocar sin bug demostrado:

```text
CenterRoomContentOnActorOrigin
UpdateRoomBounds
GetRoomBoundsData
mappings de aperturas
mapping de DoorWorldLocation
fórmulas físicas de paredes
orden de GenerateRoom
```
