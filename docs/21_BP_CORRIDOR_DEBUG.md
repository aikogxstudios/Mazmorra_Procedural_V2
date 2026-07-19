# 21 — BP_Corridor_Debug

## Estado

```text
✅ Funcional.
✅ Validado visualmente por los cuatro lados.
🛑 No tocar durante la primera fase de placement.
⚪ Gráfico interno completo no documentado todavía.
```

## Responsabilidad conocida

`BP_Corridor_Debug` recibe dos puntos world y construye un pasillo entre ellos.

Conceptualmente:

```text
StartLocation
EndLocation
→ orientar corredor
→ calcular longitud
→ representar conexión física
```

## Consumidor principal

```text
BP_DungeonGenerator_V2.SpawnCorridorsFromConnections
```

## Flujo de entrada actual

### Conexión North

```text
Start = CurrentRoom.GetDoorWorldLocation(North)
End = NeighborRoom.GetDoorWorldLocation(South)
→ Spawn BP_Corridor_Debug
```

### Conexión East

```text
Start = CurrentRoom.GetDoorWorldLocation(East)
End = NeighborRoom.GetDoorWorldLocation(West)
→ Spawn BP_Corridor_Debug
```

El generador procesa solo North y East para no duplicar corredores.

## Validaciones visuales realizadas

```text
✅ Pasillo corto entre paredes interiores.
✅ North correcto.
✅ East correcto.
✅ Las conexiones South/West aparecen mediante el procesamiento del vecino.
✅ No atraviesa una sala completa después de corregir DoorWorldLocation.
```

## Debug histórico

Se usaron:

```text
Esfera blanca en Start.
Esfera naranja en End.
Línea debug entre Start y End.
```

Esto confirmó que los puntos world entregados al corredor eran correctos.

## Futuro con salas variables

No se espera que el corredor necesite conocer `DungeonCellLinks`.

Después de colocar las salas:

```text
DoorPoint padre
DoorPoint hija
```

estarán separados por la longitud física elegida. El corredor seguirá conectando ambos puntos.

Variedad V1:

```text
pasillo corto
pasillo medio
pasillo largo
```

La longitud resultará de:

```text
longitud inicial
+
alargamiento por colisión
```

## Información pendiente de capturas

Antes de modificar el actor, confirmar:

- nombres exactos de variables Start/End;
- componentes;
- mesh usado;
- si usa spline, mesh escalado o instancias;
- cálculo exacto de midpoint;
- cálculo exacto de yaw;
- tratamiento de Z;
- collision preset;
- si genera suelo/paredes;
- si posee función de inicialización o usa Expose on Spawn;
- cómo se guarda en `SpawnedCorridors`.

## Riesgos

```text
Cambiar el pivote del mesh
→ desplaza pasillo.

Cambiar DoorPoints
→ cambia longitud/orientación.

Procesar South/West
→ duplica corredores.
```

## Política

```text
No refactorizar BP_Corridor_Debug
hasta que el placement padre-hija funcione con dos salas.
```
