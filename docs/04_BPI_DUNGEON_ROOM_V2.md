# 04 — BPI_DungeonRoomV2

`BPI_DungeonRoomV2` es el contrato común entre `BP_DungeonGenerator_V2` y cualquier actor de habitación compatible con la Mazmorra V2.

## Objetivo

El generador no debe depender del interior concreto de una sala. Solo necesita poder:

```text
Inicializarla desde una celda lógica.
Obtener el punto de una puerta.
Obtener sus bounds.
```

## Funciones confirmadas

```text
InitRoomFromCell
GetDoorWorldLocation
GetRoomBoundsData
```

## InitRoomFromCell

### Firma

```text
Input:
CellData : ST_DungeonCell
```

### Responsabilidad

Configurar la habitación física según el grafo lógico:

- tipo de sala si la implementación lo necesita;
- conexiones North/East/South/West;
- aperturas visibles;
- seed o datos propios;
- generación procedural interna.

### En BP_RoomMaster_Dungeon

Flujo confirmado:

```text
Set CurrentCellData
→ Break ST_DungeonCell
→ Set bUseDungeonConnections = true
→ Set bRandomizeExits = false
→ mapear las cuatro aperturas
→ GenerateRoom
```

Mapping protegido:

```text
Cell.bNorth → bSouthOpening
Cell.bEast  → bWestOpening
Cell.bSouth → bNorthOpening
Cell.bWest  → bEastOpening
```

Este mapping parece invertido por los nombres heredados del RoomMaster, pero fue probado visualmente.

## GetDoorWorldLocation

### Firma

```text
Input:
Direction : E_DungeonDirection

Output:
World Location : Vector
```

### Responsabilidad

Devolver la posición física exacta de la puerta correspondiente a una dirección lógica de mazmorra.

Usos actuales:

- spawn de pasillos;
- líneas debug entre puertas;
- spawn de Boss Door;
- futura alineación padre-hija.

### Mapping confirmado en BP_RoomMaster_Dungeon

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

No unificar este mapping con el de `InitRoomFromCell`. Son adaptadores distintos.

## GetRoomBoundsData

### Firma

```text
Outputs:
BoundsCenter : Vector
BoundsExtent : Vector
```

### Responsabilidad

Entregar un volumen oficial y controlado de la sala para colocación y solapamientos.

### Implementación confirmada en BP_RoomMaster_Dungeon

```text
RoomBounds
→ Get Component Bounds
   Origin → BoundsCenter
   Box Extent → BoundsExtent
```

No usar `GetActorBounds` mientras `RoomBounds` siga siendo la referencia oficial, porque `GetActorBounds` puede incluir:

- flechas;
- componentes debug;
- elementos auxiliares;
- componentes que no representan el volumen real de la sala.

## Requisitos para una sala compatible

Cualquier sala procedural o preconstruida que entre en `SpawnedRooms` debe implementar correctamente:

```text
InitRoomFromCell
GetDoorWorldLocation
GetRoomBoundsData
```

Además:

- sus DoorPoints deben representar los centros reales de conexión;
- sus bounds deben cubrir el volumen físico de la sala;
- la inicialización debe ocurrir antes de consultar DoorPoints o bounds;
- no debe regenerarse cada vez que se mueve.

## Uso futuro en colocación padre-hija

```text
ParentRoom.GetDoorWorldLocation(DirectionFromParent)
ChildRoom.GetDoorWorldLocation(GetOppositeDirection(DirectionFromParent))
ChildRoom.GetRoomBoundsData()
```

El generador no necesita conocer cómo están nombrados internamente los Arrow Components de cada clase.

## Regla de seguridad

Si una nueva sala no implementa la interfaz:

```text
No añadirla silenciosamente a SpawnedRooms.
```

Debe detectarse durante pruebas o validación antes de extender el sistema.
