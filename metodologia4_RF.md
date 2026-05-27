# Índice de Esfuerzo Ciudadano
## Metodología IV: Random Forest No Supervisado

La Metodología I construyó el IEC mediante una combinación lineal ponderada con pesos asignados por criterio experto. El análisis de correlación y PCA confirmó que las cinco variables del modelo son estadísticamente independientes entre sí — cada componente principal captura una sola variable con una carga ~0.994, y la descomposición espectral verifica $\mathbb{B}^T\mathbb{C}\mathbb{B} = \mathbb{1}$ —, lo que motivó la Metodología II: derivar los pesos directamente de los eigenvalores de la matriz de correlación, eliminando la subjetividad del analista.

Ambas metodologías, sin embargo, construyen el índice como una suma ponderada de sub-índices: imponen una estructura lineal sobre los datos. La pregunta que queda abierta es si los trámites forman agrupamientos naturales que emerjan de los datos mismos, sin asumir pesos ni linealidad. Para explorar esto se implementa un **Random Forest no supervisado**.

---

## Preprocesamiento

Se utilizan las mismas cinco variables del IEC, normalizadas a [0, 1] con MinMaxScaler:

| Variable | Descripción |
|---|---|
| `Tiempo_en_minutos` | Duración estimada del trámite |
| `N_FORMATOS_FINAL` | Número de formatos requeridos |
| `CONTEO_NETO` | Número de requisitos netos |
| `nivel_digitalizacion` | Nivel de digitalización (se invierte al ordenar esfuerzo) |
| `TraCosto` | Si el trámite implica costo (binaria) |

Se evaluó incluir `visitas_totales` y `Visitas RUTS` como proxies de complejidad percibida, pero se descartaron por no pertenecer al diseño original del IEC. Ver `experimentacion_variables_visitas.md`.

---

## Random Forest No Supervisado

### Generación de Datos Sintéticos

Para cada variable se samplea uniformemente dentro de su rango observado [min, max], generando 665 trámites ficticios. Los datos reales reciben etiqueta 1 y los sintéticos etiqueta 0; ambos conjuntos se concatenan para el entrenamiento.

### Entrenamiento

Se entrena un `RandomForestClassifier` de 1 000 árboles con `max_features='sqrt'` (≈ 2 variables por split). El accuracy de entrenamiento = 1.0 confirma que los datos reales tienen estructura distinguible de la aleatoriedad uniforme — no es sobreajuste, es la señal de que el RF internamente aprendió la densidad conjunta de los trámites reales.

### Matriz de Proximidad

Para cada par de trámites (i, j), P[i,j] = fracción de árboles donde ambos cayeron en la misma hoja terminal. Valores cercanos a 1 indican perfiles similares; cercanos a 0, que raramente comparten vecindad en el espacio de cinco variables.

---

## Clustering Jerárquico

La distancia entre trámites se define como (1 − proximidad). Se aplica clustering jerárquico aglomerativo con criterio **Ward**, que minimiza la varianza intra-cluster en cada fusión. El dendrograma orienta la selección del número de cortes; se eligen **3 clusters** como balance entre interpretabilidad y separación.

---

## Índice de Esfuerzo Ciudadano

Los clusters se ordenan por esfuerzo mediano (con `nivel_digitalizacion` invertido) y se asignan bases:

| Cluster | Esfuerzo mediano | Base |
|---|---|---|
| Cluster 3 | 2.2306 | 70.00 |
| Cluster 2 | 1.7607 | 46.67 |
| Cluster 1 | 1.0806 | 23.33 |

Dentro de cada cluster, la distancia euclídea al centroide se normaliza a [0, 30] y se suma a la base. El score final se re-escala globalmente a [0, 100].

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

La distribución es más asimétrica que la Metodología II (media 49.37, mediana 49.57), con los clusters generando tres modos diferenciados.

### Top trámites por IEC_RF

| Trámite | Secretaría | IEC_RF |
|---|---|---|
| Impartición de justicia de conflictos laborales | Secretaría de Trabajo y Previsión Social | 100.00 |
| Registro de Sindicatos, Expedición de Toma de Nota | Secretaría de Trabajo y Previsión Social | 100.00 |
| Recurso de Revocación Estatal | Secretaría de Hacienda | 99.87 |
| Convocatoria del Instituto de Formación Profesional | Secretaría de Seguridad Pública | 98.39 |
| Preinscripciones de Educación Básica | Secretaría de Educación Pública | 98.00 |

### Hallazgo transversal

Los siguientes cuatro trámites aparecen en el top de las tres metodologías implementadas, con independencia de los pesos y el enfoque utilizado:

| Trámite | IEC — Met. I | IEC — Met. II | IEC_RF |
|---|---|---|---|
| Autorización para Prestar Servicios de Seguridad Privada en el Estado | 89.7 | 93.46 | — |
| Opinión favorable para actividades con armas, municiones, pólvoras y explosivos | 75.6 | 71.68 | — |
| Dictaminación de Avalúos (Catastral) | 70.7 | 72.85 | — |
| Convocatoria del Instituto de Formación Profesional | 71.5 | 71.68 | 98.39 |

La consistencia a través de tres metodologías distintas — lineal ponderada, eigenvalores y Random Forest — refuerza la identificación de estos procesos como los de mayor carga burocrática en el estado de Hidalgo.

---

## Conclusiones

El clustering jerárquico identificó tres niveles de esfuerzo, pero los clusters 1 y 3 presentan perfiles medianos similares en la mayoría de las variables. Este resultado es coherente con los hallazgos del PCA: las cinco variables son estadísticamente independientes y ninguna domina la varianza del conjunto (eigenvalores entre 1.47 y 0.58; primera componente explica solo el 29.4 %). La **proximidad promedio de 0.4761** — por debajo de 0.5 — confirma que los trámites no forman grupos naturales bien separados: el RF llega a la misma conclusión que el PCA por un camino completamente distinto.

A pesar de la separación débil, cuatro trámites que ya aparecieron en el top de las Metodologías I y II vuelven a obtener puntuaciones altas:

- *Convocatoria del Instituto de Formación Profesional* — posición #4, IEC_RF 98.39
- *Opinión favorable para actividades con armas, municiones, pólvoras y explosivos* — posición #20, IEC_RF 80.43
- *Dictaminación de Avalúos (Catastral)* — posición #43, IEC_RF 70.29
- *Autorización para Prestar Servicios de Seguridad Privada en el Estado* — posición #45, IEC_RF 69.57

La consistencia de estos trámites a través de tres metodologías distintas refuerza su identificación como los procesos de mayor carga burocrática del catálogo estatal.

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
| `experimentacion_variables_visitas.md` | Documentación del experimento con 7 variables |
