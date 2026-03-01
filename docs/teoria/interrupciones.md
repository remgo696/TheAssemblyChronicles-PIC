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

## Fuentes de interrupción

Cada fuente de interrupción del Q10 se gestiona mediante tres bits distribuidos en los registros `PIRx`, `PIEx` e `IPRx`[^PIR] [^PIE] [^IPR]:

- **`PIRx`** — Peripheral Interrupt Request (bandera): se activa por hardware cuando ocurre el evento. Se debe limpiar por software en la ISR.
- **`PIEx`** — Peripheral Interrupt Enable (habilitación): habilita la interrupción individual de cada fuente.
- **`IPRx`** — Interrupt Priority Register (prioridad): `1` = alta prioridad · `0` = baja prioridad. Solo tiene efecto si `IPEN = 1`.

Los tres registros siguen la misma estructura: el mismo número de registro (`x`) y el mismo bit controlan la misma fuente de interrupción. Por ejemplo, `TMR0IF` (bandera), `TMR0IE` (habilitación) y `TMR0IP` (prioridad) están todos en el bit 5 de `PIR0`, `PIE0` e `IPR0` respectivamente.

### Mapa de registros de interrupción

La siguiente tabla resume todas las fuentes de interrupción periféricas del Q10 y los bits asociados en los registros `PIRx`/`PIEx`/`IPRx`. Todos estos registros están en el banco `0Eh`[^regsummary].

<figcaption markdown="span">Resumen de registros de interrupción [^regsummary]</figcaption>

| #Reg | Bit 7 | Bit 6 | Bit 5 | Bit 4 | Bit 3 | Bit 2 | Bit 1 | Bit 0 |
|:---------:|:--------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| `0` | — | — | TMR0 | IOC | — | INT2 | INT1 | INT0 |
| `1` | OSCF | CSW | — | — | — | — | ADT | AD |
| `2` | HLVD | ZCD | — | — | — | — | C2 | C1 |
| `3` | RC2 | TX2 | RC1 | TX1 | BCL2 | SSP2 | BCL1 | SSP1 |
| `4` | — | — | TMR6 | TMR5 | TMR4 | TMR3 | TMR2 | TMR1 |
| `5` | CLC4 | CLC3 | CLC2 | CLC1 | — | TMR5G | TMR3G | TMR1G |
| `6` | CLC8 | CLC7 | CLC6 | CLC5 | — | — | CCP2 | CCP1 |
| `7` | SCAN | CRC | — | NVM | — | — | — | CWG1 |


??? tip "¿Cómo leer esta tabla?"
    Para encontrar los bits asociados a una fuente de interrupción, busca el sufijo del periférico. Por ejemplo, para el TMR0:

    - **Bandera:** `TMR0IF` → `PIR0`, bit 5 (dirección `0x0EC5`).
    - **Habilitación:** `TMR0IE` → `PIE0`, bit 5 (dirección `0x0EBD`).
    - **Prioridad:** `TMR0IP` → `IPR0`, bit 5 (dirección `0x0EB5`).

    Los tres registros siempre comparten el mismo número (`PIR0`/`PIE0`/`IPR0`) y el mismo bit (5) para la misma fuente.

---

### Pasos para configurar interrupciones con prioridad
Normalmente, se usan las interrupciones con prioridad en el Q10:

0. Configurar el periférico que generará la interrupción (TMR0, CCP, EUSART, etc.).

1. Habilitar las interrupciones con prioridad en `INTCON`.

    ??? example "Ejemplo: Habilitar interrupciones globales"
        Con prioridades habilitadas (`IPEN = 1`), para habilitar alta y baja prioridad:

        ```c
        INTCONbits.IPEN = 1;    // Habilitar prioridades
        INTCONbits.GIEH = 1;    // Habilitar interrupciones de alta prioridad
        INTCONbits.GIEL = 1;    // Habilitar interrupciones de baja prioridad
        ```
        ```asm
        setf INTCON, a        ; IPEN = 1, GIE/GIEH = 1, PEIE/GIEL = 1
        ```

        Sin prioridades (`IPEN = 0`):

        ```c
        INTCONbits.GIE = 1;     // Habilitar interrupciones globales
        INTCONbits.PEIE = 1;    // Habilitar interrupciones periféricas
        ```
        ```asm
        bsf INTCON, 7, a        ; GIE = 1
        bsf INTCON, 6, a        ; PEIE = 1
        ```

1. Bajar la bandera de interrupción correspondiente en `PIRx`.

    ??? example "Ejemplo: Bajar bandera del TMR0"
        ```c
        PIR0bits.TMR0IF = 0;
        ```
        ```asm
        movlb 0Eh
        bcf PIR0, 5, b          ; TMR0IF = 0
        ```

2. Habilitar la interrupción individual en `PIEx`.

    ??? example "Ejemplo: Habilitar interrupción del TMR0"
        ```c
        PIE0bits.TMR0IE = 1;
        ```
        ```asm
        bsf PIE0, 5, b          ; TMR0IE = 1
        ```

3. Configurar la prioridad en `IPRx`.

    ??? example "Ejemplo: Alta prioridad para TMR0"
        ```c
        IPR0bits.TMR0IP = 1;    // Alta prioridad
        ```
        ```asm
        bsf IPR0, 5, b          ; TMR0IP = 1 (alta prioridad)
        ```


5. En la ISR, limpiar la bandera `PIRx` correspondiente antes de salir con `RETFIE`.

    !!! warning "Limpiar la bandera en la ISR"
        Si no se limpia la bandera de interrupción (`PIRx`) dentro de la ISR, la interrupción se disparará continuamente al salir de la rutina con `RETFIE`.

## Referencias
[^vectors]: {{ q10('15.2', 192) }}
[^INTCON]: {{ q10('15.13.1', 195) }}
[^priority]: {{ q10('15.2', 192) }}
[^PIR]: {{ q10('15.5', 195) }}
[^PIE]: {{ q10('15.6', 195) }}
[^IPR]: {{ q10('15.7', 195) }}
[^regsummary]: {{ q10('15.12', 197) }}