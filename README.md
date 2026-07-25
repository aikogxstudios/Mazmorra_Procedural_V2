# Mazmorra Procedural V2 — Base técnica viva

Documentación técnica central de la **Mazmorra Procedural V2** de **Caos Entre Reinos / Chaos Among Realms: Reborn**, desarrollada en **Unreal Engine 5.4** principalmente con **Blueprints**.

Este repositorio es la memoria técnica oficial del sistema.

## Fuente de verdad

Orden de prioridad:

1. Capturas y pruebas actuales de Unreal Engine.
2. `docs/00_ESTADO_ACTUAL.md`.
3. Documentación técnica estructurada.
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

## Estado operativo actual

### Resultado confirmado

```text
Max Rooms = 10
→ 10 Normal
+ 1 Start
+ 1 Key
+ 1 Boss
= 13 habitaciones físicas
```

```text
KEY SPECIAL ADDED
BOSS SPECIAL ADDED
Total = 13 habitaciones
```

### Regla de cantidad

```text
Max Rooms = habitaciones Normal
```

Start, Key, Boss y futuros tipos especiales se añaden fuera de ese límite.

### Flujo actual

```text
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
```

### Invariante crítica

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

### Placement

```text
✅ PlaceChildRoomFromParent
✅ DoesRoomOverlapPlacedRooms
✅ reintentos controlados
✅ overlap global contra SpawnedRooms
✅ misma candidata movida sin repetir SpawnActor/Init
```

Valores de prueba:

```text
Corridor Length = 1000
Placement Retry Step = 500
Max Placement Attempts = 10
```

### Salas especiales

```text
✅ TryAddSpecialCellFromParent
✅ Key adicional
✅ Boss adicional
✅ prueba de cuatro direcciones
✅ módulo % 4
```

Clases actuales:

```text
BP_Room_PreBuilt_Base_Child_Key
BP_Room_PreBuilt_Base_Child_Boss
```

### Seeds conocidas

```text
Seed 12345 → South
Seed 12346 → East
```

### Próxima fase

Compatibilidad entre conexiones lógicas y puertas físicas de habitaciones prefabricadas.

```text
Dead End
Straight
Corner/L
T
Cross
```

Las prebuilt no podrán inventar puertas. El generador deberá elegir clase y rotación compatibles.

## Arquitectura híbrida

```text
Habitaciones procedurales comunes
→ BP_RoomMaster_Dungeon
→ HISM
→ RoomBounds dinámico
→ 1–4 conexiones
→ abre las necesarias y tapa las no usadas
```

```text
Habitaciones preconstruidas
→ BP_Room_PreBuilt_Base y Blueprints hijos
→ RoomBounds manual
→ DoorPoints reales
→ patrón fijo de puertas
→ Level Instance o Packed Level Blueprint futuro
```

## Índice principal

| Archivo | Contenido |
|---|---|
| [`docs/00_ESTADO_ACTUAL.md`](docs/00_ESTADO_ACTUAL.md) | Estado confirmado y punto exacto de continuación |
| [`docs/01_ARQUITECTURA_GENERAL.md`](docs/01_ARQUITECTURA_GENERAL.md) | Filosofía y responsabilidades |
| [`docs/02_DATOS_STRUCTS_ENUMS.md`](docs/02_DATOS_STRUCTS_ENUMS.md) | Structs, enums e invariantes |
| [`docs/03_BP_DUNGEON_GENERATOR_V2.md`](docs/03_BP_DUNGEON_GENERATOR_V2.md) | Generador actual |
| [`docs/04_BPI_DUNGEON_ROOM_V2.md`](docs/04_BPI_DUNGEON_ROOM_V2.md) | Contrato común de habitaciones |
| [`docs/05_BP_ROOMMASTER_DUNGEON.md`](docs/05_BP_ROOMMASTER_DUNGEON.md) | Habitación procedural integrada |
| [`docs/09_COLOCACION_PADRE_HIJA.md`](docs/09_COLOCACION_PADRE_HIJA.md) | Arquitectura padre-hija |
| [`docs/10_PRUEBAS_Y_REGRESION.md`](docs/10_PRUEBAS_Y_REGRESION.md) | Matriz de pruebas |
| [`docs/13_ROADMAP.md`](docs/13_ROADMAP.md) | Orden de implementación |
| [`docs/14_PROMPT_TRASPASO.md`](docs/14_PROMPT_TRASPASO.md) | Prompt para continuar en otro chat |
| [`docs/29_BP_ROOM_PREBUILT_BASE.md`](docs/29_BP_ROOM_PREBUILT_BASE.md) | Base de habitaciones prebuilt |
| [`docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md`](docs/31_GENERACION_COMPLETA_Y_SALAS_ESPECIALES.md) | Placement completo y especiales adicionales |
| [`sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md`](sessions/2026-07-25_GENERACION_COMPLETA_ESPECIALES.md) | Cierre de sesión actual |
| [`knowledge/state.yaml`](knowledge/state.yaml) | Estado machine-readable |
| [`knowledge/bp_dungeon_generator_v2.yaml`](knowledge/bp_dungeon_generator_v2.yaml) | Funciones estructuradas |
| [`knowledge/variables.yaml`](knowledge/variables.yaml) | Variables y tipos confirmados |
| [`CHANGELOG.md`](CHANGELOG.md) | Historial cronológico |

## Seguimiento

```text
Issue #1 → SpawnStartRoom completada
Issue #2 → primera hija alineada completada
Issue #3 → separación inicial completada
Issue #4 → RoomBounds/AABB completada
Issue #5 → reintentos completados dentro del refactor
Issue #7 → placement completo y especiales en progreso/documentado
```

## Principios

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

Reglas:

- Sin PCG ni plugin procedural.
- HISM para habitaciones comunes procedurales.
- Sin Tick para generación.
- Las salas se generan una vez y se mueven; no se regeneran por intento.
- Pedir captura cuando no esté visible el montaje exacto.
- No borrar funciones antiguas hasta comprobar referencias y regresión.
