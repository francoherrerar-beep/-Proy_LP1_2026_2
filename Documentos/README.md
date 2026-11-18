# Proyecto 08: Simulador de Riego y Balance Hídrico del Suelo

**Curso:** SW303 – Lenguajes de Programación I (Imperativo)  
**Ciclo:** 2026-2  

---

## 1. Descripción del Proyecto
Aplicación de consola desarrollada en lenguaje C estándar que simula la evolución diaria de la humedad del suelo para un cultivo a partir de registros climáticos diarios (precipitación y evapotranspiración) y parámetros físicos configurables (capacidad de campo, punto de marchitez permanente y agua fácilmente aprovechable).

El simulador calcula el balance hídrico día a día, estima las pérdidas de agua por drenaje cuando se supera la capacidad máxima del suelo, emite alertas preventivas ante escasez de humedad y permite comparar dos estrategias de riego: **riego periódico fijo** frente a **riego activado por umbrales**.

---

## 2. Beneficios de la Aplicación
* **Optimización del uso del agua:** Permite evaluar cuánto volumen de agua se ahorra al aplicar riego por umbrales frente a un calendario fijo.
* **Prevención de estrés hídrico:** Identifica oportunamente los días en que el suelo cae por debajo del umbral crítico y contabiliza los días con déficit hídrico.
* **Análisis reproducible y configurable:** Facilita probar distintos tipos de suelo y series meteorológicas mediante archivos CSV sin modificar el código fuente.
* **Visualización clara de resultados:** Exporta reportes detallados en formato CSV y presenta gráficas de evolución diaria de humedad, lluvia y riego.

---

## 3. Integrantes del Equipo

| Código | Apellidos y Nombres | Rol en el Proyecto |
| :--- | :--- | :--- |
| `20250225D` | Herrera Rios, Franco | Diseñador de Flujos, Datos y Documentación |
| `20250134I` | Riquez Chacon, Samuel Abraham | Especialista en Lógica y Modelado Matemático |
| `20241156C` | Yllanes Quispe, Bryan Miguel | Arquitecto de Software / Líder Técnico |
| `20250159A` | Cornejo Gómez, Giovanni Rafael | Analista de Requerimientos y Calidad (QA) |

---

## 4. Estructura del Repositorio (`Proy_LP1_2026_2`)
* `README.md` – Información general del proyecto e integrantes.
* `01_requerimientos.md` – Requerimientos funcionales, no funcionales y escenarios de prueba.
* `02_modelo_matematico.md` – Ecuaciones del balance hídrico, drenaje, estrategias e indicadores.
* `03_diagramas.md` – Diagrama de casos de uso y diagrama de flujo del simulador.
* `04_modulos.md` – Arquitectura modular, estructuras y prototipos en C (`.h` / `.c`).