# 29 — BP_Room_PreBuilt_Base

**Estado:** base técnica confirmada  
**Última actualización:** 2026-07-25

## Objetivo

`BP_Room_PreBuilt_Base` es el actor controlador común para habitaciones construidas a mano.

No sustituye a `BP_RoomMaster_Dungeon`.

```text
BP_RoomMaster_Dungeon
→ habitaciones procedurales comunes
→ HISM
→ tamaño variable
→ RoomBounds dinámico
→ puede adaptarse a 1–4 conexiones
```

```text
BP_Room_PreBuilt_Base
→ habitaciones especiales/hechas a mano
→ tamaño definido por Blueprint hijo
→ RoomBounds manual
→ DoorPoints manuales
→ patrón de puertas físicas futuro
```

## Hijos confirmados

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

Ambos se crearon como hijos de la base y cambiaron material/nombre visual de debug.

Resultado:

```text
✅ implementan el contrato heredado
✅ devuelven RoomBounds válidos
✅ pueden ser colocados por PlaceChildRoomFromParent
✅ participan en DoesRoomOverlapPlacedRooms
```

## Bug corregido

La antigua clase:

```text
BP_Room_Debug_Key_C
```

devolvía:

```text
Bounds Center = 0,0,0
Bounds Extent = 0,0,0
```

Esto provocaba falsos overlaps incluso cuando la habitación estaba lejos.

La solución fue sustituirla por un hijo real de `BP_Room_PreBuilt_Base`.

## Estado actual de la base

La base todavía contiene componentes visuales/debug heredados.

No limpiar sin revisar referencias:

```text
SM_Floor_Debug
TextRender
PointLight...
WallFill_...
DoorMarker_...
```

## Contrato obligatorio

Interfaz:

```text
BPI_DungeonRoomV2
```

Funciones:

```text
Init Room from Cell
Get Door World Location
Get Room Bounds Data
```

Componentes técnicos:

```text
Scene
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
RoomBounds
```

## Get Room Bounds Data

```text
RoomBounds
→ Get Component Bounds
   Origin      → Bounds Center
   Box Extent  → Bounds Extent
```

No usar `Sphere Radius`.

## RoomBounds

Representa el volumen lógico ocupado por la habitación para placement.

No es:

```text
colisión del jugador
trigger jugable
colisión exacta de paredes
```

Uso:

```text
DoesRoomOverlapPlacedRooms
→ comparar candidata contra SpawnedRooms aceptadas
```

Cada Blueprint hijo debe ajustar:

```text
RoomBounds.BoxExtent
RoomBounds.RelativeLocation
```

Valor de Start validado en runtime:

```text
Bounds Extent = 980,980,400
```

`RelativeLocation` final sigue pendiente de revisión visual.

## DoorPoints

Los DoorPoints marcan el centro físico real de cada puerta posible.

```text
ParentDoor = Get Door World Location(Parent Room Actor, Parent Direction)
ChildDoor  = Get Door World Location(Child Room Actor, Child Entry Direction)
```

No conectar habitaciones desde el centro del actor.

## Integración visual futura

Actor controlador:

```text
BP_Room_PreBuilt_Base / hijo
├ BPI_DungeonRoomV2
├ DoorPoints
├ RoomBounds
├ patrón de puertas
├ datos de celda
└ referencia/contenido visual
```

Contenido visual:

```text
Level Instance o Packed Level Blueprint
├ suelo
├ paredes
├ columnas
├ props
└ decoración estática
```

Elementos jugables pueden permanecer fuera del empaquetado estático:

```text
cofres
NPC
triggers
enemigos
pickups
puertas interactivas
eventos
```

## Compatibilidad de puertas — próxima fase

Regla aprobada:

```text
Una habitación prebuilt no puede inventar puertas.
```

Debe declarar las puertas físicas que realmente existen.

Patrones iniciales:

```text
Dead End → 1 puerta
Straight → 2 puertas opuestas
Corner/L → 2 puertas contiguas
T        → 3 puertas
Cross    → 4 puertas
```

Ejemplo Corner/L local:

```text
North + East
```

Rotaciones:

```text
0°   → North + East
90°  → East + South
180° → South + West
270° → West + North
```

Antes de `SpawnActor`, el generador deberá:

```text
leer conexiones requeridas de ST_DungeonCell
→ probar clases prebuilt candidatas
→ probar rotaciones 0/90/180/270
→ aceptar solo coincidencia compatible
```

Una prebuilt con dos puertas no puede utilizarse para una celda que requiere tres.

## Diferencia procedural/prebuilt

Procedural:

```text
recibe conexiones de la celda
abre solo las necesarias
tapa paredes no utilizadas
no deja huecos al vacío
```

Prebuilt:

```text
usa huecos físicos existentes
no abre paredes nuevas
no inventa DoorPoints
se rechaza si no coincide
```

## Flujo futuro para crear una prebuilt

```text
1. Crear hijo de BP_Room_PreBuilt_Base.
2. Añadir contenido visual.
3. Ajustar RoomBounds.
4. Colocar DoorPoints reales.
5. Declarar patrón de puertas.
6. Configurar rotaciones permitidas.
7. Compilar.
8. Probar Get Door World Location.
9. Probar Get Room Bounds Data.
10. Probar selección compatible y placement.
```

## Reglas

```text
No añadir decoración a SpawnedRooms.
No usar centro del actor como DoorPoint.
No confiar en bounds automáticos del contenido visual.
No borrar debug hasta sustituir referencias.
No convertir todo a Level Instance sin validar una habitación.
No declarar puertas que no existan físicamente.
```

## Pruebas pendientes

```text
[ ] Verificar RoomBounds.RelativeLocation final.
[ ] Separar geometría debug de la base.
[ ] Crear una prebuilt Dead End real.
[ ] Crear una prebuilt Corner/L real.
[ ] Validar rotaciones.
[ ] Rechazar patrón incompatible.
[ ] Integrar Level Instance de prueba.
[ ] Medir rendimiento.
```

## Limpieza

Solo borrar assets debug cuando:

```text
nuevas clases asignadas
→ compila
→ genera correctamente
→ DoorPoints funcionan
→ RoomBounds funcionan
→ no quedan referencias
```
