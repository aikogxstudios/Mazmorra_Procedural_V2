# 09 — Colocación padre-hija por DoorPoints y RoomBounds

## Problema actual

`SpawnRoomsFromCells` coloca salas con:

```text
WorldX = GridX * CellSize
WorldY = GridY * CellSize
```

Esto funciona cuando todas las salas tienen un tamaño parecido, por ejemplo 5x5.

Cuando `RoomWidth` o `RoomDepth` aumentan:

```text
la posición sigue usando CellSize fijo
pero la geometría ocupa más espacio
→ las salas pueden solaparse
```

Aumentar un único `CellSize` no resuelve bien el diseño porque:

- Start puede ser enorme;
- otras salas pueden ser pequeñas;
- se desperdicia espacio;
- los pasillos pierden variedad;
- prebuilt y procedurales pueden tener dimensiones distintas.

## Solución aprobada

Cada celda hija se colocará desde la puerta de la habitación padre que la creó.

`DungeonCellLinks` guarda:

```text
ParentCellIndex
DirectionFromParent
bHasParent
```

Como las hijas se añaden después del padre:

```text
ParentCellIndex < ChildIndex
```

se puede spawnear en orden de índice.

## Flujo objetivo completo

```text
Spawn Start
→ para ChildIndex = 1 hasta DungeonCells.LastIndex
   → Link = DungeonCellLinks[ChildIndex]
   → ParentRoom = SpawnedRooms[Link.ParentCellIndex]
   → ParentDirection = Link.DirectionFromParent
   → ChildEntryDirection = GetOppositeDirection(ParentDirection)
   → elegir clase de la hija
   → SpawnActor una sola vez
   → InitRoomFromCell
   → obtener puertas
   → alinear
   → comprobar bounds
   → si choca, mover la misma hija más lejos
   → aceptar
   → Add SpawnedRooms en el índice correcto
```

## Primera fase: SpawnStartRoom

Objetivo inmediato:

```text
Validar DungeonCells[0]
→ Spawn StartDebugRoomClass
→ colocar en origen
→ InitRoomFromCell(DungeonCells[0])
→ Add SpawnedRooms
→ confirmar SpawnedRooms[0]
```

No procesar hijas todavía.

Antes de construir esta función, revisar capturas completas de `SpawnRoomsFromCells` para reutilizar:

- selección de clase;
- `SpawnActor`;
- `InitRoomFromCell`;
- política de colisión;
- forma actual de añadir a `SpawnedRooms`.

## Segunda fase: una sola hija

Después de validar Start:

```text
procesar solo ChildIndex = 1
```

No lanzar todavía un loop sobre toda la mazmorra.

## Datos de una hija

```text
Link = DungeonCellLinks[ChildIndex]
ParentCellIndex = Link.ParentCellIndex
ParentDirection = Link.DirectionFromParent
ChildEntryDirection = GetOppositeDirection(ParentDirection)
```

Validaciones previas:

```text
DungeonCells.IsValidIndex(ChildIndex)
DungeonCellLinks.IsValidIndex(ChildIndex)
Link.bHasParent == true
SpawnedRooms.IsValidIndex(ParentCellIndex)
ParentCellIndex < ChildIndex
```

## Alineación puerta con puerta

### ParentDoorLocation

```text
ParentRoom.GetDoorWorldLocation(ParentDirection)
```

### ChildDoorLocation

Después de spawnear e inicializar la hija:

```text
ChildRoom.GetDoorWorldLocation(ChildEntryDirection)
```

### DesiredChildDoorLocation

```text
DesiredChildDoorLocation =
ParentDoorLocation
+
DirectionVector(ParentDirection) * CorridorLength
```

### MoveDelta

```text
MoveDelta = DesiredChildDoorLocation - ChildDoorLocation
```

### Nueva posición

```text
NewChildActorLocation =
CurrentChildActorLocation + MoveDelta
```

Después:

```text
SetActorLocation(NewChildActorLocation)
```

## DirectionVector

Convención lógica:

```text
North = (0, +1, 0)
East  = (+1, 0, 0)
South = (0, -1, 0)
West  = (-1, 0, 0)
```

No mezclar esta convención con los nombres internos de Arrow Components de `BP_RoomMaster_Dungeon`.

`GetDoorWorldLocation` ya actúa como adaptador.

## RoomBounds

Después de inicializar y mover la hija:

```text
ChildRoom.GetRoomBoundsData
→ BoundsCenter
→ BoundsExtent
```

Debe compararse contra las salas ya aceptadas.

Las salas aceptadas son las que ya existen en `SpawnedRooms` antes de añadir la nueva hija.

## Comprobación AABB conceptual

Para dos cajas alineadas a ejes:

```text
OverlapX = Abs(CenterA.X - CenterB.X) <= ExtentA.X + ExtentB.X + Padding
OverlapY = Abs(CenterA.Y - CenterB.Y) <= ExtentA.Y + ExtentB.Y + Padding
OverlapZ = Abs(CenterA.Z - CenterB.Z) <= ExtentA.Z + ExtentB.Z + PaddingZ

Overlap = OverlapX AND OverlapY AND OverlapZ
```

El montaje exacto en Blueprint se decidirá cuando se cree `DoesRoomOverlapPlacedRooms`.

No asumir todavía si se usará AABB manual, `BoxOverlapActors` u otra solución; `GetRoomBoundsData` es la fuente oficial de volumen.

## Reintentos de posición

Si no hay solapamiento:

```text
Aceptar posición.
```

Si hay solapamiento:

```text
CorridorLength += PlacementRetryStep
→ recalcular DesiredChildDoorLocation
→ recalcular MoveDelta
→ mover la misma habitación
→ volver a comprobar
```

Regla crítica:

```text
No volver a llamar GenerateRoom.
No destruir y spawnear otra vez en cada intento.
```

## Variables futuras recomendadas

```text
MinCorridorLength
MaxInitialCorridorLength
PlacementRetryStep
MaxPlacementAttempts
RoomPlacementPadding
```

Estado:

```text
🟡 conceptos aprobados
⏳ nombres exactos y valores pendientes
```

No crear todas hasta necesitarlas.

## Longitud inicial del pasillo

Visión V1:

```text
pasillo corto
pasillo medio
pasillo largo
```

La variedad física puede salir de:

```text
longitud inicial aleatoria
+
alargamiento obligatorio por colisión
```

La aleatoriedad debe usar `RandomStream` cuando se formalice para mantener reproducibilidad.

## Fallo tras MaxPlacementAttempts

Opciones aprobadas en este orden:

1. ampliar más el pasillo;
2. usar longitud máxima de emergencia;
3. reintentar la generación completa con contador/seed;
4. nunca entrar en un ciclo infinito;
5. no destruir y reconstruir HISM repetidamente.

El comportamiento exacto aún no está implementado.

## Funciones futuras previstas

```text
SpawnStartRoom
SpawnRemainingRoomsFromParents
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
GetDirectionVector
```

No crearlas todas de una vez.

Orden:

```text
SpawnStartRoom
→ una hija
→ alineación
→ bounds
→ reintento
→ loop completo
```

## Correspondencia de índices durante spawn

Al aceptar una hija:

```text
SpawnedRooms.Add(ChildRoom)
```

Debe ocurrir de forma que:

```text
Add ReturnValue == ChildIndex
```

No usar ese ReturnValue como padre.

Si un spawn falla, no se puede continuar silenciosamente porque rompería:

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

## Pasillos después de colocar salas

`SpawnCorridorsFromConnections` puede mantenerse porque obtiene puertas en world space.

Al separar más las salas:

```text
Start DoorPoint
End DoorPoint
```

quedarán más lejos y `BP_Corridor_Debug` podrá extenderse.

No tocar esta función hasta que la colocación de salas esté validada.

## Boss Door después de colocar salas

`SpawnBossRoomDoors` también puede mantenerse porque consulta el DoorPoint world de la Boss Room ya colocada.

## Conexión pared con pared

```text
🔮 No forma parte de V1
```

Un pasillo muy corto puede imitar visualmente esa conexión sin complicar:

- paredes;
- columnas;
- marcos;
- pivotes;
- colisiones.

## Pruebas por fase

### SpawnStartRoom

```text
Solo una sala spawneada.
Start en posición correcta.
SpawnedRooms.Num == 1.
SpawnedRooms[0] válido.
InitRoomFromCell ejecutado.
Una única apertura lógica.
```

### Primera hija

```text
ParentCellIndex == 0 o padre válido.
Puerta del padre correcta.
Entrada de hija correcta.
Pasillo corto visible.
No hay giro inesperado.
```

### Bounds

```text
5x5
8x5
5x10
10x10
alturas distintas
```

### Reintento

```text
Forzar choque.
Aumentar CorridorLength.
Mover la misma referencia.
No cambiar RoomWidth/RoomDepth.
No duplicar HISM.
```

### Layout completo

```text
DungeonCells.Num == SpawnedRooms.Num.
Todos los padres existen antes de sus hijas.
No hay salas solapadas.
Pasillos conectan puertas.
Boss Door mantiene posición.
```
