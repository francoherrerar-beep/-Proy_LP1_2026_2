# Documento de Requerimientos – Proyecto 08
**Simulador de Riego y Balance Hídrico del Suelo**  
**Curso:** SW303 – Lenguajes de Programación I (Imperativo)  

---

## 1. Propósito y Alcance del Sistema
* **Propósito:** Estimar la evolución diaria de la humedad del suelo y determinar cuándo y cuánto regar un cultivo bajo condiciones ambientales y parámetros de suelo configurados.
* **Alcance:** Simulación del balance hídrico a paso diario con parámetros configurables por el usuario. No se incluyen modelos avanzados de crecimiento vegetal, predicción meteorológica ni control físico de válvulas.

---

## 2. Requerimientos Funcionales (RF)

| Código | Nombre | Descripción | Módulo Asociado |
| :--- | :--- | :--- | :--- |
| **RF-01** | Lectura de datos climáticos | El sistema debe leer desde un archivo CSV los registros diarios de precipitación y evapotranspiración del periodo a simular. | `datos_climaticos` |
| **RF-02** | Configuración del suelo y cultivo | El sistema debe permitir configurar la capacidad de almacenamiento de agua del suelo (capacidad de campo, punto de marchitez), la humedad inicial y los umbrales de riego. | `parametros_suelo` |
| **RF-03** | Cálculo del balance hídrico diario | El sistema debe calcular la humedad del suelo día por día considerando la humedad previa, la precipitación, el riego aplicado y la evapotranspiración. | `balance_hidrico` |
| **RF-04** | Estimación de drenaje por exceso | El sistema debe calcular las pérdidas de agua por drenaje mediante un modelo simplificado cuando el agua en el suelo supere su capacidad máxima de almacenamiento. | `balance_hidrico` |
| **RF-05** | Generación de alertas de humedad | El sistema debe emitir alertas cuando el nivel de humedad del suelo caiga por debajo del umbral crítico configurado. | `indicadores` |
| **RF-06** | Comparación de estrategias de riego | El sistema debe simular y comparar una estrategia de riego periódico fijo frente a una estrategia de riego activada por umbrales de humedad. | `control_riego` |
| **RF-07** | Cálculo de indicadores acumulados | El sistema debe calcular el consumo total acumulado de agua de riego y el número total de días con déficit hídrico durante la simulación. | `indicadores` |
| **RF-08** | Reportes y visualización gráfica | El sistema debe generar reportes de resultados (exportables a CSV) y graficar la evolución de la humedad del suelo, la precipitación y el riego aplicado. | `reportes` |

---

## 3. Requerimientos No Funcionales (RNF)

| Código | Categoría | Descripción |
| :--- | :--- | :--- |
| **RNF-01** | Lenguaje y compilación | El código fuente debe estar escrito en lenguaje C estándar y ser compilable mediante GCC sin errores ni advertencias críticas. |
| **RNF-02** | Entorno de ejecución | La aplicación debe ejecutarse íntegramente desde la terminal de comandos en sistemas operativos Windows o Linux, sin requerir librerías externas de interfaz gráfica (GUI). |
| **RNF-03** | Formato de entrada/salida | Las entradas masivas de datos climáticos y los reportes de salida deben gestionarse mediante archivos de texto plano en formato CSV. |
| **RNF-04** | Visualización de gráficos | Las gráficas de series temporales (humedad, precipitación y riego) deben renderizarse mediante caracteres ASCII en terminal o utilizando la librería `raylib` de C. |
| **RNF-05** | Modularidad | El proyecto debe organizarse en archivos de cabecera (`.h`) e implementación (`.c`) separados según los 6 módulos definidos (`datos_climaticos`, `parametros_suelo`, `balance_hidrico`, `control_riego`, `indicadores`, `reportes`). |
| **RNF-06** | Validación de entradas | El programa debe validar la existencia y el formato de los archivos CSV de entrada, evitando divisiones por cero o valores negativos inválidos en parámetros físicos. |

---

## 4. Escenarios de Prueba Planificados (Criterios QA para Entregable 2)
Para validar el cumplimiento de los requerimientos durante la etapa de implementación, se evaluarán como mínimo los siguientes escenarios:
1. **Escenario de días secos (sequía prolongada):** Precipitación nula durante varios días consecutivos para verificar la caída progresiva de humedad, la activación de alertas y el conteo de días con déficit hídrico.
2. **Escenario de lluvia intensa:** Precipitaciones superiores a la capacidad de almacenamiento del suelo para comprobar el cálculo de pérdidas por drenaje y el tope de humedad máxima.
3. **Comparación de estrategias de riego:** Ejecución de una misma serie climática bajo riego fijo vs. riego por umbral para contrastar el consumo acumulado de agua y los días en déficit.