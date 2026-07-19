# 17 — Mantenimiento de la base técnica

## Objetivo

Mantener el repositorio sincronizado con el Blueprint real y evitar que la documentación vuelva a quedar obsoleta.

## Al terminar cada sesión

```text
1. Revisar qué se modificó realmente en Unreal.
2. Marcar qué quedó confirmado, pendiente o descartado.
3. Actualizar docs/00_ESTADO_ACTUAL.md.
4. Actualizar el archivo del subsistema afectado.
5. Añadir una entrada a CHANGELOG.md.
6. Actualizar docs/10_PRUEBAS_Y_REGRESION.md.
7. Cambiar docs/13_ROADMAP.md si avanzó la fase.
8. Actualizar docs/14_PROMPT_TRASPASO.md.
9. Registrar nuevas capturas en docs/16_EVIDENCIAS_2026-07-19.md o crear un archivo de nueva fecha.
```

## Cuando se añade una variable

Documentar:

```text
Blueprint propietario
Nombre exacto
Tipo
Categoría
Default
Expose on Spawn / Instance Editable si aplica
Responsabilidad
Funciones que la leen
Funciones que la escriben
Estado de certeza
Prueba realizada
```

## Cuando se añade una función

Documentar:

```text
Nombre exacto
Blueprint
Inputs
Outputs
Variables locales
Orden de nodos
Funciones llamadas
Arrays modificados
Invariantes
Errores posibles
Pruebas
Estado
```

## Cuando se cambia un struct

Actualizar:

```text
docs/02_DATOS_STRUCTS_ENUMS.md
funciones que hacen Make/Break
funciones que reconstruyen el struct
pruebas de regresión
prompt de traspaso
```

Revisar especialmente `Make ST_DungeonCell` y `Make ST_DungeonCellLink`.

## Cuando se cambia un mapping de dirección

No actualizar solo una línea.

Revisar:

```text
GetNeighborCoord
GetOppositeDirection
InitRoomFromCell
GetDoorWorldLocation
SpawnCorridorsFromConnections
SpawnBossRoomDoors
SpawnDoorAtRoomDirection
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

Repetir pruebas por los cuatro lados.

## Cuando se cambia SpawnedRooms

Comprobar siempre:

```text
DungeonCells[Index]
=
DungeonCellLinks[Index]
=
SpawnedRooms[Index]
```

No aceptar una implementación que deje huecos o cambie orden sin rediseñar toda la arquitectura.

## Capturas recomendadas

Para documentar una función grande:

```text
captura general del gráfico
+
capturas de izquierda a derecha con solapamiento visual
+
panel de inputs/outputs
+
panel de variables locales
```

Para un Blueprint:

```text
panel de funciones
panel de variables
jerarquía de componentes
Class Settings / interfaces
Defaults relevantes
```

## Convención de archivos de evidencia

Recomendado:

```text
assets/evidence/YYYY-MM-DD/Blueprint_Function_01.png
```

Ejemplos:

```text
assets/evidence/2026-07-19/BP_DungeonGenerator_V2_TryAddRandomCell_01.png
assets/evidence/2026-07-19/BP_RoomMaster_Dungeon_Components.png
assets/evidence/2026-07-19/Test_Cells_50_Links_50.png
```

## Estados de certeza

Usar siempre:

```text
✅ confirmado
🟡 confirmado conceptualmente
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 no tocar
```

No convertir un concepto futuro en una variable “existente” sin captura.

## Jerarquía de verdad

```text
1. Captura y prueba actual del proyecto.
2. docs/00_ESTADO_ACTUAL.md.
3. Archivo técnico específico.
4. Documentación histórica.
```

Si `docs/00_ESTADO_ACTUAL.md` contradice una captura nueva, actualizar el documento antes de seguir.

## Regla para Aiko/ChatGPT/Codex

```text
Si no se conoce el montaje exacto:
→ decir qué falta
→ pedir captura concreta
→ no proponer un cableado inventado
```

## Commits recomendados

Mensajes simples:

```text
docs: update SpawnStartRoom state
feat: add SpawnStartRoom blueprint flow
fix: preserve SpawnedRooms index invariant
refactor: split room spawning by parent links
test: validate 150-room layout
```

## Frecuencia de consolidación

Después de cada fase grande:

```text
crear una versión consolidada
→ actualizar prompt de traspaso
→ revisar enlaces del README
```

## Checklist antes de cerrar sesión

- [ ] Compila sin errores.
- [ ] Se ejecutó prueba mínima.
- [ ] Se ejecutó prueba media.
- [ ] Se ejecutó prueba grande si aplica.
- [ ] No se rompieron mappings protegidos.
- [ ] Se actualizaron docs.
- [ ] Se actualizó changelog.
- [ ] El siguiente paso está escrito de forma exacta.
