# Documento de Requerimientos – Proyecto 08
**Simulador de Riego y Balance Hídrico del Suelo**  
**Curso:** SW303 – Lenguajes de Programación I (Imperativo)  
**Responsable (QA / Requerimientos):** Rafael Cornejo

---

## 1. Propósito y Alcance del Sistema
* **Propósito:** Estimar la evolución diaria de la humedad del suelo y determinar cuándo y cuánto regar un cultivo bajo condiciones ambientales y parámetros de suelo configurados.
* **Alcance:** Simulación del balance hídrico a paso diario con parámetros configurables por el usuario. No se incluyen modelos avanzados de crecimiento vegetal, predicción meteorológica ni control físico de válvulas.

### 1.1. Terminología común a todos los documentos
Estas definiciones se aplican por igual en `02_modelo_matematico.md`, `03_diagramas.md` y `04_modulos.md`.

* **Unidades:** toda cantidad de agua (CC, PMP, RAW, H, P, ET, I, D) se expresa en milímetros (mm) de lámina de agua equivalente.
* **Humedad del suelo:** $H_t$ es el agua almacenada al **inicio** del día $t$; $H_{t+1}$ es el agua al **final** de ese día (y el inicio del día siguiente). La humedad inicial de la simulación es $H_1$.
* **RAW:** parámetro del suelo/cultivo (se configura junto con CC y PMP). Define el **umbral crítico** $CC - RAW$.
* **Alerta de estrés hídrico:** día en que $H_{t+1} \le CC - RAW$.
* **Día con déficit hídrico:** día en que $H_{t+1} < PMP$. Todo día con déficit es también día con alerta, pero no al revés.

---

## 2. Requerimientos Funcionales (RF)

| Código | Nombre | Descripción | Módulo Asociado |
| :--- | :--- | :--- | :--- |
| **RF-01** | Lectura de datos climáticos | El sistema debe leer desde un archivo CSV los registros diarios de precipitación y evapotranspiración del periodo a simular. | `datos_climaticos` |
| **RF-02** | Configuración del suelo y cultivo | El sistema debe permitir configurar, en mm, la capacidad de campo (CC), el punto de marchitez permanente (PMP), el agua fácilmente aprovechable (RAW, que define el umbral crítico $CC - RAW$) y la humedad inicial $H_1$. | `parametros_suelo` |
| **RF-03** | Cálculo del balance hídrico diario | El sistema debe calcular la humedad del suelo día por día considerando la humedad al inicio del día ($H_t$), la precipitación, el riego aplicado y la evapotranspiración, obteniendo la humedad final ($H_{t+1}$). | `balance_hidrico` |
| **RF-04** | Estimación de drenaje por exceso | El sistema debe calcular las pérdidas de agua por drenaje mediante un modelo simplificado cuando el agua provisional del suelo supere la capacidad de campo (CC). | `balance_hidrico` |
| **RF-05** | Generación de alertas de humedad | El sistema debe emitir una alerta de estrés hídrico cada día en que la humedad final cumpla $H_{t+1} \le CC - RAW$ (umbral crítico definido en la sección 1.1). | `indicadores` |
| **RF-06** | Comparación de estrategias de riego | El sistema debe simular y comparar una estrategia de riego periódico fijo (intervalo y dosis configurables) frente a una estrategia de riego activada por umbral (usa el RAW del suelo y una dosis configurable), sobre la misma serie climática. | `control_riego` |
| **RF-07** | Cálculo de indicadores acumulados | El sistema debe calcular el consumo total acumulado de agua de riego ($I_{total}$), el drenaje total acumulado, el número de días con alerta de estrés y el número total de días con déficit hídrico ($N_{deficit}$, días con $H_{t+1} < PMP$). | `indicadores` |
| **RF-08** | Reportes y visualización gráfica | El sistema debe generar reportes de resultados (exportables a CSV) y graficar la evolución de la humedad del suelo, la precipitación y el riego aplicado, con referencias de CC, $CC - RAW$ y PMP. | `reportes` |

---

## 3. Requerimientos No Funcionales (RNF)

| Código | Categoría | Descripción |
| :--- | :--- | :--- |
| **RNF-01** | Lenguaje y compilación | El código fuente debe estar escrito en lenguaje C estándar (C99 o C11) y ser compilable mediante GCC sin errores ni advertencias críticas (verificado con `-Wall -Wextra`). |
| **RNF-02** | Entorno de ejecución | La aplicación debe ejecutarse íntegramente desde la terminal de comandos en sistemas operativos Windows o Linux. La versión base no debe requerir librerías externas de interfaz gráfica (GUI). |
| **RNF-03** | Formato de entrada/salida | Las entradas masivas de datos climáticos y los reportes de salida deben gestionarse mediante archivos de texto plano en formato CSV. |
| **RNF-04** | Visualización de gráficos | Las gráficas de series temporales (humedad, precipitación y riego) deben renderizarse mediante caracteres ASCII en terminal en la versión base. El uso de la librería `raylib` es una extensión opcional y no debe ser requisito para compilar ni ejecutar la versión base (coherente con RNF-02). |
| **RNF-05** | Modularidad | El proyecto debe organizarse en archivos de cabecera (`.h`) e implementación (`.c`) separados según los 6 módulos definidos (`datos_climaticos`, `parametros_suelo`, `balance_hidrico`, `control_riego`, `indicadores`, `reportes`). |
| **RNF-06** | Validación de entradas | El programa debe validar la existencia y el formato de los archivos CSV de entrada, evitando divisiones por cero o valores negativos inválidos en parámetros físicos. Debe validar además: $CC > PMP$, $0 < RAW \le CC - PMP$, $0 \le H_1 \le CC$, intervalo de riego fijo entero positivo y dosis de riego $\ge 0$. |

---

## 4. Escenarios de Prueba Planificados (Criterios QA para Entregable 2)
Para validar el cumplimiento de los requerimientos durante la etapa de implementación, se evaluarán como mínimo los siguientes escenarios:
1. **Escenario de días secos (sequía prolongada):** Precipitación nula durante varios días consecutivos para verificar la caída progresiva de humedad, la activación de alertas de estrés ($H_{t+1} \le CC - RAW$) y el conteo de días con déficit hídrico ($H_{t+1} < PMP$).
2. **Escenario de lluvia intensa:** Precipitaciones superiores a la capacidad de almacenamiento del suelo para comprobar el cálculo de pérdidas por drenaje y el tope de humedad máxima ($H_{t+1} \le CC$).
3. **Comparación de estrategias de riego:** Ejecución de una misma serie climática bajo riego fijo vs. riego por umbral para contrastar el consumo acumulado de agua, el drenaje total, los días con alerta y los días en déficit.