# 🧠 TheAssemblyChronicles-PIC
## Guia del PIC18F57Q43

Repositorio educativo con prácticas en ensamblador para el microcontrolador **PIC18F57Q43**, orientado a consolidar los fundamentos vistos en el curso y facilitar la consulta rápida de cada tema. El material está diseñado para estudiantes de **Ingeniería Electrónica**, **Mecatrónica** y **Biomédica** que cursan la asignatura de **1AEL0256 Microcontroladores** en la UPC, y está basado en el [material del curso](https://github.com/tocache/Microchip-PIC18F57Q43).

---

## 📘 Temario

### 🟩 Unidad 1 – Introducción y entorno de trabajo
- Conceptos básicos de microcontroladores 
- Entorno **MPLAB X IDE** y **PIC AS Assembler**  
- Configuración de bits (**CONFIG**) y cabeceras (`.inc`)

---

### 🟩 Unidad 2 – Arquitectura interna del PIC18F57Q43
- Bloques principales y flujo de datos  
- Memorias internas: **Flash de programa**, **EEPROM** y **SRAM**  
- Registros **SFR**, **GPR** y bancos de memoria  
- **STATUS Register** y banderas del ALU  
- Instrucciones básicas y modos de direccionamiento  

---

### 🟩 Unidad 3 – E/S digitales
- Configuración de pines: `TRIS`, `LAT`, `PORT`, `ANSEL`  
- Entradas con *weak pull-ups*  
- Botones y antirrebote (software y hardware)  
- Displays de 7 segmentos y multiplexado  
- Uso de macros y `CBLOCK` para variables  

---

### 🟩 Unidad 4 – Temporizadores
- Tiempo de instrucción y frecuencia del sistema  
- **TMR0**, **TMR1**, **TMR2**: diferencias y configuración básica  
- Generación de retardos con **TMR0**  
- Base de tiempo y eventos periódicos  
- Introducción a interrupciones por **TMR0**  

---

### 🟩 Unidad 5 – Interrupciones
- Vector de interrupción y `RETFIE`  
- INT, IOC y temporizadores  
- Priorización, banderas y habilitación global  
- Diseño de rutinas ISR eficientes  
- Ejemplo: control de LED o contador con botón  

---

### 🟩 Unidad 6 – Osciladores y configuración del reloj
- Tipos de osciladores (**HFINTOSC**, **LFINTOSC**, **EXTOSC**, **PLL**)  
- Configuración con `OSCCON1`, `OSCFRQ` y `OSCEN`  
- Cálculo de frecuencias y divisores (`NDIV`)  
- Selección del reloj y estabilidad  

---

### 🟩 Unidad 7 – Periféricos avanzados
- Módulo **PPS (Peripheral Pin Select)**  
- Introducción al **EUSART**  
- PWM básico con **TMR2** y **CCP**  
- Conceptos de **ADC** y configuración básica  
- Comunicación con sensores simples  

---

### 🟩 Unidad 8 – Memoria no volátil
- Acceso a **EEPROM** (`NVMCON`, `NVMDAT`, `NVMADR`)  
- Lectura/escritura en memoria de programa con `TBLPTR` y `TABLAT`  
- Almacenamiento de datos o configuraciones persistentes  

---

### 🟩 Unidad 9 – Estructuras lógicas
- Máquinas de estados (**Moore** y **Mealy**)  
- Tablas de búsqueda (`TBLRD`, `RETLW`)  
- Implementación de menús y modos de operación  

---

### 🟩 Unidad 10 – Buenas prácticas y documentación
- Organización de proyectos en MPLAB  
- Uso de **Git** y control de versiones  
- Estructura de repositorios educativos  
- Comentarios, macros y estilo de código  
- Depuración con simulador y Curiosity Nano  

---

> ‼️ Este repositorio busca servir como guía de referencia rápida y no como un curso completo. Se recomienda complementar este material con la documentación oficial y otros recursos educativos.
