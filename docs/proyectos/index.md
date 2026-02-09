---
title: Proyectos
description: Ejemplos prácticos y proyectos con PIC18F en Assembly y C
---

# :material-chip: Proyectos

Sección dedicada a **proyectos prácticos** con los microcontroladores PIC18F57Q43 y PIC18F47Q10. Cada proyecto incluye código fuente, esquemáticos y explicaciones paso a paso.

---

!!! note "En construcción"

    Los proyectos se irán agregando conforme avance el curso. Algunos proyectos planeados:

    - [ ] Blink LED — primer programa en Assembly
    - [ ] Lectura de botón con antirrebote
    - [ ] Display 7 segmentos multiplexado
    - [ ] Temporizador con TMR0
    - [ ] Interrupción externa con INT
    - [ ] PWM para control de brillo
    - [ ] Comunicación EUSART

---

## Estructura de cada proyecto

Cada proyecto seguirá esta estructura estándar:

```
proyectos/
└── nombre-del-proyecto/
    ├── README.md          # Descripción y objetivos
    ├── src/               # Código fuente (.asm / .c)
    ├── sim/               # Archivos de simulación
    └── docs/              # Diagramas y fotos
```

!!! tip "¿Cómo compilar?"

    Todos los proyectos están diseñados para compilarse con **MPLAB X IDE** y **PIC AS Assembler**. Consulta la guía de configuración del entorno en la sección de [Recursos](../recursos/index.md).
