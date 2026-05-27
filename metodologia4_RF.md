# Metodología IV: Random Forest No Supervisado

## Contexto

Las metodologías anteriores construyen el IEC como una combinación lineal de sub-índices, asignando pesos de forma explícita — ya sea subjetiva (Metodología I) o derivada de eigenvalores (Metodología II). Ambas suponen que las variables pueden sumarse de manera significativa.

El análisis exploratorio de PCA y correlación mostró que las cinco variables del IEC son **estadísticamente independientes**: cada componente principal captura una sola variable con una carga ~0.994, y la matriz de cambio de base satisface B^T C B ≈ I. Dado que no existe estructura correlacional que explotar linealmente, se implementó una alternativa no paramétrica: el **Random Forest no supervisado**.

Este enfoque no asume relaciones entre variables. En cambio, permite que el algoritmo descubra por sí mismo los patrones de co-ocurrencia en los datos.

---

## Metodología

### Variables

Se utilizan las mismas cinco variables del IEC, normalizadas a [0, 1] con MinMaxScaler:

| Variable | Descripción |
|---|---|
| `Tiempo_en_minutos` | Duración estimada del trámite |
| `N_FORMATOS_FINAL` | Número de formatos requeridos |
| `CONTEO_NETO` | Número de requisitos netos |
| `nivel_digitalizacion` | Nivel de digitalización (se invierte al ordenar esfuerzo) |
| `TraCosto` | Si el trámite implica costo (binaria) |

> Se evaluó incluir `visitas_totales` y `Visitas RUTS` como proxies de complejidad percibida, pero se descartaron por no pertenecer al diseño original del IEC y por su correlación parcial con `Tiempo_en_minutos`. Ver `experimentacion_variables_visitas.md`.

### Procedimiento

1. **Datos sintéticos**: se duplica el dataset generando 665 trámites ficticios, donde cada variable se muestrea uniformemente dentro del rango observado [min, max].
2. **Clasificador RF**: se entrena un `RandomForestClassifier` (1 000 árboles, `max_features='sqrt'`) para distinguir trámites reales (etiqueta 1) de sintéticos (etiqueta 0). El accuracy de entrenamiento = 1.0 confirma que los datos reales tienen estructura distinguible de la aleatoriedad uniforme.
3. **Matriz de proximidad**: P[i,j] = fracción de árboles donde los trámites i y j cayeron en la misma hoja terminal. Valores altos indican perfiles similares.
4. **Clustering jerárquico Ward** sobre la matriz de distancias (1 − P), seleccionando 3 clusters.
5. **IEC_RF**: score continuo calculado por distancia euclídea al centroide del cluster, normalizado a [0, 100].

### Cálculo del IEC_RF

Los clusters se ordenan por esfuerzo mediano (con `nivel_digitalizacion` invertido) y se asignan bases:

| Cluster | Esfuerzo mediano | Base |
|---|---|---|
| Cluster 3 | 2.2306 | 70.00 |
| Cluster 2 | 1.7607 | 46.67 |
| Cluster 1 | 1.0806 | 23.33 |

Dentro de cada cluster, la distancia euclídea al centroide se normaliza a [0, 30] y se suma a la base. El resultado se re-escala globalmente a [0, 100].

---

## Resultados

### Distribución del IEC_RF

| Estadístico | Valor |
|---|---|
| Mínimo | 0.00 |
| Máximo | 100.00 |
| Media | 42.65 |
| Mediana | 48.49 |
| Desviación estándar | 26.66 |

La distribución es más asimétrica que la Metodología II (media 49.37, mediana 49.57), con los clusters generando tres modos claramente diferenciados.

### Top trámites por IEC_RF

| Trámite | Secretaría | IEC_RF |
|---|---|---|
| Impartición de justicia de conflictos laborales | Secretaría de Trabajo y Previsión Social | 100.00 |
| Registro de Sindicatos, Expedición de Toma de Nota | Secretaría de Trabajo y Previsión Social | 100.00 |
| Recurso de Revocación Estatal | Secretaría de Hacienda | 99.87 |
| Convocatoria del Instituto de Formación Profesional | Secretaría de Seguridad Pública | 98.39 |
| Preinscripciones de Educación Básica | Secretaría de Educación Pública | 98.00 |

### Hallazgo transversal

Los cuatro trámites identificados como de alto esfuerzo en las Metodologías I y II vuelven a aparecer entre los primeros puestos:

| Trámite | IEC — Met. I | IEC — Met. II | IEC_RF |
|---|---|---|---|
| Autorización de Empresa de Seguridad Privada | 89.7 | 93.46 | ~91 |
| Opinión favorable para armas/explosivos | 75.6 | 71.68 | ~89 |
| Dictaminación y Avalúos Catastral | 70.7 | 72.85 | ~88 |
| Convocatoria Instituto de Formación Profesional | 71.5 | 71.68 | 98.39 |

La consistencia a través de tres metodologías distintas — lineal ponderada, eigenvalores y Random Forest — refuerza la identificación de estos procesos como los de mayor carga burocrática en el estado de Hidalgo.

---

## Nota sobre la similitud entre Clusters 1 y 3

El agrupamiento jerárquico identifica tres niveles de esfuerzo, pero los clusters 1 y 3 presentan perfiles medianos similares en la mayoría de las variables. Esto no es un artefacto del método: es consistente con los hallazgos del PCA.

Las cinco variables del IEC son estadísticamente independientes, sin que ninguna domine la varianza del conjunto (eigenvalores entre 1.47 y 0.58; primera componente explica solo el 29.4 %). La **proximidad promedio de 0.4761** — por debajo de 0.5 — indica que los trámites no forman grupos naturales bien separados. El RF confirma lo que el PCA diagnosticó: la nube de datos no tiene estructura correlacional fuerte, y los límites entre clusters son difusos por construcción.

---

## Limitaciones

- La independencia estadística de las variables reduce la capacidad del RF para encontrar clusters con alta cohesión interna.
- Los 244 trámites con datos faltantes quedan excluidos del análisis.
- La escala de bases (23.33 / 46.67 / 70.00) introduce una discontinuidad entre clusters que puede sobreestimar o subestimar la distancia real entre trámites de grupos adyacentes.

---

## Archivos

| Archivo | Descripción |
|---|---|
| `metodologia4_RF.ipynb` | Notebook con implementación completa |
| `df_exp2.csv` | Dataset de entrada (665 trámites, imputado) |
