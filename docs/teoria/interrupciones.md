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

## Referencias
[^vectors]: {{ q10('15.2', 192) }}