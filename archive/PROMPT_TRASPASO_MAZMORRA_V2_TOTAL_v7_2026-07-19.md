# Prompt de traspaso — Mazmorra Procedural V2 TOTAL v7

Estamos continuando la **Mazmorra Procedural V2 de Caos Entre Reinos / Chaos Among Realms: Reborn** en **Unreal Engine 5.4**, principalmente con Blueprints.

## Fuente de verdad

Lee completo antes de responder:

```text
Mazmorra_Procedural_V2_MASTER_TOTAL_v7_2026-07-19.md
```

Ese archivo contiene el estado completo de:

- `BP_DungeonGenerator_V2`;
- `ST_DungeonCell` y `ST_DungeonCellLink`;
- todas las funciones del generador;
- Boss, Key, pasillos e inventario temporal;
- `BPI_DungeonRoomV2`;
- `BP_RoomMaster` original;
- `BP_RoomMaster_Dungeon`;
- `BP_FloorActor`, `BP_WallActor` y `BP_CeilingActor`;
- HISM, aperturas, columnas, flechas, centrado y bounds;
- errores corregidos y mappings finales;
- visión futura de habitaciones variables y pasillos extensibles.

Consulta también el chat anterior de este mismo proyecto para recuperar capturas y pruebas, pero no contradigas el documento maestro sin comprobarlo conmigo.

Distingue siempre:

```text
✅ confirmado
🟡 confirmado conceptualmente
⚪ no visible
⏳ pendiente
🔮 futuro
🛑 no tocar
```

## Punto exacto de continuación de v7 histórico

Dentro de:

```text
BP_DungeonGenerator_V2
→ TryAddRandomCell
→ Make ST_DungeonCellLink
```

Conectar:

```text
Get Random Cell Index
→ Parent Cell Index

Get Selected Direction
→ Direction from Parent

Has Parent = true
```

No conectar:

```text
Return Value del Add de DungeonCells
Random Direction Index
Opposite Direction
```

Después validar:

```text
DungeonCells.Num == DungeonCellLinks.Num
```

Probar con 10, 20 y 50 habitaciones y revisar los links de cada hija.

## Mappings que no se deben cambiar

`InitRoomFromCell`, variables de apertura:

```text
Cell North → SouthOpening
Cell East  → WestOpening
Cell South → NorthOpening
Cell West  → EastOpening
```

`GetDoorWorldLocation`:

```text
Dungeon North → Arrow_Entrance_South
Dungeon East  → Arrow_Exit_East
Dungeon South → Arrow_Exit_North
Dungeon West  → Arrow_Exit_West
```

## Visión V1

```text
Start preconstruida, cualquier tamaño, una sola salida.
Normal y Key procedurales o preconstruidas.
Boss preconstruida, terminal, una entrada.
Pasillos siempre presentes, de longitud variable.
Colocación física padre-hija mediante DoorPoints y RoomBounds.
Si la hija choca, aumentar longitud del pasillo y moverla.
No regenerar la sala en cada intento.
```

Explica cada modificación Blueprint con variables, nodos, pins, flujo, valores y pruebas. No avances varias fases a la vez.

> Nota: este prompt se conserva como archivo histórico. El prompt operativo actualizado está en `docs/14_PROMPT_TRASPASO.md`.
