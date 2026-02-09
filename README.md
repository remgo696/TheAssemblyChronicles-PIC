# 🧠 The Assembly Chronicles – PIC

> **Guía técnica de referencia para microcontroladores PIC18F57Q43 y PIC18F47Q10 en Assembly**

[![Deploy MkDocs](https://github.com/remgo696/TheAssemblyChronicles-PIC/actions/workflows/publish.yml/badge.svg)](https://github.com/remgo696/TheAssemblyChronicles-PIC/actions/workflows/publish.yml)

📖 **Sitio en vivo:** [https://remgo696.github.io/TheAssemblyChronicles-PIC/](https://remgo696.github.io/TheAssemblyChronicles-PIC/)

---

## Sobre este proyecto

Este repositorio consolida los fundamentos de programación en Assembly para microcontroladores PIC de la familia 18F. Está diseñado como una referencia rápida y práctica, orientado a estudiantes de **Ingeniería Electrónica**, **Mecatrónica** y **Biomédica** de la UPC, y basado en el [material del curso](https://github.com/tocache/Microchip-PIC18F57Q43).

## Estructura del repositorio

```
.
├── docs/                          # Fuente de la documentación (MkDocs)
│   ├── index.md                   # Página de inicio
│   ├── teoria/                    # Fundamentos teóricos
│   │   ├── temporizadores.md
│   │   └── interrupciones.md
│   ├── proyectos/                 # Ejemplos y proyectos prácticos
│   ├── recursos/                  # Datasheets y herramientas
│   └── assets/                    # Imágenes, CSS y JS
├── mkdocs.yml                     # Configuración de MkDocs
├── Pipfile                        # Dependencias Python (Pipenv)
├── .github/workflows/publish.yml  # CI/CD → GitHub Pages
└── LICENSE
```

## Desarrollo local

### Requisitos previos

- Python 3.12+
- [Pipenv](https://pipenv.pypa.io/)

### Instalación

```bash
# Clonar el repositorio
git clone https://github.com/remgo696/TheAssemblyChronicles-PIC.git
cd TheAssemblyChronicles-PIC

# Instalar dependencias
pipenv install

# Levantar el servidor de desarrollo
pipenv run mkdocs serve
```

El sitio estará disponible en `http://127.0.0.1:8000`.

### Build de producción

```bash
pipenv run mkdocs build
```

Los archivos estáticos se generarán en la carpeta `site/`.

## Despliegue

El sitio se despliega **automáticamente** a GitHub Pages en cada push a la rama `main` mediante GitHub Actions. La URL del sitio es:

> [https://remgo696.github.io/TheAssemblyChronicles-PIC/](https://remgo696.github.io/TheAssemblyChronicles-PIC/)

## Licencia

Este proyecto está bajo la licencia [MIT](LICENSE).

---

> ‼️ Este repositorio busca servir como guía de referencia rápida y no como un curso completo. Se recomienda complementar este material con la documentación oficial de [Microchip](https://www.microchip.com/).
