# 32 — Fase G: solapamiento global contra habitaciones aceptadas

**Fecha:** 2026-07-25  
**Motor:** Unreal Engine 5.4  
**Blueprint principal:** `BP_DungeonGenerator_V2`

## Estado

```text
✅ Fase G completada
```

La comprobación local padre-hija de `SpawnFirstChildRoom` fue generalizada en una función reutilizable que compara una habitación candidata contra todas las habitaciones ya aceptadas de `SpawnedRooms`.

## Función confirmada

```text
DoesRoomOverlapPlacedRooms
```

Firma:

```text
Input:
Candidate Room Actor : Actor Object Reference

Output:
Overlaps Placed Rooms : Boolean
```

La función no es Pure porque usa flujo de ejecución y recorre `SpawnedRooms`.

## Variables locales confirmadas

```text
Candidate Bounds Center : Vector
Candidate Bounds Extent : Vector
Placed Room Actor       : Actor Object Reference
Placed Bounds Center    : Vector
Placed Bounds Extent    : Vector
Found Overlap           : Boolean
```

## Flujo confirmado

```text
Candidate Room Actor
→ Is Valid
```

Si la candidata no es válida:

```text
Return False
```

Si es válida:

```text
Get Room Bounds Data
→ Candidate Bounds Center
→ Candidate Bounds Extent
→ Found Overlap = False
→ For Each Loop with Break sobre SpawnedRooms
```

Por cada elemento:

```text
Array Element
→ Is Valid
→ Array Element != Candidate Room Actor
→ Set Placed Room Actor
→ Get Room Bounds Data
→ Placed Bounds Center
→ Placed Bounds Extent
```

Las referencias inválidas y la propia candidata se ignoran terminando esa iteración.

## Fórmula AABB reutilizada

Por cada habitación aceptada:

```text
OverlapX = Abs(CandidateCenterX - PlacedCenterX)
           <= CandidateExtentX + PlacedExtentX

OverlapY = Abs(CandidateCenterY - PlacedCenterY)
           <= CandidateExtentY + PlacedExtentY

OverlapZ = Abs(CandidateCenterZ - PlacedCenterZ)
           <= CandidateExtentZ + PlacedExtentZ

CurrentOverlap = OverlapX AND OverlapY AND OverlapZ
```

Si `CurrentOverlap == True`:

```text
Found Overlap = True
→ Break
```

Al terminar el loop:

```text
Completed
→ Return Found Overlap
```

## Integración activa en SpawnFirstChildRoom

Flujo final confirmado:

```text
Set Actor Location
→ DoesRoomOverlapPlacedRooms
   Candidate Room Actor = Child Room Actor
→ Set Bounds Overlap
→ Branch de reintentos existente
```

Dato activo:

```text
Overlaps Placed Rooms
→ Bounds Overlap
```

El bloque AABB local duplicado de `SpawnFirstChildRoom` fue retirado después de validar que la función nueva daba el mismo resultado.

También se eliminaron de `SpawnFirstChildRoom`, al quedar sin referencias:

```text
Parent Bounds Center
Parent Bounds Extent
Child Bounds Center
Child Bounds Extent
```

Se conserva:

```text
Bounds Overlap
```

porque sigue alimentando el `Branch` de reintentos.

## Pruebas validadas

### Caso libre

Valores normales:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

Resultado:

```text
Global Overlap = False
Distance = 1000
SpawnedRooms Length = 2
sin FAILED
```

### Fallo controlado

Valores de prueba:

```text
Corridor Length = -500
Placement Retry Step = 0
Max Placement Attempts = 3
```

Resultado:

```text
Global Overlap = True
Global Overlap = True
Global Overlap = True
FAILED: Max Placement Attempts reached
```

La candidata se destruye y no se añade a `SpawnedRooms`.

### Regresión determinista vigente

```text
Seed 12345 → South
Seed 12346 → East
```

## Invariantes conservados

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

Además:

```text
No SpawnActor adicional durante reintentos
No Init Room from Cell adicional
No regeneración HISM durante reintentos
La candidata se añade solo después del éxito
```

## Siguiente fase

```text
Fase H — generalizar la colocación para Child Index 1, 2 y después todas las hijas
```

No se borran todavía:

```text
SpawnFirstChildRoom
SpawnRoomsFromCells
```

La limpieza final se realizará únicamente cuando el nuevo sistema genere y valide toda la mazmorra.
