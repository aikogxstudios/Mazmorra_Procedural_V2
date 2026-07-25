# 13 — Roadmap de implementación

**Última actualización:** 2026-07-25

## Estado de fases

### Fase A — DungeonCellLinks

```text
✅ ST_DungeonCellLink creado.
✅ DungeonCellLinks creado.
✅ Start y todas las hijas reciben link correspondiente.
✅ ParentCellIndex válido y menor que ChildIndex.
✅ Cantidades históricas validadas hasta 150.
```

### Fase B — Start con una sola salida

```text
✅ Implementada.
✅ Validada.
```

### Fase C — SpawnStartRoom

```text
✅ Implementada y validada.
✅ SpawnedRooms[0] = Start.
```

### Fase D — Primera hija alineada

```text
✅ SpawnFirstChildRoom creada.
✅ DoorPoints padre-hija alineados.
✅ Init una sola vez.
✅ Misma referencia movida.
```

### Fase D.1 — Separación para pasillo

```text
✅ GetDirectionVector Pure.
✅ Offset de corredor validado.
✅ Seed 12345 corregida como South.
✅ Seed 12346 validada como East.
```

### Fase E — RoomBounds y AABB

```text
✅ RoomBounds procedural.
✅ BP_Room_PreBuilt_Base.
✅ Get Room Bounds Data.
✅ AABB X/Y/Z.
✅ Caso libre y caso forzado.
```

### Fase F — Reintentos controlados

```text
✅ Corridor Length.
✅ Placement Retry Step.
✅ Placement Attempt.
✅ Max Placement Attempts.
✅ bPlacement Succeeded.
✅ For Loop with Break.
✅ Misma candidata movida.
✅ Fallo controlado con Destroy Actor.
```

Reglas validadas:

```text
No repetir SpawnActor.
No repetir Init Room from Cell.
No regenerar HISM.
Detenerse por éxito o máximo de intentos.
```

### Fase G — Overlap contra todas las salas

```text
✅ DoesRoomOverlapPlacedRooms creada.
✅ Recorre SpawnedRooms.
✅ Ignora candidata e inválidos.
✅ AABB global.
✅ Break en primer overlap.
✅ Devuelve Found Overlap.
```

### Fase H — Todas las hijas

```text
✅ PlaceChildRoomFromParent creada.
✅ Input Child Index.
✅ Output Room Placed.
✅ GenerateDungeon recorre 1..DungeonCells.Length-1.
✅ False conecta a Break.
✅ 3 habitaciones iniciales validadas.
✅ 10 habitaciones validadas.
✅ 13 habitaciones validadas con especiales.
```

### Fase H.1 — Especiales adicionales

```text
✅ Max Rooms cuenta solo Normal.
✅ BuildDungeonLayout usa Max Rooms + 1 para Start.
✅ TryAddSpecialCellFromParent creada.
✅ Key añadida como nueva celda.
✅ Boss añadida como nueva celda.
✅ Key/Boss ya no reemplazan normales.
✅ 1 Start + 10 Normal + 1 Key + 1 Boss = 13.
```

### Fase H.2 — Búsqueda robusta de dirección especial

```text
✅ Inicio aleatorio 0..3.
✅ Prueba cuatro direcciones.
✅ Fórmula (Start + Index) % 4.
✅ Error de división corregido.
✅ Solo falla si las cuatro están ocupadas.
```

### Fase H.3 — Clases prebuilt Key/Boss

```text
✅ Error de bounds cero diagnosticado.
✅ BP_Room_PreBuilt_Base_Child_Key.
✅ BP_Room_PreBuilt_Base_Child_Boss.
✅ Herencia de BPI, RoomBounds y DoorPoints.
```

### Fase I — Compatibilidad de puertas prebuilt

```text
⏳ Próxima fase principal.
```

Objetivo:

```text
Una prebuilt solo puede usar puertas físicas existentes.
El generador selecciona clase + rotación compatible.
No se inventan huecos.
```

Patrones iniciales:

```text
Dead End → 1 puerta
Straight → 2 opuestas
Corner/L → 2 contiguas
T        → 3
Cross    → 4
```

Rotaciones:

```text
0°
90°
180°
270°
```

Requisitos:

```text
[ ] Definir datos de patrón de puertas.
[ ] Declarar puertas reales por clase prebuilt.
[ ] Rotar el patrón lógicamente.
[ ] Comparar contra conexiones de ST_DungeonCell.
[ ] Elegir una clase compatible antes de SpawnActor.
[ ] Rechazar clase incompatible.
[ ] Mantener procedurales compatibles con 1–4 conexiones.
```

### Fase J — Pasillos visuales variables

```text
⏳ Pendiente.
```

El placement decide la distancia válida. Después los pasillos deben conectar DoorPoints reales y validar su volumen.

### Fase K — Limpieza del refactor

```text
⏳ Pendiente después de regresión.
```

Candidatos:

```text
SpawnFirstChildRoom
SpawnRoomsFromCells
Set Array Elem antiguos Key/Boss
Print String temporales
variables sin referencias
funciones antiguas desconectadas
```

Regla:

```text
Find References
→ compilar
→ probar
→ borrar
→ regresión
```

### Fase L — Regresión completa

```text
⏳ En curso.
```

Incluye:

```text
seed 12345
seed 12346
DungeonCells.Num == DungeonCellLinks.Num == SpawnedRooms.Num
13 habitaciones con Max Rooms 10
sin overlap
sin SPECIAL FAILED
sin Max Placement Attempts reached
regeneración limpia
```

## Arquitectura híbrida aprobada

### Procedural

```text
BP_RoomMaster_Dungeon
→ habitaciones comunes
→ HISM
→ bounds dinámicos
→ 1–4 conexiones
→ abre solo las necesarias
→ tapa las no usadas
```

### Prebuilt

```text
BP_Room_PreBuilt_Base y Blueprints hijos
→ especiales/hechas a mano
→ bounds manual
→ DoorPoints reales
→ patrón fijo
→ Level Instance/Packed Level Blueprint futuro
```

## Regla de cantidad

Implementada:

```text
Max Rooms = habitaciones Normal
Total = Max Rooms + Start + especiales activas
```

Ejemplo actual:

```text
10 Normal + Start + Key + Boss = 13
```

Objetivo V1 anterior:

```text
15 Normal + Start + Key + Boss = 18
```

Ahora puede lograrse usando `Max Rooms = 15` si las especiales se añaden correctamente.

## Regla de avance

```text
Una fase
→ captura
→ cableado pequeño
→ compilar
→ prueba positiva
→ prueba negativa
→ varias seeds
→ documentar
→ siguiente fase
```
