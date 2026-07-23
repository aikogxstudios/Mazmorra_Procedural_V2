# 28 — RoomBounds de la primera hija

## Estado

```text
✅ BP_Room_PreBuilt_Base creada como base para habitaciones preconstruidas.
✅ Get Room Bounds Data implementada mediante RoomBounds → Get Component Bounds.
✅ Bounds de Start prebuilt validados.
✅ Bounds de primera hija procedural validados.
✅ Comparación AABB padre-hija implementada.
✅ Caso libre devuelve False.
✅ Caso forzado de colisión devuelve True.
```

## Bounds confirmados

Start prebuilt de prueba:

```text
Bounds Extent = X 980, Y 980, Z 400
```

Primera hija procedural:

```text
Bounds Extent = X 995, Y 995, Z 420
```

## Variables locales en SpawnFirstChildRoom

```text
Parent Bounds Center : Vector
Parent Bounds Extent : Vector
Child Bounds Center : Vector
Child Bounds Extent : Vector
bBoundsOverlap : Boolean
```

## Fórmula AABB

```text
OverlapX = Abs(ParentCenterX - ChildCenterX)
           <= ParentExtentX + ChildExtentX

OverlapY = Abs(ParentCenterY - ChildCenterY)
           <= ParentExtentY + ChildExtentY

OverlapZ = Abs(ParentCenterZ - ChildCenterZ)
           <= ParentExtentZ + ChildExtentZ

bBoundsOverlap = OverlapX AND OverlapY AND OverlapZ
```

## Error corregido

Incorrecto:

```text
Abs(ParentCenter) - ChildCenter
```

Correcto:

```text
Abs(ParentCenter - ChildCenter)
```

## Pruebas

```text
CorridorLength = 1000
→ bBoundsOverlap = False
```

```text
CorridorLength = -500
→ bBoundsOverlap = True
```

Después de la prueba forzada, restaurar:

```text
CorridorLength = 1000
```

## Habitaciones preconstruidas

Cada Blueprint hijo de `BP_Room_PreBuilt_Base` debe ajustar manualmente:

```text
RoomBounds.BoxExtent
RoomBounds.RelativeLocation
DoorPoints
```

El contenido visual futuro puede usar `Level Instance` o `Packed Level Blueprint`, pero `RoomBounds`, DoorPoints y la interfaz permanecen en el actor controlador.

## Siguiente fase

```text
Fase F — Reintentos controlados
```

Cuando `bBoundsOverlap` sea true:

```text
CorridorLength += PlacementRetryStep
→ mover la misma Child Room Actor
→ consultar de nuevo Child Bounds
→ repetir hasta no solapar o alcanzar MaxPlacementAttempts
```

No volver a generar la habitación ni repetir `InitRoomFromCell`.
