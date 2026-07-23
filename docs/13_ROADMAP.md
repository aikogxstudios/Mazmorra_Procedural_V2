# 13 — Roadmap de implementación

**Última actualización:** 2026-07-23

## Estado de fases

### Fase A — DungeonCellLinks

```text
✅ ST_DungeonCellLink creado.
✅ DungeonCellLinks creado.
✅ CreateStartCell añade link 0.
✅ TryAddRandomCell añade un link por cada hija.
✅ ParentCellIndex = Random Cell Index.
✅ DirectionFromParent = Selected Direction.
✅ bHasParent=true para hijas.
✅ Cantidades validadas con 10, 15, 20, 50 y 150 salas.
✅ ParentCellIndex válido y menor que ChildIndex.
```

### Fase B — Start con una sola salida

```text
✅ Implementada en TryAddRandomCell.
✅ Comprueba North, East, South y West.
✅ Devuelve Added=false si Start ya tiene una salida.
✅ Validada con 10, 15, 20, 50 y 150 salas.
```

### Fase C — SpawnStartRoom

```text
✅ Función creada y compilada.
✅ DungeonCells[0] validada.
✅ RoomType Start validado.
✅ SpawnActor validado.
✅ InitRoomFromCell ejecutado una sola vez.
✅ Actor añadido como SpawnedRooms[0].
✅ Prueba aislada SpawnedRooms.Num == 1.
✅ Start conserva una sola apertura.
```

### Fase D — Primera hija alineada

```text
✅ SpawnFirstChildRoom creada.
✅ Trabaja con ChildIndex = 1.
✅ Link padre validado.
✅ Parent Room Actor obtenido desde SpawnedRooms.
✅ Hija spawneada e inicializada una sola vez.
✅ Child Entry Direction opuesta calculada.
✅ ParentDoor y ChildDoor obtenidos por interfaz.
✅ Misma hija movida sin regenerar HISM.
✅ Añadida como SpawnedRooms[1].
✅ SpawnedRooms.Num == 2.
✅ Error de alineación base = 0.0.
```

### Fase D.1 — Separación inicial para pasillo

```text
✅ GetDirectionVector convertida en Pure.
✅ Mapping North/East/South/West confirmado.
✅ Parent Direction guardada desde Direction From Parent.
✅ DesiredChildDoor calculada con offset temporal 1000.
✅ La misma hija se mueve sin regenerarse.
✅ Distance entre DoorPoints = 1000.
✅ Seed 12345 validada en North.
✅ Seed 12346 validada en East.
✅ SpawnedRooms.Num sigue siendo 2.
```

Nota:

```text
1000 es todavía un literal temporal del nodo Vector * Float.
La variable formal Corridor Length queda para la Fase F.
```

### Fase E — RoomBounds y solapamiento

```text
✅ Get Room Bounds Data revisada en BP_RoomMaster_Dungeon.
✅ BP_Room_PreBuilt_Base creada/adaptada.
✅ Get Room Bounds Data implementada en la base prebuilt.
✅ Start prebuilt devuelve Bounds Extent 980,980,400.
✅ Primera hija procedural devuelve Bounds Extent 995,995,420.
✅ Variables locales de center/extent para padre e hija creadas.
✅ Comparación AABB implementada en X, Y y Z.
✅ bBoundsOverlap creada.
✅ Caso libre con 1000 devuelve False.
✅ Caso forzado con -500 devuelve True.
✅ Error ABS corregido: Abs(ParentCenter - ChildCenter).
```

Documentos:

```text
docs/28_ROOM_BOUNDS_FIRST_CHILD.md
sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md
```

### Fase F — Reintentos controlados

```text
⏳ Próxima fase inmediata.
```

Alcance inicial:

```text
Solo ChildIndex = 1.
Solo comparar contra Parent Room Actor.
Mover siempre la misma Child Room Actor.
```

Flujo previsto:

```text
Calcular posición candidata
→ SetActorLocation
→ Get Room Bounds Data
→ calcular bBoundsOverlap
```

Si `bBoundsOverlap == true`:

```text
Corridor Length += Placement Retry Step
→ Placement Attempt += 1
→ recalcular posición
→ mover la misma hija
→ repetir
```

Si `bBoundsOverlap == false`:

```text
aceptar posición
→ conservar SpawnedRooms[1]
→ terminar
```

Variables previstas; confirmar al crearlas:

```text
Corridor Length        : Float
Placement Retry Step   : Float
Placement Attempt      : Integer
Max Placement Attempts : Integer
```

Criterios de aceptación:

```text
[ ] Una colisión inicial provoca al menos un reintento.
[ ] Corridor Length aumenta de forma controlada.
[ ] La hija termina en posición libre.
[ ] No se llama otra vez a SpawnActor.
[ ] No se repite Init Room from Cell.
[ ] No se regenera HISM.
[ ] El loop termina por éxito o Max Placement Attempts.
[ ] SpawnedRooms.Num permanece en 2.
[ ] Validar North con seed 12345.
[ ] Validar East con seed 12346.
```

### Fase G — Comparar contra todas las salas aceptadas

```text
⏳ Pendiente.
```

Objetivo:

```text
DoesRoomOverlapPlacedRooms
→ recorrer SpawnedRooms ya aceptadas
→ ignorar la propia hija
→ rechazar si solapa cualquiera
```

Todavía no implementar junto con la Fase F.

### Fase H — Todas las hijas

```text
⏳ Pendiente.
```

Objetivo previsto:

```text
PlaceChildRoomFromParent
SpawnRemainingRoomsFromParents
```

Debe conservar:

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

### Fase I — Pasillos variables visuales

```text
⏳ Pendiente.
```

El placement decide primero la distancia física válida. Después el sistema de pasillos conecta los DoorPoints reales.

### Fase J — Regresión completa

```text
⏳ Pendiente.
```

Incluye:

```text
Key/Boss
Boss Door
pickup e inventario temporal
regeneración
layouts grandes
varias seeds
rendimiento
```

## Arquitectura híbrida aprobada

```text
Habitaciones normales/procedurales
→ BP_RoomMaster_Dungeon
→ HISM
→ tamaño variable futuro
→ RoomBounds dinámico
```

```text
Habitaciones especiales preconstruidas
→ BP_Room_PreBuilt_Base y Blueprints hijos
→ RoomBounds ajustado manualmente
→ DoorPoints ajustados manualmente
→ Level Instance o Packed Level Blueprint para contenido visual cuando se valide
```

Las habitaciones debug antiguas se mantienen únicamente durante la transición hasta sustituir sus referencias.

## Regla de cantidad V1

Aprobada conceptualmente, no implementada todavía:

```text
15 normales/procedurales
+ 1 Start
+ 1 Key
+ 1 Boss
= 18 habitaciones físicas
```

## Funciones confirmadas durante el refactor

```text
SpawnStartRoom
SpawnFirstChildRoom
GetDirectionVector
GetOppositeDirection Pure
```

## Funciones futuras previstas

```text
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
SpawnRemainingRoomsFromParents
ValidateDungeonData (opcional)
```

No crear todas a la vez.

## Fuera de alcance actual

```text
❌ Todas las hijas en la misma fase.
❌ Comparación global antes de cerrar reintentos de ChildIndex 1.
❌ Pasillos visuales finales.
❌ Regla final de 18 habitaciones.
❌ Nuevas RoomTypes.
❌ Decoración definitiva.
❌ Migración total a Level Instances.
```

## Regla de avance

```text
Una fase
→ compilar
→ prueba controlada
→ caso positivo
→ caso negativo
→ varias seeds/direcciones
→ documentar
→ siguiente fase
```
