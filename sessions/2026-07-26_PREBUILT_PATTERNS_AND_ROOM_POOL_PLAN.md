# Cierre de sesión — Patrones prebuilt y plan del pool de habitaciones

**Fecha:** 2026-07-26  
**Proyecto:** Caos Entre Reinos / Chaos Among Realms: Reborn  
**Sistema:** Aguja del Caos / Mazmorra Procedural V2  
**Motor:** Unreal Engine 5.4  
**Implementación:** Blueprints

## Objetivo de la sesión

Empezar la compatibilidad de puertas físicas para habitaciones construidas a mano, sin modificar todavía la generación activa ni romper las habitaciones flexibles actuales de Start, Key y Boss.

## Trabajo realizado

### 1. Enum de patrón físico

Creado:

```text
E_PreBuiltDoorPattern
```

Entradas:

```text
DeadEnd
Straight
Corner
TShape
Cross
```

Significado:

```text
DeadEnd  → 1 puerta
Straight → 2 puertas opuestas
Corner   → 2 puertas contiguas
TShape   → 3 puertas
Cross    → 4 puertas
```

### 2. Metadatos añadidos a BP_Room_PreBuilt_Base

Variables creadas:

```text
bUseFixedDoorPattern
Type: Boolean
Category: PreBuilt Doors
Default: false
```

```text
DoorPattern
Type: E_PreBuiltDoorPattern
Category: PreBuilt Doors
Default: DeadEnd
```

Regla protegida:

```text
bUseFixedDoorPattern = false
→ conserva el comportamiento flexible actual mediante WallFill
→ Start, Key y Boss siguen funcionando como antes
```

```text
bUseFixedDoorPattern = true
→ la habitación declara puertas físicas fijas
→ no podrá inventar nuevas aperturas
```

### 3. Habitación Corner de prueba

Creada como hija de `BP_Room_PreBuilt_Base`:

```text
BP_Room_PreBuilt_Test_Corner
```

Configuración:

```text
bUseFixedDoorPattern = true
DoorPattern = Corner
```

La habitación permanece aislada y no está asignada al generador.

### 4. Orientación física confirmada

Mediante capturas en vista superior se confirmó que las aperturas físicas de la habitación de prueba están orientadas localmente hacia:

```text
South + West
```

El patrón canónico Corner se considera:

```text
Rot0   → North + East
Rot90  → East + South
Rot180 → South + West
Rot270 → West + North
```

Por tanto, la habitación de prueba corresponde a:

```text
DoorPattern = Corner
DoorPatternBaseRotation = Rot180
```

### 5. Enum de orientación base

Creado:

```text
E_PreBuiltDoorRotation
```

Entradas:

```text
Rot0
Rot90
Rot180
Rot270
```

Variable añadida a `BP_Room_PreBuilt_Base`:

```text
DoorPatternBaseRotation
Type: E_PreBuiltDoorRotation
Category: PreBuilt Doors
Default: Rot0
```

Configuración confirmada por el usuario en `BP_Room_PreBuilt_Test_Corner`:

```text
DoorPatternBaseRotation = Rot180
```

### 6. Función de conversión creada

Creada en `BP_DungeonGenerator_V2`:

```text
GetPreBuiltPatternDoors
```

Propiedad:

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

Estado actual:

```text
✅ firma creada y compilada según confirmación del usuario
⏳ gráfico interno todavía vacío
⏳ no se ha probado su resultado
```

Resultado esperado para el primer caso de prueba:

```text
Corner + Rot180
→ North = false
→ East  = false
→ South = true
→ West  = true
```

## Decisión de arquitectura aceptada

La primera versión mantendrá la estrategia:

```text
LAYOUT LÓGICO PRIMERO
→ SELECCIÓN DE HABITACIÓN COMPATIBLE DESPUÉS
→ SPAWN Y PLACEMENT AL FINAL
```

Las habitaciones Normal no usarán para siempre una única clase. Existirá un pool con variedad física real:

```text
habitaciones procedurales flexibles y de tamaño variable
habitaciones prebuilt Corner/L
habitaciones Straight
habitaciones DeadEnd
habitaciones TShape
habitaciones Cross
salas estrechas tipo pasillo
salas amplias
salas con puentes, trampas, parkour o mini combate
transiciones verticales futuras
```

La selección futura será aleatoria, pero filtrada por compatibilidad:

```text
leer conexiones requeridas por ST_DungeonCell
→ filtrar clases compatibles
→ considerar patrón y rotación
→ elegir una variante por peso aleatorio
→ colocar mediante DoorPoints
→ validar RoomBounds y overlap
```

La habitación procedural flexible funcionará como fallback seguro cuando no exista una prebuilt compatible.

## Dinamismo buscado para la Aguja del Caos

El objetivo no es solo cambiar decoración. Cada tipo de habitación debe alterar el ritmo jugable:

```text
sala procedural grande
→ corredor estrecho
→ habitación en L
→ sala rectangular
→ escalera ascendente
→ puente elevado
→ bifurcación T
→ habitación especial
```

Esto permitirá que la mazmorra deje de sentirse como una cadena de habitaciones cuadradas idénticas.

## Verticalidad futura

Se acepta conceptualmente añadir habitaciones que suben o bajan mediante escaleras, rampas, puentes o desniveles.

No está implementado todavía. Para que sea lógico y reproducible, la altura deberá existir en los datos del layout, probablemente mediante uno de estos conceptos futuros:

```text
GridZ
```

o:

```text
FloorLevel
```

Las transiciones verticales no deben ser solo decoración: la salida elevada debe conducir a una celda colocada en esa altura.

## Primera versión que se probará

Antes de diseñar un sistema enorme:

```text
1. Completar compatibilidad Pattern + Rotation.
2. Crear un pequeño grupo de habitaciones prebuilt.
3. Mantener varias habitaciones procedurales flexibles.
4. Crear un pool inicial de habitaciones Normal.
5. Generar varias seeds.
6. Evaluar visualmente si la mazmorra gana ritmo y variedad.
7. Ajustar probabilidades y tipos después de sentir la primera versión.
```

Habitaciones prebuilt iniciales previstas para la prueba:

```text
Corner/L
Straight o corredor estrecho
DeadEnd
TShape si el layout ya produce una celda compatible
```

No es obligatorio crear todos los patrones antes de probar el sistema.

## Partes que no se modificaron

```text
BP_RoomMaster_Dungeon
Init Room from Cell
Get Door World Location
Get Room Bounds Data
PlaceChildRoomFromParent
DoesRoomOverlapPlacedRooms
SpawnStartRoom
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

## Estado de seguridad

```text
✅ bUseFixedDoorPattern sigue false por defecto
✅ Key y Boss conservan comportamiento flexible
✅ la Corner de prueba no está integrada en generación
✅ no se han cambiado mappings protegidos
✅ no se han abierto puertas artificiales
```

## Plan para la próxima sesión

### Paso 1 — completar GetPreBuiltPatternDoors

Implementar el mapeo de:

```text
DoorPattern + BaseRotation
→ North/East/South/West
```

Empezar solo por `Corner`, probar `Rot180` y después completar el resto.

### Paso 2 — prueba aislada

Comprobar:

```text
Corner + Rot180
→ South + West
```

La prueba debe realizarse antes de integrar la función en el placement.

### Paso 3 — completar patrones

Añadir:

```text
DeadEnd
Straight
TShape
Cross
```

Validar las cuatro rotaciones y reconocer simetrías:

```text
Straight: Rot0 = Rot180; Rot90 = Rot270
Cross: todas las rotaciones equivalentes
```

### Paso 4 — compatibilidad contra una celda

Crear después una función separada que compare exactamente:

```text
puertas físicas rotadas de la prebuilt
=
conexiones requeridas por ST_DungeonCell
```

Una coincidencia parcial no será válida.

### Paso 5 — integración controlada

Solo después de las pruebas aisladas:

```text
seleccionar clase compatible
→ calcular rotación necesaria
→ SpawnActor
→ Init Room from Cell
→ alinear DoorPoints
→ validar overlap
```

### Paso 6 — pool Normal V1

Crear la estructura de datos mínima para combinar:

```text
Room Class
Pattern / Flexible
Weight
Can Rotate
categoría o tipo jugable básico
```

Los nombres y campos exactos se decidirán cuando la compatibilidad de puertas esté validada.

## Regla de reanudación

Al volver:

```text
Abrir BP_DungeonGenerator_V2
→ abrir GetPreBuiltPatternDoors
→ enviar captura del gráfico y firma actual
→ implementar solo el caso Corner
```

No integrar todavía `BP_Room_PreBuilt_Test_Corner` en el array de habitaciones Normal.
