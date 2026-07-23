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
docs/05_BP_ROOMMASTER_DUNGEON.md
docs/09_COLOCACION_PADRE_HIJA.md
docs/10_PRUEBAS_Y_REGRESION.md
docs/13_ROADMAP.md
docs/26_SPAWN_START_ROOM.md
docs/27_SPAWN_FIRST_CHILD_ROOM.md
docs/28_ROOM_BOUNDS_FIRST_CHILD.md
sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md

Jerarquía de verdad:
1. docs/00_ESTADO_ACTUAL.md
2. Capturas y pruebas actuales de Unreal
3. Documentación estructurada del repositorio
4. Documentación histórica

Distingue siempre:
✅ confirmado
🟡 aprobado conceptualmente
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 no tocar

REGLA DE COLABORACIÓN
Antes de proponer cables, pins, variables o nodos para una función no visible, pedir captura. No inventar la estructura exacta.
Explicar de dónde sale cada dato, usar nombres en inglés y trabajar una acción pequeña cada vez.

BLUEPRINTS IMPORTANTES
- BP_RoomMaster = original, no modificado.
- BP_RoomMaster_Dungeon = variante procedural integrada.
- BP_Room_PreBuilt_Base = base/controlador de habitaciones preconstruidas.
- BP_DungeonGenerator_V2 = generador actual.

INVARIANTE CRÍTICA
DungeonCells[Index] = DungeonCellLinks[Index] = SpawnedRooms[Index]
No reordenar SpawnedRooms ni insertar decoración en ese array.

ESTADO LÓGICO CONFIRMADO
- DungeonCells.Num == DungeonCellLinks.Num validado con 10, 15, 20, 50 y 150.
- Link 0: ParentCellIndex=-1 y bHasParent=false.
- Hijas: bHasParent=true, ParentCellIndex válido y menor que ChildIndex.
- Start limitada a una salida lógica.

FLUJO TEMPORAL ACTUAL
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ SpawnFirstChildRoom

El spawn antiguo, pasillos, Boss Door y debug físico completo siguen desconectados temporalmente, no eliminados.

SPAWN START
SpawnStartRoom:
- valida DungeonCells[0];
- comprueba RoomType == Start;
- usa GetActorLocation del generador;
- transform Rotation 0, Scale 1,1,1;
- valida SpawnActor Return Value;
- InitRoomFromCell(DungeonCells[0]) una sola vez;
- añade como SpawnedRooms[0].

Durante la última sesión se usó BP_Room_PreBuilt_Base como Start Room Class de prueba.

PRIMERA HIJA
SpawnFirstChildRoom trabaja solo con ChildIndex=1.

Variables locales confirmadas:
- Parent Room Actor : Actor Object Reference
- Child Room Actor : Actor Object Reference
- Child Entry Direction : E_DungeonDirection
- Parent Direction : E_DungeonDirection
- Parent Door Location : Vector
- Child Door Location : Vector
- Parent Bounds Center : Vector
- Parent Bounds Extent : Vector
- Child Bounds Center : Vector
- Child Bounds Extent : Vector
- bBoundsOverlap : Boolean

Direction From Parent sale de:
DungeonCellLinks[1]
→ Break ST_DungeonCellLink
→ Direction From Parent

Se guarda como Parent Direction.

Child Entry Direction:
Direction From Parent
→ GetOppositeDirection Pure
→ Child Entry Direction

GET DIRECTION VECTOR
GetDirectionVector es Pure.
Mapping:
North → (0,1,0)
East  → (1,0,0)
South → (0,-1,0)
West  → (-1,0,0)

SEPARACIÓN VALIDADA
Montaje actual:
Parent Direction
→ GetDirectionVector
→ Vector * Float con literal temporal 1000

DesiredChildDoor = ParentDoorLocation + DirectionVector * 1000
MoveDelta = DesiredChildDoor - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma Child Room Actor

Pruebas:
Seed 12345 → North → Distance=1000
Seed 12346 → East  → Distance=1000
SpawnedRooms.Num=2 en ambas.

Importante: 1000 sigue siendo un literal temporal. La variable formal Corridor Length se creará/confirmará en Fase F.

BP_ROOM_PREBUILT_BASE
Contrato confirmado:
- BPI_DungeonRoomV2
- Init Room from Cell
- Get Door World Location
- Get Room Bounds Data
- DoorPoint_North/East/South/West
- RoomBounds

Get Room Bounds Data:
RoomBounds
→ Get Component Bounds
   Origin → Bounds Center
   Box Extent → Bounds Extent

Bounds confirmados:
Start prebuilt: 980,980,400
Primera hija procedural: 995,995,420

La base todavía contiene componentes visuales/debug heredados del prototipo. No limpiar sin revisar dependencias.

Arquitectura aprobada:
- normales procedurales → BP_RoomMaster_Dungeon + HISM + bounds dinámicos;
- especiales prebuilt → BP_Room_PreBuilt_Base/Blueprints hijos + RoomBounds y DoorPoints manuales;
- contenido visual futuro → Level Instance o Packed Level Blueprint, después de validar el flujo.

AABB VALIDADA
Por eje:
Abs(ParentCenter - ChildCenter) <= ParentExtent + ChildExtent

bBoundsOverlap = OverlapX AND OverlapY AND OverlapZ

Error corregido:
Incorrecto: Abs(ParentCenter) - ChildCenter
Correcto: Abs(ParentCenter - ChildCenter)

Pruebas:
Valor 1000 → bBoundsOverlap=False
Valor -500 → bBoundsOverlap=True

Antes de continuar, confirmar que el valor temporal volvió a 1000.

PUNTO EXACTO DE CONTINUACIÓN — FASE F
Crear reintentos controlados solo para ChildIndex=1.

Objetivo:
1. Crear/confirmar Corridor Length : Float.
2. Crear/confirmar Placement Retry Step : Float.
3. Crear/confirmar Placement Attempt : Integer.
4. Crear/confirmar Max Placement Attempts : Integer.
5. Calcular posición candidata.
6. Mover la misma Child Room Actor.
7. Obtener Child Bounds otra vez.
8. Calcular bBoundsOverlap.
9. Si True: aumentar distancia e intentar de nuevo.
10. Si False: aceptar.
11. Detenerse si alcanza Max Placement Attempts.

Reglas:
- no volver a SpawnActor;
- no repetir Init Room from Cell;
- no regenerar HISM;
- mover la misma referencia;
- mantener SpawnedRooms.Num=2;
- probar seed 12345 North y seed 12346 East.

NO HACER TODAVÍA
- todas las hijas;
- comparación contra todo SpawnedRooms;
- pasillos visuales finales;
- regla de 18 habitaciones;
- nuevas RoomTypes;
- tamaños aleatorios completos.

MAPPINGS PROTEGIDOS
InitRoomFromCell:
North → SouthOpening
East  → WestOpening
South → NorthOpening
West  → EastOpening

GetDoorWorldLocation:
North → Arrow_Entrance_South
East  → Arrow_Exit_East
South → Arrow_Exit_North
West  → Arrow_Exit_West

Explica cada cambio nodo por nodo y prueba una fase antes de avanzar.
```

## Archivos por tema

```text
Estado actual → docs/00_ESTADO_ACTUAL.md
Generador → docs/03_BP_DUNGEON_GENERATOR_V2.md
Room procedural → docs/05_BP_ROOMMASTER_DUNGEON.md
Placement → docs/09_COLOCACION_PADRE_HIJA.md
Pruebas → docs/10_PRUEBAS_Y_REGRESION.md
Roadmap → docs/13_ROADMAP.md
Spawn Start → docs/26_SPAWN_START_ROOM.md
Primera hija → docs/27_SPAWN_FIRST_CHILD_ROOM.md
Bounds primera hija → docs/28_ROOM_BOUNDS_FIRST_CHILD.md
Cierre de sesión → sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md
```
