# 31 — Fase F: reintentos controlados para la primera hija

**Fecha de validación:** 2026-07-25  
**Blueprint:** `BP_DungeonGenerator_V2`  
**Función:** `SpawnFirstChildRoom`  
**Alcance:** `ChildIndex = 1`

## Estado

```text
✅ Compilado
✅ Probado
✅ Éxito directo validado
✅ Reintentos validados
✅ Fallo controlado validado
✅ Issue #5 cerrada
```

## Corrección de regresión oficial

Las pruebas actuales en Unreal Engine confirman:

```text
Seed 12345 → South
Seed 12346 → East
```

Este dato reemplaza cualquier referencia anterior que indicase `North` para la seed `12345`.

## Variables locales confirmadas

```text
Corridor Length         : Float
Placement Retry Step    : Float
Placement Attempt       : Integer
Max Placement Attempts  : Integer
Placement Succeeded     : Boolean
```

Valores normales:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
Placement Succeeded = False al comenzar
```

## Flujo confirmado

La habitación hija se genera e inicializa una sola vez:

```text
SpawnActor Child Room
→ Init Room from Cell
```

Después se ejecuta:

```text
For Loop with Break
First Index = 0
Last Index = Max Placement Attempts - 1
```

Cada intento hace:

```text
Index
→ Placement Attempt
→ Get Door World Location de Child Room Actor
→ Child Door Location
→ calcular Desired Child Door
→ calcular Move Delta
→ Set Actor Location sobre la misma Child Room Actor
→ obtener Parent Bounds
→ obtener Child Bounds
→ calcular Bounds Overlap
```

La posición de la puerta hija debe consultarse dentro del loop. No se reutiliza una ubicación anterior porque la habitación cambia de posición entre intentos.

## Fórmula de colocación

```text
Desired Child Door =
Parent Door Location
+ GetDirectionVector(Parent Direction) * Corridor Length
```

```text
Move Delta = Desired Child Door - Child Door Location
```

```text
New Child Location =
Child Room Actor.GetActorLocation + Move Delta
```

## AABB confirmada

```text
OverlapX = Abs(ParentCenterX - ChildCenterX)
           <= ParentExtentX + ChildExtentX

OverlapY = Abs(ParentCenterY - ChildCenterY)
           <= ParentExtentY + ChildExtentY

OverlapZ = Abs(ParentCenterZ - ChildCenterZ)
           <= ParentExtentZ + ChildExtentZ

Bounds Overlap = OverlapX AND OverlapY AND OverlapZ
```

## Resultado cuando existe solapamiento

```text
Bounds Overlap = True
→ Corridor Length = Corridor Length + Placement Retry Step
→ terminar la rama
→ siguiente iteración del For Loop with Break
```

No se vuelve a ejecutar `SpawnActor`, `Init Room from Cell` ni la generación HISM.

## Resultado cuando la posición es válida

```text
Bounds Overlap = False
→ Placement Succeeded = True
→ Break
```

Desde `Completed`:

```text
Branch Placement Succeeded
True
→ medir distancia final
→ Add Child Room Actor a SpawnedRooms
```

Resultado:

```text
SpawnedRooms[0] = Start
SpawnedRooms[1] = primera hija
SpawnedRooms.Num = 2
```

## Fallo controlado

Desde `Completed`:

```text
Branch Placement Succeeded
False
→ Print String: FAILED: Max Placement Attempts reached
→ Destroy Actor sobre Child Room Actor
```

La candidata fallida no se añade a `SpawnedRooms` y no queda como actor huérfano.

## Prueba negativa validada

Configuración:

```text
Corridor Length = -500
Placement Retry Step = 0
Max Placement Attempts = 3
```

Resultado:

```text
True
True
True
FAILED: Max Placement Attempts reached
```

Esto confirma:

- exactamente tres intentos;
- sin loop infinito;
- la misma posición se prueba tres veces porque el incremento es cero;
- la hija se destruye al fallar;
- no se añade a `SpawnedRooms`.

## Regresión normal

Restaurar siempre:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

Pruebas oficiales:

```text
Seed 12345 → South
Seed 12346 → East
```

En ambas:

```text
Bounds Overlap final = False
Distance = 1000
SpawnedRooms.Num = 2
sin FAILED
```

## Partes protegidas

No modificar sin bug demostrado:

```text
GetOppositeDirection
GetDirectionVector
Init Room from Cell mappings
Get Door World Location mappings
fórmula AABB validada
flujo de reintentos de Fase F
```

## Sistema antiguo durante la transición

```text
SpawnRoomsFromCells
```

permanece desconectada y conservada como referencia mientras el nuevo sistema todavía no genera todas las habitaciones. No volver a conectarla al flujo actual y no borrarla hasta completar y validar el reemplazo.

## Limpieza futura acordada

Cuando el nuevo sistema genere toda la mazmorra:

```text
revisar referencias
→ borrar funciones reemplazadas
→ borrar variables sin uso
→ retirar Print Strings y debug temporal
→ compilar
→ ejecutar regresión completa
```

## Siguiente fase

```text
Fase G
→ DoesRoomOverlapPlacedRooms
→ comparar la candidata contra todas las habitaciones aceptadas en SpawnedRooms
```

Durante la Fase G todavía se mantiene `ChildIndex = 1`. La generalización de todas las hijas pertenece a una fase posterior.