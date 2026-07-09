# Codificación de variables categóricas

## Objetivo

Preparar las variables categóricas del dataset limpio para las etapas de
modelado de detección de anomalías, verificando categorías, inconsistencias y la representación numérica resultante.

## Dataset de entrada

- Archivo: `data/processed/dataset_limpio.csv`
- Registros: 117.922
- Columnas: 37
- Variable objetivo: `Attack_type`

## Variables categóricas identificadas

| Variable | Rol | Tipo de dato | Categorías | Valores faltantes |
|---|---|---|---:|---:|
| `proto` | Feature | `str` | 3 | 0 |
| `service` | Feature | `str` | 10 | 0 |
| `Attack_type` | Target | `str` | 12 | 0 |

## Revisión de categorías

Se compararon los valores originales con una versión normalizada mediante
eliminación de espacios y conversión a minúsculas.

- `proto`: no presentó inconsistencias.
- `service`: no presentó inconsistencias.
- `Attack_type`: usa mayúsculas y minúsculas mixtas en sus etiquetas, pero no presenta clases duplicadas por formato. Se conservó el formato original para mantener nombres interpretables.

## Estrategia de codificación

| Variable | Método | Justificación |
|---|---|---|
| `proto` | One-Hot Encoding | Es una variable nominal sin orden natural y tiene baja cardinalidad. |
| `service` | One-Hot Encoding | Es una variable nominal sin orden natural y tiene baja cardinalidad. |
| `Attack_type` | Label Encoding auxiliar | Es la variable objetivo. Se mantiene su versión textual para interpretación y se genera una versión numérica cuando un algoritmo o métrica lo requiera. |

## Consideración sobre fuga de información

Las columnas `id.orig_p`, `id.resp_p`, `service` y `proto` están marcadas como potencialmente leaky. Por tanto, el baseline recomendado utiliza:
```python
X_sin_leaky = X.drop(columns=["id.orig_p", "id.resp_p", "service", "proto"])