# Cierre de sesión — 2026-07-21

## Resumen normal

Hoy dimos un paso enorme en el nuevo sistema físico de la mazmorra.

Empezamos revisando la función antigua que generaba todas las habitaciones mediante una cuadrícula fija. Después dejamos el flujo principal limpio para que, durante las pruebas, solo generase la habitación inicial y una única habitación hija.

Primero confirmamos que `SpawnStartRoom` debía ejecutarse directamente desde `GenerateDungeon`, no dentro del sistema antiguo de habitaciones. La Start aparece correctamente donde está colocado el generador, conserva una sola salida y ocupa el primer lugar dentro de la lista de habitaciones generadas.

Después construimos una función nueva para generar únicamente la primera habitación hija. El sistema consulta quién es su habitación padre, averigua por qué lado sale del padre y calcula por qué lado debe entrar en la hija. La habitación hija se crea una sola vez, genera su suelo, paredes, techo y columnas, y después se mueve completa hasta quedar conectada con la Start.

Durante la primera prueba visual, las dos habitaciones aparecieron juntas y con sus aperturas enfrentadas. Para comprobar que no era solo una coincidencia visual, medimos la distancia exacta entre las dos puertas.

La primera medición mostró `2950`, pero descubrimos que no era un fallo de colocación: estábamos comparando la puerta con el centro de la habitación. Corregimos la medición para comparar la puerta del padre con la puerta de la hija y el resultado fue exactamente `0.0`.

Esto confirma que el nuevo sistema ya puede colocar una habitación de tamaño diferente tomando como referencia a su padre y alineando sus puertas perfectamente, sin depender de una cuadrícula fija.

También comprobamos que existen exactamente dos habitaciones físicas:

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
```

Finalmente creamos `GetDirectionVector`, que convierte North, East, South y West en vectores. Esta función se utilizará para alejar la habitación hija de su padre y dejar espacio para un pasillo.

## Trabajo técnico completado

```text
✅ GenerateDungeon limpiado para la prueba controlada.
✅ SpawnStartRoom conectado después de ChooseKeyAndBossCells.
✅ SpawnFirstChildRoom creada.
✅ DungeonCells[1] validada.
✅ DungeonCellLinks[1] validada.
✅ bHasParent validado.
✅ ParentCellIndex usado para obtener SpawnedRooms[ParentCellIndex].
✅ Parent Room Actor corregido para recibir el actor real del array.
✅ Clase de la hija elegida según RoomType.
✅ Hija spawneada con Always Spawn, Ignore Collisions.
✅ Return Value de la hija validado.
✅ InitRoomFromCell(DungeonCells[1]) ejecutado una sola vez.
✅ ChildEntryDirection calculada con GetOppositeDirection.
✅ GetOppositeDirection convertida a Pure.
✅ ParentDoor obtenida con DirectionFromParent.
✅ ChildDoor obtenida con ChildEntryDirection.
✅ MoveDelta calculado correctamente.
✅ SetActorLocation mueve la misma referencia.
✅ Hija añadida como SpawnedRooms[1].
✅ SpawnedRooms.Num == 2.
✅ Error de alineación final = 0.0.
✅ GetDirectionVector creada con los cuatro mappings correctos.
```

## Fórmula validada hoy

```text
MoveDelta = ParentDoorLocation - ChildDoorLocation
```

```text
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
```

```text
SetActorLocation(ChildRoomActor, NewChildLocation)
```

Después:

```text
Distance(ChildDoorAfterMove, ParentDoorLocation) = 0.0
```

## GetDirectionVector

Mapping creado:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

La captura final todavía muestra pines blancos de ejecución. Por tanto, al retomar hay que confirmar si `Pure` está activado. El mapping está correcto.

## Flujo temporal actual

```text
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ SpawnFirstChildRoom
```

El sistema antiguo de spawn, pasillos, Boss Door y debug físico completo sigue desconectado temporalmente, pero no se ha eliminado.

## Punto exacto para continuar

### Paso 1

Abrir `GetDirectionVector` y confirmar o activar:

```text
Pure = true
```

### Paso 2

Crear una longitud temporal de prueba para el pasillo.

El nombre definitivo se confirmará al crear la variable, pero el concepto es:

```text
CorridorLength
```

### Paso 3

Calcular:

```text
DirectionVector = GetDirectionVector(DirectionFromParent)
Offset = DirectionVector * CorridorLength
DesiredChildDoor = ParentDoorLocation + Offset
```

### Paso 4

Cambiar solo la resta actual:

```text
Antes:
MoveDelta = ParentDoorLocation - ChildDoorLocation

Después:
MoveDelta = DesiredChildDoor - ChildDoorLocation
```

Mantener el resto:

```text
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma hija
```

### Paso 5

Comprobar:

```text
Distance(ParentDoor, ChildDoorAfterMove) == CorridorLength
SpawnedRooms.Num == 2
```

Probar al menos dos direcciones distintas antes de pasar a RoomBounds.

## Después de esa prueba

```text
RoomBounds
→ comprobar solapamiento
→ si choca, aumentar longitud del pasillo
→ mover la misma habitación
→ repetir con límite de intentos
```

No generar todavía todas las hijas de golpe.

## Documentación actualizada

```text
docs/00_ESTADO_ACTUAL.md
docs/10_PRUEBAS_Y_REGRESION.md
docs/13_ROADMAP.md
docs/14_PROMPT_TRASPASO.md
docs/15_INDICE_TECNICO.md
docs/27_SPAWN_FIRST_CHILD_ROOM.md
knowledge/state.yaml
knowledge/bp_dungeon_generator_v2.yaml
CHANGELOG.md
README.md
```
