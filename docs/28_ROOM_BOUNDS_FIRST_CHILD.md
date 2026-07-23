# 28 — RoomBounds de la primera hija

**Última actualización:** 2026-07-23

## Estado

```text
✅ BP_Room_PreBuilt_Base creada/adaptada.
✅ Get Room Bounds Data implementada mediante RoomBounds → Get Component Bounds.
✅ Bounds de Start prebuilt validados.
✅ Bounds de primera hija procedural validados.
✅ Comparación AABB padre-hija implementada.
✅ Caso libre devuelve False.
✅ Caso forzado de colisión devuelve True.
```

## Firma confirmada

```text
Get Room Bounds Data
Outputs:
- Bounds Center : Vector
- Bounds Extent : Vector
```

Implementación:

```text
RoomBounds
→ Get Component Bounds
   ├ Origin      → Bounds Center
   └ Box Extent  → Bounds Extent
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

El extent de la Start está confirmado por Print en runtime. La posición relativa final de `RoomBounds` debe revisarse visualmente al limpiar la base prebuilt.

## Variables locales en SpawnFirstChildRoom

```text
Parent Bounds Center : Vector
Parent Bounds Extent : Vector
Child Bounds Center  : Vector
Child Bounds Extent  : Vector
bBoundsOverlap       : Boolean
```

Flujo padre:

```text
Parent Room Actor
→ Get Room Bounds Data
→ Set Parent Bounds Center
→ Set Parent Bounds Extent
```

Flujo hija:

```text
Child Room Actor
→ Get Room Bounds Data
→ Set Child Bounds Center
→ Set Child Bounds Extent
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

El nodo `ABS` debe ir después de la resta de centros.

## Pruebas

Caso libre:

```text
Literal temporal del Vector * Float = 1000
bBoundsOverlap = False
```

Caso forzado:

```text
Literal temporal del Vector * Float = -500
bBoundsOverlap = True
```

Después de la prueba forzada:

```text
restaurar/confirmar el literal en 1000
```

Importante:

```text
Todavía no se ha confirmado una variable formal llamada Corridor Length.
1000 y -500 fueron valores escritos temporalmente en el nodo Vector * Float.
```

## Habitaciones preconstruidas

Cada Blueprint hijo de `BP_Room_PreBuilt_Base` deberá ajustar:

```text
RoomBounds.BoxExtent
RoomBounds.RelativeLocation
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
```

El contenido visual futuro puede usar `Level Instance` o `Packed Level Blueprint`, pero `RoomBounds`, DoorPoints y la interfaz permanecen en el actor controlador.

Documento dedicado:

```text
docs/29_BP_ROOM_PREBUILT_BASE.md
```

## Siguiente fase

```text
Fase F — reintentos controlados para ChildIndex 1
```

Si `bBoundsOverlap == true`:

```text
Corridor Length += Placement Retry Step
→ mover la misma Child Room Actor
→ consultar otra vez Child Bounds
→ repetir hasta quedar libre o alcanzar Max Placement Attempts
```

Reglas:

```text
No volver a SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
No generar todavía todas las hijas.
```
