---
title: CCP
description: Configuración y uso del módulo CCP del PIC18F47Q10 en modo PWM
---
# Captura/Comparación/PWM (CCP)

El módulo CCP (Capture/Compare/PWM) es un periférico versátil que se encuentra en los µC PIC18F, incluyendo el Q10 (CCP1/2) y Q43 (CCP1/2/3). El nombre se debe a los tres modos en los que opera. Por ahora, nos enfocaremos en el modo PWM (pulse width modulation). 

## Modo PWM

![Señal PWM](https://emariete.com/wp-content/uploads/2023/12/Diagrama-Senal-PWM.png){width="280" align=left}
Antes de nada, recuerdemos los dos elementos de una señal (cuadrada) PWM. Primero, el periodo es el tiempo que transcurre entre dos flancos ascendentes/descendentes consecutivos de la señal. Segundo, el tiempo de servicio (duty cycle) es el porcentaje de tiempo que la señal está en alto respecto a un período. Eso es todo.
<div style="clear: both;"></div> 

<figure markdown="span" style="text-align: center;">
    ![Diagrama de bloques del módulo CCP](../assets/images/ccp_block_diagram.png){width="80%" align=center}
<figcaption  markdown="span">Diagrama de bloques del módulo CCP del PIC18F47Q10 [^3]</figcaption>
</figure>

*Puedes saltearte la explicación del cómo funciona internamente el módulo CCP si quieres y continuar leyendo "Periodo de la señal PWM"*. Empecemos viendo el bloque del modulo TMR2 (timer asociado por defecto al CPPx). Primero, podemos ver que TMR2 (de 8 bits) cuenta hasta que alcanza el valor de `PRx` (referencia al registro `TxPR`). Cuando esto sucede (han pasado `PRx`$+1$ cuentas), TMR2 se reinicia a cero, manda una señal al *10-bit Latch* y *setea* el flip-flop SR. Así, el *10-bit Latch* se actualiza con el valor que representa el registro `CCPRx` y la señal PWM se mantiene en alto. Luego, cuando TMR2 concatenado a 2 bits generados por $F_OSC$ (o el prescaler) es igual al valor del *10-bit Latch* (han pasado `CCPRx` cuentas), se *resetea* el flip-flop SR, lo que hace que la señal PWM se ponga en bajo. Finalmente, el ciclo se repite, generando una señal PWM con un periodo definido por `PRx` y un *duty cycle* definido por `CCPRx`.

### Periodo de la señal PWM
El periodo de la señal PWM está definido por la cuenta máxima del temporizador TMR2/4/6 de 10 bits asociado al módulo CCPx. 

En el Q10, el timer se asocia al módulo CCPx mediante los bits `CxTSEL` del registro `CCPTMRS`. En el Q43, mediante los bits `CxTSEL` del registro `CCPTMRS0`.

<figcaption markdown="span">CCPx Timer Selection bits [^1] [^2]</figcaption>
| CxTSEL[1:0] | Timer Asociado |
| :---: | :---: |
| 01 | TMR2 |
| 10 | TMR4 |
| 11 | TMR6 |

??? info "`CCPTMRS` vs. `CCPTMRS0`"
    El Q10 comparte el registo `CCPTMRS` entre los módulos CCP1/2 y PWM3/4. En cambio, el Q43 usa el registro `CCPTMRS0` solo para los módulos CCP1/2/3. Esto se debe a que:
    - El Q43 tiene más módulos CCP
    - El Q43 tiene más módulos PWM, cada uno con su propio registro de selección de timer.
    Así, un solo registo no es suficiente para configurar todos los módulos CCP y PWM del Q43, mientras que sí lo es para el Q10.

Los timers TMR2/4/6 son de 8 bits, lo que significa que cuentan desde 0 hasta 255. Para configurar la cuenta máxima, se carga el byte más significativo el valor máximo deseado en el registro `TxPR` del módulo TMRx. 
Por ejemplo, si se carga el valor $200$, el timer contará desde $0$ hasta $200$ antes de reiniciarse, lo que define el periodo de la señal PWM. En ese tiempo, se realizan $200+1=201$ cuentas. 
De esta forma, el periodo de la señal PWM se puede calcular con la siguiente fórmula:

$$T_{PWM} = (TxPR + 1) \times T_{CLK\ INPUT} \times Prescaler$$

??? danger "El postcaler"
    El postscaler del TMRx no afecta el periodo de la señal PWM, ya que solo influye en la generación de interrupciones, no en el conteo del timer.

En el caso de que se use $\frac{F_{osc}}{4}$ como fuente de reloj para el timer, la fórmula del periodo de la señal PWM se puede expresar como:

$$T_{PWM} = \frac{4 \times (TxPR+1) \times TMRx\ Prescaler}{F_{osc}}$$

y, si sabes el periodo de la señal PWM que quieres generar, puedes despejar el valor de `TxPR`:

$$TxPR = \frac{T_{PWM} \times F_{osc}}{4 \times TMRx\ Prescaler} - 1$$

### Duty cycle de la señal PWM
![Registro CCPRx](../assets/images/ccp_register_pwm.png){width="280" align=left}
El tiempo en la señal PWM está activa se define por el valor que representa el registro `CCPRx`, conformado por `CCPRxH` y `CCPRxL`. Además, el bit `FMT` del registro `CCPxCON` define si *10-bit Duty Cycle* toma los 10 bits más significativos (`FMT = 1`) o los 10 bits menos significativos (`FMT = 0`) del registro `CCPRx`.

<div style="clear: both;"></div>

Por lo tanto, la duración en activo de la señal PWM es:

$$T_{ON} = CCPRx\ value\times \frac{T_{CLK\ INPUT}\times TMRx\ Prescaler}{4}$$

En concordancia con lo explicado sobre el bit `FMT`, `CCPRx value = CCPRxH[1:0]:CCPRxL` para `FMT = 0` y `CCPRx value = CCPRxH:CCPRxL[7:6]` para `FMT = 1`. Además, para el caso de que se use $\frac{F_{osc}}{4}$ como fuente de reloj para el timer, la duración en activo de la señal PWM será:

$$T_{ON} = CCPRx\ value\times \frac{TMRx\ Prescaler}{F_{osc}}$$



## Referencias
[^1]: {{ q10('22.6.4', 359) }}
[^2]: {{ q43('29.1.1', 463) }}
[^3]: {{ q10('22.4.1', 351) }}