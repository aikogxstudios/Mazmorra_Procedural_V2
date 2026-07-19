# Knowledge database

Esta carpeta contiene una representación machine-readable del sistema para Aiko, ChatGPT, Codex y herramientas futuras.

## Archivos

```text
state.yaml
→ estado operativo, validaciones y siguiente paso

assets.yaml
→ inventario de assets y responsabilidades

bp_dungeon_generator_v2.yaml
→ funciones del generador

variables.yaml
→ variables por Blueprint

invariants.yaml
→ reglas que no deben romperse
```

## Regla de uso

La documentación Markdown contiene explicaciones completas.

Los YAML sirven para:

- búsquedas rápidas;
- agentes automáticos;
- validaciones futuras;
- generar índices;
- evitar depender de contexto conversacional.

## Jerarquía

```text
1. docs/00_ESTADO_ACTUAL.md
2. Capturas/pruebas actuales
3. knowledge/*.yaml
4. Resto de docs
```

Si se modifica un dato crítico, actualizar tanto Markdown como YAML.

## Estados

Los YAML usan valores como:

```text
confirmed
confirmed_conceptual
protected
pending
future
not_visible
```

## No asumir

Un campo marcado `documented`, `not_visible` o `exact_name_pending` no debe tratarse como confirmado visualmente.
