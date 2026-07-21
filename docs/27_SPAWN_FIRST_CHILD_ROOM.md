# 27 — SpawnFirstChildRoom y primera alineación padre-hija

## Estado

```text
✅ Función creada en BP_DungeonGenerator_V2.
✅ Compila y se ejecuta desde GenerateDungeon después de SpawnStartRoom.
✅ Genera únicamente ChildIndex = 1.
✅ SpawnedRooms.Num == 2 confirmado en pantalla.
✅ SpawnedRooms[0] continúa siendo Start.
✅ SpawnedRooms[1] es la primera habitación hija.
✅ La hija se inicializa una sola vez y después se mueve.
✅ Error final entre DoorPoints = 0.0.
✅ GetDirectionVector creado con los cuatro vectores correctos.
🟡 Pure de GetDirectionVector pendiente de confirmar: la captura actual todavía muestra pines de ejecución.
```

Fase realizada el **2026-07-21**.

## Objetivo

Probar la arquitectura física padre-hija con una sola habitación antes de recorrer todo el layout.

La prueba usa exclusivamente:

```text
ChildIndex = 1
```

La Start ya existe como:

```text
SpawnedRooms[0]
```

## Función

```text
Nombre: SpawnFirstChildRoom
Blueprint: BP_DungeonGenerator_V2
Inputs: ninguno
Outputs: ninguno
```

## Validaciones de entrada

La función comprueba en orden:

```text
DungeonCells.IsValidIndex(1)
DungeonCellLinks.IsValidIndex(1)
DungeonCellLinks[1].bHasParent == true
SpawnedRooms.IsValidIndex(ParentCellIndex)
Actor padre válido
```

Mensajes de diagnóstico usados:

```text
Dungeon V2 | ERROR: DungeonCells[1] is invalid
Dungeon V2 | ERROR: DungeonCellLinks[1] is invalid
Dungeon V2 | ERROR: Child 1 has no parent
Dungeon V2 | ERROR: Parent room index is invalid
Dungeon V2 | ERROR: Parent room actor is invalid
Dungeon V2 | ERROR: Failed to spawn Child Room 1
Dungeon V2 | ERROR: Child 1 cannot be Start
```

## Datos del link

```text
DungeonCellLinks[1]
→ Break ST_DungeonCellLink
```

Se usan:

```text
ParentCellIndex
DirectionFromParent
bHasParent
```

`ParentCellIndex` es un Integer, no un array.

`Parent Room Actor` es una variable local de tipo:

```text
Actor Object Reference
Array = false
```

Su valor correcto procede de:

```text
SpawnedRooms[ParentCellIndex]
```

No debe asignarse a sí misma.

## Variables locales confirmadas

```text
Parent Room Actor : Actor Object Reference
Child Room Actor : Actor Object Reference
Child Entry Direction : E_DungeonDirection
Parent Door Location : Vector
Child Door Location : Vector
```

Ninguna de estas variables es un array.

## Dirección de entrada de la hija

`GetOppositeDirection` se cambió a función Pure durante esta fase.

```text
DirectionFromParent
→ GetOppositeDirection
→ ChildEntryDirection
```

Ejemplos:

```text
Parent East  → Child West
Parent North → Child South
Parent South → Child North
Parent West  → Child East
```

## Selección de clase

Se lee:

```text
DungeonCells[1].RoomType
```

Y mediante `Switch on E_DungeonRoomType` se selecciona la clase correspondiente:

```text
Normal → clase Normal / procedural
Key    → Key Debug Room Class
Boss   → Boss Debug Room Class
Start  → error y Return
```

La celda 1 no se asume siempre Normal porque `ChooseKeyAndBossCells` puede convertirla en Key o Boss.

## Spawn e inicialización

La hija se crea temporalmente en la ubicación del generador:

```text
GetActorLocation(self)
→ Make Transform
Rotation = 0,0,0
Scale = 1,1,1
```

Configuración usada:

```text
SpawnActor
Collision Handling Override = Always Spawn, Ignore Collisions
```

Esto permite crearla aunque nazca temporalmente encima de la Start, porque será movida inmediatamente.

Después:

```text
SpawnActor Return Value
→ Is Valid
→ Set Child Room Actor
→ InitRoomFromCell(DungeonCells[1])
```

La habitación se genera e inicializa una sola vez. Los HISM no se regeneran durante el movimiento.

## Lectura de DoorPoints

### Puerta del padre

```text
Target = Parent Room Actor
Direction = DirectionFromParent
→ GetDoorWorldLocation
→ Parent Door Location
```

### Puerta de la hija

```text
Target = Child Room Actor
Direction = ChildEntryDirection
→ GetDoorWorldLocation
→ Child Door Location
```

Ambas llamadas usan `BPI_DungeonRoomV2`.

## Fórmula de alineación exacta

Primera prueba sin separación de pasillo:

```text
MoveDelta = ParentDoorLocation - ChildDoorLocation
```

```text
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
```

```text
SetActorLocation(
  Target = Child Room Actor,
  New Location = NewChildLocation,
  Sweep = false,
  Teleport = false
)
```

Después del movimiento:

```text
SpawnedRooms.Add(Child Room Actor)
```

Como la Start ya ocupaba el índice 0, la hija queda en el índice 1.

## Pruebas confirmadas

```text
✅ Solo aparecen dos habitaciones.
✅ SpawnedRooms.Num == 2.
✅ La hija queda junto a la Start.
✅ Las aperturas quedan enfrentadas.
✅ La misma referencia de actor es movida.
✅ No se vuelve a ejecutar InitRoomFromCell después del movimiento.
```

## Medición del error

Después de mover la hija se vuelve a consultar su DoorPoint y se calcula:

```text
Distance(
  ChildDoorWorldLocationAfterMove,
  ParentDoorLocation
)
```

Resultado final confirmado:

```text
0.0
```

Esto valida matemáticamente que los DoorPoints coinciden exactamente.

### Error de medición corregido

La primera medición mostró:

```text
2950
```

No era un error de colocación. Se estaba comparando accidentalmente:

```text
ChildDoorWorldLocation
contra
NewChildActorLocation / centro del actor
```

Ese valor representaba aproximadamente la distancia entre el centro de la habitación grande y su puerta.

La conexión correcta es:

```text
V1 = ChildDoorWorldLocation después de mover
V2 = ParentDoorLocation
```

## GetDirectionVector

Función creada:

```text
Nombre: GetDirectionVector
Input: Direction : E_DungeonDirection
Output: Direction Vector : Vector
```

Implementación mediante `Select`:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

El mapping está confirmado por captura.

La captura actual todavía muestra cable blanco de ejecución entre entrada y Return, por lo que la propiedad `Pure` de esta función queda pendiente de confirmar en la siguiente sesión. El resultado matemático es correcto con o sin Pure; Pure solo simplifica su uso visual.

## Flujo temporal de GenerateDungeon

Durante esta prueba:

```text
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ SpawnFirstChildRoom
```

El spawn antiguo, los pasillos, Boss Door y debug físico permanecen desconectados temporalmente, no eliminados.

## Partes protegidas

```text
🛑 No invertir ParentDoorLocation - ChildDoorLocation.
🛑 No usar ChildEntryDirection en la puerta del padre.
🛑 No usar DirectionFromParent en la puerta de la hija.
🛑 No regenerar la habitación durante el movimiento.
🛑 No añadir la hija antes de validar el SpawnActor.
🛑 No reordenar SpawnedRooms.
```

## Siguiente paso exacto

Añadir separación para el pasillo usando el vector de dirección:

```text
DesiredChildDoor =
ParentDoorLocation
+ GetDirectionVector(DirectionFromParent) * CorridorLength
```

Después:

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
```

Pruebas siguientes:

```text
DoorDistance == CorridorLength
SpawnedRooms.Num == 2
probar al menos dos direcciones
```

Todavía no implementar el loop de todas las hijas ni el sistema completo de bounds/reintentos.
