# Preprocesamiento y Limpieza — RT_IOT2022

**Responsable:** Edwin  
**Dataset fuente:** `data/rt-iot2022.zip` → archivo interno `RT_IOT2022`  
**Notebook:** `notebooks/03_dataset_limpio.ipynb`  
**Decisiones de referencia:** `reports/decisiones_preparacion.md`  
**Fecha:** 2026-07-05

---

## Resumen ejecutivo

| Métrica | Valor |
|---|---|
| Columnas originales | **84** |
| Columnas tras limpieza | **37** |
| Columnas eliminadas | **47** (1 constante + 46 redundantes) |
| Filas originales | **123,117** |
| Filas tras drop_duplicates | **117,922** |
| Filas duplicadas eliminadas | **5,195** (4.22%) |
| Valores faltantes | **0** |
| Dataset de salida | `data/processed/dataset_limpio.csv` |

---

## Respuestas a las preguntas del enunciado

### ¿Cuántas columnas tenía originalmente el dataset?

**84 columnas.** El dataset se carga con `index_col=0`, lo que absorbe la columna de índice residual del CSV sin contarla entre las features.

---

### ¿Cuántas columnas quedaron después del preprocesamiento?

**37 columnas.** Se eliminaron 47 en total: 1 constante + 46 redundantes.

---

### ¿Qué columnas fueron eliminadas?

#### Columna constante (1)

| Columna | Razón |
|---|---|
| `bwd_URG_flag_count` | 100% ceros — varianza cero, no aporta información, rompe Pearson (NaN). Decisión E1. |

#### Columnas redundantes por grupo — `REDUNDANT_COLS_DROP` (46)

La lista fue definida en `notebooks/03_matriz_correlacion.ipynb` mediante clustering jerárquico Spearman con complete linkage (corte |ρ|=0.9). De cada grupo se conservó un representante aplicando la regla: **no-leaky → más denso (menos ceros) → más interpretable**.

| Grupo | Representante(s) conservado(s) | Columnas eliminadas |
|---|---|---|
| G1 | `fwd_pkts_tot`, `bwd_pkts_payload.avg` | `bwd_data_pkts_tot`, `bwd_pkts_payload.max`, `bwd_pkts_payload.tot`, `bwd_pkts_payload.std`, `fwd_iat.min`, `fwd_iat.max`, `fwd_iat.tot`, `fwd_iat.avg`, `bwd_iat.min`, `bwd_iat.max`, `bwd_iat.tot`, `bwd_iat.avg`, `flow_iat.std`, `fwd_subflow_pkts`, `bwd_subflow_bytes` |
| G2 | `flow_duration` | `flow_iat.min`, `flow_iat.max`, `flow_iat.tot`, `flow_iat.avg`, `active.min`, `active.max`, `active.tot`, `active.avg` |
| G3 | `bwd_init_window_size`, `fwd_iat.std` | `bwd_PSH_flag_count`, `fwd_pkts_payload.std`, `bwd_iat.std` |
| G4 | `flow_pkts_per_sec` | `fwd_pkts_per_sec`, `bwd_pkts_per_sec`, `payload_bytes_per_second` |
| G5 | `fwd_header_size_max`, `fwd_init_window_size` | `fwd_header_size_tot`, `fwd_header_size_min` |
| G6 | `bwd_header_size_max`, `flow_ACK_flag_count` | `bwd_header_size_tot`, `bwd_header_size_min` |
| G7 | `idle.tot` | `idle.min`, `idle.max`, `idle.avg` |
| G8 | `fwd_pkts_payload.avg` | `fwd_pkts_payload.min` |
| G9 | `fwd_subflow_bytes` | `fwd_pkts_payload.max`, `fwd_pkts_payload.tot` |
| G10 | `fwd_bulk_bytes` | `fwd_bulk_packets`, `fwd_bulk_rate` |
| G11 | *(familia entera eliminada)* | `bwd_bulk_bytes`, `bwd_bulk_packets`, `bwd_bulk_rate` |
| G12 | `bwd_pkts_tot` | `bwd_subflow_pkts` |
| G13 | `flow_CWR_flag_count` | `flow_ECE_flag_count` |

> **Nota G11:** la familia `bwd_bulk.*` se eliminó completamente porque sus valores no-cero están mezclados entre tráfico normal y ataque (46%/54%) sin señal discriminativa clara. Ver decisión G1 en `reports/decisiones_preparacion.md`.

---

### ¿Qué problemas persistieron después de la limpieza?

#### 1. Desbalance extremo en `Attack_type`

| Clase | Filas | % |
|---|---|---|
| DOS_SYN_Hping | ~79,464 | ~67.4% |
| Thing_Speak | ~8,108 | ~6.9% |
| ARP_poisioning | ~7,750 | ~6.6% |
| ... (demás clases) | ... | ... |

El ratio entre la clase mayoritaria y la minoritaria supera 2,000:1. Esto debe manejarse con `class_weight="balanced"` y métricas adecuadas (F1-macro, PR-AUC). Ver decisiones D1 y D2.

#### 2. Columnas leaky conservadas (toggle — decisión H1)

Las columnas `id.orig_p`, `id.resp_p`, `service` y `proto` **están presentes** en el dataset limpio pero deben usarse con precaución:

- `id.resp_p` (puerto destino) correlaciona >0.9 con `fwd_pkts_payload.avg` → predice el payload casi 1:1, inflando métricas.
- `service` y `proto` pueden codificar el tipo de ataque directamente.
- **Acción:** usar `include_leaky=False` en `build_preprocessor()` para el baseline. Correr una variante con y sin leaky para medir su impacto real.

---

## Columnas que permanecen en el dataset limpio (37)

| # | Columna | Tipo | Grupo / Rol |
|---|---|---|---|
| 1 | `id.orig_p` | int64 | Leaky — puerto origen |
| 2 | `id.resp_p` | int64 | Leaky — puerto respuesta |
| 3 | `proto` | category | Leaky — protocolo |
| 4 | `service` | category | Leaky — servicio |
| 5 | `flow_duration` | float64 | G2 rep — duración del flujo |
| 6 | `fwd_pkts_tot` | int64 | G1 rep — paquetes forward totales |
| 7 | `bwd_pkts_tot` | int64 | G12 rep — paquetes backward totales |
| 8 | `fwd_data_pkts_tot` | int64 | Solitaria |
| 9 | `flow_pkts_per_sec` | float64 | G4 rep — tasa de paquetes |
| 10 | `down_up_ratio` | float64 | Solitaria |
| 11 | `fwd_header_size_max` | int64 | G5 rep — cabecera fwd máxima |
| 12 | `bwd_header_size_max` | int64 | G6 rep — cabecera bwd máxima |
| 13 | `flow_FIN_flag_count` | int64 | Solitaria — flag FIN |
| 14 | `flow_SYN_flag_count` | int64 | Solitaria — flag SYN |
| 15 | `flow_RST_flag_count` | int64 | Solitaria — flag RST |
| 16 | `fwd_PSH_flag_count` | int64 | Solitaria — flag PSH fwd |
| 17 | `flow_ACK_flag_count` | int64 | G6 rep — flag ACK |
| 18 | `fwd_URG_flag_count` | int64 | Solitaria — flag URG fwd |
| 19 | `flow_CWR_flag_count` | float64 | G13 rep — marcador de tráfico normal |
| 20 | `fwd_pkts_payload.avg` | float64 | G8 rep — payload fwd promedio |
| 21 | `bwd_pkts_payload.min` | float64 | Solitaria |
| 22 | `bwd_pkts_payload.avg` | float64 | G1 rep — payload bwd promedio |
| 23 | `flow_pkts_payload.min` | float64 | Solitaria |
| 24 | `flow_pkts_payload.max` | float64 | Solitaria |
| 25 | `flow_pkts_payload.tot` | float64 | Solitaria |
| 26 | `flow_pkts_payload.avg` | float64 | Solitaria |
| 27 | `flow_pkts_payload.std` | float64 | Solitaria — variabilidad payload |
| 28 | `fwd_iat.std` | float64 | G3 rep — variabilidad tiempos fwd |
| 29 | `fwd_subflow_bytes` | float64 | G9 rep — bytes subflujo fwd |
| 30 | `fwd_bulk_bytes` | float64 | G10 rep — marcador ARP |
| 31 | `active.std` | float64 | Solitaria |
| 32 | `idle.tot` | float64 | G7 rep — tiempo idle (señal normal) |
| 33 | `idle.std` | float64 | Solitaria |
| 34 | `fwd_init_window_size` | int64 | G5 rep — ventana TCP inicial fwd |
| 35 | `bwd_init_window_size` | int64 | G3 rep — ventana TCP inicial bwd |
| 36 | `fwd_last_window_size` | int64 | Solitaria |
| 37 | `Attack_type` | category | **Target** (variable objetivo) |

---

## Instrucciones de uso del dataset limpio

```python
import pandas as pd

df = pd.read_csv('data/processed/dataset_limpio.csv', index_col=0)

# Separar features del target
y = df['Attack_type']
X = df.drop(columns=['Attack_type'])

# Excluir columnas leaky al modelar (recomendado para baseline)
LEAKY = ['id.orig_p', 'id.resp_p', 'service', 'proto']
X_sin_leaky = X.drop(columns=LEAKY)
```

---

*Documento generado en base a `notebooks/03_dataset_limpio.ipynb` | RT_IOT2022 | 117,922 filas × 37 columnas*
