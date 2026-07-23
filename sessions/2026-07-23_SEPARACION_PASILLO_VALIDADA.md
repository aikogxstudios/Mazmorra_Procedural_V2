# Sesión 2026-07-23 — Separación inicial del pasillo validada

## Resultado

La fase D.1 queda validada para la primera habitación hija (`ChildIndex = 1`).

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
```

La separación temporal utilizada fue:

```text
CorridorLength = 1000.0
```

La fórmula implementada quedó:

```text
DirectionVector = GetDirectionVector(ParentDirection)
CorridorOffset = DirectionVector * 1000.0
DesiredChildDoor = ParentDoorLocation + CorridorOffset
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation(ChildRoomActor, NewChildLocation)
```

## Variables y funciones confirmadas

```text
Parent Direction : E_DungeonDirection (variable local)
Parent Door Location : Vector (variable local)
Child Door Location : Vector (variable local)
GetDirectionVector : Pure Function
```

`Parent Direction` recibe el valor del pin `Direction From Parent` de `Break ST_DungeonCellLink`.

## Pruebas superadas

### Seed 12345

```text
Use Random Seed = false
Dungeon Seed = 12345
Dirección = North
Distance(ParentDoor, ChildDoorAfterMove) = 1000
SpawnedRooms Length = 2
```

### Seed 12346

```text
Use Random Seed = false
Dungeon Seed = 12346
Dirección = East
Distance(ParentDoor, ChildDoorAfterMove) = 1000
SpawnedRooms Length = 2
```

## Conclusión

```text
✅ GetDirectionVector es Pure.
✅ El mapping North/East funciona.
✅ La hija se mueve en Parent Direction.
✅ La misma referencia de hija se conserva.
✅ InitRoomFromCell no se repite.
✅ La distancia final coincide con CorridorLength.
✅ SpawnedRooms mantiene los índices 0 y 1.
```

## Próximo paso

Fase E — `RoomBounds` para la primera hija:

```text
obtener bounds de la hija después de mover
→ obtener bounds de las salas aceptadas
→ comprobar intersección
```

Antes de cablear esta fase se deben revisar capturas actuales de:

```text
BP_RoomMaster_Dungeon.Get Room Bounds Data
RoomBounds y sus valores actuales
zona final de SpawnFirstChildRoom
```

Todavía no implementar reintentos, todas las hijas ni pasillos visuales finales.
