# Changelog — Mazmorra Procedural V2

Todos los cambios técnicos importantes deben añadirse aquí y reflejarse también en `docs/00_ESTADO_ACTUAL.md`.

## 2026-07-21 — Primera habitación hija alineada

### Añadido

- Creada `SpawnFirstChildRoom` dentro de `BP_DungeonGenerator_V2`.
- Añadidas validaciones de `DungeonCells[1]`, `DungeonCellLinks[1]`, `bHasParent`, `ParentCellIndex`, actor padre y actor hija.
- Añadidas variables locales:
  - `Parent Room Actor`;
  - `Child Room Actor`;
  - `Child Entry Direction`;
  - `Parent Door Location`;
  - `Child Door Location`.
- Añadida selección de clase de la hija según `RoomType`.
- Añadidos mensajes de diagnóstico para fallos de datos, padre y spawn.
- Creada `GetDirectionVector`.
- Creado el documento técnico `docs/27_SPAWN_FIRST_CHILD_ROOM.md`.

### Implementación confirmada

```text
DungeonCellLinks[1]
→ ParentCellIndex
→ SpawnedRooms[ParentCellIndex]
→ Parent Room Actor
```

```text
DirectionFromParent
→ GetOppositeDirection
→ ChildEntryDirection
```

```text
ParentDoor = GetDoorWorldLocation(ParentRoomActor, DirectionFromParent)
ChildDoor = GetDoorWorldLocation(ChildRoomActor, ChildEntryDirection)
```

```text
MoveDelta = ParentDoorLocation - ChildDoorLocation
NewLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma hija
```

### Validado

```text
✅ Solo se genera una hija además de Start.
✅ SpawnedRooms.Num == 2.
✅ SpawnedRooms[0] = Start.
✅ SpawnedRooms[1] = primera hija.
✅ InitRoomFromCell se ejecuta una sola vez para la hija.
✅ La hija se mueve sin regenerar HISM.
✅ Distancia final entre DoorPoints = 0.0.
```

### Error corregido

La primera medición mostró `2950` porque comparaba la puerta de la hija contra la ubicación/centro del actor.

La medición correcta quedó:

```text
Distance(
  ChildDoorWorldLocation después de mover,
  ParentDoorLocation
)
= 0.0
```

### GetDirectionVector

Mapping confirmado:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

La captura final aún muestra pines de ejecución; la propiedad `Pure` queda pendiente de confirmar en la próxima sesión.

### Siguiente paso

```text
Confirmar/activar Pure en GetDirectionVector
→ añadir CorridorLength de prueba
→ DesiredChildDoor = ParentDoor + DirectionVector * CorridorLength
→ mover la misma hija
→ comprobar DoorDistance == CorridorLength
```

---

## 2026-07-21 — SpawnStartRoom completada

### Añadido

- Creada la función `SpawnStartRoom` dentro de `BP_DungeonGenerator_V2`.
- Añadida validación de `DungeonCells[0]`.
- Añadida comprobación de que `DungeonCells[0].RoomType == Start`.
- Añadida validación del `Return Value` de `SpawnActor`.
- Añadidos prints de diagnóstico para índice inválido, tipo incorrecto y fallo de spawn.
- Añadido documento técnico `docs/26_SPAWN_START_ROOM.md`.

### Implementación confirmada

```text
GetActorLocation del generador
→ Make Transform
→ SpawnActor con Start Room Class
→ Is Valid
→ InitRoomFromCell(DungeonCells[0])
→ Add SpawnedRooms
```

### Corregido durante el montaje

```text
Scale Z = 0
→ Scale Z = 1
```

También se corrigió el mensaje del error de spawn para distinguirlo del error `DungeonCells[0] is not Start`.

### Validado

```text
✅ Compila.
✅ Solo aparece Start.
✅ SpawnedRooms.Num == 1.
✅ SpawnedRooms[0] es válido.
✅ La Start nace en GetActorLocation del generador.
✅ InitRoomFromCell se ejecuta con DungeonCells[0].
✅ Start conserva una única apertura.
✅ No aparecen hijas durante la prueba aislada.
```

### Decisión de diseño

Aprobado conceptualmente para V1:

```text
15 habitaciones normales/procedurales
+ 1 Start
+ 1 Key
+ 1 Boss
= 18 habitaciones físicas
```

La regla de conteo todavía no se implementa hasta estabilizar la colocación padre-hija.

---

## 2026-07-20 — Cierre de sesión y memoria permanente

### Confirmado

- Se mantiene validado `DungeonCells.Num == DungeonCellLinks.Num` con 10, 15, 20, 50 y 150 habitaciones.
- `ParentCellIndex < ChildIndex` continúa confirmado.
- La Start conserva exactamente una salida lógica.
- Los prints temporales de cantidades y links quedan desactivados, pero conservados.

### Memoria técnica

- El repositorio queda establecido como memoria técnica oficial de la Mazmorra Procedural V2.
- Añadida política permanente de actualización tras funciones probadas, bugs corregidos, fases cerradas y decisiones importantes.
- Añadido cierre de sesión en `sessions/2026-07-20_CIERRE_SESION.md`.
- Actualizado `knowledge/state.yaml` con el punto exacto de continuación.

---

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
