# 29 — BP_Room_PreBuilt_Base

**Estado:** prototipo técnico confirmado  
**Última actualización:** 2026-07-23

## Objetivo

`BP_Room_PreBuilt_Base` será el actor controlador común para habitaciones especiales construidas a mano.

Ejemplos futuros:

```text
BP_Room_PreBuilt_Start
BP_Room_PreBuilt_Key
BP_Room_PreBuilt_Boss
BP_Room_PreBuilt_Shop
BP_Room_PreBuilt_Event
```

No sustituye a `BP_RoomMaster_Dungeon`.

```text
BP_RoomMaster_Dungeon
→ habitaciones comunes procedurales
→ tamaño variable futuro
→ geometría HISM
→ RoomBounds dinámico
```

```text
BP_Room_PreBuilt_Base
→ habitaciones especiales hechas a mano
→ tamaño definido por cada Blueprint hijo
→ RoomBounds y DoorPoints manuales
→ contenido visual mediante Level Instance o Packed Level Blueprint cuando se valide
```

## Estado actual del asset

El prototipo actual fue creado/adaptado a partir de la habitación debug Start y todavía contiene componentes visuales y de prueba.

No limpiar todavía:

```text
SM_Floor_Debug
TextRender
PointLight...
WallFill_...
DoorMarker_...
```

Antes de eliminar cualquiera, revisar si participa en:

```text
Init Room from Cell
Get Door World Location
cierre de huecos
marcadores de puertas
visualización debug
```

## Contrato obligatorio

La base implementa:

```text
BPI_DungeonRoomV2
```

Funciones visibles:

```text
Init Room from Cell
Get Door World Location
Get Room Bounds Data
```

Componentes técnicos confirmados:

```text
Scene
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
RoomBounds
```

Cada habitación prebuilt debe poder responder al mismo contrato que una habitación procedural.

## Get Room Bounds Data

Firma:

```text
Bounds Center : Vector
Bounds Extent : Vector
```

Implementación:

```text
RoomBounds
→ Get Component Bounds
   ├ Origin     → Bounds Center
   └ Box Extent → Bounds Extent
```

No usar `Sphere Radius`.

## RoomBounds

`RoomBounds` es el volumen lógico que representa el espacio físico ocupado por la habitación.

No es:

```text
la colisión del jugador
la colisión de paredes
un trigger jugable
```

Se usa para placement:

```text
¿la habitación candidata invade una habitación aceptada?
Sí → mover más lejos
No  → aceptar posición
```

### Configuración por Blueprint hijo

Cada habitación hija debe ajustar manualmente:

```text
RoomBounds.BoxExtent
RoomBounds.RelativeLocation
```

`Box Extent` representa la mitad del tamaño total.

Ejemplo:

```text
Habitación total aproximada: 6000 × 4000 × 1000
Box Extent:                3000 × 2000 × 500
Relative Location Z:       500
```

El volumen debe cubrir aproximadamente:

```text
suelo
paredes
techo
geometría decorativa grande que ocupe espacio real
```

No necesita crecer por:

```text
luces
partículas pequeñas
decoración puramente visual sin volumen importante
```

### Prototipo Start validado

Valor confirmado en runtime:

```text
Bounds Extent = X 980, Y 980, Z 400
```

El valor final de `Relative Location` debe revisarse visualmente antes de convertir esta sala en plantilla definitiva.

## DoorPoints

Cada habitación hija debe colocar manualmente:

```text
DoorPoint_North
DoorPoint_East
DoorPoint_South
DoorPoint_West
```

Los DoorPoints deben marcar el centro físico de cada posible entrada/salida.

El generador no debe usar el centro del actor para conectar habitaciones.

```text
ParentDoor = Get Door World Location(Parent Room Actor, Parent Direction)
ChildDoor  = Get Door World Location(Child Room Actor, Child Entry Direction)
```

## Integración con Level Instance

La propuesta aprobada es separar responsabilidades.

### Actor controlador

```text
BP_Room_PreBuilt_Base / Blueprint hijo
├ BPI_DungeonRoomV2
├ DoorPoints
├ RoomBounds
├ datos de la celda
├ lógica de apertura/cierre
└ referencia o contenido visual prebuilt
```

### Contenido visual

```text
Level Instance o Packed Level Blueprint
├ suelo
├ paredes
├ columnas
├ props
├ escombros
├ muebles
└ decoración estática
```

### Elementos jugables

Mantener fuera del empaquetado puramente estático cuando sea necesario:

```text
cofres
NPC
triggers
enemigos
pickups
puertas interactivas
eventos
```

## Level Instance frente a Packed Level Blueprint

### Level Instance

Adecuado cuando la habitación necesita:

```text
edición cómoda como un nivel
actores variados
lógica o elementos no puramente estáticos
```

### Packed Level Blueprint

Candidato para una habitación formada principalmente por mallas estáticas repetidas y decoración fija.

Debe validarse antes de adoptarlo como regla porque algunos actores y componentes no estáticos no se empaquetan de la misma manera.

## Flujo futuro para crear una habitación prebuilt

```text
1. Crear Blueprint hijo de BP_Room_PreBuilt_Base.
2. Añadir o asignar contenido visual prebuilt.
3. Colocar DoorPoints.
4. Ajustar RoomBounds.
5. Configurar aperturas permitidas.
6. Compilar.
7. Probar Get Door World Location.
8. Probar Get Room Bounds Data.
9. Asignar la clase al generador.
10. Probar placement y solapamiento.
```

## Reglas

```text
No añadir habitaciones decorativas a SpawnedRooms.
No usar el centro del actor como DoorPoint.
No confiar en bounds automáticos del contenido visual para el placement.
No borrar las habitaciones debug hasta sustituir sus referencias.
No convertir todas las habitaciones a Level Instance antes de validar una sola.
```

## Pruebas pendientes

```text
[ ] Crear BP_Room_PreBuilt_Start como Blueprint hijo real.
[ ] Mover la geometría debug fuera de la base común.
[ ] Verificar que propiedades heredadas pueden editarse en hijos.
[ ] Validar RoomBounds.RelativeLocation final.
[ ] Validar cuatro DoorPoints.
[ ] Integrar un Level Instance de prueba.
[ ] Confirmar que al mover el actor controlador se mueve todo el contenido.
[ ] Medir rendimiento frente a actores sueltos.
[ ] Evaluar Packed Level Blueprint para decoración estática.
```

## Estado de transición

Las habitaciones debug antiguas se conservan temporalmente porque todavía pueden estar referenciadas por:

```text
Start Room Class
Room Class
Key Debug Room Class
Boss Debug Room Class
```

Solo se borrarán cuando:

```text
las nuevas clases prebuilt/procedurales estén asignadas
→ compile
→ genere correctamente
→ DoorPoints funcionen
→ RoomBounds funcionen
→ no queden referencias
```
