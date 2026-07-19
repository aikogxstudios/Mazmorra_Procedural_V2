# 22 — BP_BossDoor_Lock, Key Room y mini inventario

## Estado global

```text
✅ Loop funcional completo de prueba.
✅ Pickup entra al mini inventario.
✅ Puerta permanece cerrada sin llave.
✅ Puerta abre con llave.
🟡 Implementación experimental, no final.
⚪ Variables y funciones internas exactas pendientes de capturas completas.
```

# BP_BossDoor_Lock

## Responsabilidad conocida

```text
Bloquear la conexión hacia la Boss Room.
Consultar si el jugador/inventario tiene el item requerido.
Permanecer cerrada sin llave.
Abrirse con llave.
```

## Spawn

La genera `BP_DungeonGenerator_V2` mediante:

```text
SpawnBossRoomDoors
→ SpawnDoorAtRoomDirection
```

No está colocada de forma fija dentro de la Boss Room.

## Posición

```text
BossRoomActor.GetDoorWorldLocation(Direction)
```

## Rotación

```text
North = 0
East = -90
South = 180
West = 90
```

## Collision Handling conocido del spawn

Documentado:

```text
Always Spawn, Ignore Collisions
```

Verificar en captura antes de modificar.

## Item requerido

La puerta conoce su item requerido.

Decisión de arquitectura:

```text
BossKeyItemData no pertenece a BP_DungeonGenerator_V2.
```

El generador decide:

- clase;
- posición;
- rotación;
- referencia para limpieza.

La puerta decide:

- item requerido;
- condición de apertura;
- comportamiento visual/físico.

## Información interna pendiente

Falta registrar nombres exactos de:

- variable de item requerido;
- función de interacción;
- función de consulta al inventario;
- animación/movimiento de apertura;
- componentes;
- collision presets;
- sonido/VFX;
- si consume o no la llave.

# BP_Room_Debug_Key

## Estado

```text
✅ Spawnea pickup temporalmente.
✅ Delay aproximado de 10 segundos.
✅ Usa Arrow/punto central.
```

## Flujo conceptual

```text
Sala Key inicializada/spawneada
→ Delay ~10 s
→ obtener punto central
→ SpawnActor clase pickup
```

## Motivo del Delay

Implementación temporal para probar el loop sin un sistema formal de contenido.

No representa el diseño final.

## Pendiente futuro

```text
Eliminar Delay.
Integrar spawn en contenido de sala.
Evitar duplicados.
Elegir posición según sala.
Separar lógica de layout y contenido.
```

# Pickup de llave

## Responsabilidad conocida

```text
Detectar recogida/interacción.
Añadir Data Asset o item al mini inventario.
Desaparecer o marcarse recogido.
```

## Información pendiente

- nombre exacto de clase;
- Data Asset usado;
- ID del item;
- función AddItem;
- prevención de recogida duplicada;
- replicación si aplica;
- actor/componente que recibe el item.

# Mini Inventory Component

## Estado

```text
✅ Funcional para la prueba.
🟡 Experimental.
```

## Responsabilidad conocida

```text
Guardar items temporales.
Responder si contiene el item requerido.
```

## Información pendiente

- nombre exacto de clase del componente;
- tipo de array/map;
- funciones Add/Has/Remove;
- owner actual;
- persistencia;
- replicación;
- relación con el inventario final.

# UI mini inventario

## Estado

```text
✅ Se abre con O.
```

## Uso

Permite comprobar visualmente que la llave fue añadida.

No representa la UI final.

# Loop de prueba

```text
GenerateDungeon
→ ChooseKeyAndBossCells
→ Spawn Key Room
→ Delay
→ Spawn pickup
→ jugador recoge pickup
→ mini inventario guarda item
→ jugador llega a Boss Door
→ Boss Door consulta inventario
→ sin item: cerrada
→ con item: abre
```

# Pruebas de regresión

```text
[ ] Key Room no es Start.
[ ] Key Room no es Boss.
[ ] Pickup aparece una vez.
[ ] Pickup aparece en el punto correcto.
[ ] Inventario recibe item.
[ ] UI lo muestra.
[ ] Door cerrada sin item.
[ ] Door abre con item.
[ ] Puertas antiguas se destruyen al regenerar.
[ ] Pickup antiguo no queda huérfano al regenerar.
```

# Futuro de puertas

Datos reservados:

```text
DoorType
RequiredKeyID
DoorGroupID
bConsumesKey
bOpensWholeGroup
bOptional
```

No implementar durante la fase `SpawnStartRoom`/placement.

# Relación con el inventario final

El proyecto tiene planes separados para:

- cartas;
- consumibles;
- inventario de run.

Este mini inventario se conserva solo como infraestructura de prueba hasta que el sistema definitivo lo sustituya.
