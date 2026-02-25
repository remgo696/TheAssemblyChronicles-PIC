# Librería LCD1602

Librería para el control de pantallas LCD de 16x2 caracteres con interfaz paralela de 4 bits en microcontroladores PIC18F.

## 📋 Características

- Interfaz de 4 bits (modo ahorro de pines)
- **Puerto de control y puerto de datos configurables por separado** mediante dos `#define` en `LCD.h`
- Compatible con LCD HD44780 y compatibles
- Funciones para control de cursor, escritura de texto y valores numéricos
- Creación de caracteres personalizados
- Desplazamiento de texto

## 🔧 Instalación

### 1. Copiar los archivos

Copia `LCD.c` y `LCD.h` a tu proyecto en MPLAB X IDE:

```
Tu_Proyecto/
├── main.c
├── LCD.c
└── LCD.h
```

### 2. Incluir la librería

```c
#include "LCD.h"
```

### 3. Configurar los puertos

En `LCD.h` encontrarás **dos defines** que controlan en qué puerto físico se conectan los pines de control y los de datos. Ambos pueden apuntar al mismo puerto o a puertos distintos:

```c
#define LCD_CTRL_PORT   D       // Puerto para RS, RW, E
#define LCD_DATA_PORT   D       // Puerto para D4, D5, D6, D7
```

**Pines de control** (3 pines consecutivos del puerto `LCD_CTRL_PORT`):

| Bit del puerto | Función LCD |
|:---------------|:------------|
| bit 0 | RS (Register Select) |
| bit 1 | RW (Read/Write) |
| bit 2 | E (Enable) |

**Pines de datos** (nibble alto del puerto `LCD_DATA_PORT`):

| Bit del puerto | Función LCD |
|:---------------|:------------|
| bit 4 | D4 |
| bit 5 | D5 |
| bit 6 | D6 |
| bit 7 | D7 |

> **Ejemplo — puertos separados:**
> Si quieres los pines de control en PORTB y los datos en PORTD:
> ```c
> #define LCD_CTRL_PORT   B
> #define LCD_DATA_PORT   D
> ```
> Esto asigna RB0→RS, RB1→RW, RB2→E y RD4-RD7→D4-D7.

### ⚠️ Advertencia — Pines del debugger

| Microcontrolador | Pines compartidos con debugger |
|:-----------------|:-------------------------------|
| **PIC18F47Q10** | **RD0** y **RD1** |
| **PIC18F57Q43** | **RF0** y **RF1** |

Los pines del debugger coinciden con los bits 0 y 1 del puerto, que son **RS** y **RW** en esta librería. Si `LCD_CTRL_PORT` se configura como `D` (en el Q10) o `F` (en el Q43), **el debugger on-board no funcionará correctamente**.

**Soluciones:**

- Usar un puerto diferente para los pines de control (ej. `B` o `C`).
- Aceptar que no se podrá usar el debugger on-board.

### 4. Configurar la frecuencia del oscilador

En `LCD.h`, verifica que `_XTAL_FREQ` coincida con tu reloj:

```c
#define _XTAL_FREQ 4000000UL    // 4 MHz (ajustar según tu configuración)
```

### 5. Inicializar el LCD

`LCD_INIT()` configura automáticamente los pines de ambos puertos como salidas digitales (TRIS y ANSEL), por lo que **no necesitas configurar TRISD/ANSELD manualmente** para los pines del LCD:

```c
void main(void) {
    LCD_INIT();     // Configura pines y arranca el LCD

    // Tu código aquí
    while(1) {
        // ...
    }
}
```

## 📖 Uso básico

### Escribir texto

```c
ESCRIBE_MENSAJE("Hola Mundo", 10);     // Con tamaño explícito
ESCRIBE_MENSAJE2("PIC18F57Q43");       // Calcula tamaño automáticamente
```

### Posicionar el cursor

```c
POS_CURSOR(1, 0);           // Fila 1, columna 0
ESCRIBE_MENSAJE2("Linea 1");
POS_CURSOR(2, 0);           // Fila 2, columna 0
ESCRIBE_MENSAJE2("Linea 2");
```

Filas: **1–4** (LCD 16x2 usa 1 y 2). Columnas: **0–15**.

### Borrar pantalla

```c
BORRAR_LCD();
```

### Escribir números

```c
LCD_ESCRIBE_VAR_CHAR(25, 2);       // Escribe "25" (unsigned char, 2 dígitos)
LCD_ESCRIBE_VAR_INT(1234, 4);      // Escribe "1234" (unsigned int, 4 dígitos)
```

### Control del cursor y pantalla

Las funciones de activación/desactivación usan `bool` (`true`/`false` de `<stdbool.h>`, incluido automáticamente por la librería):

```c
CURSOR_HOME();              // Cursor al inicio
CURSOR_ONOFF(true);         // Mostrar cursor
CURSOR_ONOFF(false);        // Ocultar cursor
BLINK_CURSOR(true);         // Cursor parpadeante
BLINK_CURSOR(false);        // Cursor fijo
DISPLAY_ONOFF(true);        // Encender pantalla
DISPLAY_ONOFF(false);       // Apagar pantalla
DISPLAY_SHIFTLEFT();        // Desplazar contenido ←
DISPLAY_SHIFTRIGHT();       // Desplazar contenido →
```

## 📚 Ejemplo completo

```c
#include <xc.h>
#include "LCD.h"

void main(void) {
    // LCD_INIT configura TRIS y ANSEL de los puertos seleccionados
    LCD_INIT();

    POS_CURSOR(1, 0);
    ESCRIBE_MENSAJE2("  PIC18F57Q43  ");
    POS_CURSOR(2, 0);
    ESCRIBE_MENSAJE2(" Assembly Code ");

    __delay_ms(2000);
    BORRAR_LCD();

    unsigned int contador = 0;
    while(1) {
        POS_CURSOR(1, 0);
        ESCRIBE_MENSAJE2("Contador:");
        POS_CURSOR(2, 5);
        LCD_ESCRIBE_VAR_INT(contador, 4);
        contador++;
        if(contador > 9999) contador = 0;
        __delay_ms(500);
    }
}
```

## 🔍 Referencia de funciones

| Función | Descripción |
|:--------|:------------|
| `LCD_INIT()` | Configura pines (TRIS/ANSEL) e inicializa el LCD |
| `BORRAR_LCD()` | Borra la pantalla completa |
| `POS_CURSOR(fila, col)` | Posiciona el cursor (fila 1-4, col 0-15) |
| `CURSOR_HOME()` | Cursor al inicio |
| `CURSOR_ONOFF(bool)` | Muestra/oculta cursor (`true`/`false`) |
| `BLINK_CURSOR(bool)` | Cursor parpadeante (`true`/`false`) |
| `DISPLAY_ONOFF(bool)` | Enciende/apaga pantalla (`true`/`false`) |
| `CURSOR_SHIFTLEFT()` | Mueve cursor ← |
| `CURSOR_SHIFTRIGHT()` | Mueve cursor → |
| `DISPLAY_SHIFTLEFT()` | Desplaza contenido ← |
| `DISPLAY_SHIFTRIGHT()` | Desplaza contenido → |
| `ENVIA_CHAR(dato)` | Envía un carácter |
| `ESCRIBE_MENSAJE(cadena, tam)` | Escribe texto (tamaño explícito) |
| `ESCRIBE_MENSAJE2(cadena)` | Escribe texto (tamaño automático) |
| `LCD_ESCRIBE_VAR_CHAR(num, dig)` | Escribe `unsigned char` con N dígitos |
| `LCD_ESCRIBE_VAR_INT(num, dig)` | Escribe `unsigned int` con N dígitos |
| `LCD_CHAR_GRADO()` | Escribe el símbolo ° |
| `LCD_VARCHAR_BITS(dato)` | Muestra un byte en binario |
| `GENERACARACTER(vector, pos)` | Crea carácter personalizado (CGRAM) |


## 📝 Créditos

- **Desarrollo original:** Sergio Salas
- **Modificaciones:** Kalun Jose Lau Gan
- **Documentación y mejoras:** remgo696
