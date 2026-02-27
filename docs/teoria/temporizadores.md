---
title: Temporizadores (Timer 0)
description: Configuración y uso del módulo TMR0 del PIC18F47Q10 en modo 8 y 16 bits
---

# Temporizadores

Los Q10 y Q43 cuentan con tres tipos de módulos de temporización:

| Módulo | Resolución | Uso típico |
|:------:|:----------:|:----------:|
| TMR0 | 8 o 16 bits | General |
| TMR1 | 16 bits | Tiempo preciso, CCP |
| TMR2 | 8 bits | PWM, ADC |

A continuación se detalla el funcionamiento del módulo TMR0. Se mencionarán el TMR1/3/5 y TMR2/4/6 en los módulos que usan esos temporizadores.

---

## Timer 0 (TMR0)

<figure markdown="span" style="text-align: center;">
    ![Diagrama de bloque del TMR0](../assets/images/Timer0_Block_Diagram.png){width="80%" align=center}
<figcaption markdown="span">Diagrama de bloques del módulo TMR0 del PIC18F47Q10 [^1]</figcaption>
</figure>

El módulo TMR0 es un temporizador/contador de **8 o 16 bits** que incrementa su valor en cada flanco positivo de la fuente de reloj. Cuando llega al valor máximo ocurre un **overflow** (16 bits) o un **match** (8 bits); en ambos casos, se reinicia la cuenta.
Si un evento de overflow/match ocurre la cantidad de veces configurada en el postscaler, se levanta la bandera `TMR0IF` del registro `PIR0`[^PIR0] y la señal `T0_out` se alterna.
La cuenta actual está en los registros `TMR0L` y `TMR0H` (en 8 bits solo `TMR0L`).

??? info "TMR0 en el Q43"
    En el Q43, la bandera de interrupción `TMR0IF` se encuentra en el bit 7 del registro `PIR3` (banco 4) en lugar de `PIR0`. Análogamente, `TMR0IE` está en `PIE3` y `TMR0IP` en `IPR3`.

De este modo, la duración total de un ciclo del temporizador se puede calcular con:

$$
\text{Duracion} = \frac{\text{PreScaler} \times \text{Num. Cuentas} \times \text{PostScaler}}{F_{\text{TMR0}}}
$$

Pero, lo que normalmente se calcula es el valor de `TMR0L` y `TMR0H` para obtener la cantidad de cuentas deseada. Esto se calcula de forma diferente para  cada modo (8 o 16 bits), como se detalla a continuación.

### Modo 8 bits

<figure markdown="span">
  ![Diagrama de bloque del TMR0 en modo 8 bits](../assets/images/Timer0_Block_Diagram_8bit.png){ width="50%" }
  <figcaption>Diagrama de bloque del TMR0 en modo 8 bits</figcaption>
</figure>

Como se aprecia en su diagrama de bloques, el `TMR0L` es comparado con `TMR0H` en cada cuenta. Si son iguales, se genera un **match** y el `TMR0L` se reinicia a 0. Por lo tanto, en modo de 8 bits, **se puede contar hasta 254** porque al llegar a 255 se genera el match y se reinicia a 0. En este sentido, se afirma que:

$$
\text{Num. Cuentas} \leq 255 \land \mathtt{TMR0H} = \text{Num. Cuentas}
$$

$$
\mathtt{TMR0H} = \frac {\text{Duracion} \times F_{\text{T0CS}}} {\text{PreScaler} \times \text{PostScaler}} \leq 255
$$

??? example "Ejemplo"
    Si se escribe `100` en `TMR0H`, el `TMR0L` contará desde 0 hasta 99 (100 cuentas) y luego ocurrirá el match.

### Modo 16 bits

<figure markdown="span">
  ![Diagrama de bloque del TMR0 en modo 16 bits](../assets/images/Timer0_Block_Diagram_16bit.png){ width="50%" }
  <figcaption>Diagrama de bloque del TMR0 en modo 16 bits</figcaption>
</figure>

Como se ve en el diagrama de bloques, la cuenta inicia desde lo escrito en `TMR0H:TMR0L` y termina en `FFFF` (65 535). 
En este modo, para contar una cantidad específica de tiempo, se puede precargar un valor inicial en `TMR0H:TMR0L`.

$$ \text{Num. Cuentas} \leq 65536 \land \text{Num. Cuentas} = 65536 - \mathtt{TMR0H:L} $$

$$ \mathtt{TMR0H:L} = 65536 - \frac {\text{Duracion} \times F_{\text{T0CS}}} {\text{PreScaler} \times \text{PostScaler}} \geq 0$$

!!! warning "Orden de lectura / escritura"
    - **Leer:** primero `TMR0L`, luego `TMR0H`.
    - **Escribir:** primero `TMR0H`, luego `TMR0L`.

---

### Pasos para configurar el TMR0

Como guía práctica, se detallan los pasos necesarios para configurar el módulo TMR0:

1. Configurar `T0CON1`: seleccionar la fuente de reloj (`T0CS`), sincronía (`T0ASYNC`) y prescaler (`T0CKPS`).

    <figcaption markdown="span">Timer0 Control Register 1 (T0CON1)[^T0CON1]</figcaption>

    | Bit | 7:5 | 4 | 3:0 |
    |:---:|:---:|:---:|:---:|
    | **Campo** | T0CS[2:0] | T0ASYNC | T0CKPS[3:0] |
    | **Permisos** | R/W | R/W | R/W |
    | **Reset** | 000 | 0 | 0000 |

    <figcaption markdown="span">Fuentes de reloj del TMR0 (T0CS)[^clk]</figcaption>

    | T0CS[2:0] | Fuente |
    |:---------:|:-------|
    | 000 | T0CKIPPS |
    | 001 | $\overline{\text{T0CKIPPS}}$ |
    | 010 | Fosc/4 |
    | 011 | HFINTOSC |
    | 100 | LFINTOSC |
    | 101 | MFINTOSC 500 kHz |
    | 110 | SOSC |
    | 111 | CLC1_OUT |

    `T0ASYNC = 1` si la fuente es asíncrona con el reloj del sistema.

    `T0CKPS[3:0]` representa el prescaler de $2^{\text{T0CKPS}}$.

    ??? example "Ejemplo: Configuración de T0CON1"
        Para el Q10: Si quieres usar $F_{osc}/4$ como fuente de reloj, modo síncrono y prescaler de 1:16 ($2^4$):

        ```c
        T0CON1 = 0x24;         // Fosc/4, síncrono, prescaler 1:16
        ```
        ```asm
        movlw 0x24              ; 0010 0100
        movwf T0CON1, a         ; Fosc/4, síncrono, prescaler 1:16
        ```

2. Configurar `T0CON0`: seleccionar el modo (`T016BIT`) y el postscaler (`T0OUTPS`). **No habilitar aún** el timer (`T0EN = 0`).

    <figcaption markdown="span">Timer0 Control Register 0 (T0CON0)[^T0CON0]</figcaption>

    | Bit | 7 | 6 | 5 | 4 | 3:0 |
    |:---:|:---:|:---:|:---:|:---:|:---:|
    | **Campo** | T0EN | — | T0OUT | T016BIT | T0OUTPS[3:0] |
    | **Permisos** | R/W | — | RO | R/W | R/W |
    | **Reset** | 0 | — | 0 | 0 | 0000 |

    `T0EN` habilita el timer. 
    Y `T016BIT` habilita el modo de 16 bits.
    `T0OUT` es la salida del temporizador que se alterna al ocurrir la interrupción (solo lectura).

    Los bits `T0OUTPS[3:0]` representan el postscaler de `1:T0OUTPS+1`.

    ??? example "Ejemplo: Configuración de T0CON0"
        Para el Q10: Si quieres modo 8 bits, sin postscaler. **Sin habilitar aún** el timer (`T0EN = 0`):

        ```c
        T0CON0 = 0x00;         // 8 bits, postscaler 1:1, deshabilitado
        ```
        ```asm
        clrf T0CON0, a          ; 8 bits, postscaler 1:1, deshabilitado
        ```

3. Cargar la cuenta `TMR0` del timer:
    - Escribir el valor calculado para `TMR0H` según el modo (8 o 16 bits).
    - Si el modo es **16 bits**, escribir el valor calculado para `TMR0L`.

    ??? example "Ejemplo: Carga del timer (8 bits)"
        Para el Q10: Con la configuración de los ejemplos anteriores (modo 8 bits, sin postscaler, prescaler de 1:16 y reloj $\frac{F_{\text{osc}}}{4} = 1MHz$), si quieres configurar la duración máxima:

        $$
        \text{Duracion} = \frac{1}{F_{\text{T0CS}}} \times \text{PreScaler} \times \mathtt{TMR0H} \times \text{PostScaler} 
        = \frac {16 \times 255 \times 1}{1MHz} = 4.08\mu s 
        $$

        ```c
        TMR0H = 255;           // Cuenta máxima = 255 (TMR0L: 0 → 255)
        ```
        ```asm
        setf TMR0H, a          ; Cuenta máxima = 255 (TMR0L: 0 → 255)
        ```

4. **(Opcional) Interrupción:** Configurar los registros de interrupción del TMR0.
    - Bajar la bandera `TMR0IF` del registro `PIR0`.
    - Habilitar la interrupción `TMR0IE` del registro `PIE0`.
    - Configurar la prioridad `TMR0IP` del registro `IPR0` (`1` = alta prioridad).
    En todos los casos, se hace referencia al bit 5 de cada registro.

   
    ??? example "Ejemplo: Configuración de interrupción"
        Para el Q10: Habilitar la interrupción del TMR0 con alta prioridad:

        ```c
        PIR0bits.TMR0IF = 0;    // Bajar bandera de interrupción
        PIE0bits.TMR0IE = 1;    // Habilitar interrupción del TMR0
        IPR0bits.TMR0IP = 1;    // Alta prioridad
        ```
        ```asm
        movlb 0Eh               ; Banco 0Eh
        bcf PIR0, 5, b          ; Bajar bandera de interrupción
        bsf PIE0, 5, b          ; Habilitar interrupción del TMR0
        bsf IPR0, 5, b          ; Alta prioridad
        ```

        ??? info "Interrupción del TMR0 en el Q43"
            En el Q43, los bits `TMR0IF`, `TMR0IE` y `TMR0IP` se encuentran en los registros `PIR3` (bit 7), `PIE3` (bit 7) e `IPR3` (bit 7) respectivamente.


6. **(Opcional) Salida PPS:** Escribir en `RxyPPS` el valor `0x13` para redirigir `T0_out` a un pin[^PPS]. Solo disponible en puertos **RB** y **RC**. Divide ÷2 la frecuencia de `T0_out`.

    ??? example "Ejemplo: Configuración PPS"
        Para el Q10: Redirigir `T0_out` al pin RC0:

        ```c
        RC0PPS = 0x13;          // RC0 -> T0_out
        TRISCbits.TRISC0 = 0;   // RC0 como salida
        ```
        Todos los registros RBxPPS y RCxPPS del Q10 están en el banco `0Eh`.
        ```asm
        movlb 0Eh               ; Banco 0Eh
        movlw 0x13
        movwf RC0PPS, b         ; RC0 -> T0_out
        bcf TRISC, 0, a         ; RC0 como salida
        ```
        ??? info "PPS del TMR0 en el Q43"
            En el Q43, el código PPS para `T0_out` es `0x39` y solo está disponible en puertos **RC** y **RF**.

4. Habilitar el timer: activar el bit `T0EN` del registro `T0CON0`.

    ??? example "Ejemplo: Habilitar el TMR0"
        ```c
        T0CON0bits.T0EN = 1;    // Habilitar TMR0
        ```
        ```asm
        bsf T0CON0, 7, a        ; Habilitar TMR0 (T0EN = 1)
        ```

## Referencias
[^1]: {{ q10('19', 295) }}
[^T0CON0]: {{ q10('19.6.1', 300) }}
[^T0CON1]: {{ q10('19.6.2', 301) }}
[^clk]: {{ q10('19.2.1', 296) }}
[^TMR0]: {{ q10('19.6.3', 302) }}
[^PIR0]: {{ q10('15.13.2', 199) }}
[^PIE0]: {{ q10('15.13.10', 207) }}
[^IPR0]: {{ q10('15.13.18', 215) }}
[^PPS]: {{ q10('18.2', 287) }}
