# 10 — Pruebas y regresión

**Última actualización:** 2026-07-23

Este archivo reúne las pruebas que deben repetirse después de cambios importantes.

## 1. Layout lógico

- [x] Genera 10 habitaciones.
- [x] Genera 15 habitaciones.
- [x] Genera 20 habitaciones.
- [x] Genera 50 habitaciones.
- [x] Genera 150 habitaciones.
- [x] Start mantiene una sola salida.
- [ ] Probar muchas seeds con el mismo tamaño.
- [ ] Confirmar determinismo completo de layout, tamaños y longitudes.
- [ ] No hay dos celdas con el mismo `GridX/GridY`.
- [ ] Cada conexión lógica existe en ambos lados.
- [ ] Boss es dead-end cuando existe candidato.
- [ ] Key no es Start ni Boss.

## 2. DungeonCellLinks

- [x] `DungeonCells.Num == DungeonCellLinks.Num` con 10.
- [x] `DungeonCells.Num == DungeonCellLinks.Num` con 15.
- [x] `DungeonCells.Num == DungeonCellLinks.Num` con 20.
- [x] `DungeonCells.Num == DungeonCellLinks.Num` con 50.
- [x] `DungeonCells.Num == DungeonCellLinks.Num` con 150.
- [x] Link 0: `ParentCellIndex=-1`.
- [x] Link 0: `bHasParent=false`.
- [x] Links > 0: `bHasParent=true`.
- [x] `ParentCellIndex >= 0` para hijas.
- [x] `ParentCellIndex < ChildIndex`.
- [x] `DirectionFromParent` recibe `Selected Direction`.
- [ ] Validación automática de conexión simétrica padre-hija.

## 3. SpawnStartRoom

- [x] `DungeonCells.IsValidIndex(0)`.
- [x] `DungeonCells[0].RoomType == Start`.
- [x] Clase Start conectada a `SpawnActor`.
- [x] Transform con escala `1,1,1`.
- [x] Return Value validado.
- [x] `InitRoomFromCell(DungeonCells[0])` una sola vez.
- [x] `SpawnedRooms.Num == 1` en prueba aislada.
- [x] `SpawnedRooms[0]` válido.
- [x] Start con una apertura.
- [x] Mensajes de diagnóstico para índice, tipo y spawn.

## 4. Primera hija padre-hija

- [x] `DungeonCells.IsValidIndex(1)`.
- [x] `DungeonCellLinks.IsValidIndex(1)`.
- [x] `bHasParent == true`.
- [x] `ParentCellIndex` válido.
- [x] `SpawnedRooms.IsValidIndex(ParentCellIndex)`.
- [x] Parent Room Actor válido.
- [x] Child Room Actor válido.
- [x] Hija generada una sola vez.
- [x] `InitRoomFromCell(DungeonCells[1])` una sola vez.
- [x] `Child Entry Direction = GetOppositeDirection(Direction From Parent)`.
- [x] ParentDoor usa `Direction From Parent`.
- [x] ChildDoor usa `Child Entry Direction`.
- [x] Misma referencia movida con `SetActorLocation`.
- [x] `SpawnedRooms.Num == 2`.
- [x] `SpawnedRooms[0] = Start`.
- [x] `SpawnedRooms[1] = primera hija`.
- [x] Alineación base sin pasillo: distancia `0.0`.
- [x] Error de medición `2950` diagnosticado y corregido.

## 5. GetDirectionVector

- [x] Función creada.
- [x] Función Pure confirmada.
- [x] North = `(0,1,0)`.
- [x] East = `(1,0,0)`.
- [x] South = `(0,-1,0)`.
- [x] West = `(-1,0,0)`.
- [x] Usa `Parent Direction`, no `Child Entry Direction`, para separar la hija.

## 6. Separación inicial de pasillo

- [x] `Parent Direction` guardada desde `Direction From Parent`.
- [x] `DesiredChildDoor = ParentDoor + DirectionVector * 1000`.
- [x] `MoveDelta = DesiredChildDoor - ChildDoorLocation`.
- [x] Se mueve la misma Child Room Actor.
- [x] No se repite `SpawnActor`.
- [x] No se repite `InitRoomFromCell`.
- [x] No se regenera HISM.
- [x] Seed 12345 → North → distancia 1000.
- [x] Seed 12346 → East → distancia 1000.
- [x] `SpawnedRooms.Num == 2` en ambas.
- [ ] Crear variable formal `Corridor Length`; ahora 1000 es literal temporal.

## 7. BP_Room_PreBuilt_Base

- [x] Base creada/adaptada.
- [x] Implementa `BPI_DungeonRoomV2`.
- [x] Tiene DoorPoint North/East/South/West.
- [x] Tiene `RoomBounds`.
- [x] `Get Room Bounds Data` usa `Get Component Bounds`.
- [x] `Origin → Bounds Center`.
- [x] `Box Extent → Bounds Extent`.
- [x] Start de prueba devuelve `980,980,400`.
- [ ] Verificar visualmente `RoomBounds.RelativeLocation` final.
- [ ] Limpiar componentes debug solo después de revisar dependencias.
- [ ] Validar un Blueprint hijo real de Start.
- [ ] Validar Level Instance o Packed Level Blueprint visual.

## 8. RoomBounds procedural

- [x] `Update Room Bounds` calcula X desde `Room Width`.
- [x] `Update Room Bounds` calcula Y desde `Room Depth`.
- [x] `Update Room Bounds` calcula Z desde altura de paredes.
- [x] `Set Box Extent` recibe vector calculado.
- [x] `Set Relative Location` centra Z.
- [x] Primera hija devuelve `995,995,420`.
- [ ] Probar 5x5.
- [ ] Probar 8x5.
- [ ] Probar 5x10.
- [ ] Probar 10x10.
- [ ] Probar tamaños aleatorios min/max.

## 9. Comparación AABB padre-hija

Variables locales confirmadas:

```text
Parent Bounds Center : Vector
Parent Bounds Extent : Vector
Child Bounds Center  : Vector
Child Bounds Extent  : Vector
bBoundsOverlap       : Boolean
```

Pruebas:

- [x] Bounds del padre obtenidos después de mover la hija.
- [x] Bounds de la hija obtenidos después de moverla.
- [x] X usa `Abs(ParentCenterX - ChildCenterX) <= ParentExtentX + ChildExtentX`.
- [x] Y usa `Abs(ParentCenterY - ChildCenterY) <= ParentExtentY + ChildExtentY`.
- [x] Z usa `Abs(ParentCenterZ - ChildCenterZ) <= ParentExtentZ + ChildExtentZ`.
- [x] `bBoundsOverlap = X AND Y AND Z`.
- [x] Caso libre con 1000 devuelve `False`.
- [x] Caso forzado con -500 devuelve `True`.
- [x] Error de `ABS` antes de la resta corregido.
- [ ] Confirmar que el literal volvió a 1000 antes de Fase F.

## 10. Reintentos controlados — próxima fase

- [ ] Crear/confirmar `Corridor Length : Float`.
- [ ] Crear/confirmar `Placement Retry Step : Float`.
- [ ] Crear/confirmar `Placement Attempt : Integer`.
- [ ] Crear/confirmar `Max Placement Attempts : Integer`.
- [ ] Una colisión inicial provoca un reintento.
- [ ] Cada reintento aumenta Corridor Length.
- [ ] Se mueve la misma referencia.
- [ ] No se llama otra vez a `SpawnActor`.
- [ ] No se repite `InitRoomFromCell`.
- [ ] No se regenera HISM.
- [ ] Termina cuando `bBoundsOverlap == false`.
- [ ] Termina al alcanzar `Max Placement Attempts`.
- [ ] `SpawnedRooms.Num == 2`.
- [ ] Probar seed 12345 North.
- [ ] Probar seed 12346 East.

## 11. Direcciones y DoorPoints protegidos

### Aperturas

- [ ] Cell North abre SouthOpening.
- [ ] Cell East abre WestOpening.
- [ ] Cell South abre NorthOpening.
- [ ] Cell West abre EastOpening.

### DoorWorldLocation

- [ ] Dungeon North devuelve `Arrow_Entrance_South`.
- [ ] Dungeon East devuelve `Arrow_Exit_East`.
- [ ] Dungeon South devuelve `Arrow_Exit_North`.
- [ ] Dungeon West devuelve `Arrow_Exit_West`.

## 12. Spawn completo futuro

- [ ] `DungeonCells.Num == SpawnedRooms.Num`.
- [ ] Todas las salas aceptadas.
- [ ] Sin solapamientos.
- [ ] Pasillos conectan DoorPoints.
- [ ] Boss Door sigue correcta.
- [ ] Regeneración limpia todos los actores y arrays.

## 13. Rendimiento

- [ ] Sin Tick en generador para generar.
- [ ] Sin Tick en salas para generar.
- [ ] HISM mantiene instancias agrupadas.
- [x] Primera hija usa una llamada de inicialización.
- [x] Las pruebas de posición mueven la misma hija.
- [ ] Reintentos solo mueven actor.
- [ ] Medir 10, 20, 50 y 150 salas.
- [ ] Medir prebuilt con Level Instance/Packed Level Blueprint.

## Política de pruebas

```text
compilar
→ prueba controlada
→ caso libre
→ caso de colisión
→ varias direcciones/seeds
→ revisar Output Log
→ revisar visualmente
→ documentar
```

No avanzar a la siguiente fase si la actual no está validada.
