# Sesión 2026-07-23 — RoomBounds y solapamiento validados

## Estado previo

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
CorridorLength temporal = 1000
```

Pruebas de separación ya confirmadas:

```text
Seed 12345 → North → Distance = 1000
Seed 12346 → East  → Distance = 1000
```

## BP_Room_PreBuilt_Base

Se creó una base compatible con `BPI_DungeonRoomV2` para habitaciones preconstruidas.

`Get Room Bounds Data` quedó implementada así:

```text
RoomBounds
→ Get Component Bounds
   ├ Origin     → Bounds Center
   └ Box Extent → Bounds Extent
```

La habitación Start de prueba ajustó manualmente su `RoomBounds` a:

```text
Box Extent = X 980, Y 980, Z 400
Relative Location Z = 400
```

Resultado validado en ejecución:

```text
Parent Bounds Extent = X 980, Y 980, Z 400
```

La primera hija procedural devolvió:

```text
Child Bounds Extent = X 995, Y 995, Z 420
```

## Variables locales añadidas en SpawnFirstChildRoom

```text
Parent Bounds Center : Vector
Parent Bounds Extent : Vector
Child Bounds Center : Vector
Child Bounds Extent : Vector
bBoundsOverlap : Boolean
```

## Comparación AABB

La comprobación validada por cada eje es:

```text
X = Abs(ParentCenterX - ChildCenterX)
    <= ParentExtentX + ChildExtentX

Y = Abs(ParentCenterY - ChildCenterY)
    <= ParentExtentY + ChildExtentY

Z = Abs(ParentCenterZ - ChildCenterZ)
    <= ParentExtentZ + ChildExtentZ

bBoundsOverlap = X AND Y AND Z
```

### Error corregido

La primera versión colocó `ABS` antes de la resta:

```text
Abs(ParentCenter) - ChildCenter
```

Eso producía un falso positivo.

Se corrigió a:

```text
Abs(ParentCenter - ChildCenter)
```

## Pruebas confirmadas

Caso libre:

```text
CorridorLength = 1000
bBoundsOverlap = False
```

Caso forzado de colisión:

```text
CorridorLength = -500
bBoundsOverlap = True
```

Después de la prueba, el valor temporal debe volver a:

```text
CorridorLength = 1000
```

## Decisión de arquitectura

```text
Habitaciones procedurales comunes
→ BP_RoomMaster_Dungeon
→ RoomBounds dinámico

Habitaciones especiales preconstruidas
→ BP_Room_PreBuilt_Base y Blueprints hijos
→ contenido visual futuro mediante Level Instance o Packed Level Blueprint cuando se valide
→ RoomBounds ajustado manualmente en cada habitación hija
```

Las habitaciones debug antiguas se mantienen solo durante la transición hasta sustituir todas sus referencias.

## Siguiente fase

Fase F — reintentos controlados para la primera hija:

```text
Si bBoundsOverlap == True
→ aumentar CorridorLength con PlacementRetryStep
→ mover la misma Child Room Actor
→ volver a obtener Child Bounds
→ repetir
```

Reglas obligatorias:

```text
No volver a SpawnActor.
No repetir InitRoomFromCell.
No regenerar HISM.
Usar MaxPlacementAttempts para evitar loops infinitos.
Seguir trabajando solo con ChildIndex = 1.
```
