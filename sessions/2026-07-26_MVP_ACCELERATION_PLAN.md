# Plan de aceleración — Primera versión funcional

**Fecha:** 2026-07-26  
**Proyecto:** Aguja del Caos / Mazmorra Procedural V2

## Decisión

Después de cuatro semanas de trabajo de base, estabilidad y arquitectura, la prioridad cambia a entregar una primera versión funcional y jugable lo antes posible.

A partir de ahora:

```text
menos planificación futura
menos limpieza no crítica
menos sistemas avanzados antes de probar
más integración
más pruebas en juego
más progreso visible
```

## Objetivo de la primera versión funcional

La versión se considerará lista para primera prueba cuando:

```text
1. El layout actual siga generando Start + Normal + Key + Boss.
2. Las habitaciones Normal puedan salir de un pool pequeño.
3. El pool mezcle la procedural flexible con varias prebuilt.
4. Las prebuilt respeten patrón físico y rotación.
5. Existan al menos Corner/L, Straight y DeadEnd de prueba.
6. No haya puertas inventadas ni overlaps visibles.
7. Las habitaciones estén conectadas por un pasillo V1 transitable.
8. Varias seeds puedan recorrerse en PIE para evaluar ritmo y variedad.
```

## Orden inmediato

```text
1. Terminar GetPreBuiltPatternDoors.
2. Crear comprobación exacta contra ST_DungeonCell.
3. Integrar BP_Room_PreBuilt_Test_Corner.
4. Crear pool mínimo de habitaciones Normal.
5. Añadir una Straight y una DeadEnd de prueba.
6. Mantener BP_RoomMaster_Dungeon como fallback flexible.
7. Crear pasillo recto procedural V1 entre DoorPoints.
8. Ejecutar prueba completa jugable con varias seeds.
```

## Recortes temporales de alcance

No bloquean la primera versión:

```text
- verticalidad real y GridZ
- escaleras entre plantas
- backtracking avanzado
- room pools complejos por bioma
- pesos avanzados o reglas narrativas
- múltiples RoomBounds para salas irregulares
- Level Instance/Packed Level Blueprint final
- optimización artística definitiva
- TShape y Cross prebuilt si la procedural ya cubre esos casos
- limpieza de debug que no afecte al funcionamiento
```

## Regla de trabajo

Cada sesión debe terminar con al menos uno de estos resultados:

```text
- una función compilada y probada
- una habitación integrada en generación
- un bug de placement corregido
- un tramo jugable nuevo
- una regresión completa superada
```

La documentación se actualizará al final de cada bloque probado, sin interrumpir el avance principal.
