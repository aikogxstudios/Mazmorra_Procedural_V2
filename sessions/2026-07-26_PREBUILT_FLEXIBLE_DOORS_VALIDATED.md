# Validación de puertas flexibles — BP_Room_PreBuilt_Base

**Fecha:** 2026-07-26

## Resultado confirmado

La habitación `BP_Room_PreBuilt_Base_Child_Key` fue comprobada en ejecución.

```text
1 apertura correspondiente a su conexión lógica
3 WallFill visibles cerrando las direcciones no conectadas
sin aperturas adicionales
```

## Implementación observada en Init Room from Cell

Para cada dirección de `ST_DungeonCell`:

```text
Connection = true
→ DoorMarker_[Direction] visible
→ WallFill_[Direction] oculto
→ WallFill_[Direction] con No Collision
```

```text
Connection = false
→ DoorMarker_[Direction] oculto
→ WallFill_[Direction] visible
→ la pared permanece cerrada
```

## Conclusión

`BP_Room_PreBuilt_Base` funciona actualmente como una base prebuilt flexible capaz de abrir/cerrar cualquiera de las cuatro direcciones mediante `WallFill`.

Esta validación no sustituye el futuro sistema de habitaciones artísticas de patrón fijo. Las futuras prebuilt de patrón fijo deberán declarar sus puertas físicas reales y solo podrán usarse con una rotación compatible.

## Próximo paso

Diseñar el contrato mínimo para patrones físicos:

```text
Dead End
Straight
Corner/L
T
Cross
```

Mantener intactos por ahora:

```text
Init Room from Cell
DoorPoint_North/East/South/West
WallFill_North/East/South/West
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```
