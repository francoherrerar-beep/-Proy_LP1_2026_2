# Modelo Matemático - Proyecto 08
**Simulador de Riego y Balance Hídrico del Suelo**  
**Curso:** SW303 - Lenguaje de Programación I (Imperativo)  
**Responsable:** Especialista en Lógica y Modelado Matemático - Samuel Riquez


## 1. Propósito del modelo

El modelo matemático define las operaciones que utilizará el simulador para estimar la evolución diaria del agua almacenada en el suelo y determinar cuándo aplicar riego.

Se consideran los datos diarios de precipitación y evapotranspiración, el agua almacenada al inicio del día, los parámetros del suelo y la estrategia de riego seleccionada.

## 2. Variables y parámetros

Todas las cantidades de agua se expresan en milímetros (mm) de lámina de agua equivalente.

| Símbolo | Nombre | Descripción |
| :--- | :--- | :--- |
| P_t | Precipitación | Agua que ingresa al suelo por lluvia durante el día t. |
| ET_t | Evapotranspiración | Agua que sale del suelo durante el día t. |
| I_t | Riego aplicado | Agua incorporada mediante riego durante el día t. |
| H_t | Agua almacenada | Agua almacenada al **inicio** del día t. |
| H_1 | Almacenamiento inicial | Agua almacenada al inicio de la simulación (primer día). |
| H_t* | Almacenamiento provisional | Agua calculada antes de descontar el drenaje. |
| CC | Capacidad de campo | Límite superior de agua que el suelo puede retener en el modelo. |
| PMP | Punto de marchitez permanente | Referencia para identificar déficit hídrico. |
| RAW | Agua fácilmente aprovechable | Parámetro del suelo/cultivo: agotamiento máximo admisible desde CC sin estrés. Define el umbral crítico `CC - RAW`, usado para activar el riego por umbrales y para emitir alertas. |
| A_t | Agotamiento | Agua faltante respecto de la capacidad de campo al inicio del día t. |
| D_t | Drenaje | Agua que supera la capacidad de campo durante el día t. |
| H_(t+1) | Agua final | Agua almacenada al **final** del día t (inicio del día t+1). |
| I_fijo | Dosis fija | Cantidad de agua aplicada en cada evento de la estrategia fija. |
| I_umbral | Dosis por umbral | Cantidad de agua aplicada en cada evento de la estrategia por umbrales. |

**Convención de notación:** en todos los documentos del proyecto, `H_t` es el agua al inicio del día `t` y `H_(t+1)` es el agua al final de ese mismo día. Las decisiones de riego del día `t` se toman con `H_t`; las alertas y el déficit se evalúan con `H_(t+1)`.

### 2.1. Parámetros de las estrategias

- **Parámetros del suelo (compartidos por ambas estrategias):** CC, PMP, RAW y H_1. RAW pertenece al suelo/cultivo; la estrategia por umbrales lo consume, pero no lo define.
- **Estrategia fija:** intervalo entre riegos (en días) y cantidad de agua aplicada en cada evento (`I_fijo`).
- **Estrategia por umbrales:** cantidad de agua aplicada cuando se activa el riego (`I_umbral`). El umbral de activación es `CC - RAW`, con RAW tomado de los parámetros del suelo.

Estos valores deben ser configurables; el modelo no establece valores únicos.

## 3. Condiciones iniciales y validaciones

Antes de iniciar la simulación, el usuario proporciona los parámetros del suelo y el almacenamiento inicial.

1. La capacidad de campo debe ser mayor que el punto de marchitez permanente: `CC > PMP`.
2. El agua fácilmente aprovechable debe ser positiva y no superar el agua útil total del suelo: `0 < RAW <= CC - PMP`. Esto garantiza que el umbral crítico no quede por debajo del PMP (`PMP <= CC - RAW`).
3. El almacenamiento inicial debe cumplir `0 <= H_1 <= CC`.
4. La precipitación, evapotranspiración y el riego no pueden ser negativos.
5. El intervalo de riego fijo debe ser un entero positivo.
6. La dosis configurada para cada riego (`I_fijo`, `I_umbral`) debe ser mayor o igual que cero.
7. Los datos climáticos deben corresponder a registros diarios válidos.

Para este modelo simplificado, la precipitación y el riego se consideran ingresos al almacenamiento del suelo, mientras que la evapotranspiración representa una salida de agua.

## 4. Modelo de balance hídrico diario

El balance hídrico representa la variación del agua almacenada en el suelo durante un día.

### 4.1. Almacenamiento provisional

Primero se suma el agua que ingresa al suelo y se resta el agua perdida por evapotranspiración:

`H_t* = H_t + P_t + I_t - ET_t`

### 4.2. Cálculo del drenaje

Si el almacenamiento provisional supera la capacidad de campo, el exceso se considera drenaje:

`D_t = max(0, H_t* - CC)`

Si `H_t*` no supera `CC`, el drenaje es cero.

### 4.3. Almacenamiento final

El almacenamiento al finalizar el día se obtiene descontando el drenaje:

`H_(t+1) = max(0, H_t* - D_t)`

Así, el almacenamiento final queda siempre entre 0 y la capacidad de campo (`0 <= H_(t+1) <= CC`) y se utiliza como almacenamiento inicial del día siguiente. El piso en 0 evita almacenamientos negativos cuando la evapotranspiración supera el agua disponible: en ese caso `H_t*` es negativo y el suelo simplemente queda sin agua.

## 5. Agotamiento, umbral de riego, alerta y déficit

### 5.1. Agotamiento

Para determinar si el suelo necesita riego, se calcula el agotamiento de agua respecto de la capacidad de campo al inicio del día:

`A_t = CC - H_t`

### 5.2. Umbral de riego (estrategia por umbrales)

En la estrategia por umbrales, el riego del día `t` se activa cuando el agotamiento alcanza o supera RAW:

`Activar riego si A_t >= RAW`

Esto equivale a comprobar:

`H_t <= CC - RAW`

La decisión se toma con el agua al inicio del día, antes de sumar `P_t`, `I_t` y restar `ET_t`.

### 5.3. Alerta de estrés hídrico

El umbral crítico de alerta es `CC - RAW`. Un día se marca con alerta de estrés cuando el agua al finalizar el día no supera ese umbral:

`Día con alerta si H_(t+1) <= CC - RAW`

### 5.4. Déficit hídrico

El punto de marchitez permanente se utiliza como referencia para identificar días con déficit hídrico:

`Día con déficit si H_(t+1) < PMP`

El modelo contabiliza un día con déficit cuando el almacenamiento al finalizar ese día se encuentra por debajo del PMP.

### 5.5. Relación entre alerta y déficit

Por la validación `RAW <= CC - PMP` se cumple `PMP <= CC - RAW`. Por lo tanto, todo día con déficit (`H_(t+1) < PMP`) es también un día con alerta (`H_(t+1) <= CC - RAW`), pero un día con alerta no necesariamente tiene déficit. La alerta es una señal temprana; el déficit es la condición más severa.

## 6. Estrategias de riego

El simulador contempla dos estrategias que pueden ejecutarse sobre una misma serie de datos climáticos para comparar sus resultados.

### 6.1. Estrategia de riego periódico fijo

El riego se aplica siguiendo un intervalo establecido por el usuario, independientemente del nivel de humedad del suelo.

`I_t = I_fijo`, si corresponde un día de riego.

`I_t = 0`, en caso contrario.

El intervalo determina cada cuántos días se aplica el riego.

### 6.2. Estrategia de riego por umbrales

El riego se aplica cuando el agotamiento de agua alcanza o supera RAW (parámetro del suelo).

`I_t = I_umbral`, si `A_t >= RAW`.

`I_t = 0`, si `A_t < RAW`.

`I_umbral` es la cantidad de agua configurada para cada evento. En ambas estrategias, el agua aplicada se incorpora al balance hídrico del día y el drenaje se calcula después, si se supera CC.

## 7. Indicadores derivados del modelo

Aunque el cálculo de indicadores corresponde al módulo `indicadores`, el modelo define las cantidades que se utilizarán.

### 7.1. Consumo acumulado de riego

Es la suma de toda el agua aplicada durante el periodo de `n` días:

`I_total = I_1 + I_2 + ... + I_n`

### 7.2. Días con déficit hídrico

Es la cantidad de días en los que el almacenamiento final se encuentra por debajo del punto de marchitez permanente:

`N_deficit = cantidad de días en que H_(t+1) < PMP`

### 7.3. Drenaje acumulado

Es la suma de toda el agua perdida por drenaje durante el periodo:

`D_total = D_1 + D_2 + ... + D_n`

### 7.4. Días con alerta de estrés

Es la cantidad de días en los que el almacenamiento final no supera el umbral crítico:

`N_alerta = cantidad de días en que H_(t+1) <= CC - RAW`

Estos indicadores permitirán comparar el consumo de agua, las pérdidas por drenaje, los días con alerta y los días con déficit entre las estrategias.

## 8. Supuestos y limitaciones

1. La simulación se realiza en intervalos diarios.
2. La precipitación y la evapotranspiración se reciben como datos de entrada; el simulador no pronostica el clima.
3. La lluvia y el riego se consideran ingresos directos al almacenamiento del suelo.
4. El drenaje se estima cuando el almacenamiento provisional supera la capacidad de campo.
5. La estrategia fija utiliza un intervalo y una dosis configurados por el usuario.
6. La estrategia por umbrales utiliza RAW (parámetro del suelo) y una dosis configurada por el usuario.
7. El modelo no representa el crecimiento del cultivo, la distribución del agua en distintas capas del suelo ni el funcionamiento físico de válvulas de riego.
8. La evapotranspiración efectiva no puede superar el agua disponible: el piso `H_(t+1) >= 0` limita la pérdida, pero el modelo no reduce la ET en función del estrés hídrico.

## 9. Relación con los módulos

| Módulo | Relación con el modelo matemático |
| :--- | :--- |
| `balance_hidrico` | Aplica las ecuaciones de almacenamiento provisional, drenaje y almacenamiento final; marca la alerta diaria (`H_(t+1) <= CC - RAW`). |
| `control_riego` | Determina la cantidad de riego según la estrategia fija o por umbrales; lee RAW desde los parámetros del suelo. |
| `parametros_suelo` | Proporciona CC, PMP, RAW y el almacenamiento inicial H_1, y valida sus restricciones. |
| `datos_climaticos` | Proporciona precipitación y evapotranspiración diaria. |
| `indicadores` | Calcula `I_total`, `D_total`, `N_alerta` y `N_deficit`. |
| `reportes` | Presenta o exporta los resultados de la simulación. |

---
