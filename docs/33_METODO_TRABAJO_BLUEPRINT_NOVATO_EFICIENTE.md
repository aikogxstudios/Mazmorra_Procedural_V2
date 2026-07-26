# 33 — Método de trabajo Blueprint para avanzar rápido sin perder claridad

**Fecha:** 2026-07-26  
**Proyecto:** Aguja del Caos / Mazmorra Procedural V2  
**Motor:** Unreal Engine 5.4  
**Objetivo inmediato:** versión privada 0.0.5.9 en itch.io

## Principio general

No se recorta la explicación técnica necesaria. Se recorta únicamente:

```text
teoría que no desbloquea la tarea actual
casos futuros que todavía no hacen falta
sistemas opcionales antes de validar el caso mínimo
reorganización estética antes de probar
```

El usuario es principiante en Unreal Engine. Por tanto, cada tarea Blueprint debe explicarse de forma exacta, visual y verificable.

## Formato obligatorio para cada función o bloque Blueprint

Cada instrucción técnica debe incluir:

```text
1. Objetivo del bloque.
2. Blueprint y función exactos donde trabajar.
3. Nombre exacto en inglés de cada nodo.
4. Variables, inputs y outputs necesarios, con tipo.
5. De dónde sale cada dato o pin importante.
6. Conexión exacta entre nodos.
7. Qué no tocar.
8. Compile + Save.
9. Prueba mínima inmediata.
10. Resultado esperado.
11. Texto de comentario para dejar dentro del Blueprint.
```

## Comentarios obligatorios dentro de Unreal

Cada función nueva debe tener una explicación permanente para poder entenderla en el futuro sin depender del asistente.

Usar dos niveles cuando sea útil:

### Function Description

En la descripción de la función, escribir una frase breve que explique su responsabilidad general.

Ejemplo:

```text
Convierte un patrón de puertas prebuilt y su rotación base en las cuatro direcciones físicas disponibles.
```

### Comment Box dentro del gráfico

Cada grupo importante de nodos debe quedar dentro de un Comment Box con un título claro.

Ejemplos:

```text
RESOLVE BASE DOORS FROM PATTERN
APPLY BASE ROTATION
COMPARE AGAINST CELL CONNECTIONS
SUCCESS — PATTERN MATCHES CELL
FAILURE — INCOMPATIBLE PREBUILT
```

El asistente debe proporcionar siempre el texto exacto sugerido para estos comentarios cuando se construya una función.

## Ritmo de trabajo

```text
hacer un bloque pequeño
→ Compile
→ Save
→ probar
→ revisar captura o resultado
→ continuar
```

No construir varias funciones grandes sin probar la anterior.

## Capturas

Pedir capturas cuando sean necesarias para confirmar:

```text
estructura actual de una función
nombres reales de variables
pins disponibles
flujo de ejecución existente
resultado visual o error
```

No inventar la estructura exacta de una función que no se haya visto.

## Método MVP 0.0.5.9

```text
hacer el caso mínimo
→ integrarlo
→ probarlo jugando
→ corregir
→ ampliar
```

La claridad técnica se mantiene. La velocidad se consigue evitando trabajo no imprescindible, no omitiendo pasos que el usuario necesita para entender Unreal.

## Regla para decidir tareas

Antes de empezar una tarea:

```text
¿Es necesaria para probar la primera mazmorra jugable?
```

Si no lo es:

```text
se documenta para después
no bloquea la versión 0.0.5.9
```

## Estado de aplicación

Este método entra en vigor desde la siguiente continuación en:

```text
BP_DungeonGenerator_V2
→ GetPreBuiltPatternDoors
```

La función se construirá por bloques pequeños, con nodos exactos, origen de datos, prueba aislada y comentarios dentro del Blueprint.
