---
name: Bug de Mazmorra V2
title: "[Bug] "
about: Registrar un error reproducible del sistema procedural
labels: ""
assignees: ""
---

## Resumen

```text
Qué ocurre:
Qué debería ocurrir:
```

## Seed y configuración

```text
DungeonSeed:
bUseRandomSeed:
MaxRooms:
MaxGenerationAttempts:
CellSize:
```

## Sala/celda afectada

```text
ChildIndex:
ParentCellIndex:
DirectionFromParent:
RoomType:
GridX:
GridY:
RoomSeed:
```

## Reproducción

```text
1.
2.
3.
```

## Capturas / vídeo

- [ ] Captura del resultado.
- [ ] Captura del Blueprint afectado.
- [ ] Output Log.
- [ ] DebugPrintLayout.
- [ ] Debug Cells/Links si aplica.
- [ ] Debug DoorPoints si aplica.

## Invariantes

```text
DungeonCells.Num:
DungeonCellLinks.Num:
SpawnedRooms.Num:
Start ConnectionCount:
```

## Tipo de fallo

- [ ] Layout lógico.
- [ ] Parent link.
- [ ] Start.
- [ ] Spawn de sala.
- [ ] Dirección/apertura.
- [ ] DoorPoint.
- [ ] Bounds.
- [ ] Solapamiento.
- [ ] Pasillo.
- [ ] Boss/Key.
- [ ] Boss Door.
- [ ] Inventario/llave.
- [ ] Limpieza/regeneración.
- [ ] Rendimiento.

## Prueba mínima

```text
Número mínimo de habitaciones que reproduce el fallo:
```

## Solución propuesta

```text
No modificar una parte protegida sin confirmar el diagnóstico.
```
