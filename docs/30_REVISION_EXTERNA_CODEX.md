# 30 — Revisión externa de Codex y decisiones del proyecto

**Fecha de la revisión:** 2026-07-23  
**Estado:** referencia externa, no fuente de verdad  
**Origen:** opinión solicitada por el usuario después de que Codex leyera el repositorio sin modificarlo

## Regla principal

Este documento conserva ideas útiles de una revisión externa.

```text
No representa funcionalidad implementada.
No sustituye docs/00_ESTADO_ACTUAL.md.
No modifica por sí solo ninguna parte estable.
```

Toda recomendación debe pasar por:

```text
captura actual
→ cambio aislado
→ compilar
→ prueba positiva y negativa
→ regresión
→ documentación como confirmado
```

## Corrección temporal de la revisión

La revisión entendió que la siguiente fase era validar `RoomBounds`. Al recibirla, el proyecto ya había avanzado un paso más:

```text
✅ Fase D.1 — separación de 1000 validada en North y East.
✅ Fase E — RoomBounds y comparación AABB validados.
⏳ Fase F — reintentos controlados para ChildIndex 1.
```

Por tanto, la revisión no cambia el punto de continuación.

## Valoración aceptada

La revisión considera acertadas estas decisiones actuales:

```text
DATOS PRIMERO
→ SPAWN DESPUÉS
```

- `DungeonCellLinks` conserva padre y dirección.
- `DungeonCells[Index]`, `DungeonCellLinks[Index]` y `SpawnedRooms[Index]` mantienen correspondencia.
- `BPI_DungeonRoomV2` desacopla el generador de cada tipo de habitación.
- Cada habitación se genera una vez y después se mueve la misma referencia.
- HISM y generación por eventos evitan reconstrucciones y Tick innecesario.
- El desarrollo por fases pequeñas y pruebas controladas debe mantenerse.

## Recomendaciones aceptadas para el backlog futuro

### 1. Validar el espacio físico de los corredores

Después de estabilizar la colocación de todas las habitaciones:

```text
nuevo corredor vs habitaciones aceptadas
nueva habitación vs corredores aceptados
nuevo corredor vs corredores aceptados, si las pruebas muestran que es necesario
```

Primera solución prevista para V1:

```text
Box Bounds alargado que represente el volumen del corredor
```

No se implementa durante la Fase F.

### 2. Distancia real de grafo para Boss y Key

La distancia Manhattan actual es estable, pero no siempre representa la distancia jugable real.

Mejora futura aceptada conceptualmente:

```text
Boss
→ dead-end con mayor distancia real desde Start mediante BFS

Key
→ habitación que produzca un desvío jugable real antes de Boss
```

Antes de sustituir `ChooseKeyAndBossCells`:

```text
crear prueba comparativa aislada
→ comparar Manhattan contra distancia de grafo
→ cambiar solo si mejora el diseño y supera regresión
```

`ChooseKeyAndBossCells` permanece protegido por ahora.

### 3. Determinismo completo

Meta futura:

```text
misma DungeonSeed
→ mismo grafo
→ mismos RoomTypes
→ mismos tamaños
→ mismas longitudes iniciales
→ mismo contenido procedural
→ mismas posiciones finales
```

No declarar la generación completamente reproducible hasta probar todos esos niveles.

### 4. Validaciones automáticas de invariantes

Después de completar el placement físico:

```text
DungeonCells.Num == DungeonCellLinks.Num == SpawnedRooms.Num
coordenadas lógicas únicas
conexiones simétricas
ParentCellIndex < ChildIndex
Start con una salida
Boss terminal
referencias físicas válidas
ningún solapamiento de habitaciones
misma seed = mismo resultado
```

Las pruebas automáticas complementarán, no sustituirán, la revisión visual.

### 5. Mejorar la verificabilidad del repositorio

Cuando el sistema físico esté estable, añadir de forma gradual:

- capturas completas de funciones críticas;
- resultados visuales de pruebas;
- GIF o vídeo corto de una generación;
- diagramas actualizados;
- exportación de Blueprint a texto cuando sea práctica y fiable.

Esto se considera documentación y portfolio, no requisito para continuar la Fase F.

### 6. Reducir contradicciones documentales

Jerarquía operativa mantenida:

```text
1. Capturas y pruebas actuales de Unreal Engine.
2. docs/00_ESTADO_ACTUAL.md.
3. knowledge/state.yaml y documentación estructurada.
4. sessions/ y documentos históricos.
```

Después de cerrar una fase, actualizar como mínimo:

```text
docs/00_ESTADO_ACTUAL.md
knowledge/state.yaml
docs/10_PRUEBAS_Y_REGRESION.md
docs/13_ROADMAP.md
CHANGELOG.md
README.md cuando cambie el próximo paso
```

Una comprobación automática de coherencia documental queda como mejora futura y no debe bloquear el desarrollo del Blueprint.

## Recomendaciones aplazadas hasta demostrar necesidad

### Lista de celdas con salidas libres

Podría reducir intentos fallidos de `TryAddRandomCell` y controlar mejor profundidad y bifurcaciones.

Decisión actual:

```text
No implementar sin medición que demuestre un problema.
```

El layout ya alcanzó 10, 15, 20, 50 y 150 habitaciones.

### Optimización adicional del layout

No refactorizar `TryAddRandomCell`, `BuildDungeonLayout` ni la selección de padres únicamente por intuición. Primero medir intentos, tiempo y fallos reales.

## Decisiones que no cambian

```text
🛑 GetNeighborCoord
🛑 IsCellOccupied
🛑 SetConnectionOnCell
🛑 BuildDungeonLayout base
🛑 ChooseKeyAndBossCells actual
🛑 mappings de InitRoomFromCell
🛑 mappings de GetDoorWorldLocation
🛑 SpawnCorridorsFromConnections
🛑 Boss Door
```

También se mantiene:

```text
No saltar de ChildIndex 1 a toda la mazmorra.
No regenerar HISM durante reintentos.
No repetir SpawnActor ni InitRoomFromCell.
No introducir BFS, corredores físicos finales o pruebas automáticas dentro de la Fase F.
```

## Punto de continuación sin cambios

```text
Fase F — reintentos controlados para ChildIndex 1
```

Preflight:

```text
Confirmar que el literal temporal de Vector * Float volvió de -500 a 1000.
```

Después:

```text
crear variables de reintento
→ provocar colisión inicial
→ aumentar Corridor Length
→ mover la misma Child Room Actor
→ volver a consultar bounds
→ terminar al quedar libre o alcanzar Max Placement Attempts
```

## Resumen de decisión

La revisión externa se considera útil y varias recomendaciones pasan al backlog futuro. No se realiza ningún cambio de arquitectura ni de Blueprint como consecuencia inmediata de esta opinión. La prioridad permanece en completar las fases físicas en el orden ya aprobado.