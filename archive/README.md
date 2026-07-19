# Archivo histórico

Esta carpeta conserva documentos de traspaso anteriores.

## Disponible

```text
PROMPT_TRASPASO_MAZMORRA_V2_TOTAL_v7_2026-07-19.md
```

## Documento maestro v7

El contenido técnico del antiguo:

```text
Mazmorra_Procedural_V2_MASTER_TOTAL_v7_2026-07-19.md
```

fue reorganizado e integrado en:

```text
docs/00_ESTADO_ACTUAL.md
docs/01_ARQUITECTURA_GENERAL.md
docs/02_DATOS_STRUCTS_ENUMS.md
docs/03_BP_DUNGEON_GENERATOR_V2.md
docs/04_BPI_DUNGEON_ROOM_V2.md
docs/05_BP_ROOMMASTER_DUNGEON.md
docs/06_BP_ROOMMASTER_ORIGINAL.md
docs/07_ACTORES_HISM.md
docs/08_SALAS_PASILLOS_PUERTAS.md
docs/09_COLOCACION_PADRE_HIJA.md
docs/10_PRUEBAS_Y_REGRESION.md
docs/11_ERRORES_Y_DECISIONES.md
docs/12_FUNCIONES_DEBUG.md
docs/13_ROADMAP.md
```

Además, la documentación estructurada contiene cambios posteriores al v7 histórico:

```text
DungeonCellLinks validados hasta 150 salas.
Start limitada a una salida.
Identidad exacta BP_RoomMaster/BP_RoomMaster_Dungeon aclarada.
GenerateDungeon confirmado por captura.
Funciones debug aclaradas.
Siguiente paso cambiado a SpawnStartRoom.
```

Por tanto, para continuar trabajo no debe usarse el prompt histórico como estado actual. Usar:

```text
docs/00_ESTADO_ACTUAL.md
docs/14_PROMPT_TRASPASO.md
knowledge/state.yaml
```
