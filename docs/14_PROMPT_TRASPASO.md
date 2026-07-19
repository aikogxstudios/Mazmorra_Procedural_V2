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

Estado confirmado:
- ST_DungeonCellLink existe.
- DungeonCellLinks existe.
- CreateStartCell añade link 0 con ParentCellIndex=-1, DirectionFromParent=North y bHasParent=false.
- TryAddRandomCell añade un link por cada hija con:
  Random Cell Index → ParentCellIndex
  Selected Direction → DirectionFromParent
  bHasParent=true
- No usa ReturnValue de Add DungeonCells, Random Direction Index ni Opposite Direction.
- DungeonCells.Num == DungeonCellLinks.Num se validó con 10, 15, 20, 50 y 150 habitaciones.
- ParentCellIndex es válido y menor que ChildIndex.
- Start ya está limitada a una salida.
- El debug de cantidades/links queda desactivado pero conservado.

La condición de Start dentro de TryAddRandomCell es:
Random Cell Index == 0
AND
SelectedCell.bNorth OR bEast OR bSouth OR bWest

True → Return Added=false
False → continuar generación.

Blueprints:
- BP_RoomMaster es el original y no se ha tocado.
- BP_RoomMaster_Dungeon es el duplicado integrado.

Mappings que no se deben cambiar:

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

Visión V1:
- Start preconstruida, cualquier tamaño y una sola salida.
- Normal y Key procedurales o preconstruidas.
- Boss preconstruida, terminal y con una entrada.
- Pasillos siempre presentes y de longitud variable.
- La colocación física será padre-hija mediante DoorPoints y RoomBounds.
- Si una hija choca, se alarga el pasillo y se mueve la misma hija.
- No regenerar HISM por cada intento.

Punto exacto de continuación:
Antes de diseñar nada, pedir capturas completas de SpawnRoomsFromCells.
Después crear solamente SpawnStartRoom:
- validar DungeonCells[0]
- usar StartDebugRoomClass
- spawnear Start en el origen acordado
- InitRoomFromCell(DungeonCells[0])
- añadirla como SpawnedRooms[0]
- probar que SpawnedRooms.Num == 1

No crear todavía todas las funciones futuras ni colocar todas las hijas.

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
```
