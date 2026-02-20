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

??? info "Fórmula de verificación"
    $$T_{PWM} = \frac{4 \times (TxPR+1) \times TMRx\ Prescaler}{F_{osc}}$$

!!! warning "Fuente de reloj"
    Es menester que la fuente de reloj del timer sea $\frac{F_{osc}}{4}$ para el correcto funcionamiento del módulo CCP. El PostScaler **no** afecta el periodo PWM.

### Duty Cycle (Tiempo en alto)

$$CCPRx\text{ value} = \frac{T_{ON} \times F_{osc}}{TMRx\text{ Prescaler}}$$

??? info "Fórmula de verificación"
    $$T_{ON} = CCPRx\text{ value} \times \frac{TMRx\text{ Prescaler}}{F_{osc}}$$

- Si `FMT = 0` (derecha): `CCPRx value = CCPRxH[1:0]:CCPRxL`
- Si `FMT = 1` (izquierda): `CCPRx value = CCPRxH:CCPRxL[7:6]`

---

## Registros a usar

| Registro | Función | Referencia |
|:---------|:--------|:-----------|
| `RxyPPS` | Asigna pin de salida al módulo CCPx | Q10 §18.2 p.287 [^CCP_codes_Q10] |
| `TRISx` | Dirección del pin (entrada/salida) | — |
| `TxPR` | Cuenta máxima del timer → define periodo PWM | — |
| `CCPxCON` | Control del CCP: habilitar (`EN`), modo (`MODE`), formato (`FMT`) | Q10 §22.6.1 p.356 [^CCPxCON] |
| `CCPRxH:L` | Valor de 10 bits que define el duty cycle | — |
| `TxCLKCON` | Fuente de reloj del timer (`CS=1` → $F_{osc}/4$) | — |
| `TxCON` | Control del timer: habilitar (`ON`), prescaler (`CKPS`) | Q10 §21.9.3 p.341 [^TMRxCON] |
| `PIR4` | Banderas de interrupción TMR2/4/6 (`TMR2IF` bit 1) | Q10 §15.13.6 p.203 [^PIR4] |
| `CCPTMRS` | Selección del timer asociado al CCPx | Q10 §22.6.4 p.359 [^1] |

### PPS – Códigos de salida CCP

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

### Timer asociado (CxTSEL)

| CxTSEL[1:0] | Timer |
|:---:|:---:|
| 01 | TMR2 |
| 10 | TMR4 |
| 11 | TMR6 |

---

## Configuración rápida

No evita "pulsos basura" durante la configuración.

1. Asignar pin al CCPx vía PPS (`RxyPPS`).
2. Configurar pin como **salida** (`TRISx = 0`).
3. Cargar periodo en `TxPR`.
4. Configurar `CCPxCON`: modo PWM + formato (`EN`, `MODE`, `FMT`).
5. Cargar duty cycle en `CCPRxH:L`.
6. Seleccionar reloj del timer: `TxCLKCON = 1` ($F_{osc}/4$).
7. Configurar prescaler y habilitar timer: `TxCON` (`ON = 1`).

---

## Configuración segura

Evita "pulsos basura" configurando el pin como entrada hasta que el timer esté listo[^setup_pwm].

1. Asignar pin al CCPx vía PPS (`RxyPPS`).
2. Configurar pin como **entrada** (`TRISx = 1`).
3. Cargar periodo en `TxPR`.
4. Configurar `CCPxCON`: modo PWM + formato (`EN`, `MODE`, `FMT`).
5. Cargar duty cycle en `CCPRxH:L`.
6. Bajar bandera `TMRxIF` en `PIR4`.
7. Seleccionar reloj del timer: `TxCLKCON = 1` ($F_{osc}/4$).
8. Configurar prescaler y habilitar timer: `TxCON` (`ON = 1`).
9. Esperar a que `TMRxIF == 1`.
10. Habilitar salida del pin (`TRISx = 0`).

---

## Referencias
[^1]: {{ q10('22.6.4', 359) }}
[^CCP_codes_Q10]: {{ q10('18.2', 287) }}
[^CCPxCON]: {{ q10('22.6.1', 356) }}
[^PIR4]: {{ q10('15.13.6', 203) }}
[^TMRxCON]: {{ q10('21.9.3', 341) }}
[^setup_pwm]: {{ q10('22.4.2', 351) }}
