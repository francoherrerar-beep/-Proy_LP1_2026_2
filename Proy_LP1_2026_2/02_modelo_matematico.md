# Modelo Matemático - Proyecto 08
**Simulador de Riego y Balance Hí­drico del Suelo**  
**Curso:** SW303 - Lenguaje de Programación I (Imperativo)  
**Responsable:** Especialista en Lógica y Modelado Matemático - Samuel Riquez


## 1. Propósito del modelo

El modelo matemático define las operaciones que utilizará el simulador para estimar la evolución diaria del agua almacenada en el suelo y determinar cuándo aplicar riego.

Se consideran los datos diarios de precipitación y evapotranspiracón, el agua almacenada al inicio del dí­a, los parámetros del suelo y la estrategia de riego seleccionada.

## 2. Variables y parámetros

Todas las cantidades de agua se expresan en milí­metros (mm).

| Sí­mbolo | Nombre | Descripción |
| :--- | :--- | :--- |
| P_t | Precipitación | Agua que ingresa al suelo por lluvia durante el dí­a t. |
| ET_t | Evapotranspiración | Agua que sale del suelo durante el dí­a t. |
| I_t | Riego aplicado | Agua incorporada mediante riego durante el dí­a t. |
| H_t | Agua almacenada | Agua almacenada al inicio del dí­a t. |
| H_t* | Almacenamiento provisional | Agua calculada antes de descontar el drenaje. |
| CC | Capacidad de campo | Lí­mite superior de agua que el suelo puede retener en el modelo. |
| PMP | Punto de marchitez permanente | Referencia para identificar déficit hí­drico. |
| RAW | Agua fácilmente aprovechable | Umbral de agotamiento utilizado para activar el riego. |
| D_t | Drenaje | Agua que supera la capacidad de campo durante el día t. |
| H_(t+1) | Agua final | Agua almacenada al terminar el dí­a t. |

### 2.1. Parámetros de las estrategias

- **Estrategia fija:** intervalo entre riegos y cantidad de agua aplicada en cada evento.
- **Estrategia por umbrales:** valor de RAW y cantidad de agua aplicada cuando se activa el riego.

Estos valores deben ser configurables; el modelo no establece valores únicos.

## 3. Condiciones iniciales y validaciones

Antes de iniciar la simulación, el usuario proporciona los parámetros del suelo y el almacenamiento inicial.

1. La capacidad de campo debe ser mayor que el punto de marchitez permanente: `CC > PMP`.
2. El almacenamiento inicial debe cumplir `0 <= H_1 <= CC`.
3. La precipitación, evapotranspiración y el riego no pueden ser negativos.
4. El intervalo de riego fijo debe ser un entero positivo.
5. La dosis configurada para cada riego debe ser mayor o igual que cero.
6. Los datos climáticos deben corresponder a registros diarios válidos.

Para este modelo simplificado, la precipitación y el riego se consideran ingresos al almacenamiento del suelo, mientras que la evapotranspiración representa una salida de agua.

## 4. Modelo de balance hí­drico diario

El balance hí­drico representa la variación del agua almacenada en el suelo durante un dí­a.

### 4.1. Almacenamiento provisional

Primero se suma el agua que ingresa al suelo y se resta el agua perdida por evapotranspiración:

`H_t* = H_t + P_t + I_t - ET_t`

### 4.2. Cálculo del drenaje

Si el almacenamiento provisional supera la capacidad de campo, el exceso se considera drenaje:

`D_t = max(0, H_t* - CC)`

Si `H_t*` no supera `CC`, el drenaje es cero.

### 4.3. Almacenamiento final

El almacenamiento al finalizar el dí­a se obtiene descontando el drenaje:

`H_(t+1) = H_t* - D_t`

Así, el almacenamiento final no supera la capacidad de campo y se utiliza como almacenamiento inicial del dí­a siguiente.

## 5. Déficit hídrico y umbral de riego

Para determinar si el suelo necesita riego, se calcula el agotamiento de agua respecto de la capacidad de campo:

`A_t = CC - H_t`

En la estrategia por umbrales, el riego se activa cuando el agotamiento alcanza o supera RAW:

`Activar riego si A_t >= RAW`

Esto equivale a comprobar:

`H_t <= CC - RAW`

El punto de marchitez permanente se utiliza como referencia para identificar días con déficit hí­drico:

`Día con déficit si H_(t+1) < PMP`

El modelo contabiliza un dí­a con déficit cuando el almacenamiento al finalizar ese día se encuentra por debajo del PMP.


## 6. Estrategias de riego

El simulador contempla dos estrategias que pueden ejecutarse sobre una misma serie de datos climáticos para comparar sus resultados.

### 6.1. Estrategia de riego periódico fijo

El riego se aplica siguiendo un intervalo establecido por el usuario, independientemente del nivel de humedad del suelo.

`I_t = I_fijo`, si corresponde un dí­a de riego.

`I_t = 0`, en caso contrario.

El intervalo determina cada cuántos dí­as se aplica el riego.

### 6.2. Estrategia de riego por umbrales

El riego se aplica cuando el agotamiento de agua alcanza o supera RAW.

`I_t = I_umbral`, si `A_t >= RAW`.

`I_t = 0`, si `A_t < RAW`.

`I_umbral` es la cantidad de agua configurada para cada evento. En ambas estrategias, el agua aplicada se incorpora al balance hÃ­drico del dí­a y el drenaje se calcula después, si se supera CC.

## 7. Indicadores derivados del modelo

Aunque el cálculo de indicadores corresponde al módulo `indicadores`, el modelo define las cantidades que se utilizarán.

### 7.1. Consumo acumulado de riego

Es la suma de toda el agua aplicada durante el periodo:

`I_total = I_1 + I_2 + ... + I_n`

### 7.2. Dí­as con déficit hí­drico

Es la cantidad de dí­as en los que el almacenamiento final se encuentra por debajo del punto de marchitez permanente:

`N_deficit = cantidad de días en que H_(t+1) < PMP`

Estos indicadores permitirán comparar el consumo de agua y los dí­as con déficit entre las estrategias.

## 8. Supuestos y limitaciones

1. La simulación se realiza en intervalos diarios.
2. La precipitación y la evapotranspiración se reciben como datos de entrada; el simulador no pronostica el clima.
3. La lluvia y el riego se consideran ingresos directos al almacenamiento del suelo.
4. El drenaje se estima cuando el almacenamiento provisional supera la capacidad de campo.
5. La estrategia fija utiliza un intervalo y una dosis configurados por el usuario.
6. La estrategia por umbrales utiliza RAW y una dosis configurada por el usuario.
7. El modelo no representa el crecimiento del cultivo, la distribución del agua en distintas capas del suelo ni el funcionamiento fí­sico de válvulas de riego.

## 9. Relación con los módulos

| Módulo | Relación con el modelo matemático |
| :--- | :--- |
| `balance_hidrico` | Aplica las ecuaciones de almacenamiento provisional, drenaje y almacenamiento final. |
| `control_riego` | Determina la cantidad de riego según la estrategia fija o por umbrales. |
| `parametros_suelo` | Proporciona CC, PMP, RAW y el almacenamiento inicial. |
| `datos_climaticos` | Proporciona precipitación y evapotranspiración diaria. |
| `indicadores` | Calcula el consumo acumulado de riego y los dí­as con déficit hí­drico. |
| `reportes` | Presenta o exporta los resultados de la simulación. |

---
