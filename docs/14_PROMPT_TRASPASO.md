# 14 — Prompt de traspaso actualizado

Copia este bloque al iniciar un chat nuevo.

```text
Estamos continuando la Mazmorra Procedural V2 de Caos Entre Reinos / Chaos Among Realms: Reborn en Unreal Engine 5.4, principalmente con Blueprints.

La fuente de verdad técnica es el repositorio:
aikogxstudios/Mazmorra_Procedural_V2

Lee primero:
README.md
docs/00_ESTADO_ACTUAL.md
docs/01_ARQUITECTURA_GENERAL.md
docs/03_BP_DUNGEON_GENERATOR_V2.md
docs/05_BP_ROOMMASTER_DUNGEON.md
docs/09_COLOCACION_PADRE_HIJA.md
docs/10_PRUEBAS_Y_REGRESION.md
docs/13_ROADMAP.md
docs/26_SPAWN_START_ROOM.md
docs/27_SPAWN_FIRST_CHILD_ROOM.md
sessions/2026-07-21_CIERRE_SESION.md

Consulta el resto del repositorio cuando la tarea toque ese sistema.

Jerarquía de verdad:
1. docs/00_ESTADO_ACTUAL.md
2. Capturas y pruebas actuales de Unreal
3. Documentación estructurada del repositorio
4. Documentación histórica

Distingue siempre:
✅ confirmado
🟡 confirmado conceptualmente
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 no tocar

Regla de colaboración:
Si una función o gráfico no está visible o documentado nodo por nodo, pide una captura antes de proponer cables, nodos, pins o nombres. No inventes estructura visual.

ESTADO LÓGICO CONFIRMADO
- ST_DungeonCellLink existe.
- DungeonCellLinks existe.
- CreateStartCell añade link 0 con ParentCellIndex=-1, DirectionFromParent=North y bHasParent=false.
- TryAddRandomCell añade un link por cada hija con:
  Random Cell Index → ParentCellIndex
  Selected Direction → DirectionFromParent
  bHasParent=true
- DungeonCells.Num == DungeonCellLinks.Num se validó con 10, 15, 20, 50 y 150 habitaciones.
- ParentCellIndex es válido y menor que ChildIndex.
- Start está limitada a una salida.
- El debug de cantidades/links queda desactivado pero conservado.

La condición de Start en TryAddRandomCell es:
Random Cell Index == 0
AND
SelectedCell.bNorth OR bEast OR bSouth OR bWest

True → Return Added=false
False → continuar generación.

SPAWN ANTIGUO
SpawnRoomsFromCells fue revisada mediante capturas. Su flujo antiguo es:
ForEach DungeonCells
→ Break ST_DungeonCell
→ GridX/GridY * CellSize
→ MakeTransform
→ Switch E_DungeonRoomType
→ SelectedRoomClass
→ SpawnActor
→ InitRoomFromCell
→ Add SpawnedRooms

Este sistema permanece como referencia y está temporalmente desconectado durante el refactor padre-hija.

SPAWN START CONFIRMADO
SpawnStartRoom está creada, compilada y probada:
- valida DungeonCells[0];
- comprueba RoomType == Start;
- usa GetActorLocation del generador;
- MakeTransform con Rotation 0 y Scale 1,1,1;
- SpawnActor con Start Room Class / StartDebugRoomClass;
- valida Return Value;
- InitRoomFromCell(DungeonCells[0]);
- Add SpawnedRooms.

Resultado:
- solo aparece Start;
- SpawnedRooms.Num == 1;
- SpawnedRooms[0] es válido;
- Start conserva una sola apertura.

PRIMERA HIJA CONFIRMADA
SpawnFirstChildRoom está creada y probada únicamente con ChildIndex = 1.

Validaciones:
- DungeonCells[1] válido;
- DungeonCellLinks[1] válido;
- bHasParent=true;
- ParentCellIndex válido;
- SpawnedRooms[ParentCellIndex] válido;
- Actor padre válido;
- Actor hija válido.

Variables locales:
- Parent Room Actor : Actor Object Reference, no array;
- Child Room Actor : Actor Object Reference, no array;
- Child Entry Direction : E_DungeonDirection;
- Parent Door Location : Vector;
- Child Door Location : Vector.

Direcciones:
DirectionFromParent
→ GetOppositeDirection
→ ChildEntryDirection

GetOppositeDirection se cambió a Pure.

Clase de la hija:
- Normal → clase Normal/procedural;
- Key → Key Debug Room Class;
- Boss → Boss Debug Room Class;
- Start → error y Return.

Spawn de hija:
- nace temporalmente en GetActorLocation del generador;
- Collision Handling = Always Spawn, Ignore Collisions;
- SpawnActor Return Value validado;
- InitRoomFromCell(DungeonCells[1]) una sola vez.

DoorPoints:
ParentDoor = ParentRoomActor.GetDoorWorldLocation(DirectionFromParent)
ChildDoor = ChildRoomActor.GetDoorWorldLocation(ChildEntryDirection)

Movimiento validado sin separación de pasillo:
MoveDelta = ParentDoorLocation - ChildDoorLocation
NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
SetActorLocation sobre la misma referencia
Add Child Room Actor a SpawnedRooms

Resultado confirmado:
- SpawnedRooms.Num == 2;
- SpawnedRooms[0] = Start;
- SpawnedRooms[1] = primera hija;
- la hija se mueve y no se regenera;
- distancia entre DoorPoints después de mover = 0.0.

Error histórico de esta prueba:
La primera medición mostró 2950 porque comparaba ChildDoorWorldLocation contra la ubicación/centro del actor. La medición correcta compara ChildDoorWorldLocation después del movimiento contra ParentDoorLocation.

GET DIRECTION VECTOR
GetDirectionVector está creada con:
Input: Direction : E_DungeonDirection
Output: Direction Vector : Vector

Mapping:
North → (0,1,0)
East → (1,0,0)
South → (0,-1,0)
West → (-1,0,0)

El mapping está confirmado. La captura final todavía muestra pines de ejecución, por lo que Pure no debe darse por confirmado. Al retomar, revisar la propiedad Pure y activarla si sigue desmarcada.

FLUJO TEMPORAL DE GENERATE DUNGEON
GenerateDungeon
→ Switch Has Authority
→ ResetDungeon
→ InitRandomStream
→ CreateStartCell
→ BuildDungeonLayout
→ ChooseKeyAndBossCells
→ SpawnStartRoom
→ SpawnFirstChildRoom

No conectar todavía el spawn antiguo, pasillos, Boss Door ni debug físico completo.

MAPPINGS QUE NO SE DEBEN CAMBIAR
InitRoomFromCell:
Cell North → SouthOpening
Cell East → WestOpening
Cell South → NorthOpening
Cell West → EastOpening

GetDoorWorldLocation:
Dungeon North → Arrow_Entrance_South
Dungeon East → Arrow_Exit_East
Dungeon South → Arrow_Exit_North
Dungeon West → Arrow_Exit_West

SpawnCorridorsFromConnections está validado y procesa solo North/East para evitar duplicados. No tocar.

BLUEPRINTS
- BP_RoomMaster es el original y no se ha tocado.
- BP_RoomMaster_Dungeon es el duplicado integrado.

VISIÓN V1
- Start preconstruida, cualquier tamaño y una salida.
- 15 habitaciones normales/procedurales.
- 1 Key y 1 Boss aparte de las 15 normales.
- Total físico previsto: 18 habitaciones incluyendo Start.
- La regla de cantidad está aprobada conceptualmente, no implementada todavía.
- Pasillos siempre presentes y de longitud variable.
- Colocación padre-hija mediante DoorPoints y RoomBounds.
- Si una hija choca, alargar el pasillo y mover la misma hija.
- No regenerar HISM por cada intento.

PUNTO EXACTO DE CONTINUACIÓN
1. Abrir GetDirectionVector y confirmar/activar Pure.
2. Definir un CorridorLength temporal de prueba.
3. Calcular:
   DirectionVector = GetDirectionVector(DirectionFromParent)
   DesiredChildDoor = ParentDoorLocation + DirectionVector * CorridorLength
4. Sustituir únicamente:
   MoveDelta = ParentDoorLocation - ChildDoorLocation
   por:
   MoveDelta = DesiredChildDoor - ChildDoorLocation
5. Mantener:
   NewChildLocation = ChildRoomActor.GetActorLocation + MoveDelta
   SetActorLocation sobre la misma hija.
6. Medir:
   Distance(ParentDoor, ChildDoorAfterMove) == CorridorLength
7. Confirmar SpawnedRooms.Num == 2.
8. Probar al menos dos direcciones diferentes.

No crear todavía:
- loop de todas las hijas;
- bounds globales;
- reintentos por solapamiento;
- pasillos variables completos;
- nuevas RoomTypes.

Explica cada cambio Blueprint nodo por nodo, usando nombres reales de las capturas, y prueba una sola fase antes de avanzar.
```

## Archivos por tema

```text
Generador → docs/03_BP_DUNGEON_GENERATOR_V2.md
Structs/enums → docs/02_DATOS_STRUCTS_ENUMS.md
Interfaz → docs/04_BPI_DUNGEON_ROOM_V2.md
Room procedural integrada → docs/05_BP_ROOMMASTER_DUNGEON.md
Room original → docs/06_BP_ROOMMASTER_ORIGINAL.md
HISM → docs/07_ACTORES_HISM.md
Salas/pasillos/puertas → docs/08_SALAS_PASILLOS_PUERTAS.md
Placement → docs/09_COLOCACION_PADRE_HIJA.md
Pruebas → docs/10_PRUEBAS_Y_REGRESION.md
Errores/decisiones → docs/11_ERRORES_Y_DECISIONES.md
Debug → docs/12_FUNCIONES_DEBUG.md
Roadmap → docs/13_ROADMAP.md
SpawnStartRoom → docs/26_SPAWN_START_ROOM.md
Primera hija → docs/27_SPAWN_FIRST_CHILD_ROOM.md
```
