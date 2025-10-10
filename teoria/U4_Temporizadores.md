<!-- Fecha de última modificación: 10/10/2025 -->
### 🟩 Unidad 4 – Temporizadores
- **TMR0**, **TMR1**, **TMR2**: diferencias y configuración básica  
- Generación de retardos con **TMR0**  
- Base de tiempo y eventos periódicos  
- Introducción a interrupciones por **TMR0**  

 ---

TMR0 | TMR1 | TMR2
-----|------|-----
8 o 16 bits | 16 bits | 8 bits
General | Tiempo preciso | PWM, ADC


# Timer 0 (TMR0)
![Diagrama de bloque del TMR0](../recursos/datasheet/Timer0_Block_Diagram.png)

## Descripción general
- Es un temporizador/contador de 8 o 16 bits que incrementa su valor en cada flanco positivo de la fuente de reloj.
- Cuando llega al valor máximo ocurre un overflow en el caso del contador de 16 bits o un match en el caso del contador de 8 bits. En ambos casos, se reinicia la cuenta.
- Si un evento de overflow/match ocurre la cantidad de veces configurada en el postscaler, se levanta la bandera T0IF (PIR3<7>) y T0_out (CON) se alterna.
- La cuenta actual está en los registros TMR0L y TMR0H (en 8 bits solo TMR0L).
- La configuración básica es:
    - Configurar T0CON1 (fuente de reloj) y T0CON0 (modo y prescaler).
    - 8bits: Escribir en TMR0H el valor final + 1.
    - 16bits: La cuenta máxima es 65535. Se puede inicializar la cuenta escribiendo en TMR0H:TMR0L con EN=0.
    - PPS: Escribir en RxyPPS 0x39. Solo para RC y RF. Divide ÷2 la frecuencia de T0_out.
    - T0IF: Encender PIE3<7>. Bajar PIR3<7> al atender la interrupción.

$$ duración = \frac{4}{F_{osc}}\times prescaler \times cuenta\ máxima \times postscaler$$

## 8-Bit mode
Como se aprecia en su diagrama de bloques, el TMR0L es comparado son TMR0H en cada cuenta. Si son iguales, se genera un match y el TMR0L se reinicia a 0. Por lo tanto, en modo de 8 bits, **se puede contar hasta 254** porque al llegar a 255 se genera el match y se reinicia a 0.
$$cuenta\ máxima \leq 254$$
Para configurar la $cuenta\ máxima$, se debe escribir en TMR0H el $valor\ final+1$ que se desea. Por ejemplo, si se escribe 100, el TMR0L contará desde 100 hasta 254 (155 cuentas) y luego ocurrirá el match.
<div align="center">
<img src="../recursos/datasheet/Timer0_Block_Diagram_8bit.png" alt="Diagrama de bloque del TMR0 en modo 8 bits" width="50%">
</div>

## 16-Bit mode
La cuenta máxima en modo de 16 bits es 65535, ya que TMR0H:TMR0L incrementa hasta llegar a su valor máximo (FFFF) y luego ocurre un overflow, reiniciándose ambos registros a 0.

Para leer TMR0 en modo de 16 bits, se debe leer primero TMR0L y luego TMR0H. Para escribir, se debe escribir primero TMR0H y luego TMR0L.

<div align="center">
<img src="../recursos/datasheet/Timer0_Block_Diagram_16bit.png" alt="Diagrama de bloque del TMR0 en modo 16 bits" width="50%">
</div>

## Registros para configurar el TMR0
### T0CON0 (BSR=3)
| Bit | 7 | 6 | 5   | 4    | 3:0 |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **Name** | EN | — | OUT | MD16 | OUTPS[3:0] |
| **Access** | R/W | — | R | R/W | R/W |
| **Reset** | 0 | — | 0 | 0 | 0 |

- **EN:** Habilita el temporizador. La cuenta se pausa si se deshabilita.
- **OUT:** T0_out. Salida del temporizador que se alterna al ocurrir la interrupción.  
- **MD16:** Selecciona modo 8 o 16 bits (1 = 16 bits).  
- **OUTPS[3:0]:** Postescalador de salida. Cuenta ${(valor)}$ eventos.

### T0CON1 (BSR=3)
| Bit | 7-5   | 4    | 3 | 2-0 |
|:---:|:---:|:---:|:---:|:---:|
| **Name** | CS[2:0] | ASYNC | CKPS[3] | CKPS[2:0] |
| **Access** | R/W | R/W | R/W | R/W |
| **Reset** | 0 | 0 | 0 | 0 |

- **CS[2:0]:** Selecciona la fuente de reloj (CLC1_OUT, SOSC, MFINTOSC 500kHz, LFINTOSC, HFINTOSC, Fosc/4, $\overline{\text{T0CKIPPS}}$, T0CKIPPS).
- **ASYNC:** Selecciona si la fuente de reloj es asíncrona (1) o síncrona (0) con el reloj del sistema.
- **CKPS[3:0]:** Prescaler. Divisor de frecuencia entre $2^{(valor)}$.

### TMR0H y TMR0L (BSR=3)
| Bit | 7-0 | 7-0 |
|:---:|:---:|:---:|
| **Name** | TMR0H[7:0] | TMR0L[7:0] |
| **Access** | R/W | R/W |
| **Reset** | FFH | 00H |


### PIRn (BSR=4, a), PIE (BSR=4, a), IPR (BSR=3)
- PIR3<7>: TMR0IF. Flag de interrupción.
- PIE3<7>: TMR0IE. Habilita la interrupción.
- IPR3<7>: TMR0IP. 1: Alta prioridad de interrupción.