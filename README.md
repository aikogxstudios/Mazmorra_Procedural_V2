# Mazmorra Procedural V2 — Base técnica viva

Documentación técnica central de la **Mazmorra Procedural V2** de **Caos Entre Reinos / Chaos Among Realms: Reborn**, desarrollada en **Unreal Engine 5.4** principalmente con **Blueprints**.

Este repositorio existe para que el creador, Aiko/ChatGPT, Codex u otro chat puedan recuperar el estado real del sistema sin inventar nombres, repetir errores de orientación ni perder decisiones ya probadas.

## Fuente de verdad

Orden de prioridad cuando exista una contradicción:

1. `docs/00_ESTADO_ACTUAL.md`.
2. Capturas y pruebas actuales de Unreal Engine.
3. Documentación técnica estructurada de este repositorio.
4. Documentos históricos y diseño original.

Leyenda usada:

- **✅ Confirmado:** probado en Unreal, visible en capturas o confirmado expresamente.
- **🟡 Confirmado conceptualmente:** el diseño está aprobado, pero falta verificar algún nodo, pin o nombre exacto.
- **⚪ No visible:** no hay captura suficiente del montaje exacto.
- **⏳ Pendiente:** creado parcialmente o aún sin implementar.
- **🔮 Futuro:** reservado para más adelante.
- **🛑 No tocar:** estable; modificar solo con bug demostrado y prueba de regresión.

## Estado operativo inmediato

Ya está validado:

```text
DungeonCells.Num == DungeonCellLinks.Num
```

Pruebas superadas:

```text
10 habitaciones
15 habitaciones
20 habitaciones
50 habitaciones
150 habitaciones
```

También está validado:

```text
Start tiene una sola salida lógica.
DungeonCellLinks[0].bHasParent = false.
Todas las hijas guardan ParentCellIndex y DirectionFromParent.
ParentCellIndex < ChildIndex.
```

El debug temporal de cantidades y links se conserva, pero queda desactivado hasta necesitarlo.

### Próximo paso exacto

```text
Crear SpawnStartRoom
→ spawnear solo DungeonCells[0]
→ usar StartDebugRoomClass
→ colocar Start en el origen
→ InitRoomFromCell
→ añadirla como SpawnedRooms[0]
→ validar correspondencia de índices
```

Todavía no se debe colocar toda la mazmorra padre-hija de una vez.

## Índice de documentación

| Archivo | Contenido |
|---|---|
| [`docs/00_ESTADO_ACTUAL.md`](docs/00_ESTADO_ACTUAL.md) | Estado confirmado, punto actual y siguiente paso |
| [`docs/01_ARQUITECTURA_GENERAL.md`](docs/01_ARQUITECTURA_GENERAL.md) | Filosofía, flujo y responsabilidades |
| [`docs/02_DATOS_STRUCTS_ENUMS.md`](docs/02_DATOS_STRUCTS_ENUMS.md) | Structs, enums e invariantes |
| [`docs/03_BP_DUNGEON_GENERATOR_V2.md`](docs/03_BP_DUNGEON_GENERATOR_V2.md) | Funciones, variables y flujo completo del generador |
| [`docs/04_BPI_DUNGEON_ROOM_V2.md`](docs/04_BPI_DUNGEON_ROOM_V2.md) | Contrato común de las habitaciones |
| [`docs/05_BP_ROOMMASTER_DUNGEON.md`](docs/05_BP_ROOMMASTER_DUNGEON.md) | Variante procedural integrada en la mazmorra |
| [`docs/06_BP_ROOMMASTER_ORIGINAL.md`](docs/06_BP_ROOMMASTER_ORIGINAL.md) | Sistema procedural original no modificado |
| [`docs/07_ACTORES_HISM.md`](docs/07_ACTORES_HISM.md) | Floor, Wall, Ceiling, columnas y aperturas |
| [`docs/08_SALAS_PASILLOS_PUERTAS.md`](docs/08_SALAS_PASILLOS_PUERTAS.md) | Salas debug, pasillos, Boss Door, llave e inventario temporal |
| [`docs/09_COLOCACION_PADRE_HIJA.md`](docs/09_COLOCACION_PADRE_HIJA.md) | Arquitectura futura de DoorPoints, bounds y pasillos variables |
| [`docs/10_PRUEBAS_Y_REGRESION.md`](docs/10_PRUEBAS_Y_REGRESION.md) | Matriz de pruebas y comprobaciones |
| [`docs/11_ERRORES_Y_DECISIONES.md`](docs/11_ERRORES_Y_DECISIONES.md) | Errores históricos, soluciones y partes protegidas |
| [`docs/12_FUNCIONES_DEBUG.md`](docs/12_FUNCIONES_DEBUG.md) | Herramientas visuales y prints de diagnóstico |
| [`docs/13_ROADMAP.md`](docs/13_ROADMAP.md) | Orden de implementación aprobado |
| [`docs/14_PROMPT_TRASPASO.md`](docs/14_PROMPT_TRASPASO.md) | Prompt operativo para continuar en otro chat |
| [`docs/15_INDICE_TECNICO.md`](docs/15_INDICE_TECNICO.md) | Lista rápida de assets, funciones y variables |
| [`docs/16_EVIDENCIAS_2026-07-19.md`](docs/16_EVIDENCIAS_2026-07-19.md) | Capturas, vídeos y pruebas confirmadas de la sesión |
| [`docs/17_MANTENIMIENTO_DOCUMENTACION.md`](docs/17_MANTENIMIENTO_DOCUMENTACION.md) | Cómo actualizar la base técnica en futuras sesiones |
| [`CHANGELOG.md`](CHANGELOG.md) | Historial cronológico de cambios y validaciones |
| [`archive/PROMPT_TRASPASO_MAZMORRA_V2_TOTAL_v7_2026-07-19.md`](archive/PROMPT_TRASPASO_MAZMORRA_V2_TOTAL_v7_2026-07-19.md) | Prompt histórico v7 |

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
- HISM para suelo, paredes, columnas y techo.
- Sin Tick para la generación.
- Las salas se generan una vez y se mueven; no se regeneran por cada intento de colocación.
- `DungeonCells[Index]`, `DungeonCellLinks[Index]` y `SpawnedRooms[Index]` deben representar siempre la misma habitación.
- Las capturas actuales del Blueprint mandan sobre una deducción textual.
- Si falta información visual de una función, se pide captura antes de cambiarla.

## Proyecto

```text
Juego: Caos Entre Reinos / Chaos Among Realms: Reborn
Sistema: Aguja del Caos / Mazmorra Procedural V2
Motor: Unreal Engine 5.4
Implementación principal: Blueprints
```
