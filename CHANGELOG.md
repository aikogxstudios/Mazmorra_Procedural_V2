# Changelog — Mazmorra Procedural V2

Todos los cambios técnicos importantes deben añadirse aquí y reflejarse también en `docs/00_ESTADO_ACTUAL.md`.

## 2026-07-21 — SpawnStartRoom completada

### Añadido

- Creada la función `SpawnStartRoom` dentro de `BP_DungeonGenerator_V2`.
- Añadida validación de `DungeonCells[0]`.
- Añadida comprobación de que `DungeonCells[0].RoomType == Start`.
- Añadida validación del `Return Value` de `SpawnActor`.
- Añadidos prints de diagnóstico para índice inválido, tipo incorrecto y fallo de spawn.
- Añadido documento técnico `docs/18_SPAWN_START_ROOM.md`.

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

### Siguiente paso

```text
Fase D
→ generar únicamente ChildIndex 1
→ leer DungeonCellLinks[1]
→ obtener su padre
→ spawnear e inicializar una sola hija
→ alinear DoorPoint de padre e hija
→ comprobar SpawnedRooms.Num == 2
```

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

### Siguiente paso

```text
Revisar capturas completas de SpawnRoomsFromCells
→ crear y probar únicamente SpawnStartRoom
```

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