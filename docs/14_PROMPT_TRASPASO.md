# 14 — Prompt de traspaso actualizado

Copia este bloque al iniciar un chat nuevo.

```text
Estamos continuando la Mazmorra Procedural V2 de Caos Entre Reinos / Chaos Among Realms: Reborn en Unreal Engine 5.4, principalmente con Blueprints.

La fuente de verdad técnica es:
aikogxstudios/Mazmorra_Procedural_V2

Lee primero:
README.md
docs/00_ESTADO_ACTUAL.md
docs/03_BP_DUNGEON_GENERATOR_V2.md
docs/09_COLOCACION_PADRE_HIJA.md
docs/10_PRUEBAS_Y_REGRESION.md
docs/13_ROADMAP.md
docs/29_BP_ROOM_PREBUILT_BASE.md
docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md
sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md

Jerarquía de verdad:
1. Capturas y pruebas actuales de Unreal.
2. docs/00_ESTADO_ACTUAL.md.
3. Documentación estructurada del repositorio.
4. Documentación histórica.

Leyenda:
✅ confirmado
🟡 conceptual
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 protegido

REGLA DE COLABORACIÓN
Antes de proponer cables, pins, variables o nodos para una función no visible, pedir capturas. No inventar estructuras exactas. Explicar de dónde sale cada dato, usar nombres en inglés y avanzar por pasos pequeños.

IDENTIDADES
- BP_RoomMaster = original, sin tocar.
- BP_RoomMaster_Dungeon = habitación procedural integrada.
- BP_Room_PreBuilt_Base = controlador/base de prebuilt.
- BP_DungeonGenerator_V2 = generador actual.

INVARIANTE CRÍTICA
DungeonCells[Index] = DungeonCellLinks[Index] = SpawnedRooms[Index]
No reordenar SpawnedRooms ni insertar decoración.

FLUJO ACTUAL
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ For Loop with Break
   First Index = 1
   Last Index = DungeonCells.Length - 1
   Loop Body → PlaceChildRoomFromParent(Index)
   Room Placed = False → Break

El flujo antiguo SpawnRoomsFromCells/pasillos/Boss Door/debug permanece desconectado, no eliminado.

MAX ROOMS
Max Rooms cuenta únicamente habitaciones Normal.
BuildDungeonLayout usa Max Rooms + 1 para incluir Start.
Key, Boss y futuros tipos especiales se añaden después.

Ejemplo confirmado:
Max Rooms = 10
→ 10 Normal + 1 Start + 1 Key + 1 Boss = 13 físicas.

PLACE CHILD ROOM FROM PARENT
Firma:
Input Child Index : Integer
Output Room Placed : Boolean

Responsabilidades:
- validar celda/link/padre;
- seleccionar clase Normal/Key/Boss;
- SpawnActor una sola vez;
- Init Room from Cell una sola vez;
- resolver Parent Direction;
- Child Entry Direction = opuesta;
- consultar DoorPoints;
- mover la misma candidata;
- comprobar overlap global;
- reintentar;
- añadir a SpawnedRooms o destruir al fallar.

Variables locales confirmadas:
Parent Room Actor : Actor Reference
Child Room Actor : Actor Reference
Child Entry Direction : E_DungeonDirection
Parent Direction : E_DungeonDirection
Parent Door Location : Vector
Child Door Location : Vector
Corridor Length : Float
Placement Attempt : Integer
Placement Retry Step : Float
Max Placement Attempts : Integer
bPlacement Succeeded : Boolean
Bounds Overlap : Boolean

Valores de prueba:
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10

Fórmula:
DesiredChildDoor = ParentDoorLocation + GetDirectionVector(Parent Direction) * Corridor Length
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewLocation = ChildRoomActor.GetActorLocation + MoveDelta

En cada intento hay que volver a consultar Child Door Location.

DOES ROOM OVERLAP PLACED ROOMS
Input Candidate Room Actor
Output Overlaps Placed Rooms

Recorre SpawnedRooms, ignora inválidos y la propia candidata, obtiene bounds y aplica AABB:
Abs(CandidateCenter - PlacedCenter) <= CandidateExtent + PlacedExtent
por X/Y/Z y AND final.

REINTENTOS
For Loop with Break:
First 0
Last Max Placement Attempts - 1

Overlap True:
Corridor Length += Placement Retry Step

Overlap False:
bPlacementSucceeded = true
→ Break

Fallo:
Print
→ Destroy Actor
→ Room Placed = false

TRY ADD SPECIAL CELL FROM PARENT
Inputs:
Parent Cell Index
Special Room Type

Outputs:
bAdded
New Cell Index

Rechaza Start y Normal.
Todos los fallos devuelven New Cell Index = -1.
New Cell Index sale del Add de DungeonCells.

Prueba las cuatro direcciones con:
(Direction Start Index + Loop Index) % 4

0 North
1 East
2 South
3 West

No usar división; se corrigió a módulo %.

CHOOSE KEY AND BOSS CELLS
Key Cell Index y Boss Cell Index ahora representan padres normales seleccionados.
Los Set Array Elem antiguos que convertían normales están desconectados.

Al final:
Sequence Then 0 → añadir Key
Sequence Then 1 → añadir Boss

Clases actuales:
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss

Estas clases solucionaron un bug de bounds cero de BP_Room_Debug_Key_C.

RESULTADO CONFIRMADO
KEY SPECIAL ADDED
BOSS SPECIAL ADDED
Total = 13 habitaciones

SEEDS CONOCIDAS
Seed 12345 → South
Seed 12346 → East
No documentar 12345 como North sin nueva prueba.

PRÓXIMA FASE
Compatibilidad de puertas para prebuilt.

Procedural:
- soporta 1,2,3,4 conexiones;
- abre las necesarias;
- tapa las no usadas.

Prebuilt:
- declara puertas físicas reales;
- no inventa huecos;
- puede rotar 0/90/180/270;
- solo se selecciona si el patrón encaja.

Patrones:
Dead End, Straight, Corner/L, T, Cross.

LIMPIEZA PENDIENTE
No borrar todavía:
SpawnFirstChildRoom
SpawnRoomsFromCells
Set Array Elem antiguos Key/Boss
Print String temporales
variables sin referencias

Primero Find References, compilar, probar y luego limpiar.

MAPPINGS PROTEGIDOS
InitRoomFromCell:
North → SouthOpening
East → WestOpening
South → NorthOpening
West → EastOpening

GetDoorWorldLocation:
North → Arrow_Entrance_South
East → Arrow_Exit_East
South → Arrow_Exit_North
West → Arrow_Exit_West
```

## Archivos por tema

```text
Estado actual → docs/00_ESTADO_ACTUAL.md
Generador → docs/03_BP_DUNGEON_GENERATOR_V2.md
Pruebas → docs/10_PRUEBAS_Y_REGRESION.md
Roadmap → docs/13_ROADMAP.md
Prebuilt → docs/29_BP_ROOM_PREBUILT_BASE.md
Fase completa → docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md
Cierre de sesión → sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md
```
