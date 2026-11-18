# Arquitectura y Definición de Módulos – Proyecto 08
**Simulador de Riego y Balance Hídrico del Suelo** | **Curso:** SW303 – LP1  
**Responsable:** Bryan Yllanes

---

## 1. Flujo de Interacción Modular

El sistema aplica programación **modular estructurada en C estándar** (`C99/C11`, compilable con GCC y `-Wall -Wextra`), dividiendo la arquitectura en 6 pares de archivos (`.h` para interfaces/estructuras y `.c` para implementación):

```text
[ datos_climaticos ] ──(P_t, ET_t)──┐
                                    ├──> [ balance_hidrico ] <──(I_t)── [ control_riego ]
[ parametros_suelo ] ──(CC,PMP,RAW)─┴─────────────────────────────────────────┘
                                                 │
                                                 ▼ (Historial EstadoDia)
                                    ┌────────────────────────┐
                                    ▼                        ▼
                             [ indicadores ]           [ reportes ]
                  (I_total, D_total, N_alerta,      (Salida CSV / Gráfica)
                          N_deficit)
```

`parametros_suelo` también alimenta a `indicadores` (umbrales de alerta y déficit) y a `reportes` (líneas de referencia CC, CC − RAW y PMP en la gráfica).

### 1.1. Convenciones comunes (alineadas con `02_modelo_matematico.md`)

* **Unidades:** todas las cantidades de agua (CC, PMP, RAW, H, P, ET, I, D) están en mm de lámina de agua equivalente.
* **Notación:** $H_t$ = humedad al **inicio** del día $t$ (campo `humedad_inicial`); $H_{t+1}$ = humedad al **final** del día (campo `humedad_final`). La humedad inicial de la simulación es $H_1$.
* **Rango de humedad:** $H_{t+1} = \max(0, H_t^* - D_t)$, por lo que siempre $0 \le H_{t+1} \le CC$.
* **RAW** es un parámetro del suelo (vive en `ParametrosSuelo`). La estrategia por umbral lo **lee** desde el suelo; `ConfiguracionRiego` no lo duplica.
* **Alerta de estrés:** $H_{t+1} \le CC - RAW$.
* **Día con déficit:** $H_{t+1} < PMP$ (siempre implica alerta, ya que $RAW \le CC - PMP$).

---

## 2. Especificación de Módulos

### 2.1 `datos_climaticos` (`.h` / `.c`)
* **Función:** Lee y valida ($P_t \ge 0$, $ET_t \ge 0$) los registros diarios desde el CSV (RF-01, RNF-06).
* **Estructura y Prototipos:**
```c
typedef struct {
    int dia;
    double precipitacion;       // P_t (mm)
    double evapotranspiracion;  // ET_t (mm)
} RegistroClima;

int cargar_datos_climaticos(const char *ruta_csv, RegistroClima **registros, int *total_dias);
void liberar_datos_climaticos(RegistroClima *registros);
```

### 2.2 `parametros_suelo` (`.h` / `.c`)
* **Función:** Configura y valida las propiedades físicas del suelo: $CC > PMP$, $0 < RAW \le CC - PMP$ y $0 \le H_1 \le CC$ (RF-02, RNF-06). La condición sobre RAW asegura que el umbral crítico $CC - RAW$ nunca quede por debajo del PMP.
* **Estructura y Prototipos:**
```c
typedef struct {
    double capacidad_campo;    // CC (mm)
    double punto_marchitez;    // PMP (mm)
    double agua_aprovechable;  // RAW (mm); umbral crítico = CC - RAW
    double humedad_inicial;    // H_1 (mm)
} ParametrosSuelo;

ParametrosSuelo inicializar_suelo(double cc, double pmp, double raw, double h_inicial);
int validar_parametros_suelo(const ParametrosSuelo *suelo);
```

### 2.3 `control_riego` (`.h` / `.c`)
* **Función:** Calcula el riego diario $I_t$ según estrategia fija (por intervalo) o por umbral (RF-06). En la estrategia por umbral, el riego se activa si $A_t = CC - H_t \ge RAW$, con $H_t$ = humedad al inicio del día y RAW leído de `ParametrosSuelo`.
* **Validaciones de `validar_configuracion_riego`:** `intervalo_dias` entero positivo y dosis ($I_{fijo}$, $I_{umbral}$) mayores o iguales que cero (RNF-06).
* **Estructuras y Prototipos:**
```c
typedef enum { ESTRATEGIA_FIJA, ESTRATEGIA_UMBRAL } TipoEstrategia;

typedef struct {
    TipoEstrategia estrategia;
    int intervalo_dias;   // estrategia fija: cada cuántos días se riega
    double dosis_fija;    // I_fijo (mm)
    double dosis_umbral;  // I_umbral (mm); el umbral CC - RAW viene del suelo
} ConfiguracionRiego;

int validar_configuracion_riego(const ConfiguracionRiego *config);
// humedad_actual = H_t (inicio del día)
double calcular_riego_diario(const ConfiguracionRiego *config, const ParametrosSuelo *suelo, double humedad_actual, int dia);
```

### 2.4 `balance_hidrico` (`.h` / `.c`)
* **Función:** Motor diario de conservación de masa ($H_t^* = H_t + P_t + I_t - ET_t$), drenaje ($D_t = \max(0, H_t^* - CC)$) y humedad final ($H_{t+1} = \max(0, H_t^* - D_t)$, acotada a $0 \le H_{t+1} \le CC$) (RF-03, RF-04). Marca `alerta_estres` cuando $H_{t+1} \le CC - RAW$ usando `verificar_alerta_humedad`.
* **Estructura y Prototipos:**
```c
typedef struct {
    int dia;
    double humedad_inicial;       // H_t (mm)
    double precipitacion;         // P_t (mm)
    double riego;                 // I_t (mm)
    double evapotranspiracion;    // ET_t (mm)
    double drenaje;               // D_t (mm)
    double humedad_final;         // H_(t+1) (mm), acotada a [0, CC]
    int alerta_estres;            // 1 si H_(t+1) <= CC - RAW, 0 en caso contrario
} EstadoDia;

// h_actual = H_t (inicio del día); la humedad_final devuelta es H_(t+1)
EstadoDia simular_dia(int dia, double h_actual, RegistroClima clima, const ParametrosSuelo *suelo, const ConfiguracionRiego *riego);

// Parte de H_1 = suelo->humedad_inicial. Devuelve un arreglo dinámico de total_dias
// elementos; el llamador debe liberarlo con free().
EstadoDia* ejecutar_simulacion_completa(const RegistroClima *clima, int total_dias, const ParametrosSuelo *suelo, const ConfiguracionRiego *riego);
```

### 2.5 `indicadores` (`.h` / `.c`)
* **Función:** Evalúa la alerta de estrés y calcula $I_{total}$, $D_{total}$, los días con alerta ($N_{alerta}$) y los días con déficit hídrico $N_{deficit}$ ($H_{t+1} < PMP$) (RF-05, RF-07).
* **Estructura y Prototipos:**
```c
typedef struct {
    double consumo_total_riego;  // I_total (mm)
    double drenaje_total;        // D_total = suma de D_t (mm)
    int dias_deficit_hidrico;    // N_deficit: días con H_(t+1) < PMP
    int dias_alerta_umbral;      // N_alerta: días con H_(t+1) <= CC - RAW
} ResumenIndicadores;

// Retorna 1 si humedad_final <= CC - RAW (alerta de estrés), 0 en caso contrario
int verificar_alerta_humedad(double humedad_final, const ParametrosSuelo *suelo);
ResumenIndicadores calcular_indicadores(const EstadoDia *historial, int total_dias, const ParametrosSuelo *suelo);
```

### 2.6 `reportes` (`.h` / `.c`)
* **Función:** Exporta la tabla diaria a CSV, compara estrategias en consola y grafica la evolución en ASCII (RF-08, RNF-03, RNF-04). La gráfica incluye la humedad ($H_{t+1}$), la precipitación y el riego, con líneas de referencia para CC, $CC - RAW$ y PMP tomadas de `ParametrosSuelo`. El uso de Raylib es opcional y no es requisito de la versión base (RNF-02, RNF-04).
* **Prototipos:**
```c
int exportar_resultados_csv(const char *ruta_salida, const EstadoDia *historial, int total_dias, const ResumenIndicadores *indicadores);
void mostrar_comparacion_consola(const ResumenIndicadores *res_fijo, const ResumenIndicadores *res_umbral);
void graficar_evolucion_ascii(const EstadoDia *historial, int total_dias, const ParametrosSuelo *suelo);
```

---

## 3. Trazabilidad Rápida

| Módulo | Cabecera | RF / RNF | Rol Principal |
| :--- | :--- | :--- | :--- |
| `datos_climaticos` | `datos_climaticos.h` | RF-01, RNF-03, RNF-06 | Lectura y validación de CSV climático |
| `parametros_suelo` | `parametros_suelo.h` | RF-02, RNF-06 | Parámetros físicos del suelo ($CC, PMP, RAW, H_1$) y su validación |
| `control_riego` | `control_riego.h` | RF-06, RNF-06 | Reglas de riego fijo vs. por umbral |
| `balance_hidrico` | `balance_hidrico.h` | RF-03, RF-04 | Cálculo diario de humedad, drenaje y marca de alerta |
| `indicadores` | `indicadores.h` | RF-05, RF-07 | Alertas y métricas acumuladas ($I_{total}$, $D_{total}$, $N_{alerta}$, $N_{deficit}$) |
| `reportes` | `reportes.h` | RF-08, RNF-03, RNF-04 | Salida CSV y gráficos ASCII (Raylib opcional) |
