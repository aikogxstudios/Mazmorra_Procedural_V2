# 12 — Funciones y herramientas de debug

## Objetivo

Las funciones de debug permiten separar errores de:

```text
layout lógico
≠
DoorPoints
≠
colocación física
≠
pasillos
```

No deben confundirse con lógica de producción.

## DebugPrintLayout

### Estado

✅ Montaje visible por captura.

### Flujo

```text
DebugPrintLayout
→ Branch bDebugDungeon
   True:
   → ForEach DungeonCells
   → Break ST_DungeonCell
   → Enum to String(RoomType)
   → Format Text
   → Print String
```

Formato observado:

```text
Cell {ID} | X={X} Y={Y} | N={N} E={E} S={S} W={W} | T={Name}
```

Datos impresos:

```text
RoomID
GridX
GridY
bNorth
bEast
bSouth
bWest
RoomType
```

Configuración observada del Print:

```text
Print to Screen = true
Print to Log = true
Duration = 10.0
Development Only
```

## Debug de cantidades Cells/Links

Se añadió temporalmente después de `BuildDungeonLayout`:

```text
DungeonCells.Length
DungeonCellLinks.Length
→ Format Text
"Cells: {Cells} | Links: {Links}"
→ Print String
```

Resultados validados:

```text
10 | 10
15 | 15
20 | 20
50 | 50
150 | 150
```

Estado actual:

```text
Desactivado, no eliminado.
```

## Debug detallado de DungeonCellLinks

Para cada `ArrayIndex`:

```text
DungeonCellLinks.IsValidIndex(ArrayIndex)
→ Get DungeonCellLinks[ArrayIndex]
→ Break ST_DungeonCellLink
→ Format Text
```

Formato usado:

```text
Link Child={Child}
| Parent={Parent}
| Dir={Direction}
| HasParent={HasParent}
```

Validaciones realizadas:

```text
Child 0 → Parent -1, HasParent=false
Child > 0 → Parent válido, HasParent=true
Parent < Child
Direction enum válida
```

Estado actual:

```text
Desactivado, conservado para regresión.
```

## DebugDrawConnections

Definición confirmada por el usuario:

```text
[Debug] Dibuja líneas entre celdas conectadas para comprobar visualmente el layout.
```

Responsabilidad:

- representar conexiones lógicas;
- comprobar simetría;
- visualizar forma del grafo antes de depender de DoorPoints.

No asumir su montaje exacto sin captura si se modifica.

## DebugDrawDoorPoints

Definición confirmada:

```text
[Debug] Dibuja puntos en las puertas activas de cada sala spawneada.
```

Uso:

- comprobar que la apertura y el Arrow coinciden;
- verificar world location;
- distinguir errores de orientación frente a errores de corredor.

Forma parte del flujo real mostrado al final de `GenerateDungeon`.

## DebugDrawDoorToDoorConnections

Definición confirmada:

```text
[Debug] Dibuja líneas entre DoorPoints reales de salas conectadas.
```

Uso:

- comprobar pares de puertas;
- detectar si un pasillo atravesaría una sala;
- validar mappings internos;
- comparar conexión lógica con conexión física.

Forma parte del flujo real mostrado al final de `GenerateDungeon`.

## Debug visual usado durante orientación

Se usaron temporalmente:

```text
Esferas en las cuatro Arrows.
Esfera morada/magenta en GetActorLocation.
Esfera blanca en Start del corredor.
Esfera naranja en End del corredor.
Línea debug entre Start y End.
```

Diagnósticos obtenidos:

```text
DoorWorldLocation del pasillo era correcto.
SpawnCorridorsFromConnections era correcto.
El origen del RoomMaster estaba en una esquina.
La orientación necesitaba dos mappings distintos.
```

## Uso recomendado por tipo de error

### El layout tiene forma rara

```text
DebugPrintLayout
DebugDrawConnections
```

### Una puerta parece estar en pared incorrecta

```text
DebugDrawDoorPoints
revisar InitRoomFromCell
revisar GetDoorWorldLocation
```

### El pasillo atraviesa una sala

```text
DebugDrawDoorToDoorConnections
esfera Start/End
```

### Las salas desaparecen o se desordenan

```text
Print DungeonCells.Num
Print DungeonCellLinks.Num
Print SpawnedRooms.Num
Print ArrayIndex y ParentCellIndex
```

### Start tiene varias salidas

```text
DebugPrintLayout de Cell 0
contar N/E/S/W true
```

## Política de debug

```text
Conservar herramientas aunque estén desactivadas.
No dejar prints masivos activos en build final.
Usar bDebugDungeon cuando sea posible.
Print to Log para revisar muchos links.
Probar con pocas salas antes de 150.
```

## Futuro recomendado

Crear una validación central opcional:

```text
ValidateDungeonData
```

Podría comprobar:

```text
Cells.Num == Links.Num
Cells.Num == SpawnedRooms.Num después del spawn
Parent válido
Parent < Child
Start sin padre
Start con una salida
conexiones simétricas
referencias físicas válidas
```

Estado:

```text
🔮 futuro; no crear antes de SpawnStartRoom
```
