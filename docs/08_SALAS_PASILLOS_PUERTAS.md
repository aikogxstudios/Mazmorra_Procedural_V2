# 08 — Salas, pasillos, Boss Door, llave e inventario temporal

## Inventario de actores y sistemas

| Elemento | Tipo | Estado | Función |
|---|---|---:|---|
| `BP_DungeonGenerator_V2` | Blueprint Actor | ✅ | Generador principal |
| `BP_Room_Debug_Start` | Actor | ✅ | Start preconstruida/debug |
| `BP_Room_Debug_Normal` | Actor | ✅ histórico | Normal debug anterior |
| `BP_Room_Debug_Key` | Actor | ✅ temporal | Key debug con pickup temporal |
| `BP_Room_Debug_Boss` | Actor | ✅ | Boss debug/preconstruida |
| `BP_RoomMaster` | Actor | ✅ original | Sala procedural independiente |
| `BP_RoomMaster_Dungeon` | Actor | ✅ integrado | Sala procedural compatible con Dungeon V2 |
| `BP_Corridor_Debug` | Actor | ✅ | Pasillo entre DoorPoints |
| `BP_BossDoor_Lock` | Actor | ✅ | Puerta bloqueada del Boss |
| Pickup de llave | Actor | ✅ | Añade llave al mini inventario |
| Mini Inventory Component | Actor Component | ✅ experimental | Inventario temporal |
| UI mini inventario | Widget | ✅ experimental | Se abre con `O` |

# Salas debug

## Estructura histórica

Las salas debug incluyen elementos como:

```text
Floor
WallFill_North
WallFill_East
WallFill_South
WallFill_West
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
DoorMarker_* opcionales
```

Una conexión `true` oculta o desactiva el tapón de pared correspondiente.

## Start Debug

Características actuales:

```text
Color morado.
Preconstruida.
Puede tener cualquier tamaño.
Solo una salida lógica.
No puede ser Key ni Boss.
```

La regla de una salida está implementada en `TryAddRandomCell`, no dentro de la sala.

## Normal Debug

Sala normal histórica usada antes de integrar `BP_RoomMaster_Dungeon`.

Puede conservarse para pruebas comparativas.

## Key Debug

Estado actual:

```text
✅ La Key Room spawnea temporalmente el pickup.
✅ Usa un Delay aproximado de 10 segundos.
✅ Usa un Arrow o punto central.
✅ Spawnea la clase del pickup de llave.
```

Esta implementación sirve para validar el loop completo, no es el sistema final de contenido de sala.

Pendiente futuro:

- eliminar `Delay`;
- evitar duplicados de forma formal;
- integrar contenido de sala;
- elegir posición según sala procedural o prebuilt;
- separar lógica de contenido de la selección de layout.

## Boss Debug

Características:

```text
Color rojo.
Preconstruida.
Terminal.
No contiene una Boss Door fija.
```

La puerta la genera `BP_DungeonGenerator_V2`.

# Selección de Boss

La Boss se elige preferentemente como dead-end:

```text
RoomType != Start
ConnectionCount == 1
```

Entre candidatos, se elige la mayor distancia Manhattan desde el origen:

```text
Abs(GridX) + Abs(GridY)
```

Resultado esperado:

```text
Boss terminal con una entrada.
```

# Selección de Key

Condiciones:

```text
No Start.
No Boss.
DistanceFromStart >= 3.
```

Puntuación:

```text
DistanceFromBoss =
Abs(GridX - BossGridX)
+ Abs(GridY - BossGridY)

KeyScore = DistanceFromBoss * DistanceFromStart
```

La mejor puntuación se convierte en Key.

Objetivo de diseño:

```text
Obligar a explorar una rama antes de acceder al Boss.
```

# BP_Corridor_Debug

## Responsabilidad

Construir un pasillo entre dos posiciones world:

```text
Start DoorWorldLocation
End DoorWorldLocation
```

`SpawnCorridorsFromConnections` ya lo usa correctamente.

## Prevención de duplicados

El generador procesa solo:

```text
North
East
```

Porque cada conexión lógica existe en ambos sentidos.

Procesar también South/West duplicaría pasillos.

## Estado visual

```text
✅ Pasillos verdes correctos.
✅ Conexiones por los cuatro lados.
✅ DoorPoints reales validados.
```

## Futuro

La misma clase podrá extender pasillos cuando las salas estén más separadas:

```text
pasillo corto
pasillo medio
pasillo largo
```

No hace falta cambiar el layout lógico para alargar el corredor.

# Boss Door

## Clase

```text
BP_BossDoor_Lock
```

## Spawn

`BP_DungeonGenerator_V2` llama:

```text
SpawnBossRoomDoors
→ SpawnDoorAtRoomDirection
```

La puerta se coloca en el DoorPoint de la conexión de Boss.

## Rotaciones aprobadas

```text
North = 0
East = -90
South = 180
West = 90
```

## Reglas

```text
La Boss Room no debe incluir una puerta fija.
La Boss Door es un actor separado.
SpawnedDungeonDoors debe limpiarse al regenerar.
```

# Mini inventario

Estado funcional:

```text
Pickup añade Data Asset/item.
UI se abre con O.
Boss Door consulta si el item requerido existe.
Sin llave permanece cerrada.
Con llave se abre.
```

Este sistema es experimental.

No confundirlo con:

- inventario final de cartas;
- inventario final de consumibles;
- sistema de progresión definitivo.

# Loop funcional actual

```text
Generar mazmorra
→ seleccionar Key Room
→ esperar Delay temporal
→ spawnear pickup
→ recoger llave
→ añadir al mini inventario
→ llegar a Boss Door
→ validar item requerido
→ abrir puerta
```

# Visión V1 de salas

## Start

```text
Siempre prebuilt.
Una salida.
Cualquier tamaño.
```

## Normal

```text
Procedural o prebuilt.
Puede bifurcar.
```

## Key

```text
Procedural o prebuilt.
Contiene la llave de la run.
```

## Boss

```text
Siempre prebuilt.
Terminal.
Una entrada.
```

# RoomTypes futuras

Reservadas:

```text
Combat
Treasure
Puzzle
Safe
Shop
Rest
Secret
Event
Storage
Trade
Portal
Stairs Up
Stairs Down
```

Preferencia de diseño:

- opcionales terminales o claramente separadas;
- no romper progresión Start → Key → Boss;
- la sala decide contenido interno;
- el generador decide grafo, clase y posición.

# Puertas futuras

Datos reservados:

```text
DoorType
RequiredKeyID
DoorGroupID
bConsumesKey
bOpensWholeGroup
bOptional
```

No implementar durante la fase de colocación padre-hija.

# Pruebas de regresión

```text
Boss es dead-end cuando hay candidato.
Boss Door aparece en la única entrada.
Rotación correcta por dirección.
No quedan puertas antiguas al regenerar.
Key no es Start ni Boss.
Pickup aparece.
Pickup entra al mini inventario.
Puerta no abre sin llave.
Puerta abre con llave.
Pasillos no se duplican.
Pasillos conectan DoorPoints interiores.
```
