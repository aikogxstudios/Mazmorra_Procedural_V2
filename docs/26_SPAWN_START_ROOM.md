# 26 — SpawnStartRoom

## Estado

```text
✅ Creada en BP_DungeonGenerator_V2.
✅ Compila.
✅ Probada aisladamente en Unreal Engine.
✅ Fase cerrada el 2026-07-21.
```

## Objetivo

Generar únicamente la habitación inicial antes de comenzar la colocación padre-hija del resto de la mazmorra.

La función reemplaza, para la prueba aislada, el comportamiento antiguo de `SpawnRoomsFromCells`, que generaba todas las habitaciones de una vez mediante `GridX/GridY * CellSize`.

## Firma

```text
Nombre: SpawnStartRoom
Blueprint: BP_DungeonGenerator_V2
Inputs: ninguno
Outputs: ninguno
```

## Flujo confirmado por capturas

### 1. Validar la celda inicial

```text
DungeonCells
→ Is Valid Index
   Index = 0
→ Branch
```

Si el índice no es válido, termina la función y muestra un mensaje de error de desarrollo.

### 2. Confirmar el tipo Start

```text
DungeonCells[0]
→ Break ST_DungeonCell
→ RoomType == Start
→ Branch
```

Si la celda 0 no es Start:

```text
Print String
Dungeon V2 | ERROR: DungeonCells[0] is not Start
→ Return
```

### 3. Crear transform

```text
GetActorLocation(self)
→ Make Transform.Location
```

Valores confirmados:

```text
Location = BP_DungeonGenerator_V2.GetActorLocation
Rotation = 0,0,0
Scale = 1,1,1
```

Corrección realizada durante el montaje:

```text
Scale Z estaba en 0
→ corregido a 1
```

La Start nace donde esté colocado el generador, no obligatoriamente en el origen mundial.

### 4. Spawnear la Start

```text
Start Room Class / StartDebugRoomClass
→ SpawnActor.Class

Make Transform
→ SpawnActor.Spawn Transform
```

Configuración visible:

```text
Collision Handling Override = Default
```

No cambiar todavía este ajuste sin una prueba específica.

### 5. Validar el actor

```text
SpawnActor.ReturnValue
→ Is Valid.Input Object
```

Si no es válido:

```text
Print String
Dungeon V2 | ERROR: Failed to spawn Start Room
→ Return
```

### 6. Inicializar y guardar

```text
Is Valid
→ Init Room from Cell
   Target = SpawnActor.ReturnValue
   Cell Data = DungeonCells[0]
→ SpawnedRooms.Add(SpawnActor.ReturnValue)
```

## Invariantes

```text
DungeonCells[0]
=
DungeonCellLinks[0]
=
SpawnedRooms[0]
```

Significado:

```text
DungeonCells[0]      → datos lógicos de Start
DungeonCellLinks[0]  → Start no tiene padre
SpawnedRooms[0]      → actor físico de Start
```

## Prueba aislada superada

Resultado confirmado por el usuario:

```text
✅ Todo compila.
✅ Solo aparece una habitación.
✅ SpawnedRooms.Num == 1.
✅ SpawnedRooms[0] es válido.
✅ La habitación corresponde a Start.
✅ La Start conserva una única apertura.
✅ No aparecen hijas.
✅ No aparecen pasillos ni Boss Door durante la prueba aislada.
```

## Partes protegidas

```text
🛑 No volver a usar DebugRoomClass para la Start.
🛑 No poner Scale Z en 0.
🛑 No añadir el Return Value a SpawnedRooms sin validarlo.
🛑 No ejecutar InitRoomFromCell más de una vez.
🛑 No reordenar SpawnedRooms.
```

## Siguiente fase

```text
Fase D — Una sola hija
```

Objetivo:

```text
ChildIndex = 1
→ leer DungeonCellLinks[1]
→ obtener ParentCellIndex
→ usar SpawnedRooms[ParentCellIndex]
→ spawnear una sola hija
→ InitRoomFromCell(DungeonCells[1])
→ alinear DoorPoint de padre e hija
→ guardar hija como SpawnedRooms[1]
```

Todavía no incluir:

```text
bounds globales
reintentos por solapamiento
loop de todas las hijas
pasillos variables
regeneración repetida de HISM
```