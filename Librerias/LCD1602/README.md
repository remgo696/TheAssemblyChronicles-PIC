# Librería LCD1602

Librería para el control de pantallas LCD de 16x2 caracteres con interfaz paralela de 4 bits en microcontroladores PIC18F.

## 📋 Características

- Interfaz de 4 bits (modo ahorro de pines)
- Compatible con LCD HD44780 y compatibles
- Funciones para control de cursor, escritura de texto y valores numéricos
- Creación de caracteres personalizados
- Desplazamiento de texto

## 🔧 Instalación

### 1. Copiar los archivos

Agrega los archivos `LCD.c` y `LCD.h` a tu proyecto en MPLAB X IDE:

```
Tu_Proyecto/
├── main.c
├── LCD.c          ← Archivo de implementación
└── LCD.h          ← Archivo de cabecera
```

### 2. Incluir la librería en tu código

En tu archivo `main.c` o donde necesites usar el LCD:

```c
#include "LCD.h"
```

### 3. Configurar los pines

La librería utiliza **PORTD** por defecto:

| Pin PIC | Función LCD | Pin LCD |
|:--------|:------------|:--------|
| RD0 | RS (Register Select) | Pin 4 |
| RD1 | RW (Read/Write) | Pin 5 |
| RD2 | E (Enable) | Pin 6 |
| RD4 | D4 (Dato) | Pin 11 |
| RD5 | D5 (Dato) | Pin 12 |
| RD6 | D6 (Dato) | Pin 13 |
| RD7 | D7 (Dato) | Pin 14 |

!!! warning "Importante"
    - **Verifica que los pines RD0, RD1, RD2, RD4, RD5, RD6 y RD7 estén disponibles** en tu diseño.
    - RD3 **NO se usa**, por lo que puede usarse para otras funciones.
    - Si necesitas usar otros pines, modifica los `#define` en `LCD.h`.

### 4. Configurar el reloj

En `LCD.h`, verifica que `_XTAL_FREQ` coincida con la frecuencia de tu oscilador:

```c
#define _XTAL_FREQ 4000000UL    // 4 MHz (ajustar según tu configuración)
```

## 📖 Uso básico

### Inicialización

```c
LCD_INIT();                 // Inicializar el LCD (obligatorio al inicio)
```

### Escribir texto

```c
// Escribir un mensaje con tamaño conocido
ESCRIBE_MENSAJE("Hola Mundo", 10);

// Escribir un mensaje sin especificar tamaño (calcula automáticamente).
ESCRIBE_MENSAJE2("PIC18F57Q43");
```

### Posicionar el cursor

```c
POS_CURSOR(1, 0);           // Fila 1, columna 0 (inicio de la primera línea)
ESCRIBE_MENSAJE2("Linea 1");

POS_CURSOR(2, 0);           // Fila 2, columna 0 (inicio de la segunda línea)
ESCRIBE_MENSAJE2("Linea 2");
```

Las filas van de **1 a 4** (LCD 16x2 usa solo 1 y 2), las columnas de **0 a 15**.

### Borrar pantalla

```c
BORRAR_LCD();               // Borra todo el contenido y regresa cursor al inicio
```

### Escribir números

```c
unsigned char temperatura = 25;
unsigned int contador = 1234;

// Escribir variable tipo char con 2 dígitos
LCD_ESCRIBE_VAR_CHAR(temperatura, 2);

// Escribir variable tipo int con 4 dígitos
LCD_ESCRIBE_VAR_INT(contador, 4);
```

### Control del cursor

```c
CURSOR_HOME();              // Regresa el cursor al inicio (fila 1, columna 0)
CURSOR_ONOFF(OON);          // Mostrar cursor
CURSOR_ONOFF(OFF);          // Ocultar cursor
BLINK_CURSOR(OON);          // Cursor parpadeante
BLINK_CURSOR(OFF);          // Cursor fijo
```

### Control de la pantalla

```c
DISPLAY_ONOFF(OON);         // Encender pantalla
DISPLAY_ONOFF(OFF);         // Apagar pantalla
DISPLAY_SHIFTLEFT();        // Desplazar contenido a la izquierda
DISPLAY_SHIFTRIGHT();       // Desplazar contenido a la derecha
```

## 📚 Ejemplo completo

```c
#include <xc.h>
#include "LCD.h"

void main(void) {
    // Configuración del sistema
    TRISD = 0x00;           // PORTD como salida
    ANSELD = 0x00;          // PORTD digital
    
    // Inicializar LCD
    LCD_INIT();
    
    // Mostrar mensaje de bienvenida
    POS_CURSOR(1, 0);
    ESCRIBE_MENSAJE2("  PIC18F57Q43  ");
    POS_CURSOR(2, 0);
    ESCRIBE_MENSAJE2(" Assembly Code ");
    
    __delay_ms(2000);
    BORRAR_LCD();
    
    // Mostrar contador
    unsigned int contador = 0;
    while(1) {
        POS_CURSOR(1, 0);
        ESCRIBE_MENSAJE2("Contador:");
        POS_CURSOR(2, 5);
        LCD_ESCRIBE_VAR_INT(contador, 4);
        
        contador++;
        __delay_ms(500);
        
        if(contador > 9999) contador = 0;
    }
}
```

## 🔍 Funciones disponibles

| Función | Descripción |
|:--------|:------------|
| `LCD_INIT()` | Inicializa el LCD (llamar al inicio) |
| `BORRAR_LCD()` | Borra la pantalla completa |
| `POS_CURSOR(fila, columna)` | Posiciona el cursor (fila: 1-4, columna: 0-15) |
| `CURSOR_HOME()` | Regresa el cursor al inicio |
| `ENVIA_CHAR(dato)` | Envía un carácter individual |
| `ESCRIBE_MENSAJE(cadena, tam)` | Escribe texto con tamaño especificado |
| `ESCRIBE_MENSAJE2(cadena)` | Escribe texto (calcula tamaño automáticamente) |
| `LCD_ESCRIBE_VAR_CHAR(numero, digitos)` | Escribe variable unsigned char |
| `LCD_ESCRIBE_VAR_INT(numero, digitos)` | Escribe variable unsigned int |
| `CURSOR_ONOFF(estado)` | Muestra/oculta el cursor (OON/OFF) |
| `BLINK_CURSOR(estado)` | Cursor parpadeante (OON/OFF) |
| `DISPLAY_ONOFF(estado)` | Enciende/apaga la pantalla (OON/OFF) |
| `CURSOR_SHIFTLEFT()` | Mueve cursor a la izquierda |
| `CURSOR_SHIFTRIGHT()` | Mueve cursor a la derecha |
| `DISPLAY_SHIFTLEFT()` | Desplaza contenido a la izquierda |
| `DISPLAY_SHIFTRIGHT()` | Desplaza contenido a la derecha |
| `LCD_CHAR_GRADO()` | Escribe el símbolo de grado (°) |
| `LCD_VARCHAR_BITS(dato)` | Muestra un byte en formato binario |
| `GENERACARACTER(vector, pos)` | Crea carácter personalizado (CGRAM) |

## ⚙️ Personalización de pines

Si necesitas usar otros pines diferentes a PORTD, modifica las siguientes líneas en `LCD.h`:

```c
#define RS  LATDbits.LATD0      // Cambiar LATD0 por el pin deseado
#define RW  LATDbits.LATD1      // Cambiar LATD1 por el pin deseado
#define E   LATDbits.LATD2      // Cambiar LATD2 por el pin deseado
```

También deberás modificar la función `ENVIA_NIBBLE()` en `LCD.c` para usar el puerto correspondiente.

## 🐛 Solución de problemas

### El LCD no muestra nada

- Verifica las conexiones físicas del LCD
- Verifica el contraste del LCD (potenciómetro en pin 3)
- Confirma que `_XTAL_FREQ` coincida con tu frecuencia de reloj
- Revisa que PORTD esté configurado como salida digital

### Caracteres extraños

- Verifica que la frecuencia `_XTAL_FREQ` sea correcta
- Asegúrate de llamar `LCD_INIT()` al inicio
- Verifica las conexiones de los pines de datos (D4-D7)

### El texto no se actualiza

- Llama a `BORRAR_LCD()` antes de escribir nuevo texto
- Usa `POS_CURSOR()` para posicionar correctamente

## 📝 Créditos

- **Desarrollo original:** Prof. Sergio Salas
- **Modificaciones:** Kalun Jose Lau Gan
- **Documentación y mejoras:** remgo696

## 📄 Licencia

Librería de uso educativo para el curso de Microcontroladores PIC18F.
