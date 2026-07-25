# 10 — Pruebas y regresión

**Última actualización:** 2026-07-25

Este archivo reúne las pruebas obligatorias después de cambios importantes.

## 1. Layout lógico

- [x] Genera 10, 15, 20, 50 y 150 celdas históricamente.
- [x] Start mantiene una sola salida.
- [x] `DungeonCells.Num == DungeonCellLinks.Num`.
- [x] Link 0 usa `ParentCellIndex=-1` y `bHasParent=false`.
- [x] Hijas usan `bHasParent=true`.
- [x] `ParentCellIndex < ChildIndex`.
- [ ] Probar muchas seeds con el mismo tamaño.
- [ ] Confirmar determinismo completo.
- [ ] Automatizar validación de coordenadas únicas.
- [ ] Automatizar conexión simétrica padre-hija.

## 2. Max Rooms

Regla actual:

```text
Max Rooms = habitaciones Normal
```

- [x] `BuildDungeonLayout` usa `Max Rooms + 1` para incluir Start.
- [x] `Max Rooms=10` genera 1 Start + 10 Normal antes de especiales.
- [x] Key no consume `Max Rooms`.
- [x] Boss no consume `Max Rooms`.
- [x] Resultado final actual con 10: 13 habitaciones.

## 3. SpawnStartRoom

- [x] Valida `DungeonCells[0]`.
- [x] Comprueba `RoomType == Start`.
- [x] Transform con escala `1,1,1`.
- [x] Valida `SpawnActor Return Value`.
- [x] Ejecuta `InitRoomFromCell` una sola vez.
- [x] Añade `SpawnedRooms[0]`.

## 4. PlaceChildRoomFromParent

- [x] Input `Child Index : Integer`.
- [x] Output `Room Placed : Boolean`.
- [x] Valida `DungeonCells[Child Index]`.
- [x] Valida `DungeonCellLinks[Child Index]`.
- [x] Comprueba `bHasParent`.
- [x] Resuelve `Parent Cell Index`.
- [x] Resuelve padre desde `SpawnedRooms`.
- [x] Selecciona clase Normal/Key/Boss.
- [x] Rechaza Start como hija.
- [x] Genera el actor una sola vez.
- [x] Inicializa una sola vez.
- [x] Mueve la misma referencia.
- [x] Añade a `SpawnedRooms` solo después del éxito.
- [x] Destruye la candidata al fallar.

## 5. Direcciones

- [x] `Child Entry Direction = GetOppositeDirection(Parent Direction)`.
- [x] ParentDoor usa `Parent Direction`.
- [x] ChildDoor usa `Child Entry Direction`.
- [x] `GetDirectionVector` es Pure.
- [x] North = `(0,1,0)`.
- [x] East = `(1,0,0)`.
- [x] South = `(0,-1,0)`.
- [x] West = `(-1,0,0)`.
- [x] Corrección actual: seed 12345 → South.
- [x] Seed 12346 → East.

## 6. Separación y reintentos

Valores de prueba:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

- [x] `Corridor Length` existe como Float.
- [x] `Placement Retry Step` existe como Float.
- [x] `Placement Attempt` existe como Integer.
- [x] `Max Placement Attempts` existe como Integer.
- [x] `bPlacement Succeeded` existe como Boolean.
- [x] El loop usa `Max Placement Attempts - 1`.
- [x] Una colisión aumenta `Corridor Length`.
- [x] Cada intento vuelve a consultar `Child Door Location`.
- [x] No se repite `SpawnActor`.
- [x] No se repite `InitRoomFromCell`.
- [x] No se regenera HISM.
- [x] Caso libre termina sin FAILED.
- [x] Caso forzado termina con `Max Placement Attempts reached`.
- [x] Caso forzado destruye el actor.

## 7. DoesRoomOverlapPlacedRooms

- [x] Función creada.
- [x] Input `Candidate Room Actor`.
- [x] Output `Overlaps Placed Rooms`.
- [x] Valida candidata.
- [x] Obtiene candidate bounds.
- [x] Recorre `SpawnedRooms` con `For Each Loop with Break`.
- [x] Ignora actores inválidos.
- [x] Ignora la propia candidata.
- [x] Obtiene placed bounds.
- [x] Aplica AABB X/Y/Z.
- [x] `Found Overlap` se devuelve al completar.
- [x] Caso libre devuelve False.
- [x] Caso forzado devuelve True.

Fórmula protegida:

```text
Abs(CandidateCenter - PlacedCenter)
<= CandidateExtent + PlacedExtent
```

por cada eje y AND final.

## 8. BP_Room_PreBuilt_Base

- [x] Implementa `BPI_DungeonRoomV2`.
- [x] Tiene DoorPoints N/E/S/W.
- [x] Tiene `RoomBounds`.
- [x] `Get Room Bounds Data` usa `Get Component Bounds`.
- [x] Start de prueba devuelve `980,980,400`.
- [x] Key hija de la base devuelve bounds válidos.
- [x] Boss hija de la base devuelve bounds válidos.
- [x] Error de clase debug Key con bounds cero diagnosticado.
- [x] Clases actuales:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

- [ ] Revisar `RoomBounds.RelativeLocation` final.
- [ ] Limpiar componentes debug solo tras revisar referencias.
- [ ] Validar Level Instance o Packed Level Blueprint.

## 9. TryAddSpecialCellFromParent

- [x] Input `Parent Cell Index`.
- [x] Input `Special Room Type`.
- [x] Output `bAdded`.
- [x] Output `New Cell Index`.
- [x] Rechaza Start.
- [x] Rechaza Normal.
- [x] Valida Parent Cell Index.
- [x] Fallos devuelven `New Cell Index=-1`.
- [x] `New Cell Index` sale del Add de `DungeonCells`.
- [x] Añade un `DungeonCellLink` correspondiente.
- [x] Mantiene `DungeonCells.Num == DungeonCellLinks.Num`.
- [x] Prueba las cuatro direcciones.
- [x] Usa `(Direction Start Index + Index) % 4`.
- [x] Error división/modulo corregido.
- [x] Falla solo si las cuatro direcciones están ocupadas.

## 10. Key y Boss adicionales

- [x] Los antiguos `Set Array Elem` de Room Type están desconectados.
- [x] `Key Cell Index` representa padre normal.
- [x] `Boss Cell Index` representa padre normal.
- [x] Key se añade primero.
- [x] Boss se añade después.
- [x] Key comprueba coordenada libre.
- [x] Boss comprueba coordenada libre después de Key.
- [x] Mensaje `KEY SPECIAL ADDED` confirmado.
- [x] Mensaje `BOSS SPECIAL ADDED` confirmado.
- [x] Total 13 confirmado con `Max Rooms=10`.

## 11. Generación completa

- [x] `GenerateDungeon` usa loop desde 1 hasta `DungeonCells.Length-1`.
- [x] `Room Placed=false` conecta a `Break`.
- [x] Tres habitaciones iniciales validadas antes del loop general.
- [x] Diez celdas físicas validadas antes de hacer especiales adicionales.
- [x] Trece habitaciones validadas después del cambio final.
- [ ] Repetir regresión con seed 12345.
- [ ] Repetir regresión con seed 12346.
- [ ] Confirmar visualmente cero solapamientos en ambas.
- [ ] Confirmar `DungeonCells.Num=13`.
- [ ] Confirmar `DungeonCellLinks.Num=13`.
- [ ] Confirmar `SpawnedRooms.Num=13`.

## 12. Puertas prebuilt — próxima fase

- [ ] Definir patrón de puertas reales.
- [ ] Dead End: 1 puerta.
- [ ] Straight: 2 opuestas.
- [ ] Corner/L: 2 contiguas.
- [ ] T: 3.
- [ ] Cross: 4.
- [ ] Probar rotaciones 0/90/180/270.
- [ ] Rechazar prebuilt incompatible.
- [ ] No permitir huecos inventados.
- [ ] Mantener habitaciones procedurales flexibles 1–4.

## 13. Limpieza

No borrar hasta completar la regresión:

- [ ] `SpawnFirstChildRoom`.
- [ ] `SpawnRoomsFromCells`.
- [ ] `Set Array Elem` antiguos de Key/Boss.
- [ ] Prints temporales.
- [ ] Variables sin referencias.

Política:

```text
Find References
→ compilar
→ probar
→ eliminar
→ nueva regresión
```

## Política general de pruebas

```text
compilar
→ prueba controlada
→ caso libre
→ caso de fallo
→ varias seeds/direcciones
→ revisar Output Log
→ revisar visualmente
→ documentar
```
