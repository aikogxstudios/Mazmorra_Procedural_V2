# Cómo actualizar la documentación

Este repositorio documenta un sistema Blueprint vivo. Los cambios deben reflejar el estado real de Unreal Engine.

## Antes de editar

1. Abrir el Blueprint real.
2. Confirmar nombres exactos.
3. Guardar capturas del gráfico.
4. Identificar invariantes afectadas.

## Después de editar

Actualizar como mínimo:

```text
docs/00_ESTADO_ACTUAL.md
CHANGELOG.md
knowledge/state.yaml
```

Si afecta a un subsistema, actualizar también su archivo específico.

## Cambios de funciones

Documentar:

- firma;
- inputs/outputs;
- variables locales;
- flujo de ejecución;
- arrays modificados;
- pruebas;
- errores conocidos.

## Cambios de variables

Documentar:

- nombre exacto;
- tipo;
- categoría;
- default;
- funciones lectoras/escritoras;
- estado de certeza.

## Commits

Ejemplos:

```text
feat: add SpawnStartRoom
fix: preserve parent-child index mapping
test: validate 150-room layout
docs: update room placement state
```

## Regla principal

```text
No asumir el montaje de un Blueprint sin captura.
```

Más información:

```text
docs/17_MANTENIMIENTO_DOCUMENTACION.md
```
