# Cierre parcial de limpieza — DebugDrawConnections

**Fecha:** 2026-07-26  
**Blueprint:** `BP_DungeonGenerator_V2`

## Eliminación confirmada

Se eliminó únicamente la función:

```text
DebugDrawConnections
```

Motivo:

```text
No se usa en el flujo actual.
```

Después de eliminarla:

```text
Blueprint compila
la generación completa continúa funcionando
13 habitaciones siguen apareciendo
Start, Key y Boss continúan incluidas
```

## Funciones de debug conservadas

Se decidió no borrar más herramientas de diagnóstico por ahora:

```text
DebugPrintLayout
DebugDrawDoorPoints
DebugDrawDoorToDoorConnections
```

Estas funciones pueden ser útiles para las próximas tareas:

```text
compatibilidad de puertas físicas prebuilt
rotación de habitaciones prebuilt
alineación DoorPoint a DoorPoint
pasillo procedural adaptable
comprobación de conexiones físicas
```

## Regla vigente

```text
No borrar una función de debug solo porque no esté conectada actualmente.
Mantenerla cuando pueda ayudar en una fase inmediata.
Usar Find References y una prueba controlada antes de futuras eliminaciones.
```

## Punto siguiente

La limpieza deja de ser prioritaria por ahora. El siguiente trabajo funcional será diseñar e implementar la compatibilidad entre conexiones lógicas y puertas físicas de habitaciones prefabricadas.
