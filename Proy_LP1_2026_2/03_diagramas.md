# Diagramas del Sistema - Proyecto 08

## Simulador de Riego y Balance Hídrico del Suelo

**Curso:** SW303 - Lenguaje de Programación I (Imperativo)  
**Responsable:** Diseñador de Flujos - Franco Herrera Rios

---

## 1. Propósito de los diagramas

Los diagramas describen cómo interactúa el usuario con el simulador y cómo fluye el proceso de cálculo día a día, desde la carga de los datos hasta la generación de resultados.

El diagrama de casos de uso muestra las funciones que el sistema ofrece al usuario. El diagrama de flujo detalla el orden de las operaciones de la simulación: la validación de los datos de entrada, la decisión de riego, el balance hídrico, el drenaje por exceso, la alerta de estrés hídrico y el cierre con las métricas y los reportes.

---

### 3.1 Diagrama de Casos de Uso

```mermaid
flowchart LR
    Actor(("Usuario / Agrónomo"))

    subgraph Sistema["Simulador de Riego y Balance Hídrico"]
        CU1("Cargar archivo meteorológico CSV")
        CU2("Configurar parámetros de suelo y cultivo")
        CU3("Seleccionar estrategia de riego: Fijo / Umbral")
        CU4("Ejecutar simulación diaria")
        CU5("Consultar alertas de déficit hídrico")
        CU6("Exportar reporte CSV de balance")
        CU7("Visualizar evolución gráfica")
    end

    Actor --> CU1
    Actor --> CU2
    Actor --> CU3
    Actor --> CU4
    Actor --> CU5
    Actor --> CU6
    Actor --> CU7

    CU4 -. "«include»" .-> CU5
```

### 3.2 Diagrama de Flujo del Proceso Diario de Simulación

```mermaid
flowchart TD
    Inicio(["Inicio de simulación"]) --> CargarArchivos["Lectura de registros climáticos CSV"]
    CargarArchivos --> ValidarCSV{"¿CSV válido?"}
    ValidarCSV -- No --> ErrorCSV["Mostrar error: ruta inválida, formato incorrecto o valores negativos"]
    ErrorCSV --> Fin(["Fin"])
    ValidarCSV -- Sí --> ConfigSuelo["Cargar parámetros CC, PMP, RAW y humedad inicial θ0"]

    ConfigSuelo --> ValidarSuelo{"¿Parámetros coherentes? PMP < CC"}
    ValidarSuelo -- No --> ErrorSuelo["Mostrar error de parámetros de suelo"]
    ErrorSuelo --> Fin
    ValidarSuelo -- Sí --> SelEstrategia["Establecer estrategia de riego: Calendario vs Umbral"]

    SelEstrategia --> Bucle["Iniciar bucle: para día t = 1 hasta N"]
    Bucle --> ObtenerClima["Leer P_t y ET_t del día"]
    ObtenerClima --> EvaluarRiego{"¿Aplica riego según estrategia?"}

    EvaluarRiego -- Sí --> AsignarRiego["Riego I_t > 0"]
    EvaluarRiego -- No --> CeroRiego["Riego I_t = 0"]

    AsignarRiego --> CalcBalance
    CeroRiego --> CalcBalance

    CalcBalance["Balance tentativo: θ*_t = θ_(t-1) + P_t + I_t - ET_t"]

    CalcBalance --> CheckExceso{"¿θ*_t > CC?"}
    CheckExceso -- Sí --> Drenar["D_t = θ*_t - CC, θ_t = CC"]
    CheckExceso -- No --> NoDrenar["D_t = 0"]

    Drenar --> CheckAlerta
    NoDrenar --> CheckAlerta

    CheckAlerta{"¿θ_t < RAW?"}
    CheckAlerta -- Sí --> ActivarAlerta["Marcar alerta: déficit / estrés hídrico"]
    CheckAlerta -- No --> Seguir["Operación en rango óptimo"]

    ActivarAlerta --> GuardarDia["Guardar registro diario de estado"]
    Seguir --> GuardarDia

    GuardarDia --> FinBucle{"¿t == N?"}
    FinBucle -- No --> Bucle
    FinBucle -- Sí --> Consolidar["Calcular métricas globales acumuladas"]

    Consolidar --> Salidas["Generar archivo CSV y renderizar gráfico"]
    Salidas --> Fin
```
