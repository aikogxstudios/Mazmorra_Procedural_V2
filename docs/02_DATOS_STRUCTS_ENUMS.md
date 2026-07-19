# 02 — Datos, structs, enums e invariantes

## ST_DungeonCell

Struct lógico principal de cada habitación.

| Campo | Tipo | Estado | Uso |
|---|---|---:|---|
| `GridX` | Integer | ✅ | Coordenada lógica X |
| `GridY` | Integer | ✅ | Coordenada lógica Y |
| `RoomType` | `E_DungeonRoomType` | ✅ | Tipo de sala |
| `bNorth` | Boolean | ✅ | Conexión lógica North |
| `bEast` | Boolean | ✅ | Conexión lógica East |
| `bSouth` | Boolean | ✅ | Conexión lógica South |
| `bWest` | Boolean | ✅ | Conexión lógica West |
| `RoomSeed` | Integer | ✅ | Seed propia de la sala |
| `RoomID` | Integer | ✅ | Identificador lógico |

Regla crítica:

```text
Al cambiar RoomType,
conservar GridX, GridY,
las cuatro conexiones,
RoomSeed y RoomID.
```

No reconstruir una celda cambiando solo el tipo y perdiendo los demás datos.

## ST_DungeonCellLink

Guarda desde qué habitación nació cada celda.

| Campo | Tipo | Default | Estado | Uso |
|---|---|---:|---:|---|
| `ParentCellIndex` | Integer | `-1` | ✅ | Índice del padre |
| `DirectionFromParent` | `E_DungeonDirection` | North | ✅ | Dirección de salida usada en el padre |
| `bHasParent` | Boolean | false | ✅ | Validez del vínculo |

Ejemplo:

```text
DungeonCellLinks[7]
ParentCellIndex = 3
DirectionFromParent = East
bHasParent = true
```

Significa:

```text
La habitación 7 nació desde la salida East de la habitación 3.
```

La dirección guardada es desde el padre. No es la dirección de entrada de la hija.

### Link de Start

```text
DungeonCellLinks[0]
ParentCellIndex = -1
DirectionFromParent = North
bHasParent = false
```

`North` es relleno y no debe leerse cuando `bHasParent=false`.

### Link de una hija

Dentro de `TryAddRandomCell`:

```text
Random Cell Index → ParentCellIndex
Selected Direction → DirectionFromParent
Has Parent = true
```

No conectar:

```text
❌ Add DungeonCells.ReturnValue
❌ Random Direction Index
❌ Opposite Direction
```

## E_DungeonDirection

Valores:

```text
North
East
South
West
```

Convención lógica:

```text
North → GridY + 1
East  → GridX + 1
South → GridY - 1
West  → GridX - 1
```

Vectores físicos previstos:

```text
North = (0, +1, 0)
East  = (+1, 0, 0)
South = (0, -1, 0)
West  = (-1, 0, 0)
```

Opuestos:

```text
North ↔ South
East  ↔ West
```

## E_DungeonRoomType

Valores observados:

```text
Start
Normal
Combat
Treasure
Puzzle
Safe
Shop
Rest
Key
Boss Door
Boss
Portal
Secret
Event
Storage
Trade
Corridor
Stairs Up
Stairs Down
```

Implementados actualmente:

```text
✅ Start
✅ Normal
✅ Key
✅ Boss
```

Reservados:

```text
🔮 Combat
🔮 Treasure
🔮 Puzzle
🔮 Safe
🔮 Shop
🔮 Rest
🔮 Boss Door como RoomType
🔮 Portal
🔮 Secret
🔮 Event
🔮 Storage
🔮 Trade
🔮 Corridor
🔮 Stairs Up
🔮 Stairs Down
```

No confundir valores del enum con contenido funcional ya implementado.

## Invariantes de arrays

### Correspondencia principal

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

### Regla de creación

Por cada éxito de `TryAddRandomCell`:

```text
DungeonCells añade 1
DungeonCellLinks añade 1
```

Por cada fallo:

```text
DungeonCells añade 0
DungeonCellLinks añade 0
```

### Regla de orden

```text
ParentCellIndex < ChildIndex
```

Esto permite spawnear salas en orden de índice porque el padre ya existe cuando se procesa la hija.

### Reglas de seguridad

Nunca:

- ordenar los arrays relacionados;
- insertar elementos auxiliares en `SpawnedRooms`;
- borrar un elemento intermedio;
- añadir una sala física fuera del orden lógico;
- usar `RoomID` como sustituto automático de `ArrayIndex` sin comprobarlo;
- asumir que `DirectionFromParent` es la entrada de la hija.

## Variables locales confirmadas de TryAddRandomCell

| Variable | Tipo | Uso |
|---|---|---|
| `Random Cell Index` | Integer | Índice del padre elegido |
| `Selected Cell` | `ST_DungeonCell` | Copia del padre |
| `Random Direction Index` | Integer | Índice 0-3 previo al enum |
| `Selected Direction` | `E_DungeonDirection` | Dirección final desde el padre |
| `Neighbor X` | Integer | X de la hija |
| `Neighbor Y` | Integer | Y de la hija |
| `New Room Seed` | Integer | Seed de la hija |
| `Updated Selected Cell` | `ST_DungeonCell` | Padre con conexión activada |
| `New Base Cell` | `ST_DungeonCell` | Hija antes de conectar el lado opuesto |
| `Opposite Direction` | `E_DungeonDirection` | Dirección de vuelta hacia el padre |
| `New Connected Cell` | `ST_DungeonCell` | Hija terminada |

Equivalencias conceptuales que no existen como variables reales:

```text
BaseCellIndex = Random Cell Index
ChosenDirection = Selected Direction
```

No crear variables nuevas con esos nombres salvo que se decida expresamente un refactor.

## Variables locales confirmadas de ChooseKeyAndBossCells

| Variable | Tipo | Uso |
|---|---|---|
| `ConnectionCount` | Integer | Número de conexiones |
| `DistanceFromStart` | Integer | Distancia Manhattan al origen |
| `DistanceFromBoss` | Integer | Distancia Manhattan a Boss |
| `KeyScore` | Integer | `DistanceFromBoss * DistanceFromStart` |

## Futuras variables de colocación

Aprobadas conceptualmente, nombres/valores aún pendientes:

```text
MinCorridorLength
MaxInitialCorridorLength
PlacementRetryStep
MaxPlacementAttempts
RoomPlacementPadding
```

No crearlas todas de golpe. Se introducirán cuando cada fase las necesite.
