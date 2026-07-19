# 11 — Errores históricos, soluciones y decisiones protegidas

## Tabla de errores corregidos

| Error | Diagnóstico real | Solución final | Estado |
|---|---|---|---:|
| Habitaciones procedurales no centradas | El suelo crecía desde una esquina | `RoomContentRoot` + offsets | ✅ |
| Pasillos parecían enterrados o mal alineados | Origen físico inconsistente y grid fijo | Centrado; spacing variable pendiente | ✅/⏳ |
| Puertas mirando a la nada | Mapping de aperturas incorrecto | Mapping completo invertido en `InitRoomFromCell` | ✅ |
| Pasillos atravesaban salas | `GetDoorWorldLocation` no coincidía con orientación heredada | N/S invertidos; E/W nominales | ✅ |
| East/West físicos intercambiados | Posición de Child Actors | Fórmulas físicas corregidas | ✅ |
| Bounds Z demasiado pequeño | Se aplicó `0.5` dos veces | Una mitad + `BoundsPaddingZ` | ✅ |
| Bounds permanecía en el suelo | Relative Location Z incorrecta | `RoomSizeZ * 0.5` | ✅ |
| `ParentCellIndex` apuntaba a la hija | Se usó Return Value de `Add DungeonCells` | Usar `Random Cell Index` | ✅ |
| Se buscaban variables inexistentes | Nombres conceptuales confundidos con nombres reales | Usar `Random Cell Index` y `Selected Direction` | ✅ |
| Se creía que `SetConnectionOnCell` modificaba el array | Firma inferida incorrectamente | Documentar `In Cell → Out Cell` | ✅ |
| Se decía que la llave automática estaba pendiente | Documento desactualizado | Registrar spawn temporal con Delay | ✅ |
| Start podía tener varias salidas | Podía volver a ser elegida como padre | Rechazar intento si índice 0 ya tiene conexión | ✅ |
| Primer montaje de Start devolvía Added=true | Return equivocado en rama de bloqueo | Cambiar a Added=false | ✅ |
| Primer OR de Start solo miraba South/West | Faltaban North/East | OR con las cuatro conexiones | ✅ |
| Debug de links solo contaba elementos | No validaba contenido | Print de Child/Parent/Direction/HasParent | ✅ |

## Mappings protegidos

### InitRoomFromCell

```text
Cell North → SouthOpening
Cell East  → WestOpening
Cell South → NorthOpening
Cell West  → EastOpening
```

### GetDoorWorldLocation

```text
Dungeon North → Arrow_Entrance_South
Dungeon East  → Arrow_Exit_East
Dungeon South → Arrow_Exit_North
Dungeon West  → Arrow_Exit_West
```

Estos mappings son distintos a propósito.

No “simplificarlos” ni hacerlos iguales por intuición.

## Fórmulas protegidas

### Centrado

```text
OffsetX = (RoomWidth - 1) * EffectiveTileSize * -0.5
OffsetY = (RoomDepth - 1) * EffectiveTileSize * -0.5
```

### Bounds

```text
RoomSizeX = RoomWidth * EffectiveTileSize
RoomSizeY = RoomDepth * EffectiveTileSize
RoomSizeZ = WallHeightTiles * WallTileHeight

ExtentX = RoomSizeX * 0.5 + BoundsPadding
ExtentY = RoomSizeY * 0.5 + BoundsPadding
ExtentZ = RoomSizeZ * 0.5 + BoundsPaddingZ

RelativeZ = RoomSizeZ * 0.5
```

### Edge columns

```text
Inicio = TileSize * -0.5
Final = ((LengthTiles - 1) * TileSize) + (TileSize * 0.5)
```

### Paredes

```text
South:
X=0
Y=EffectiveTileSize*-0.5

North:
X=0
Y=RoomDepth*EffectiveTileSize + EffectiveTileSize*-0.5

West:
X=EffectiveTileSize*-0.5
Y=0
Yaw=90

East:
X=RoomWidth*EffectiveTileSize + EffectiveTileSize*-0.5
Y=0
Yaw=90
```

## Sistemas marcados como no tocar

```text
🛑 GetNeighborCoord
🛑 GetOppositeDirection
🛑 IsCellOccupied
🛑 FindCellIndexByCoord
🛑 SetConnectionOnCell actual
🛑 BuildDungeonLayout base
🛑 SpawnCorridorsFromConnections
🛑 BP_Corridor_Debug mientras funciona
🛑 ChooseKeyAndBossCells estable
🛑 Boss dead-end
🛑 SpawnBossRoomDoors
🛑 SpawnDoorAtRoomDirection y rotaciones
🛑 RoomContentRoot
🛑 CenterRoomContentOnActorOrigin
🛑 fórmulas físicas de paredes
🛑 mappings finales
🛑 GetRoomBoundsData
🛑 correspondencia de índices
```

Modificar solo con:

```text
bug demostrado
→ captura
→ copia/commit previo
→ cambio aislado
→ prueba de regresión
```

## Decisiones de alcance

### Sin PCG

```text
No usar PCG ni plugin procedural.
```

Motivo: mantener control y Blueprints comprensibles.

### Pasillos siempre presentes en V1

```text
Sí: corto, medio o largo.
```

### Pared con pared

```text
Pospuesto.
```

### Level Instances

```text
No migrar toda la mazmorra.
```

Motivos:

- no garantizan mejor rendimiento;
- añaden streaming y carga asíncrona;
- actor + HISM encaja mejor con salas variables;
- 10–50 salas no justifican refactor sin mediciones.

### Child Actors

```text
Aceptables por ahora.
```

Medir antes de refactorizar.

### Regeneración durante colocación

```text
No regenerar HISM por cada intento.
```

Spawn una vez y mover.

### Inventario temporal

```text
Se mantiene como prueba del loop.
No representa el inventario final.
```

## Decisiones de documentación

### Jerarquía de verdad

```text
1. Estado actual del repositorio.
2. Capturas/pruebas actuales.
3. Documentación estructurada.
4. Documentación histórica.
```

### Regla de nuevo chat

Si falta el gráfico exacto:

```text
No asumir.
Pedir captura.
```

### Nombres reales

No crear variables conceptuales como si existieran:

```text
BaseCellIndex
ChosenDirection
```

Equivalentes reales:

```text
Random Cell Index
Selected Direction
```

## Errores comunes de Blueprint a vigilar

### Array index

```text
Length 1 → índice 0
Length 2 → índices 0 y 1
```

### Add ReturnValue

El `ReturnValue` de `Add DungeonCells` es el índice de la hija recién añadida.

No es el padre.

### Enum vs Integer

```text
Random Direction Index = Integer 0-3
Selected Direction = E_DungeonDirection
```

Guardar el enum final.

### Opposite Direction

Representa la conexión de la hija hacia el padre.

No representa la salida usada en el padre.

### Struct reconstruction

Al cambiar `RoomType`, conservar todos los demás campos.

### Debug print

Un resultado correcto de cantidades no garantiza por sí solo que Parent/Direction sean correctos.

## Decisión actual de debug

El debug de cantidades y links se conserva desactivado.

No borrarlo porque permite revalidar rápidamente:

```text
Cells
Links
ChildIndex
ParentCellIndex
DirectionFromParent
bHasParent
```

## Siguiente riesgo técnico

El próximo cambio importante será dividir `SpawnRoomsFromCells`.

Riesgo principal:

```text
romper la correspondencia de índices
```

Medida preventiva:

```text
implementar solo SpawnStartRoom
→ validar
→ luego una hija
```
