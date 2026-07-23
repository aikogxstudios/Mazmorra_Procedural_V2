# Mazmorra Procedural V2 — Base técnica viva

Documentación técnica central de la **Mazmorra Procedural V2** de **Caos Entre Reinos / Chaos Among Realms: Reborn**, desarrollada en **Unreal Engine 5.4** principalmente con **Blueprints**.

Este repositorio permite recuperar el estado real del sistema sin inventar nombres, repetir errores de orientación ni perder decisiones probadas.

## Fuente de verdad

Orden de prioridad:

1. `docs/00_ESTADO_ACTUAL.md`.
2. Capturas y pruebas actuales de Unreal Engine.
3. Documentación técnica estructurada del repositorio.
4. Documentación histórica.

Leyenda:

```text
✅ confirmado
🟡 aprobado conceptualmente
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 estable / no tocar sin bug demostrado
```

## Estado operativo inmediato

### Layout lógico

```text
✅ DungeonCells.Num == DungeonCellLinks.Num
✅ Validado con 10, 15, 20, 50 y 150 habitaciones
✅ Start tiene una sola salida lógica
✅ ParentCellIndex válido y menor que ChildIndex
```

### SpawnStartRoom

```text
✅ genera DungeonCells[0]
✅ valida SpawnActor
✅ InitRoomFromCell una sola vez
✅ SpawnedRooms[0] = Start
```

### SpawnFirstChildRoom

```text
✅ trabaja con ChildIndex = 1
✅ obtiene Parent Room Actor desde DungeonCellLinks[1]
✅ spawnea e inicializa una sola hija
✅ mueve la misma referencia
✅ SpawnedRooms.Num = 2
✅ SpawnedRooms[1] = primera hija
```

### Separación de pasillo

```text
✅ GetDirectionVector es Pure
✅ Seed 12345 → North → Distance 1000
✅ Seed 12346 → East  → Distance 1000
✅ SpawnedRooms.Num = 2 en ambas
```

`1000` sigue siendo un literal temporal. La variable formal `Corridor Length` queda para la Fase F.

### RoomBounds y solapamiento

```text
✅ BP_Room_PreBuilt_Base devuelve bounds reales
✅ Start prebuilt: 980,980,400
✅ Primera hija procedural: 995,995,420
✅ Comparación AABB padre-hija implementada
✅ 1000 → bBoundsOverlap=False
✅ -500 → bBoundsOverlap=True
```

### Próximo paso exacto

```text
Fase F — reintentos controlados para ChildIndex 1
```

```text
si bBoundsOverlap
→ aumentar Corridor Length
→ mover la misma Child Room Actor
→ consultar bounds otra vez
→ repetir hasta quedar libre o alcanzar Max Placement Attempts
```

Reglas:

```text
No volver a SpawnActor.
No repetir InitRoomFromCell.
No regenerar HISM.
Mantener SpawnedRooms.Num = 2.
```

## Arquitectura híbrida

```text
Habitaciones procedurales comunes
→ BP_RoomMaster_Dungeon
→ HISM
→ RoomBounds dinámico
```

```text
Habitaciones especiales preconstruidas
→ BP_Room_PreBuilt_Base y Blueprints hijos
→ RoomBounds y DoorPoints manuales
→ Level Instance o Packed Level Blueprint visual después de validación
```

## Índice principal

| Archivo | Contenido |
|---|---|
| [`docs/00_ESTADO_ACTUAL.md`](docs/00_ESTADO_ACTUAL.md) | Estado confirmado y punto exacto de continuación |
| [`docs/01_ARQUITECTURA_GENERAL.md`](docs/01_ARQUITECTURA_GENERAL.md) | Filosofía y responsabilidades |
| [`docs/02_DATOS_STRUCTS_ENUMS.md`](docs/02_DATOS_STRUCTS_ENUMS.md) | Structs, enums e invariantes |
| [`docs/03_BP_DUNGEON_GENERATOR_V2.md`](docs/03_BP_DUNGEON_GENERATOR_V2.md) | Generador |
| [`docs/04_BPI_DUNGEON_ROOM_V2.md`](docs/04_BPI_DUNGEON_ROOM_V2.md) | Contrato común de habitaciones |
| [`docs/05_BP_ROOMMASTER_DUNGEON.md`](docs/05_BP_ROOMMASTER_DUNGEON.md) | Habitación procedural integrada |
| [`docs/06_BP_ROOMMASTER_ORIGINAL.md`](docs/06_BP_ROOMMASTER_ORIGINAL.md) | Sistema original no modificado |
| [`docs/07_ACTORES_HISM.md`](docs/07_ACTORES_HISM.md) | Suelo, paredes, techo y columnas |
| [`docs/08_SALAS_PASILLOS_PUERTAS.md`](docs/08_SALAS_PASILLOS_PUERTAS.md) | Salas, pasillos y puertas |
| [`docs/09_COLOCACION_PADRE_HIJA.md`](docs/09_COLOCACION_PADRE_HIJA.md) | Arquitectura padre-hija |
| [`docs/10_PRUEBAS_Y_REGRESION.md`](docs/10_PRUEBAS_Y_REGRESION.md) | Matriz de pruebas |
| [`docs/11_ERRORES_Y_DECISIONES.md`](docs/11_ERRORES_Y_DECISIONES.md) | Errores y decisiones protegidas |
| [`docs/12_FUNCIONES_DEBUG.md`](docs/12_FUNCIONES_DEBUG.md) | Debug visual y prints |
| [`docs/13_ROADMAP.md`](docs/13_ROADMAP.md) | Orden de implementación |
| [`docs/14_PROMPT_TRASPASO.md`](docs/14_PROMPT_TRASPASO.md) | Prompt para continuar en otro chat |
| [`docs/15_INDICE_TECNICO.md`](docs/15_INDICE_TECNICO.md) | Índice rápido |
| [`docs/20_FORMULAS_Y_VALORES.md`](docs/20_FORMULAS_Y_VALORES.md) | Fórmulas protegidas |
| [`docs/23_SEEDS_Y_REPRODUCIBILIDAD.md`](docs/23_SEEDS_Y_REPRODUCIBILIDAD.md) | Seeds y determinismo |
| [`docs/26_SPAWN_START_ROOM.md`](docs/26_SPAWN_START_ROOM.md) | Spawn de Start |
| [`docs/27_SPAWN_FIRST_CHILD_ROOM.md`](docs/27_SPAWN_FIRST_CHILD_ROOM.md) | Primera hija y alineación |
| [`docs/28_ROOM_BOUNDS_FIRST_CHILD.md`](docs/28_ROOM_BOUNDS_FIRST_CHILD.md) | Bounds y comparación AABB |
| [`docs/29_BP_ROOM_PREBUILT_BASE.md`](docs/29_BP_ROOM_PREBUILT_BASE.md) | Base para habitaciones preconstruidas y Level Instance futuro |
| [`sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md`](sessions/2026-07-23_ROOM_BOUNDS_VALIDATED.md) | Cierre de sesión y punto de continuación |
| [`knowledge/state.yaml`](knowledge/state.yaml) | Estado machine-readable |
| [`knowledge/bp_dungeon_generator_v2.yaml`](knowledge/bp_dungeon_generator_v2.yaml) | Funciones del generador estructuradas |
| [`CHANGELOG.md`](CHANGELOG.md) | Historial cronológico |

## Seguimiento

```text
Issue #1 — SpawnStartRoom → completada
Issue #2 — primera hija alineada → completada
Issue #3 — separación inicial → completada
Issue #4 — RoomBounds y AABB → completada
Siguiente → Fase F, reintentos controlados
```

## Principios del sistema

```text
DATOS PRIMERO
→ SPAWN DESPUÉS
```

```text
Primero funcional.
Luego procedural.
Luego bonito.
Luego optimizado.
```

Reglas clave:

- Sin PCG ni plugin procedural.
- HISM para habitaciones comunes procedurales.
- Sin Tick para generación.
- Las salas se generan una vez y se mueven; no se regeneran por intento.
- `DungeonCells[Index]`, `DungeonCellLinks[Index]` y `SpawnedRooms[Index]` representan siempre la misma habitación.
- Pedir captura cuando no esté visible el montaje exacto.

## Proyecto

```text
Juego: Caos Entre Reinos / Chaos Among Realms: Reborn
Sistema: Aguja del Caos / Mazmorra Procedural V2
Motor: Unreal Engine 5.4
Implementación principal: Blueprints
```
