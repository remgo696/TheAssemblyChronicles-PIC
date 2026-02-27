---
title: TMR0 – Guía Rápida
description: Guía rápida de configuración del módulo TMR0 del PIC18F47Q10 en modo 8 y 16 bits
keywords:
  - TMR0
  - Timer0
  - temporizador
  - PIC18F47Q10
  - Q10
  - T0CON0
  - T0CON1
  - prescaler
  - postscaler
tags:
  - TMR0
  - Timer0
  - guía rápida
  - PIC18F47Q10
---

# TMR0 – Guía Rápida

Para más detalles sobre el módulo TMR0, su configuración y funcionamiento, consultar la [guía completa](../teoria/temporizadores.md).

## Cálculos Previos

### Duración del temporizador

$$
\text{Duracion} = \frac{\text{PreScaler} \times \text{Num. Cuentas} \times \text{PostScaler}}{F_{\text{TMR0}}}
$$

### Modo 8 bits
$$
\mathtt{TMR0H} = \frac {\text{Duracion} \times F_{\text{TMR0}}} {\text{PreScaler} \times \text{PostScaler}} \leq 255
$$

### Modo 16 bits

$$ \mathtt{TMR0H:L} = 65536 - \frac {\text{Duracion} \times F_{\text{TMR0}}} {\text{PreScaler} \times \text{PostScaler}} \geq 0$$

!!! warning "Orden de lectura / escritura (16 bits)"
    - **Leer:** primero `TMR0L`, luego `TMR0H`.
    - **Escribir:** primero `TMR0H`, luego `TMR0L`.

---

## Registros a usar

| Registro | Función | Banco |
|:---------|:--------|:------:|
| `T0CON0` | Control: habilitar (`T0EN`), modo (`T016BIT`), postscaler (`T0OUTPS`) | Access |
| `T0CON1` | Fuente de reloj (`T0CS`), sincronía (`T0ASYNC`), prescaler (`T0CKPS`) | Access |
| `TMR0H` | Valor de comparación (8 bits) / byte alto del contador (16 bits) | Access |
| `TMR0L` | Contador (8 bits) / byte bajo del contador (16 bits) | Access |
| `PIR0` | Bandera de interrupción `TMR0IF` (bit 5) | `0x0E` |
| `PIE0` | Habilitación de interrupción `TMR0IE` (bit 5) | `0x0E` |
| `IPR0` | Prioridad de interrupción `TMR0IP` (bit 5) | `0x0E` |
| `RxyPPS` | Salida PPS de `T0_out` (`0x13`). Solo RB y RC. Divide ÷2 la frecuencia | `0x0E` |

### T0CON0

| Bit 7 | Bit 5 | Bit 4 | Bits 3:0 |
|:-----:|:-----:|:-----:|:--------:|
| T0EN | T0OUT (RO) | T016BIT | T0OUTPS[3:0] |

- `T0EN = 1`: habilitar timer.
- `T016BIT = 1`: modo 16 bits / `T016BIT = 0`: modo 8 bits.
- `T0OUT`: salida `T0_out`, se alterna al ocurrir cada interrupción.
- `T0OUTPS`: postscaler — cuenta $(\text{valor}+1)$ eventos antes de levantar `TMR0IF`.

### T0CON1

| Bits 7:5 | Bit 4 | Bits 3:0 |
|:--------:|:-----:|:--------:|
| T0CS[2:0] | T0ASYNC | T0CKPS[3:0] |

- `T0CS[2:0]`: Selecciona la fuente de reloj.  
- `T0ASYNC = 1`: fuente asíncrona con el reloj del sistema.
- `T0CKPS[3:0]`: prescaler — divisor de frecuencia de $2^{(\text{valor})}$.

??? info "T0CS[2:0]: Fuentes de reloj"
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


---

## Configuración básica

1. Configurar `T0CON1`: fuente de reloj (`T0CS`), prescaler (`T0CKPS`) y sincronía (`T0ASYNC`).
2. Configurar `T0CON0`: modo (`T016BIT`), postscaler (`T0OUTPS`).
3. **8 bits:** escribir en `TMR0H` el valor $\text{cuenta máxima} + 1$.
4. **16 bits:** (opcional) precargar `TMR0H:TMR0L` con `EN = 0`.
5. Habilitar el timer: `T0CON0.T0EN = 1`.

## Configuraciones opcionales

6. **Interrupción:** bajar `PIR0.TMR0IF`, habilitar `PIE0.TMR0IE` y (opscional) configurar `IPR0.TMR0IP` para la prioridad. No olvidar tener configurado el INTCON para habilitar las interrupciones.
7. **PPS:** escribir `0x13` en `RxyPPS` (solo RB y RC). Divide ÷2 la frecuencia de `T0_out`.
