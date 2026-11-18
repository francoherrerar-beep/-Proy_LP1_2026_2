# Diagramas del Sistema - Proyecto 08

## Simulador de Riego y Balance Hídrico del Suelo

**Curso:** SW303 - Lenguaje de Programación I (Imperativo)  
**Responsable:** Diseñador de Flujos - Franco Herrera Rios

---

## 1. Propósito de los diagramas

Los diagramas describen cómo interactúa el usuario con el simulador y cómo fluye el proceso de cálculo día a día, desde la carga y validación de los datos hasta la comparación de estrategias y la generación de resultados.

El diagrama de casos de uso muestra las funciones que el sistema ofrece al usuario. El diagrama de flujo detalla el orden de las operaciones de la simulación: la validación de los datos de entrada, la decisión de riego, el balance hídrico, el drenaje por exceso, la alerta de estrés hídrico día a día y el cierre con las métricas comparativas y los reportes.

**Notación (idéntica a `02_modelo_matematico.md`):**

- $H_t$ es el agua almacenada al **inicio** del día $t$ y $H_{t+1}$ el agua al **final** de ese día (inicio del día siguiente). El almacenamiento inicial de la simulación es $H_1$.
- $H_{t+1} = \max(0,\ H_t^* - D_t)$, por lo que siempre se cumple $0 \le H_{t+1} \le CC$.
- $RAW$ es un parámetro del suelo/cultivo: cantidad de agua que se puede agotar desde $CC$ sin estrés.
- **Umbral de riego** (decisión del día $t$): $H_t \le CC - RAW$, equivalente a $A_t = CC - H_t \ge RAW$.
- **Alerta de estrés** (fin del día $t$): $H_{t+1} \le CC - RAW$.
- **Día con déficit** (fin del día $t$): $H_{t+1} < PMP$. Todo día con déficit es también día con alerta.

---

## 2. Diagramas del Sistema

### 2.1 Diagrama de Casos de Uso

```mermaid
flowchart LR
    Actor(("Usuario / Agrónomo"))

    subgraph Sistema["Simulador de Riego y Balance Hídrico"]
        CU1("Cargar archivo meteorológico CSV")
        CU2("Configurar parámetros de suelo y cultivo: CC, PMP, RAW, H_1")
        CU3("Configurar y seleccionar estrategias de riego: Fija / Umbral / Ambas")
        CU4("Ejecutar simulación diaria")
        CU5("Comparar estrategias de riego (Fija vs. Umbral)")
        CU6("Consultar alertas de estrés y déficit hídrico")
        CU7("Exportar reporte CSV de balance")
        CU8("Visualizar evolución gráfica")
    end

    Actor --> CU1
    Actor --> CU2
    Actor --> CU3
    Actor --> CU4
    Actor --> CU5
    Actor --> CU6
    Actor --> CU7
    Actor --> CU8

    CU4 -. "«include»" .-> CU6
    CU5 -. "«include»" .-> CU4
```

### 2.2 Diagrama de Flujo del Proceso Diario de Simulación

```mermaid
flowchart TD
    Inicio(["Inicio de simulación"]) --> CargarArchivos["Lectura de registros climáticos CSV"]
    CargarArchivos --> ValidarCSV{"¿CSV válido y P_t, ET_t >= 0?"}
    ValidarCSV -- No --> ErrorCSV["Mostrar error: ruta inválida, formato incorrecto o valores negativos"]
    ErrorCSV --> Fin(["Fin"])
    ValidarCSV -- Sí --> ConfigSuelo["Cargar parámetros CC, PMP, RAW y humedad inicial H_1"]

    ConfigSuelo --> ValidarSuelo{"¿Parámetros coherentes? CC > PMP, 0 < RAW <= CC - PMP, 0 <= H_1 <= CC"}
    ValidarSuelo -- No --> ErrorSuelo["Mostrar error de parámetros de suelo"]
    ErrorSuelo --> Fin
    ValidarSuelo -- Sí --> ConfigRiego["Configurar riego: intervalo e I_fijo (Fija), I_umbral (Umbral)"]

    ConfigRiego --> ValidarRiego{"¿Configuración de riego válida? intervalo entero > 0, dosis >= 0"}
    ValidarRiego -- No --> ErrorRiego["Mostrar error de configuración de riego"]
    ErrorRiego --> Fin
    ValidarRiego -- Sí --> SelEstrategia["Seleccionar estrategias a simular: Fija / Umbral / Ambas"]

    SelEstrategia --> TomarEstrategia["Tomar la siguiente estrategia pendiente"]
    TomarEstrategia --> InitDia["Inicializar t = 1, H_t = H_1"]
    InitDia --> ObtenerClima["Leer P_t y ET_t del día t"]
    ObtenerClima --> EvaluarRiego{"¿Aplica riego? Fija: toca día de riego. Umbral: A_t = CC - H_t >= RAW"}

    EvaluarRiego -- Sí --> AsignarRiego["Riego I_t = I_fijo o I_umbral según estrategia"]
    EvaluarRiego -- No --> CeroRiego["Riego I_t = 0"]

    AsignarRiego --> CalcBalance
    CeroRiego --> CalcBalance

    CalcBalance["Balance provisional: H*_t = H_t + P_t + I_t - ET_t"]

    CalcBalance --> CheckExceso{"¿H*_t > CC?"}
    CheckExceso -- Sí --> Drenar["D_t = H*_t - CC, H_(t+1) = CC"]
    CheckExceso -- No --> NoDrenar["D_t = 0, H_(t+1) = max(0, H*_t)"]

    Drenar --> CheckAlerta
    NoDrenar --> CheckAlerta

    CheckAlerta{"¿H_(t+1) <= CC - RAW?"}
    CheckAlerta -- Sí --> ActivarAlerta["Marcar alerta de estrés hídrico"]
    CheckAlerta -- No --> Seguir["Operación en rango óptimo"]

    ActivarAlerta --> GuardarDia["Guardar registro diario: H_t, P_t, I_t, ET_t, D_t, H_(t+1), alerta"]
    Seguir --> GuardarDia

    GuardarDia --> FinBucle{"¿t == N?"}
    FinBucle -- No --> Avanzar["Avanzar al día siguiente: t = t + 1, H_t = H_(t+1)"]
    Avanzar --> ObtenerClima
    FinBucle -- Sí --> Consolidar["Calcular métricas de la estrategia: I_total, D_total, días con alerta y N_deficit (H_(t+1) < PMP)"]

    Consolidar --> CheckComparar{"¿Falta simular otra estrategia?"}
    CheckComparar -- Sí --> TomarEstrategia
    CheckComparar -- No --> Salidas["Comparar estrategias en consola, exportar CSV y renderizar gráfico"]
    Salidas --> Fin
```
