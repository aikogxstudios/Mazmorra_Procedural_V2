# Objetivo de entrega — versión 0.0.5.9 beta privada

**Fecha:** 2026-07-26

## Meta

La primera versión funcional de la mazmorra procedural se publicará como:

```text
0.0.5.9
```

Destino:

```text
itch.io
→ acceso privado/restringido
→ solo testers elegidos por el desarrollador
```

Objetivo de la publicación:

```text
probar la mazmorra en una build real
recibir feedback de beta testers
validar estabilidad
probar parches posteriores
corregir errores antes de una apertura mayor
```

## Cambio de método de trabajo

Se mantiene la arquitectura actual, pero se acelera la ejecución.

Antes:

```text
estudiar
→ diseñar
→ preparar todos los casos
→ integrar
```

Ahora:

```text
hacer el caso mínimo
→ integrarlo
→ probarlo jugando
→ corregir
→ ampliar
```

## Reglas de eficiencia compartidas

```text
1. Una tarea técnica cada vez.
2. No diseñar sistemas futuros que no bloqueen la 0.0.5.9.
3. No crear todos los casos antes de validar el primero.
4. Probar cada bloque dentro de la mazmorra real cuanto antes.
5. Mantener una alternativa procedural cuando falten prebuilt compatibles.
6. Documentar al terminar un bloque, no interrumpir constantemente la implementación.
7. No rehacer sistemas estables salvo bug demostrado.
8. Cada sesión debe producir un resultado tangible.
```

Resultados tangibles válidos:

```text
función compilada y probada
habitación integrada
bug corregido
pasillo recorrible
generación completa validada
build privada lista para testers
```

## Alcance mínimo de la 0.0.5.9

```text
- generación completa estable
- Start, habitaciones Normal, Key y Boss
- habitaciones procedurales variables
- un pequeño pool de prebuilt compatibles
- al menos Corner/L, Straight y DeadEnd de prueba
- rotaciones correctas
- selección aleatoria compatible
- procedural como fallback
- RoomBounds y DoorPoints funcionando
- pasillos V1 transitables
- varias seeds recorribles en PIE y build
- sistema preparado para recibir parches
```

Fuera de alcance inmediato:

```text
GridZ completo
verticalidad procedural avanzada
backtracking complejo
pools por bioma
decoración final
optimización artística definitiva
pesos avanzados
TShape/Cross prebuilt si la procedural cubre esos casos
```

## Punto de continuación

```text
BP_DungeonGenerator_V2
→ GetPreBuiltPatternDoors
→ implementar caso mínimo
→ validar Corner + Rot180
→ integrar BP_Room_PreBuilt_Test_Corner en generación real
```

## Regla principal

La prioridad ya no es completar toda la arquitectura futura antes de probar. La prioridad es sacar una primera mazmorra funcional, recorrible y suficientemente variada para publicarla como beta privada 0.0.5.9 y mejorarla mediante pruebas y parches reales.
