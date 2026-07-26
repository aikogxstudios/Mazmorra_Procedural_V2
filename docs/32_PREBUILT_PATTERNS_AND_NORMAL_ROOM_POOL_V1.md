# 32 — Patrones prebuilt y pool de habitaciones Normal V1

**Estado:** diseño aceptado, implementación parcial  
**Última actualización:** 2026-07-26

## Objetivo

Permitir que la mazmorra combine habitaciones procedurales flexibles con habitaciones prebuilt de formas físicas diferentes, sin colocar clases incompatibles ni inventar puertas que no existan.

El objetivo visual y jugable es evitar una sucesión de habitaciones cuadradas iguales y conseguir ritmos como:

```text
sala amplia
→ corredor estrecho
→ giro en L
→ sala procedural
→ bifurcación
→ transición vertical
→ sala especial
```

## Arquitectura aprobada

```text
DungeonCells y DungeonCellLinks crean primero el layout lógico
→ cada celda declara sus conexiones requeridas
→ el sistema busca habitaciones compatibles
→ selecciona una variante aleatoria
→ aplica una rotación compatible
→ coloca mediante DoorPoints
→ valida RoomBounds y overlap
```

No se cambia a un crecimiento puramente físico por módulos en esta primera versión.

## Dos familias de habitaciones Normal

### Procedural flexible

```text
BP_RoomMaster_Dungeon
```

Características:

```text
tamaño variable
puede soportar 1–4 conexiones
abre solo las paredes necesarias
cierra las no utilizadas
RoomBounds dinámico
fallback seguro
```

### Prebuilt de patrón fijo

Características:

```text
tamaño y geometría diseñados a mano
puertas físicas reales
DoorPoints manuales
RoomBounds manual
solo acepta conexiones exactamente compatibles
puede rotarse en pasos de 90 grados
```

## E_PreBuiltDoorPattern

```text
DeadEnd
Straight
Corner
TShape
Cross
```

Patrones canónicos locales:

```text
DeadEnd  → North
Straight → North + South
Corner   → North + East
TShape   → North + East + West
Cross    → North + East + South + West
```

## E_PreBuiltDoorRotation

```text
Rot0
Rot90
Rot180
Rot270
```

Rotación conceptual de direcciones:

```text
North → East → South → West → North
```

## Tablas de orientación

### DeadEnd

```text
Rot0   → North
Rot90  → East
Rot180 → South
Rot270 → West
```

### Straight

```text
Rot0   → North + South
Rot90  → East + West
Rot180 → North + South
Rot270 → East + West
```

### Corner

```text
Rot0   → North + East
Rot90  → East + South
Rot180 → South + West
Rot270 → West + North
```

### TShape

```text
Rot0   → North + East + West       (cerrado South)
Rot90  → North + East + South      (cerrado West)
Rot180 → East + South + West       (cerrado North)
Rot270 → North + South + West      (cerrado East)
```

### Cross

```text
Rot0/90/180/270
→ North + East + South + West
```

## Metadatos actuales de BP_Room_PreBuilt_Base

```text
bUseFixedDoorPattern : Boolean
DoorPattern : E_PreBuiltDoorPattern
DoorPatternBaseRotation : E_PreBuiltDoorRotation
```

Comportamiento:

```text
bUseFixedDoorPattern = false
→ habitación flexible actual
→ usa WallFill según ST_DungeonCell
```

```text
bUseFixedDoorPattern = true
→ habitación artística con puertas físicas fijas
→ se valida patrón y rotación
→ no se abren huecos nuevos
```

## Prototipo actual

```text
BP_Room_PreBuilt_Test_Corner
```

Configuración:

```text
bUseFixedDoorPattern = true
DoorPattern = Corner
DoorPatternBaseRotation = Rot180
```

Aperturas físicas observadas:

```text
South + West
```

Estado:

```text
aislada
no asignada al generador
no usada por PlaceChildRoomFromParent
```

## GetPreBuiltPatternDoors

Función creada en `BP_DungeonGenerator_V2`.

```text
Pure = true
```

Inputs:

```text
Door Pattern  : E_PreBuiltDoorPattern
Base Rotation : E_PreBuiltDoorRotation
```

Outputs:

```text
North : Boolean
East  : Boolean
South : Boolean
West  : Boolean
```

Estado:

```text
firma creada
lógica interna pendiente
```

Primer test obligatorio:

```text
Corner + Rot180
→ false, false, true, true
```

## Regla de compatibilidad exacta

Una prebuilt solo es válida cuando:

```text
PhysicalNorth == CellNorth
AND PhysicalEast == CellEast
AND PhysicalSouth == CellSouth
AND PhysicalWest == CellWest
```

No se acepta una sala con puertas adicionales aunque contenga las requeridas.

Ejemplo inválido:

```text
Celda requiere North + East
Prebuilt ofrece North + East + South
→ rechazar
```

## Pool de habitaciones Normal V1

El pool futuro no será únicamente un `Array<Actor Class>` sin información. Cada entrada deberá aportar los datos mínimos necesarios para filtrar antes de hacer spawn.

Diseño conceptual mínimo:

```text
Room Class
bIsFlexible
Door Pattern
Weight
bCanRotate
```

Campos posibles para una fase posterior:

```text
Room Category
Gameplay Type
Vertical Transition
Biome Tags
Minimum Difficulty
Maximum Difficulty
```

Los nombres y tipos exactos todavía no están implementados.

## Selección V1

```text
leer ST_DungeonCell
→ obtener North/East/South/West requeridos
→ reunir entradas compatibles
→ incluir procedurales flexibles
→ incluir prebuilt con patrón/rotación exactos
→ elegir por peso aleatorio usando RandomStream
→ SpawnActor
→ Init Room from Cell
→ placement actual
```

## Fallback

La habitación procedural flexible debe permanecer disponible para evitar que el generador falle cuando el pool no tenga una prebuilt adecuada.

```text
sin prebuilt compatible
→ usar BP_RoomMaster_Dungeon
```

## Primera prueba de variedad

Crear pocas salas antes de ampliar el sistema:

```text
1 Corner/L prebuilt
1 Straight/corredor prebuilt
1 DeadEnd prebuilt
varias procedurales de tamaño aleatorio
```

Después generar varias seeds y evaluar:

```text
ritmo
repetición visual
separación
cantidad de giros
bifurcaciones
sensación de mazmorra
```

Una TShape puede añadirse cuando exista una celda lógica de tres conexiones para probarla.

## Verticalidad futura

Las habitaciones con escaleras o rampas podrán formar parte del pool, pero requieren que el layout represente altura.

Conceptos posibles:

```text
GridZ
FloorLevel
```

Ejemplo:

```text
celda nivel 0
→ habitación de escalera ascendente
→ celda nivel 1
```

La salida elevada debe tener un DoorPoint real con Z diferente y conducir a una celda lógica situada en ese nivel.

Estado:

```text
conceptual
no implementar antes de validar el pool horizontal V1
```

## Orden de implementación

```text
1. Completar GetPreBuiltPatternDoors.
2. Probar Corner + Rot180.
3. Completar todos los patrones y rotaciones.
4. Crear comparación exacta contra ST_DungeonCell.
5. Probar rechazo de incompatibles.
6. Crear pool Normal mínimo.
7. Integrar selección compatible antes de SpawnActor.
8. Crear 2–3 prebuilt adicionales.
9. Ejecutar regresión de 13 habitaciones.
10. Evaluar visualmente varias seeds.
```

## Reglas protegidas

```text
No tocar mappings actuales de Init Room from Cell.
No tocar mappings actuales de Get Door World Location.
No integrar una prebuilt fija antes de validar sus DoorPoints y RoomBounds.
No quitar el fallback procedural.
No añadir actores decorativos a SpawnedRooms.
No aceptar coincidencias parciales de puertas.
No implementar verticalidad solo como decoración.
```
