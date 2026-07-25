# 09 — Colocación padre-hija por DoorPoints y RoomBounds

**Última actualización:** 2026-07-25

## Problema resuelto

El sistema antiguo colocaba habitaciones mediante:

```text
WorldX = GridX * CellSize
WorldY = GridY * CellSize
```

Eso no es seguro cuando Start, prebuilt y procedurales tienen tamaños distintos.

La solución actual coloca cada hija desde la puerta real del padre que la creó.

## Datos de relación

`DungeonCellLinks` guarda:

```text
ParentCellIndex
DirectionFromParent
bHasParent
```

Como las hijas se añaden después:

```text
ParentCellIndex < ChildIndex
```

Por tanto se pueden colocar en orden de índice.

## Flujo actual

```text
SpawnStartRoom
→ For Loop with Break
   First Index = 1
   Last Index = DungeonCells.Length - 1
   Loop Body → PlaceChildRoomFromParent(Index)
   Room Placed = False → Break
```

## PlaceChildRoomFromParent

Firma:

```text
Input : Child Index : Integer
Output: Room Placed : Boolean
```

Flujo:

```text
validar DungeonCells[Child Index]
→ validar DungeonCellLinks[Child Index]
→ Link = DungeonCellLinks[Child Index]
→ ParentRoom = SpawnedRooms[Link.ParentCellIndex]
→ ParentDirection = Link.DirectionFromParent
→ ChildEntryDirection = GetOppositeDirection(ParentDirection)
→ elegir clase
→ SpawnActor una sola vez
→ InitRoomFromCell una sola vez
→ obtener DoorPoints
→ calcular posición candidata
→ mover misma referencia
→ comprobar overlap global
→ reintentar o aceptar
```

## Selección de clase

```text
Normal → procedural común
Key    → BP_Room_PreBuilt_Base_Child_Key
Boss   → BP_Room_PreBuilt_Base_Child_Boss
Start  → inválido como hija
```

## Alineación puerta con puerta

```text
ParentDoorLocation =
ParentRoom.GetDoorWorldLocation(ParentDirection)
```

```text
ChildDoorLocation =
ChildRoom.GetDoorWorldLocation(ChildEntryDirection)
```

```text
DesiredChildDoorLocation =
ParentDoorLocation
+ DirectionVector(ParentDirection) * CorridorLength
```

```text
MoveDelta = DesiredChildDoorLocation - ChildDoorLocation
```

```text
NewChildActorLocation =
CurrentChildActorLocation + MoveDelta
```

Después:

```text
SetActorLocation(NewChildActorLocation)
```

En cada reintento se vuelve a consultar `ChildDoorLocation` desde la posición actual. No usar una puerta obsoleta del primer intento.

## DirectionVector

```text
North = (0, +1, 0)
East  = (+1, 0, 0)
South = (0, -1, 0)
West  = (-1, 0, 0)
```

No mezclar con nombres internos de Arrow Components. `GetDoorWorldLocation` actúa como adaptador.

## Variables confirmadas

```text
Parent Room Actor      : Actor Object Reference
Child Room Actor       : Actor Object Reference
Child Entry Direction  : E_DungeonDirection
Parent Direction       : E_DungeonDirection
Parent Door Location   : Vector
Child Door Location    : Vector
Corridor Length        : Float
Placement Attempt      : Integer
Placement Retry Step   : Float
Max Placement Attempts : Integer
bPlacement Succeeded   : Boolean
Bounds Overlap         : Boolean
```

Valores de prueba:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

## Reintentos

```text
For Loop with Break
First Index = 0
Last Index = Max Placement Attempts - 1
```

Por intento:

```text
Placement Attempt = Index
→ Get Door World Location de la hija actual
→ Set Child Door Location
→ calcular nueva posición
→ SetActorLocation
→ DoesRoomOverlapPlacedRooms
```

Si hay overlap:

```text
Corridor Length += Placement Retry Step
→ siguiente intento
```

Si no hay overlap:

```text
bPlacementSucceeded = true
→ Break
```

Si se agotan intentos:

```text
Print diagnóstico
→ Destroy Actor
→ Room Placed = false
```

Reglas:

```text
No volver a SpawnActor.
No repetir InitRoomFromCell.
No regenerar HISM.
Mover la misma referencia.
```

## DoesRoomOverlapPlacedRooms

Firma:

```text
Input : Candidate Room Actor
Output: Overlaps Placed Rooms
```

Flujo:

```text
obtener bounds de candidata
→ Found Overlap = false
→ For Each Loop with Break sobre SpawnedRooms
→ ignorar inválidos
→ ignorar candidata
→ obtener bounds de colocada
→ comparar AABB
→ primer overlap: true y Break
→ Completed: devolver Found Overlap
```

## AABB confirmada

```text
OverlapX = Abs(CandidateCenter.X - PlacedCenter.X)
           <= CandidateExtent.X + PlacedExtent.X

OverlapY = Abs(CandidateCenter.Y - PlacedCenter.Y)
           <= CandidateExtent.Y + PlacedExtent.Y

OverlapZ = Abs(CandidateCenter.Z - PlacedCenter.Z)
           <= CandidateExtent.Z + PlacedExtent.Z

Overlap = X AND Y AND Z
```

`<=` considera contacto exacto como overlap.

## Correspondencia de índices

Al aceptar:

```text
SpawnedRooms.Add(ChildRoom)
```

Debe conservar:

```text
Add ReturnValue == ChildIndex
```

Si una hija falla, el loop general se detiene. No se continúa silenciosamente.

## Resultado confirmado

```text
Max Rooms = 10
1 Start + 10 Normal + 1 Key + 1 Boss = 13
```

## Seeds conocidas

```text
Seed 12345 → South
Seed 12346 → East
```

## Pasillos después del placement

`SpawnCorridorsFromConnections` sigue desconectada temporalmente.

Cuando se reactive, deberá conectar DoorPoints world ya colocados y respetar la distancia física elegida por placement.

## Puertas prebuilt — próxima fase

Una prebuilt no puede inventar puertas.

Patrones:

```text
Dead End
Straight
Corner/L
T
Cross
```

El generador deberá comparar conexiones de celda contra puertas físicas declaradas y rotaciones 0/90/180/270 antes del spawn.

Las procedurales seguirán adaptándose a 1–4 conexiones y cerrando paredes no usadas.
