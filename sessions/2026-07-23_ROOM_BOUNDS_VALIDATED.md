# Sesión 2026-07-23 — Separación, prebuilt base y RoomBounds

## Estado al comenzar

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
DoorPoint alignment base = 0.0
```

`SpawnFirstChildRoom` ya generaba e inicializaba una hija una sola vez y la movía usando su link padre.

## 1. GetDirectionVector convertida en Pure

Firma:

```text
Input : Direction        : E_DungeonDirection
Output: Direction Vector : Vector
```

Mapping confirmado:

```text
North → ( 0,  1, 0)
East  → ( 1,  0, 0)
South → ( 0, -1, 0)
West  → (-1,  0, 0)
```

La llamada aparece como nodo verde sin pins de ejecución.

## 2. Parent Direction

Se añadió una variable local:

```text
Parent Direction : E_DungeonDirection
```

Origen exacto:

```text
DungeonCellLinks[1]
→ Break ST_DungeonCellLink
→ Direction From Parent
→ Set Parent Direction
```

Se usa `Parent Direction` para desplazar la hija hacia fuera desde su padre.

No usar `Child Entry Direction` para el vector de salida.

## 3. Separación inicial de pasillo

Montaje:

```text
Parent Direction
→ GetDirectionVector
→ Vector * Float (1000 temporal)
```

```text
DesiredChildDoor =
ParentDoorLocation
+ DirectionVector * 1000
```

La resta de movimiento pasó de:

```text
ParentDoorLocation - ChildDoorLocation
```

a:

```text
DesiredChildDoor - ChildDoorLocation
```

Se mantuvo:

```text
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma Child Room Actor
```

`1000` es un literal temporal en el nodo `Vector * Float`; todavía no existe una variable formal confirmada llamada `Corridor Length`.

## 4. Pruebas de separación

Con `Use Random Seed = false`:

```text
Seed 12345 → North
Distance = 1000
SpawnedRooms Length = 2
```

```text
Seed 12346 → East
Distance = 1000
SpawnedRooms Length = 2
```

Confirmado:

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
```

No se repitió:

```text
SpawnActor
Init Room from Cell
GenerateRoom/HISM
```

## 5. BP_Room_PreBuilt_Base

Se creó/adaptó:

```text
BP_Room_PreBuilt_Base
```

Objetivo:

```text
Base/controlador de habitaciones especiales construidas a mano
```

Contrato visible:

```text
BPI_DungeonRoomV2
Init Room from Cell
Get Door World Location
Get Room Bounds Data
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
RoomBounds
```

La base todavía conserva componentes visuales/debug del prototipo Start. No se limpiaron porque primero debe revisarse qué usa `Init Room from Cell`.

## 6. Arquitectura prebuilt aprobada

```text
Habitación procedural común
→ BP_RoomMaster_Dungeon
→ HISM
→ tamaño variable futuro
→ RoomBounds dinámico
```

```text
Habitación especial prebuilt
→ BP_Room_PreBuilt_Base / Blueprint hijo
→ DoorPoints manuales
→ RoomBounds manual
→ contenido visual futuro mediante Level Instance o Packed Level Blueprint
```

Los actores jugables, triggers, NPC, cofres y puertas interactivas pueden permanecer fuera del contenido puramente empaquetado.

Las habitaciones debug antiguas se conservan durante la transición hasta sustituir sus referencias.

## 7. Get Room Bounds Data

En `BP_RoomMaster_Dungeon` ya existía:

```text
RoomBounds
→ Get Component Bounds
→ Origin / Box Extent
```

En `BP_Room_PreBuilt_Base` se implementó igual:

```text
RoomBounds
→ Get Component Bounds
   ├ Origin      → Bounds Center
   └ Box Extent  → Bounds Extent
```

## 8. Bounds confirmados

Start prebuilt de prueba:

```text
Bounds Extent = X 980, Y 980, Z 400
```

Primera hija procedural:

```text
Bounds Extent = X 995, Y 995, Z 420
```

El `Box Extent` de la Start está confirmado en runtime.

La posición relativa final del componente `RoomBounds` debe revisarse visualmente en una futura limpieza de la base; no asumirla solo a partir del Print de extent.

## 9. Variables locales de bounds

Se añadieron en `SpawnFirstChildRoom`:

```text
Parent Bounds Center : Vector
Parent Bounds Extent : Vector
Child Bounds Center  : Vector
Child Bounds Extent  : Vector
bBoundsOverlap       : Boolean
```

Flujo:

```text
Parent Room Actor
→ Get Room Bounds Data
→ Set Parent Bounds Center
→ Set Parent Bounds Extent
```

```text
Child Room Actor
→ Get Room Bounds Data
→ Set Child Bounds Center
→ Set Child Bounds Extent
```

## 10. Comparación AABB

Fórmula validada:

```text
OverlapX = Abs(ParentCenterX - ChildCenterX)
           <= ParentExtentX + ChildExtentX

OverlapY = Abs(ParentCenterY - ChildCenterY)
           <= ParentExtentY + ChildExtentY

OverlapZ = Abs(ParentCenterZ - ChildCenterZ)
           <= ParentExtentZ + ChildExtentZ

bBoundsOverlap = OverlapX AND OverlapY AND OverlapZ
```

## 11. Error corregido

Primera versión incorrecta:

```text
Abs(ParentCenter) - ChildCenter
```

Versión correcta:

```text
Abs(ParentCenter - ChildCenter)
```

El `ABS` debe ir después de la resta.

## 12. Pruebas AABB

Caso libre:

```text
Offset temporal = 1000
bBoundsOverlap = False
```

Caso forzado de colisión:

```text
Offset temporal = -500
bBoundsOverlap = True
```

Antes de retomar:

```text
restaurar/confirmar el literal temporal en 1000
```

## 13. Estado al cerrar

```text
✅ Fase D.1 completada.
✅ Fase E completada.
✅ Separación validada en North y East.
✅ Start prebuilt compatible con Get Room Bounds Data.
✅ Hija procedural compatible con Get Room Bounds Data.
✅ Caso libre y caso de colisión detectados correctamente.
✅ SpawnedRooms.Num sigue siendo 2.
```

## 14. Punto exacto para mañana — Fase F

Implementar reintentos controlados solo para:

```text
ChildIndex = 1
```

Variables previstas; confirmar nombres y tipos al crearlas:

```text
Corridor Length        : Float
Placement Retry Step   : Float
Placement Attempt      : Integer
Max Placement Attempts : Integer
```

Flujo previsto:

```text
calcular posición candidata
→ mover misma Child Room Actor
→ obtener Child Bounds
→ calcular bBoundsOverlap
```

Si `True`:

```text
Corridor Length += Placement Retry Step
Placement Attempt += 1
→ repetir
```

Si `False`:

```text
aceptar posición
→ conservar SpawnedRooms[1]
→ terminar
```

Si alcanza el máximo:

```text
imprimir error controlado
→ no entrar en loop infinito
```

Reglas obligatorias:

```text
No volver a SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Mover la misma Child Room Actor.
Mantener SpawnedRooms.Num = 2.
Probar seed 12345 North.
Probar seed 12346 East.
```

No hacer todavía:

```text
todas las hijas
comparación contra todas las salas
pasillos visuales finales
regla de 18 habitaciones
tamaños aleatorios completos
```
