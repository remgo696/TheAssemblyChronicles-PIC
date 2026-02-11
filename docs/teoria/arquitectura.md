---
title: Arquitectura de un PIC18F
description: Explica cómo se relaciona el µC con sus registros, memoria y periféricos
keywords:
  - PIC18F57Q43
  - PIC18F47Q10
  - microcontrolador
tags:
  - PIC
---

# Arquitectura de un PIC18F

## La arquitectura Harvard
![Arquitectura Harvard](../assets/images/arquitectura_Harvard.svg){width="280" align=left}
Los µC PIC siguen la arquitectura Harvard. Esto significa que tienen buses separados para instrucciones y datos, lo que permite acceder a ambos simultáneamente. 
<div style="clear: both;"></div> 

En el diagrama de bloques del Q10 encontrarás el bus de datos (en rojo) y el de instruccione (en verde) separados. Por esto, se dice que es una arquitectura Harvard.

<figure markdown="span" style="text-align: center;">
  ![FamilyBlockDiagram_Q10](../assets/images/arquitectura_Q10.png)
  <figcaption  markdown="span">Diagrama de bloques de la familia PIC18F27/47Q10 [^1]</figcaption>
</figure>

Lo mismo pasa con el Q43, pero no te voy a colorear la diferencia. Lo que sí, es importante destacar las aparentes diferencias que en realidad no lo son. Por ejemplo, el bus de datos del Q10 va conectado a sus periféricos de entrada/salida (PORT) y demás módulos, mientras que el del Q43, a pesar de también hacerlo, lo obvia en su diagrama. 

<figure markdown="span" style="text-align: center;">
  ![FamilyBlockDiagram_Q43](../assets/images/arquitectura_Q43.png){width="80%"}
  <figcaption markdown="span">Diagrama de bloques de la familia PIC18F27/47/57Q43 [^2] </figcaption>
</figure>


## Memoria y registros
---
Dado que el Q10 y Q43 son de 8 bits, su bus de datos es de 8 bits, lo que implica que sus registros y memoria también son de 8 bits (1 byte ⁓ hexadecimal de 2 dígitos). A continuación, se detalla la división de memoria de ambos µC:

<figcaption markdown="span">Distribución de memoria en los Q43 y Q10</figcaption>

| Tipo de Memoria | Tecnología | Uso Principal | 18F57Q43 | 18F47Q10 |
| :--- | :--- | :--- | :---: | :---: |
| **Program Flash** | No Volátil | Almacenamiento del código (instrucciones) | **128 KB** | **128 KB** |
| **Data SRAM** | Volátil | Variables dinámicas y *Software Stack* | **8 KB** | **⁓3,5 KB** |
| **Data EEPROM** | No Volátil | Almacenamiento de datos persistentes | **1 KB** | **1 KB** |

!!! failure "Uso del EEPROM"
    A pesar de su encanto, no veremos cómo manejar la EEPROM en este curso. Y, a fines prácticos, se entenderá como un periférico de almacenamiento.
   
### Memoria de programa
La memoria de programa es donde se almacenan las **intrucciones** que componen el programa que ejecuta el  µC. Siempre es no volátil (no se borra al apagar el dispositivo) y, para el caso del Q10 y Q43 es tipo **Flash**.

En la tabla anterior se muestra que ambos µC tienen 64 kW (1 word $=$ 2 bytes) de memoria de programa, que puede guardar hasta $65 536 = 1 024\times 64$ instrucciones. Esto se debe a que, las instrucciones ocupan, por lo general, una palabra de la memoria de programa. Para el Q10, las excepciones son las instrucciones `MOVFF`, `GOTO`, `CALL` y `LFSR`, que ocupan dos palabras. En adición de esas intrucciones, el Q43 tiene la instrucción `MOVFFL` que ocupa tres palabras.

En el Q10, la memoria de programa abarca el rango de direcciones desde `00 0000h` hasta `01 FFFFh`. Y, como se muestra en la tabla siguiente, también incluye los vectores de interrupción y el vector de reset.
El vector de reset es la dirección a la que el µC salta para iniciar la ejecución del programa después de un reinicio. En la mayoría de los casos (el Q10 y Q43 inclusive), esta dirección es `00 0000h`. Por esto, es común colocar una instrucción de salto en `00 0000h` que redirija a la dirección donde realmente se encuentra el programa. Pero, ¿dónde debería empezar el programa? Al menos, no debe de interponerse con los vectores de interrupción. Por lo tanto, el programa del Q10 puede empezar a partir de `00 001Ah` o más allá. En pic-as (ensamblador), esto se traduce en:
```assembly
  ORG 00h ; La siguiente instrucción estará en el vector de reset
  bra 1Ah ; Salto a donde empieza el programa
```
O sea, que en la dirección `00h` (vector de reset) se coloca la instrucción de salto (`bra`) que redirige a la dirección `1Ah`, donde realmente empieza el programa. Aunque, por razones de estilo, algunos profesores prefieren colocar el programa a partir de `00 0020h` o `00 0100h`. Además, se puede usar una etiqueta para marcar el inicio del programa, como `inicio`, y luego usar esa etiqueta en la instrucción de salto:
```assembly
  ORG 00h ; La siguiente instrucción estará en el vector de reset
  bra main ; Salto a donde empieza el programa
  ORG 1Ah ; La siguiente instrucción estará en la dirección 1Ah
main:
  ; Aquí va el código del programa
```

<figcaption markdown="span">Mapa de memoria de programa y datos del Q10 [^3]</figcaption>
| Rango de Direcciones | Sección de Memoria | Detalles / Tamaño |
| :--- | :--- | :--- |
| `00 0000h` | **Reset Vector** | Inicio del programa |
| `00 0008h` | **Interrupt Vector High** | Interrupciones de Alta Prioridad |
| `00 0018h` | **Interrupt Vector Low** | Interrupciones de Baja Prioridad |
| `00 001Ah` - `01 FFFFh` | **Program Flash Memory** | 64 KW (Espacio de código) |
| `30 0000h` - `30 000Bh` | **Configuration Words** | 6 Words (Configuración de Fusibles) |
| `31 0000h` - `31 00FFh` | **Data EEPROM** | 1024 Bytes (Datos No Volátiles) |



## Referencias
[^1]: Microchip Technology Inc. (2020). *PIC18F27/47Q10 Microcontroller Data Sheet*. Sección 2.3, "Details on Individual Family Members". [Ver PDF (Pág. 18)](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU08/ProductDocuments/DataSheets/PIC18F27-47Q10-Micorcontroller-Data-Sheet-DS40002043.pdf#page=18).
[^2]: Microchip Technology Inc. (2024). *PIC18F57Q43 Microcontroller Data Sheet*. Sección 7, "PIC18 CPU". [Ver PDF (Pág. 28)](https://ww1.microchip.com/downloads/aemDocuments/documents/MCU08/ProductDocuments/DataSheets/PIC18F27-47-57Q43-Microcontroller-Data-Sheet-XLP-DS40002147.pdf#page=28).
[^3]: {{ q10('11.1', 108) }}