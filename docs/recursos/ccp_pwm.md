---
title: CCP – Guía Rápida PWM
description: Guía rápida de configuración del módulo CCP en modo PWM para el PIC18F47Q10 en Assembly y C
keywords:
  - CCP
  - PWM
  - PIC18F47Q10
  - Q10
  - CCPxCON
  - TMR2
  - duty cycle
  - periodo PWM
tags:
  - CCP
  - PWM
  - guía rápida
  - PIC18F47Q10
---

# CCP – Guía Rápida (Modo PWM)

## Cálculos Previos

### Periodo PWM

$$TxPR = \frac{T_{PWM} \times F_{osc}}{4 \times TMRx\ Prescaler} - 1$$

??? info "Fórmula de verificación[^f_Tpwm]"
    $$T_{PWM} = \frac{4 \times (TxPR+1) \times TMRx\ Prescaler}{F_{osc}}$$

!!! warning "Fuente de reloj"
    Es menester que la fuente de reloj del timer sea $\frac{F_{osc}}{4}$ para el correcto funcionamiento del módulo CCP. El PostScaler **no** afecta el periodo PWM.

### Duty Cycle (Tiempo en alto)

$$\text{CCPRx value} = \frac{T_{ON} \times F_{osc}}{TMRx\text{ Prescaler}}$$

??? info "Fórmula de verificación[^f_Ton]"
    $$T_{ON} = CCPRx\text{ value} \times \frac{TMRx\text{ Prescaler}}{F_{osc}}$$

- Si `FMT = 0` (derecha): `CCPRx value = CCPRxH[1:0]:CCPRxL`
- Si `FMT = 1` (izquierda): `CCPRx value = CCPRxH:CCPRxL[7:6]`

---

## Registros a usar

| Registro | Función | Banco |
|:---------|:--------|:------:|
| `RxyPPS`[^CCP_codes_Q10] | Asigna pin de salida al módulo CCPx | `0x0E` |
| `TRISx` | Dirección del pin (entrada/salida) | Access |
| `TxPR` | Cuenta máxima del timer → define periodo PWM | Access |
| `CCPxCON`[^CCPxCON] | Control del CCP: habilitar (`EN`), modo (`MODE`), formato (`FMT`) | Access |
| `CCPRxH:L` | Valor de 10 bits que define el duty cycle | Access |
| `TxCLKCON` | Fuente de reloj del timer (`TxCLKCON=1` → $F_{osc}/4$) | Access |
| `TxCON`[^TMRxCON] | Control del timer: habilitar (`ON`), prescaler (`CKPS`) | Access |
| `PIR4`[^PIR4] | Banderas de interrupción TMR2/4/6 (`TMR2IF` bit 1) | `0x0E` |
| `CCPTMRS`[^1] | Selección del timer asociado al CCPx | Access |

### CCPTMRS: Bits CxTSEL

| Bits 3:2 | Bits 1:0 |
|:---------:|:---------:|
| C2TSEL[1:0] | C1TSEL[1:0] |

- Seleccionar TMR2 (`CxTSEL = 00`), TMR4 (`CxTSEL = 10`) o TMR6 (`CxTSEL = 11`)

### PPS: Códigos de salida CCP

| RxyPPS | Módulo | Puertos destino |
|:------:|:------:|:---------------:|
| 0x05 | CCP1 | RB, RC |
| 0x06 | CCP2 | RB, RC |

### CCPxCON

| Bit 7 | Bit 5 | Bit 4 | Bits 3:0 |
|:-----:|:-----:|:-----:|:--------:|
| EN | OUT (RO) | FMT | MODE[3:0] |

- Modo PWM: `MODE = 11xx`
- `FMT = 1`: justificado a la izquierda / `FMT = 0`: derecha



---

## Configuración rápida

No evita "pulsos basura" durante la configuración.

0. Seleccionar timer (TMR2/4/6) con [`CCPTMRS`](#ccptmrs-bits-cxtsel)[^1].
1. Asignar vía PPS un pin al CCP1 ([`RxyPPS=5`](#pps-codigos-de-salida-ccp)) o al CCP2 ([`RxyPPS=6`](#pps-codigos-de-salida-ccp)) [^CCP_codes_Q10].
2. Configurar el pin como **salida** (`TRISxbits.TRISxy = 0`).
3. Cargar el [periodo calculado](#periodo-pwm) en `TxPR`.
4. Configurar [`CCPxCON`](#ccpxcon): modo PWM + formato (`EN`, `MODE`, `FMT`)[^CCPxCON].
5. Cargar duty cycle [calculado](#duty-cycle-tiempo-en-alto) (10 bits) en `CCPRxH:L`.
6. Seleccionar reloj del timer: `TxCLKCON = 1` ($\frac{F_{osc}}{4}$).
7. Configurar prescaler y habilitar timer: `TxCON` (`ON = 1`)[^TMRxCON].

---

## Configuración segura

Evita "pulsos basura" configurando el pin como entrada hasta que el timer esté listo[^setup_pwm].

0. Seleccionar timer (TMR2/4/6) con [`CCPTMRS`](#ccptmrs-bits-cxtsel)[^1].
1. Asignar vía PPS un pin al CCP1 ([`RxyPPS=5`](#pps-codigos-de-salida-ccp)) o al CCP2 ([`RxyPPS=6`](#pps-codigos-de-salida-ccp)) [^CCP_codes_Q10].
2. Configurar el pin como **entrada** (`TRISxbits.TRISxy = 1`).
3. Cargar el [periodo calculado](#periodo-pwm) en `TxPR`.
4. Configurar [`CCPxCON`](#ccpxcon): modo PWM + formato (`EN`, `MODE`, `FMT`)[^CCPxCON].
5. Cargar duty cycle [calculado](#duty-cycle-tiempo-en-alto) (10 bits) en `CCPRxH:L`.
6. Bajar bandera `TMRxIF` en `PIR4`[^PIR4].
7. Seleccionar reloj del timer: `TxCLKCON = 1` ($\frac{F_{osc}}{4}$).
8. Configurar prescaler y habilitar timer: `TxCON` (`ON = 1`)[^TMRxCON].
9. Esperar a que `TMRxIF == 1`.
10. Habilitar salida del pin (`TRISxbits.TRISxy = 0`).

---

## Referencias
[^1]: {{ q10('22.6.4', 359) }}
[^CCP_codes_Q10]: {{ q10('18.2', 287) }}
[^CCPxCON]: {{ q10('22.6.1', 356) }}
[^PIR4]: {{ q10('15.13.6', 203) }}
[^TMRxCON]: {{ q10('21.9.3', 341) }}
[^setup_pwm]: {{ q10('22.4.2', 351) }}
[^f_Tpwm]: {{ q10('22.4.4', 352) }}
[^f_Ton]: {{ q10('22.4.5', 353) }}
