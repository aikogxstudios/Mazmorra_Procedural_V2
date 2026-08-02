# Cierre definitivo del sistema Blueprint y comienzo de un nuevo plugin C++

**Fecha:** 2026-08-02  
**Proyecto anterior:** Mazmorra Procedural V2  
**Nuevo proyecto:** DungeonLabPlugin

## Resumen

Durante varias semanas se desarrolló un generador de mazmorras en Blueprints para Unreal Engine 5.4.

El sistema anterior llegó a incluir:

- generación lógica mediante celdas y enlaces padre-hijo;
- habitación Start;
- una cantidad configurable de habitaciones Normal;
- habitaciones especiales Key y Boss añadidas fuera del límite de habitaciones normales;
- generación física de 13 habitaciones con la configuración de prueba;
- selección de clases de habitación;
- habitaciones procedurales y pruebas con habitaciones prehechas;
- DoorPoints para conectar habitaciones;
- RoomBounds para representar el espacio ocupado;
- comprobación global de solapamientos mediante AABB;
- reintentos de colocación;
- seeds reproducibles;
- selección de patrones de puertas como DeadEnd, Straight, Corner, TShape y Cross;
- pruebas de rotación y compatibilidad de puertas;
- documentación técnica y sesiones de trabajo.

También se detectaron y corrigieron problemas como:

- funciones heredadas sobrescritas accidentalmente en Blueprints hijos;
- RoomBounds que devolvían valores cero;
- mensajes de error ambiguos;
- confusión entre direcciones locales y mundiales;
- complejidad creciente al intentar adaptar habitaciones prehechas rotadas.

## Decisión final

El sistema anterior queda **abandonado definitivamente**.

No continuará su desarrollo y no se utilizará como base del nuevo generador.

Esto significa que se descartan:

- la arquitectura actual en Blueprints;
- las funciones creadas;
- los enums y structs del sistema anterior;
- la lógica de selección y colocación;
- la integración de habitaciones prehechas;
- las tareas pendientes;
- las pruebas de rotación;
- la documentación operativa del sistema anterior;
- cualquier planificación de pasillos, verticalidad o expansión basada en esta arquitectura.

Todo el trabajo anterior se considera material histórico y no una base reutilizable.

## Nuevo rumbo

Se ha decidido comenzar un sistema completamente nuevo desde cero llamado:

```text
DungeonLabPlugin
```

El nuevo sistema será un plugin para Unreal Engine 5 desarrollado principalmente en C++ y expuesto de forma cómoda a Blueprints.

Objetivos generales del nuevo proyecto:

- arquitectura modular y reutilizable;
- independencia del proyecto principal;
- configuración sencilla desde Blueprints y Data Assets;
- soporte para habitaciones prehechas y procedurales;
- habitaciones de tamaños y formas diferentes;
- Start, Normal, Key y Boss como categorías iniciales;
- pasillos procedurales;
- seeds reproducibles;
- soporte futuro para ramas, escaleras, verticalidad, parkour y habitaciones especiales;
- documentación completa como wiki y guía de usuario.

## Estado del repositorio anterior

Este repositorio queda únicamente como archivo histórico del sistema descartado.

No deben abrirse nuevas tareas ni continuar fases pendientes dentro de esta arquitectura.

Cualquier nueva implementación de mazmorras deberá realizarse en el repositorio del nuevo plugin:

```text
aikogxstudios/DungeonLabPlugin
```

## Regla final

```text
Mazmorra Procedural V2 en Blueprints
= sistema descartado y cerrado.

DungeonLabPlugin en C++
= nuevo sistema oficial desde cero.
```
