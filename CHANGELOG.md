# Changelog — Mazmorra Procedural V2

Todos los cambios técnicos importantes deben reflejarse también en `docs/00_ESTADO_ACTUAL.md`.

## 2026-07-26 — Regresión final de Fase H y primera limpieza

### Validado

```text
Seed 12345 → South → 13 habitaciones sin fallos
Seed 12346 → East  → 13 habitaciones sin fallos
```

En ambas pruebas:

```text
1 Start
10 Normal
1 Key
1 Boss
sin SPECIAL FAILED
sin Max Placement Attempts reached
sin solapamientos visibles
```

### Eliminado

- `SpawnFirstChildRoom` fue eliminado de `BP_DungeonGenerator_V2`.
- Antes de borrarlo se ejecutó `Find References` y solo apareció la definición de la propia función.
- Después de eliminarlo, el Blueprint compiló y la generación completa siguió funcionando.

### Confirmado

```text
GenerateDungeon
→ SpawnStartRoom
→ For Loop with Break
→ PlaceChildRoomFromParent(Index)
```

`PlaceChildRoomFromParent` es ahora la única función activa encargada del spawn, inicialización, colocación, reintentos y aceptación de todas las habitaciones hijas.

### Documentación

- Añadido `sessions/2026-07-26_PHASE_H_REGRESSION_COMPLETE.md`.
- Añadido `sessions/2026-07-26_SPAWN_FIRST_CHILD_REMOVED.md`.
- Issue #7 cerrada como completada.

### Limpieza pendiente

```text
SpawnRoomsFromCells
Set Array Elem antiguos de Key/Boss
Print String temporales
variables sin referencias
nodos desconectados
```

---

## 2026-07-25 — Generación física completa y salas especiales adicionales

### Añadido

- `PlaceChildRoomFromParent`.
- `DoesRoomOverlapPlacedRooms`.
- `TryAddSpecialCellFromParent`.
- Loop general en `GenerateDungeon` desde índice 1 hasta `DungeonCells.Length - 1`.
- Reintentos controlados con `For Loop with Break`.
- Búsqueda de las cuatro direcciones para habitaciones especiales.
- `BP_Room_PreBuilt_Base_Child_Key`.
- `BP_Room_PreBuilt_Base_Child_Boss`.
- Documento `docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md`.
- Cierre `sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md`.

### Variables confirmadas

`PlaceChildRoomFromParent`:

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

`DoesRoomOverlapPlacedRooms`:

```text
Candidate Bounds Center
Candidate Bounds Extent
Placed Room Actor
Placed Bounds Center
Placed Bounds Extent
Found Overlap
```

`TryAddSpecialCellFromParent`:

```text
Direction Start Index
Random Direction Index
Random Cell Index
Selected Cell
Selected Direction
Neighbor X
Neighbor Y
New Room Seed
Updated Selected Cell
New Base Cell
Opposite Direction
New Connected Cell
```

### Modificado

- `GenerateDungeon` dejó de llamar a `SpawnFirstChildRoom` como ruta principal y usa un loop general.
- `BuildDungeonLayout` usa `Max Rooms + 1` para incluir Start.
- `Max Rooms` pasa a significar exclusivamente cantidad de habitaciones `Normal`.
- `ChooseKeyAndBossCells` conserva la búsqueda de candidatos, pero Key/Boss ya no sustituyen habitaciones normales.
- `Key Cell Index` y `Boss Cell Index` representan ahora padres normales seleccionados.
- Los antiguos `Set Array Elem` que cambiaban `Room Type` quedaron desconectados temporalmente.
- Key se añade antes de Boss mediante `Sequence`.
- Boss comprueba ocupación después de añadir Key.

### Placement general

```text
SpawnStartRoom
→ For Loop with Break
   First Index = 1
   Last Index = DungeonCells.Length - 1
→ PlaceChildRoomFromParent(Index)
→ Room Placed false = Break
```

### Reintentos

Valores de prueba:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

Reglas:

```text
No repetir SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover la misma candidata.
Volver a consultar Child Door Location cada intento.
```

### Overlap global

`DoesRoomOverlapPlacedRooms` recorre todas las habitaciones aceptadas y usa:

```text
Abs(CandidateCenter - PlacedCenter)
<= CandidateExtent + PlacedExtent
```

por X/Y/Z con AND final.

### Salas especiales

`TryAddSpecialCellFromParent`:

```text
Inputs:
Parent Cell Index
Special Room Type

Outputs:
bAdded
New Cell Index
```

- Rechaza `Start` y `Normal`.
- Todos los fallos devuelven `New Cell Index=-1`.
- El índice de éxito se toma del Add de `DungeonCells`.
- Añade también un `DungeonCellLink` con el mismo índice.

Búsqueda:

```text
(Direction Start Index + For Loop.Index) % 4
```

### Corregido

- Se corrigió el uso accidental de división entre 4 por módulo `% 4`.
- Se corrigió el falso overlap de la antigua `BP_Room_Debug_Key_C`, que devolvía bounds cero.
- Key y Boss se migraron a hijos de `BP_Room_PreBuilt_Base`.
- Se corrigió la documentación de seed 12345: la dirección confirmada es `South`, no `North`.

### Validado

```text
KEY SPECIAL ADDED
BOSS SPECIAL ADDED
```

Con:

```text
Max Rooms = 10
```

resultado:

```text
1 Start
10 Normal
1 Key
1 Boss
= 13 habitaciones
```

### Decisión de arquitectura

```text
Max Rooms cuenta solo Normal.
Las salas especiales se añaden fuera de ese límite.
```

Próxima fase:

```text
Compatibilidad de patrones de puertas para habitaciones prebuilt.
```

Patrones previstos:

```text
Dead End
Straight
Corner/L
T
Cross
```

Las prebuilt no podrán inventar puertas y deberán encajar mediante clase + rotación 0/90/180/270.

---

## 2026-07-23 — Separación, prebuilt base y RoomBounds

### Añadido

- `Parent Direction` y variables de bounds en `SpawnFirstChildRoom`.
- `BP_Room_PreBuilt_Base`.
- `docs/28_ROOM_BOUNDS_FIRST_CHILD.md`.
- `docs/29_BP_ROOM_PREBUILT_BASE.md`.
- `sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md`.

### Validado

```text
Start prebuilt Bounds Extent = 980,980,400
Primera hija procedural = 995,995,420
AABB libre con 1000 = false
AABB forzada con -500 = true
```

### Corregido

```text
Incorrecto: Abs(ParentCenter) - ChildCenter
Correcto:   Abs(ParentCenter - ChildCenter)
```

---

## 2026-07-21 — Primera habitación hija alineada

- `SpawnFirstChildRoom` creada.
- `GetDirectionVector` creada.
- Primera hija inicializada una vez y movida sin regenerar HISM.
- Alineación base validada.
- Error de medición 2950 diagnosticado.

---

## 2026-07-21 — SpawnStartRoom completada

- `SpawnStartRoom` creada.
- Start validada e inicializada una sola vez.
- `SpawnedRooms[0]` confirmado.
- Scale Z corregida de 0 a 1.

---

## 2026-07-20 — Memoria permanente

- Repositorio establecido como memoria técnica oficial.
- Cells/Links validados hasta 150 salas.
- Start con una salida confirmada.
- Política de actualización permanente añadida.

---

## 2026-07-19 — Base técnica inicial

- Documentación estructurada creada.
- `ST_DungeonCellLink` y `DungeonCellLinks` consolidados.
- Arquitectura padre-hija aprobada.
- Identidades de `BP_RoomMaster` y `BP_RoomMaster_Dungeon` fijadas.
