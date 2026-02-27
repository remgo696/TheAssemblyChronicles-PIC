---
title: TMR0 – Guía Rápida
description: Guía rápida de configuración del módulo TMR0 del PIC18F57Q43 en modo 8 y 16 bits
keywords:
  - TMR0
  - Timer0
  - temporizador
  - PIC18F57Q43
  - Q43
  - T0CON0
  - T0CON1
  - prescaler
  - postscaler
tags:
  - TMR0
  - Timer0
  - guía rápida
  - PIC18F57Q43
---

# TMR0 – Guía Rápida

Para más detalles sobre el módulo TMR0, su configuración y funcionamiento, consultar la [guía completa](../teoria/temporizadores.md).

## Cálculos Previos

### Duración del temporizador

$$\text{duración} = \frac{4}{F_{osc}} \times \text{prescaler} \times \text{cuenta máxima} \times \text{postscaler}$$

### Modo 8 bits

- `TMR0L` cuenta desde 0 hasta `TMR0H - 1`. Al igualar `TMR0H` se genera un **match** y se reinicia.
- Cuenta máxima $\leq 254$. Cargar en `TMR0H` el valor $\text{cuenta máxima} + 1$.

### Modo 16 bits

- `TMR0H:TMR0L` cuenta hasta `FFFF` (65 535). Al desbordar ocurre un **overflow** y se reinicia a 0.
- Se puede precargar un valor inicial escribiendo en `TMR0H:TMR0L` con `EN = 0`.

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
| `PIR0` | Bandera de interrupción `TMR0IF` (bit 7) | `0x0E` |
| `PIE0` | Habilitación de interrupción `TMR0IE` (bit 7) | `0x0E` |
| `IPR0` | Prioridad de interrupción `TMR0IP` (bit 7) | `0x0E` |
| `RxyPPS` | Salida PPS de `T0_out` (`0x13`). Solo RB y RC. Divide ÷2 la frecuencia | — |

### T0CON0

| Bit 7 | Bit 5 | Bit 4 | Bits 3:0 |
|:-----:|:-----:|:-----:|:--------:|
| T0EN | T0OUT (RO) | T016BIT | T0OUTPS[3:0] |

- `T0EN = 1`: habilitar timer (la cuenta se **pausa** si `T0EN = 0`).
- `T016BIT = 1`: modo 16 bits / `T016BIT = 0`: modo 8 bits.
- `T0OUT`: salida `T0_out`, se alterna al ocurrir la interrupción.
- `T0OUTPS`: postscaler — cuenta $(\text{valor})$ eventos antes de levantar `TMR0IF`.

### T0CON1

| Bits 7:5 | Bit 4 | Bit 3 | Bits 2:0 |
|:--------:|:-----:|:-----:|:--------:|
| CS[2:0] | ASYNC | CKPS[3] | CKPS[2:0] |

- `CS[2:0]`: fuente de reloj.

| CS[2:0] | Fuente |
|:-------:|:-------|
| 000 | T0CKIPPS |
| 001 | $\overline{\text{T0CKIPPS}}$ |
| 010 | Fosc/4 |
| 011 | HFINTOSC |
| 100 | LFINTOSC |
| 101 | MFINTOSC 500 kHz |
| 110 | SOSC |
| 111 | CLC1_OUT |

- `ASYNC = 1`: fuente asíncrona / `0`: síncrona con el reloj del sistema.
- `CKPS[3:0]`: prescaler — divisor de frecuencia: $2^{(\text{valor})}$.

### TMR0H y TMR0L

| Registro | Reset | Descripción |
|:--------:|:-----:|:------------|
| `TMR0H` | `FFh` | Valor de comparación (8 bits) o byte alto (16 bits) |
| `TMR0L` | `00h` | Contador (8 bits) o byte bajo (16 bits) |

### Registros de interrupción

| Registro | Bit | Nombre | Descripción |
|:---------|:---:|:------:|:------------|
| `PIR3` (BSR = 4) | `<7>` | `TMR0IF` | Flag de interrupción |
| `PIE3` (BSR = 4) | `<7>` | `TMR0IE` | Habilita la interrupción |
| `IPR3` (BSR = 3) | `<7>` | `TMR0IP` | `1` = alta prioridad |

---

## Configuración básica

1. Configurar `T0CON1`: fuente de reloj (`CS`), prescaler (`CKPS`) y sincronía (`ASYNC`).
2. Configurar `T0CON0`: modo (`MD16`), postscaler (`OUTPS`).
3. **8 bits:** escribir en `TMR0H` el valor $\text{cuenta máxima} + 1$.
4. **16 bits:** (opcional) precargar `TMR0H:TMR0L` con `EN = 0`.
5. Habilitar el timer: `T0CON0.EN = 1`.
6. **(Opcional) Interrupción:** habilitar `PIE3<7>` y bajar `PIR3<7>` al atenderla.
7. **(Opcional) PPS:** escribir `0x39` en `RxyPPS` (solo RC y RF). Divide ÷2 la frecuencia de `T0_out`.
