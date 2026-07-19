# Changelog — Mazmorra Procedural V2

Todos los cambios técnicos importantes deben añadirse aquí y reflejarse también en `docs/00_ESTADO_ACTUAL.md`.

## 2026-07-19 — Base de conocimiento GitHub

### Documentación

- Creado repositorio técnico estructurado.
- Añadido índice principal.
- Separada documentación por Blueprint y subsistema.
- Añadidos estados de certeza.
- Añadido prompt de traspaso actualizado.
- Añadida matriz de regresión.
- Registrados errores históricos y decisiones protegidas.

### Estado operativo actualizado

```text
DungeonCells.Num == DungeonCellLinks.Num
```

Validado con:

```text
10
15
20
50
150 habitaciones
```

### DungeonCellLinks

Confirmado:

```text
CreateStartCell:
ParentCellIndex=-1
DirectionFromParent=North
bHasParent=false
```

Confirmado en `TryAddRandomCell`:

```text
Random Cell Index → ParentCellIndex
Selected Direction → DirectionFromParent
bHasParent=true
```

Validado:

```text
ParentCellIndex >= 0 para hijas
ParentCellIndex < ChildIndex
bHasParent=true para hijas
```

### Start una salida

Añadida condición en `TryAddRandomCell`:

```text
Random Cell Index == 0
AND
SelectedCell tiene cualquier conexión activa
```

Resultado:

```text
True → Return Added=false
```

Corregido durante montaje:

- Return estaba inicialmente en `Added=true`.
- OR comprobaba inicialmente solo South/West.
- Se cambió a `Added=false` y OR con N/E/S/W.

Validado con 10, 15, 20, 50 y 150 habitaciones.

### Debug

- Print `Cells | Links` añadido y validado.
- Print detallado de links añadido y validado.
- Ambos quedan desactivados pero conservados.

### Blueprints aclarados

```text
BP_RoomMaster = original sin tocar
BP_RoomMaster_Dungeon = duplicado integrado
```

### Funciones Debug aclaradas

```text
DebugDrawConnections
→ líneas entre celdas lógicas

DebugDrawDoorPoints
→ puntos en puertas activas

DebugDrawDoorToDoorConnections
→ líneas entre DoorPoints físicos
```

### Siguiente paso

```text
Revisar SpawnRoomsFromCells
→ crear SpawnStartRoom
```

---

## 2026-07-19 — Documento maestro TOTAL v7

- Consolidado `BP_DungeonGenerator_V2`.
- Consolidado `ST_DungeonCellLink`.
- Consolidado `BP_RoomMaster` original.
- Consolidado `BP_RoomMaster_Dungeon`.
- Consolidado `BP_FloorActor`, `BP_WallActor`, `BP_CeilingActor`.
- Documentados mappings finales.
- Documentado `RoomContentRoot`.
- Documentado `RoomBounds`.
- Aprobada arquitectura padre-hija.

---

## 2026-07-19 — Integración procedural y bounds

### Añadido

```text
RoomContentRoot
RoomBounds
bSouthOpening
bUseDungeonConnections
CurrentCellData
BoundsPadding
BoundsPaddingZ
BoundsHalfHeight
GetRoomBoundsData
CenterRoomContentOnActorOrigin
UpdateRoomBounds
```

### Corregido

- origen de sala procedural;
- aperturas por layout;
- DoorWorldLocation;
- paredes East/West;
- altura y posición de bounds.

### Validado

- pasillos por cuatro lados;
- Boss Door;
- llave temporal;
- mini inventario.

---

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
