# Changelog — Mazmorra Procedural V2

Todos los cambios técnicos importantes deben reflejarse también en `docs/00_ESTADO_ACTUAL.md`.

## 2026-07-23 — Separación, prebuilt base y RoomBounds

### Añadido

- `Parent Direction : E_DungeonDirection` como variable local de `SpawnFirstChildRoom`.
- `Parent Bounds Center : Vector`.
- `Parent Bounds Extent : Vector`.
- `Child Bounds Center : Vector`.
- `Child Bounds Extent : Vector`.
- `bBoundsOverlap : Boolean`.
- `BP_Room_PreBuilt_Base` como prototipo controlador de habitaciones preconstruidas.
- Documento `docs/28_ROOM_BOUNDS_FIRST_CHILD.md`.
- Documento `docs/29_BP_ROOM_PREBUILT_BASE.md`.
- Cierre `sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md`.

### Modificado

- `GetDirectionVector` confirmado como función Pure.
- `SpawnFirstChildRoom` dejó de alinear las puertas a distancia cero durante la prueba actual y usa una separación temporal.
- `Start Room Class` se probó con `BP_Room_PreBuilt_Base`.
- `Get Room Bounds Data` de la base prebuilt se conectó a `RoomBounds → Get Component Bounds`.
- El roadmap avanzó a Fase F: reintentos controlados.

### Implementación confirmada

```text
DungeonCellLinks[1].DirectionFromParent
→ Parent Direction
→ GetDirectionVector
→ Vector * Float (1000 temporal)
```

```text
DesiredChildDoor =
ParentDoorLocation
+ DirectionVector * 1000
```

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma hija
```

### Separación validada

```text
Seed 12345 → North → Distance = 1000
Seed 12346 → East  → Distance = 1000
SpawnedRooms.Num = 2
```

### BP_Room_PreBuilt_Base

Contrato confirmado:

```text
BPI_DungeonRoomV2
Init Room from Cell
Get Door World Location
Get Room Bounds Data
DoorPoint_North/East/South/West
RoomBounds
```

Bounds de Start prebuilt confirmados:

```text
X 980, Y 980, Z 400
```

Bounds de primera hija procedural:

```text
X 995, Y 995, Z 420
```

### Detección AABB

```text
OverlapX = Abs(ParentCenterX - ChildCenterX)
           <= ParentExtentX + ChildExtentX

OverlapY = Abs(ParentCenterY - ChildCenterY)
           <= ParentExtentY + ChildExtentY

OverlapZ = Abs(ParentCenterZ - ChildCenterZ)
           <= ParentExtentZ + ChildExtentZ

bBoundsOverlap = OverlapX AND OverlapY AND OverlapZ
```

Pruebas:

```text
1000  → False
-500  → True
```

### Error corregido

Incorrecto:

```text
Abs(ParentCenter) - ChildCenter
```

Correcto:

```text
Abs(ParentCenter - ChildCenter)
```

### Decisión de arquitectura

```text
Procedural común
→ BP_RoomMaster_Dungeon
→ HISM
→ RoomBounds dinámico
```

```text
Especial prebuilt
→ BP_Room_PreBuilt_Base / Blueprint hijo
→ RoomBounds y DoorPoints manuales
→ Level Instance o Packed Level Blueprint visual después de validación
```

Las habitaciones debug se mantienen temporalmente hasta sustituir referencias.

### Pendiente inmediato

```text
Fase F — reintentos controlados para ChildIndex 1
```

Variables previstas, todavía no creadas/confirmadas:

```text
Corridor Length : Float
Placement Retry Step : Float
Placement Attempt : Integer
Max Placement Attempts : Integer
```

---

## 2026-07-21 — Primera habitación hija alineada

### Añadido

- `SpawnFirstChildRoom`.
- Validaciones de `DungeonCells[1]`, `DungeonCellLinks[1]`, padre e hija.
- Variables locales de actores, puertas y dirección de entrada.
- `GetDirectionVector`.
- Documento `docs/27_SPAWN_FIRST_CHILD_ROOM.md`.

### Validado

```text
SpawnedRooms.Num == 2
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
InitRoomFromCell una sola vez
La hija se mueve sin regenerar HISM
Distancia final base = 0.0
```

### Error corregido

La medición `2950` comparaba la puerta de la hija contra la ubicación/centro del actor. La medición correcta usa `ParentDoorLocation`.

---

## 2026-07-21 — SpawnStartRoom completada

### Añadido

- `SpawnStartRoom`.
- Validación de `DungeonCells[0]` y `RoomType == Start`.
- Validación de `SpawnActor Return Value`.
- Documento `docs/26_SPAWN_START_ROOM.md`.

### Validado

```text
Solo aparece Start
SpawnedRooms.Num == 1
SpawnedRooms[0] válido
InitRoomFromCell(DungeonCells[0])
Una única apertura
```

### Corregido

```text
Scale Z = 0 → Scale Z = 1
```

---

## 2026-07-20 — Memoria permanente

- Repositorio establecido como memoria técnica oficial.
- Cells/Links validados hasta 150 salas.
- Start con una salida confirmada.
- Prints temporales conservados y desactivados.
- Política de actualización permanente añadida.

---

## 2026-07-19 — Base técnica inicial

- Documentación estructurada creada.
- `ST_DungeonCellLink` y `DungeonCellLinks` consolidados.
- Arquitectura padre-hija aprobada.
- `BP_RoomMaster` identificado como original sin tocar.
- `BP_RoomMaster_Dungeon` identificado como variante integrada.
- `RoomContentRoot`, `RoomBounds`, aperturas y DoorPoints documentados.
- Pasillos, Boss Door, llave e inventario temporal registrados.

## Formato para futuras entradas

```markdown
## YYYY-MM-DD — Nombre de fase

### Añadido
- ...

### Modificado
- ...

### Corregido
- ...

### Validado
- ...

### Pendiente
- ...

### Siguiente paso
- ...
```
