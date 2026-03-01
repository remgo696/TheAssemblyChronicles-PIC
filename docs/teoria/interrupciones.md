---
title: Interrupciones
description: Vectores de interrupción, priorización y diseño de ISR en el PIC18F47Q10
---

# Interrupciones

## ¿Qué es una interrupción?

Las **interrupciones** suspenden temporalmente la ejecución del μC del programa principal para atender un evento (externo o interno). Una vez atendido, se retoma la ejecución normal mediante la instrucción `RETFIE`. Así, el evento disparador de la rutina se llama **fuente de interrupción**, el código que se ejecuta para atenderla es la **rutina de servicio de interrupción** (ISR) y la dirección de memoria donde se encuentra la ISR es el **vector de interrupción**.

## Vectores de interrupción
El Q10 tiene 2 vectores de interrupción[^vectors]: `0x0008` para interrupciones de alta prioridad y `0x0018` para las de baja prioridad. Es más, al desactivar las prioridades, todas las interrupciones van al vector de alta prioridad `0x0008`.

??? info "Vectores de interrupción en el Q43"
    El Q43 utiliza un **controlador de interrupciones vectorizado** (VIC) con una tabla de vectores configurable a través de `IVTBASE`, en lugar de los dos vectores fijos del Q10.

### Registro de control de interrupciones (INTCON)

El registro `INTCON` es el registro principal de control de interrupciones del Q10[^INTCON]. Reside en el Access bank y controla la habilitación global de interrupciones, el modo de prioridad y la selección de flanco para las interrupciones externas.

<figcaption markdown="span">Interrupt Control Register (INTCON)[^INTCON]</figcaption>

| Bit | 7 | 6 | 5 | 2 | 1 | 0 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Campo** | GIE/GIEH | PEIE/GIEL | IPEN | INT2EDG | INT1EDG | INT0EDG |
| **Permisos** | R/W | R/W | R/W | R/W | R/W | R/W |
| **Reset** | 0 | 0 | 0 | 1 | 1 | 1 |

`GIE/GIEH`
:   Habilitación global de interrupciones. Su comportamiento depende del modo de prioridad:

    - Si `IPEN = 1`: habilita las interrupciones de **alta prioridad**.
    - Si `IPEN = 0`: habilita las interrupciones **no periféricas**. 

`PEIE/GIEL`
:   Habilitación de interrupciones periféricas / de baja prioridad:

    - Si `IPEN = 1`: habilita todas las interrupciones de **baja prioridad**.
    - Si `IPEN = 0`: habilita todas las interrupciones **periféricas**.

    No se puede activar si no está activado `GIE/GIEH`.

`IPEN`
:   Habilitación de prioridades de interrupción. `1` = prioridades habilitadas y configurables, como altas o bajas, en su registro de prioridad de interrupción `IPRx`.· `0` = prioridades deshabilitadas y se ignoran los registros de prioridad `IPRx`.

`INTxEDG` (x = 0, 1, 2)
:   Selección de flanco para la interrupción externa INTx. `1` = flanco ascendente · `0` = flanco descendente.

---

## Referencias
[^vectors]: {{ q10('15.2', 192) }}
[^INTCON]: {{ q10('15.13.1', 195) }}