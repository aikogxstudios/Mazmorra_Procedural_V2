# 31 — Generación física completa y salas especiales adicionales

**Última actualización:** 2026-07-25  
**Estado:** implementación compilada y probada

## Objetivo alcanzado

El sistema ya no coloca únicamente la Start y la primera hija. Ahora puede recorrer todas las celdas lógicas, colocar sus actores físicos siguiendo la relación padre-hija, evitar solapamientos contra todas las habitaciones aceptadas y añadir Key/Boss como salas adicionales.

## Resultado confirmado

Con:

```text
Max Rooms = 10
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

se obtiene:

```text
1 Start
10 Normal
1 Key
1 Boss
= 13 habitaciones físicas
```

Mensajes confirmados:

```text
KEY SPECIAL ADDED
BOSS SPECIAL ADDED
Total = 13 habitaciones
```

## Nuevo significado oficial de Max Rooms

```text
Max Rooms = cantidad de habitaciones Normal
```

No cuenta:

```text
Start
Key
Boss
ni futuros Room Types especiales
```

`BuildDungeonLayout` genera:

```text
Max Rooms + 1
```

El `+1` corresponde exclusivamente a Start.

Después, `ChooseKeyAndBossCells` añade Key y Boss como nuevas celdas.

Ejemplo:

```text
Max Rooms = 10
BuildDungeonLayout termina con 11 celdas:
[0] Start
[1..10] Normal

ChooseKeyAndBossCells añade:
[11] Key
[12] Boss
```

## Invariante crítica

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

Reglas:

```text
No reordenar SpawnedRooms.
No insertar decoración en SpawnedRooms.
No eliminar índices intermedios.
DungeonCells y DungeonCellLinks deben añadir siempre un elemento por la misma habitación.
```

## Flujo actual de GenerateDungeon

```text
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ For Loop with Break
   First Index = 1
   Last Index = DungeonCells.Length - 1
   Loop Body → PlaceChildRoomFromParent(Index)
   Room Placed = False → Break
```

El antiguo `SpawnRoomsFromCells` permanece desconectado como referencia histórica. No borrarlo hasta terminar la limpieza final y comprobar que no conserva referencias necesarias.

## PlaceChildRoomFromParent

### Firma

```text
Input:
Child Index : Integer

Output:
Room Placed : Boolean
```

### Responsabilidades

```text
validar DungeonCells[Child Index]
validar DungeonCellLinks[Child Index]
obtener Parent Cell Index
obtener Parent Room Actor desde SpawnedRooms
seleccionar clase según Room Type
SpawnActor una sola vez
Init Room from Cell una sola vez
calcular Parent Direction
calcular Child Entry Direction opuesta
obtener DoorPoints reales
mover la misma Child Room Actor
comprobar colisión global
hacer reintentos controlados
aceptar y añadir a SpawnedRooms
ó destruir el actor al fallar
```

### Selección de clase

```text
Normal → habitación procedural común
Key    → BP_Room_PreBuilt_Base_Child_Key
Boss   → BP_Room_PreBuilt_Base_Child_Boss
Start  → inválido como hija
```

Los nombres visibles confirmados de los hijos prebuilt son:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

### Alineación

```text
Parent Direction = DungeonCellLinks[Child Index].DirectionFromParent
Child Entry Direction = GetOppositeDirection(Parent Direction)
```

```text
DesiredChildDoor =
ParentDoorLocation
+ GetDirectionVector(Parent Direction) * Corridor Length
```

```text
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
```

En cada reintento se vuelve a consultar `Child Door Location` desde el actor ya movido. No se reutiliza una posición de puerta obsoleta.

## Variables locales confirmadas de placement

```text
Parent Room Actor      : Actor Object Reference
Child Room Actor       : Actor Object Reference
Child Entry Direction  : E_DungeonDirection
Parent Direction       : E_DungeonDirection
Parent Door Location   : Vector
Child Door Location    : Vector
Corridor Length        : Float
Placement Attempt      : Integer
Placement Retry Step   : Float
Max Placement Attempts : Integer
bPlacement Succeeded   : Boolean
Bounds Overlap         : Boolean
```

Valores de prueba actuales:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

## Reintentos controlados

Se usa `For Loop with Break`.

```text
First Index = 0
Last Index = Max Placement Attempts - 1
```

Por intento:

```text
1. Placement Attempt = Index
2. volver a consultar Child Door Location
3. calcular la nueva posición con Corridor Length
4. mover el mismo actor
5. DoesRoomOverlapPlacedRooms(Child Room Actor)
6. si solapa: Corridor Length += Placement Retry Step
7. si no solapa: bPlacementSucceeded = true y Break
```

Si todos los intentos fallan:

```text
Print diagnóstico
Destroy Actor(Child Room Actor)
Room Placed = false
```

Reglas de rendimiento:

```text
No repetir SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover siempre la misma referencia.
```

## DoesRoomOverlapPlacedRooms

### Firma

```text
Input:
Candidate Room Actor : Actor Object Reference

Output:
Overlaps Placed Rooms : Boolean
```

### Comportamiento

```text
validar Candidate Room Actor
obtener Candidate Bounds
Found Overlap = false
recorrer SpawnedRooms con For Each Loop with Break
ignorar actores inválidos
ignorar Candidate Room Actor si aparece en el array
obtener bounds de la habitación aceptada
comparar AABB en X/Y/Z
si solapa: Found Overlap = true y Break
Completed → devolver Found Overlap
```

### Fórmula AABB

```text
OverlapX = Abs(CandidateCenterX - PlacedCenterX)
           <= CandidateExtentX + PlacedExtentX

OverlapY = Abs(CandidateCenterY - PlacedCenterY)
           <= CandidateExtentY + PlacedExtentY

OverlapZ = Abs(CandidateCenterZ - PlacedCenterZ)
           <= CandidateExtentZ + PlacedExtentZ

Overlap = OverlapX AND OverlapY AND OverlapZ
```

`<=` considera contacto exacto como overlap.

## Error de bounds de Key/Boss y solución

El fallo diagnosticado mostraba:

```text
Room Type = Key
Child Class = BP_Room_Debug_Key_C
Candidate Center = 0,0,0
Candidate Extent = 0,0,0
```

Esto hacía que una sala situada físicamente lejos continuara marcando overlap con Start.

Solución aplicada:

```text
crear Key y Boss como hijos de BP_Room_PreBuilt_Base
mantener RoomBounds
mantener DoorPoints
heredar BPI_DungeonRoomV2
cambiar únicamente material/nombre visual de debug
```

Resultado:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

Ambas devuelven bounds válidos y pueden ser colocadas por la función general.

## TryAddSpecialCellFromParent

### Firma

```text
Inputs:
Parent Cell Index : Integer
Special Room Type : E_DungeonRoomType

Outputs:
bAdded        : Boolean
New Cell Index: Integer
```

Todos los retornos de fallo usan:

```text
bAdded = false
New Cell Index = -1
```

El índice `0` nunca representa error porque pertenece a Start.

### Validaciones

```text
DungeonCells no vacío
Parent Cell Index válido
Special Room Type != Start
Special Room Type != Normal
```

### Responsabilidad

```text
usar el padre indicado
buscar una coordenada vecina libre
actualizar la conexión del padre
crear la nueva ST_DungeonCell con Special Room Type
crear conexión opuesta de la nueva celda
añadir DungeonCells
crear ST_DungeonCellLink
añadir DungeonCellLinks
retornar el índice del Add de DungeonCells
```

El `New Cell Index` se toma oficialmente del `Return Value` de:

```text
DungeonCells → Add
```

No del Add de `DungeonCellLinks`.

### Prueba de cuatro direcciones

La función elige un inicio aleatorio entre 0 y 3 y prueba las cuatro direcciones sin repetir:

```text
(Direction Start Index + For Loop.Index) % 4
```

```text
For Loop
First Index = 0
Last Index = 3
```

Mapping del Select:

```text
0 = North
1 = East
2 = South
3 = West
```

Si una dirección está ocupada:

```text
termina esa iteración
→ prueba la siguiente
```

Si encuentra una libre:

```text
añade la sala
→ Return true
```

Si las cuatro están ocupadas:

```text
For Loop.Completed
→ Return false, -1
```

Error corregido durante el montaje:

```text
INCORRECTO: división entre 4
CORRECTO: módulo % 4
```

## ChooseKeyAndBossCells

La búsqueda de candidatos padre se conserva:

```text
Boss Cell Index = padre normal elegido para Boss
Key Cell Index  = padre normal elegido para Key
```

Estos nombres son históricos. Ya no significan que esas celdas se conviertan en Boss/Key.

Los antiguos `Set Array Elem` que cambiaban `Room Type` quedaron desconectados temporalmente.

Al terminar las búsquedas se ejecuta un `Sequence`:

```text
Then 0
→ TryAddSpecialCellFromParent(
     Parent = Key Cell Index,
     Type = Key
   )

Then 1
→ TryAddSpecialCellFromParent(
     Parent = Boss Cell Index,
     Type = Boss
   )
```

Key se añade primero. Boss comprueba ocupación después de que Key ya exista, evitando que ambas intenten ocupar la misma coordenada.

## Seeds conocidas

Corrección confirmada durante la sesión:

```text
Seed 12345 → South
Seed 12346 → East
```

No volver a documentar 12345 como North sin una nueva prueba visual que lo demuestre.

## Habitaciones prefabricadas con puertas fijas — siguiente fase

Decisión aprobada:

```text
Una habitación prebuilt no puede inventar puertas.
```

Patrones previstos:

```text
Dead End → 1 puerta
Straight → 2 puertas opuestas
Corner/L → 2 puertas contiguas
T        → 3 puertas
Cross    → 4 puertas
```

Las habitaciones procedurales seguirán siendo flexibles:

```text
BP_RoomMaster_Dungeon
→ puede recibir 1, 2, 3 o 4 conexiones
→ abre solo las necesarias
→ tapa paredes no utilizadas
```

Las habitaciones prebuilt serán estrictas:

```text
solo usan DoorPoints y huecos físicos existentes
no abren paredes nuevas
se rotan en pasos de 90 grados
se rechazan cuando su patrón no encaja
```

Ejemplo de Corner/L local `North + East`:

```text
0°   → North + East
90°  → East + South
180° → South + West
270° → West + North
```

Pendiente diseñar el dato de patrón, el selector de clase compatible y la rotación antes de `SpawnActor`.

## Limpieza pendiente

No eliminar todavía hasta terminar regresión:

```text
SpawnFirstChildRoom
SpawnRoomsFromCells
Set Array Elem antiguos de Key/Boss
Print String temporales
variables locales/debug sin referencias
```

Proceso obligatorio:

```text
Find References
→ confirmar reemplazo
→ compilar
→ probar seeds
→ borrar
→ nueva regresión
```
