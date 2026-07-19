# 10 — Pruebas y regresión

Este archivo reúne todas las pruebas que deben repetirse después de cambios importantes.

## 1. Layout lógico

- [x] Genera 10 habitaciones.
- [x] Genera 15 habitaciones.
- [x] Genera 20 habitaciones.
- [x] Genera 50 habitaciones.
- [x] Genera 150 habitaciones.
- [ ] Probar muchas seeds con el mismo tamaño.
- [ ] Confirmar que una misma seed reproduce el layout cuando la seed sea formalmente determinista para todos los subsistemas.
- [ ] No hay dos celdas con el mismo `GridX/GridY`.
- [ ] Cada conexión lógica existe en ambos lados.
- [x] Start mantiene una sola salida.
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
- [ ] Validación automática de que la conexión correspondiente del padre está en true.

## 3. Start con una sola salida

- [x] La condición comprueba `Random Cell Index == 0`.
- [x] El OR incluye North.
- [x] El OR incluye East.
- [x] El OR incluye South.
- [x] El OR incluye West.
- [x] La rama True devuelve `Added=false`.
- [x] 10 habitaciones alcanzadas.
- [x] 15 habitaciones alcanzadas.
- [x] 20 habitaciones alcanzadas.
- [x] 50 habitaciones alcanzadas.
- [x] 150 habitaciones alcanzadas.
- [x] Visualmente Start muestra un único pasillo.

## 4. Spawn actual

Antes del refactor padre-hija:

- [ ] `DungeonCells.Num == SpawnedRooms.Num`.
- [ ] Cada `SpawnedRooms[Index]` corresponde al tipo de `DungeonCells[Index]`.
- [ ] No hay referencias nulas.
- [ ] `InitRoomFromCell` se ejecuta después de `SpawnActor`.
- [ ] `SpawnedRooms` se limpia al regenerar.

## 5. SpawnStartRoom

Pruebas futuras inmediatas:

- [ ] `DungeonCells.IsValidIndex(0)`.
- [ ] `DungeonCellLinks.IsValidIndex(0)`.
- [ ] `StartDebugRoomClass` válido.
- [ ] Solo se spawnea una sala.
- [ ] `SpawnedRooms.Num == 1`.
- [ ] `SpawnedRooms[0]` válido.
- [ ] Start colocada en origen acordado.
- [ ] `InitRoomFromCell(DungeonCells[0])` ejecutado.
- [ ] Una única apertura visible.

## 6. Habitación procedural

Tamaños:

- [ ] 5x5.
- [ ] 8x5.
- [ ] 5x10.
- [ ] 10x10.
- [ ] Valores aleatorios dentro de min/max.
- [ ] Alturas distintas.

Centrado:

- [ ] `GetActorLocation` queda en el centro.
- [ ] `RoomContentRoot` usa offsets correctos.
- [ ] Rectángulos mantienen centro.

Bounds:

- [ ] Bounds cubre ancho.
- [ ] Bounds cubre fondo.
- [ ] Bounds cubre altura.
- [ ] Relative Z = media altura.
- [ ] `BoundsPadding=5` se ve correcto.
- [ ] `BoundsPaddingZ=20` se ve correcto.
- [ ] Verificar si `BoundsHalfHeight` sigue conectado.

## 7. Direcciones y DoorPoints

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

### Resultado visual

- [ ] North conecta puntos interiores.
- [ ] East conecta puntos interiores.
- [ ] South conecta puntos interiores.
- [ ] West conecta puntos interiores.
- [ ] Ningún pasillo atraviesa una sala completa.

## 8. Paredes y HISM

- [ ] `ClearInstances` antes de regenerar.
- [ ] `SetupFloor` antes de paredes.
- [ ] `EffectiveTileSize` correcto.
- [ ] South/North usan `RoomWidth`.
- [ ] East/West usan `RoomDepth`.
- [ ] `OpeningStartIndex` correcto.
- [ ] `OpeningHeightTiles=1`.
- [ ] `IsOpeningCell=true` no crea pared.
- [ ] `IsOpeningCell=false` crea pared.
- [ ] Columnas respetan huecos.
- [ ] Loop de columnas termina en `LengthTiles-1`.
- [ ] Edge inicial en `TileSize * -0.5`.
- [ ] Edge final usa fórmula correcta.
- [ ] `MeshMinZ` corrige pared.
- [ ] `ColumnMinZ` corrige columna.

## 9. Pasillos

- [ ] Se procesan solo North/East.
- [ ] No hay duplicados.
- [ ] `SpawnedCorridors` se limpia al regenerar.
- [ ] Start y End son DoorPoints reales.
- [ ] Pasillo corto correcto.
- [ ] Pasillo medio futuro.
- [ ] Pasillo largo futuro.

## 10. Boss y Key

Boss:

- [ ] `ConnectionCount==1` cuando hay dead-end.
- [ ] Es la candidata más alejada.
- [ ] `RoomType` cambia sin perder datos.
- [ ] `BossGridX/Y` correctos.

Key:

- [ ] No es Start.
- [ ] No es Boss.
- [ ] `DistanceFromStart>=3`.
- [ ] `KeyScore` calculado correctamente.
- [ ] `RoomType` cambia sin perder datos.

## 11. Boss Door

- [ ] Índice Boss válido.
- [ ] Room Boss válida.
- [ ] Una puerta en la única entrada.
- [ ] North yaw 0.
- [ ] East yaw -90.
- [ ] South yaw 180.
- [ ] West yaw 90.
- [ ] No quedan puertas antiguas.
- [ ] No abre sin llave.
- [ ] Abre con llave.

## 12. Pickup e inventario temporal

- [ ] Pickup aparece tras Delay temporal.
- [ ] Usa el punto central correcto.
- [ ] No se duplica tras regeneraciones.
- [ ] Se añade al mini inventario.
- [ ] UI abre con `O`.
- [ ] La Boss Door consulta el item requerido.

## 13. Colocación padre-hija

Primera hija:

- [ ] `ParentCellIndex` válido.
- [ ] Padre ya existe en `SpawnedRooms`.
- [ ] `ChildEntryDirection` opuesta.
- [ ] ParentDoor correcto.
- [ ] ChildDoor correcto.
- [ ] `MoveDelta` alinea puertas.

Solapamiento:

- [ ] `GetRoomBoundsData` después de mover.
- [ ] Comparación contra salas aceptadas.
- [ ] Si choca, aumenta `CorridorLength`.
- [ ] Se mueve la misma referencia.
- [ ] No regenera HISM.
- [ ] Límite de intentos evita loop infinito.

Layout completo:

- [ ] `DungeonCells.Num == SpawnedRooms.Num`.
- [ ] Todas las salas aceptadas.
- [ ] Sin solapamientos.
- [ ] Pasillos conectan puertas.
- [ ] Boss Door sigue correcta.

## 14. Regeneración

- [ ] Destruye puertas.
- [ ] Destruye pasillos.
- [ ] Destruye salas.
- [ ] Limpia `DungeonCells`.
- [ ] Limpia `DungeonCellLinks`.
- [ ] Limpia `SpawnedRooms`.
- [ ] Limpia `SpawnedCorridors`.
- [ ] Limpia `SpawnedDungeonDoors`.
- [ ] Resetea Boss/Key.
- [ ] No deja actores huérfanos.

## 15. Rendimiento

- [ ] Sin Tick en generador.
- [ ] Sin Tick en salas para generación.
- [ ] HISM mantiene instancias agrupadas.
- [ ] Una llamada `GenerateRoom` por sala.
- [ ] Reintentos solo mueven actor.
- [ ] Medir 10, 20, 50 y 150 salas.
- [ ] No refactorizar Child Actors sin perfilado.

## Política de pruebas

Después de cualquier cambio en una parte protegida:

```text
compilar
→ prueba pequeña
→ prueba media
→ prueba grande
→ varias seeds
→ revisar Output Log
→ revisar visualmente
```

No avanzar a la siguiente fase si la actual no está validada.
