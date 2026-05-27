# Índice de Esfuerzo Ciudadano
## Metodología II: Ponderación por Eigenvalores

---

## Contexto

La Metodología I (`combinacion-lineal-ponderada`) construyó el IEC asignando pesos por criterio experto: tiempo 35%, digitalización 20%, requisitos 20%, formatos 15%, costo 10%. Si bien produjo resultados coherentes en los extremos, la elección de pesos introducía un sesgo del analista sin sustento estadístico.

El análisis exploratorio posterior (`PCA-and-correlation-analysis`) reveló que las 5 variables del modelo son **estadísticamente independientes** entre sí — los loadings del PCA mostraron una alineación casi perfecta (0.994) entre cada componente principal y una única variable, y la descomposición espectral de la matriz de correlación confirmó este hallazgo: $\mathbb{B}^T\mathbb{C}\mathbb{B} = \mathbb{1}$. Esta independencia implica que cada variable aporta información única e irremplazable, y que los eigenvalores de $\mathbb{C}$ pueden usarse directamente como pesos objetivos.

---

## Metodología

Los pesos $w_i$ se derivan normalizando los eigenvalores de la matriz de correlación de Pearson $\mathbb{C}$:

$$w_i = \frac{\lambda_i}{\sum_{j=1}^{5} \lambda_j}$$

| Variable | Sub-índice | $\lambda_i$ | $w_i$ |
|---|---|:---:|:---:|
| `nivel_digitalizacion` | $I_{Digital}$ | 1.4711 | **29.42%** |
| `Tiempo_en_minutos` | $I_{Tiempo}$ | 1.1132 | **22.26%** |
| `CONTEO_NETO` | $I_{Requisitos}$ | 1.0015 | **20.03%** |
| `N_FORMATOS_FINAL` | $I_{Formatos}$ | 0.8391 | **16.78%** |
| `TraCosto` | $I_{Costo}$ | 0.5750 | **11.50%** |

El IEC resultante es:

$$IEC = 0.2942\cdot I_{Digital} + 0.2226\cdot I_{Tiempo} + 0.2003\cdot I_{Requisitos} + 0.1678\cdot I_{Formatos} + 0.1150\cdot I_{Costo}$$

Las funciones de normalización de cada sub-índice al rango [0, 100] son las mismas definidas en la Metodología I.

---

## Resultados

### Distribución del IEC

| Estadístico | Valor |
|-------------|:-----:|
| Mínimo | 1.86 |
| Media | 49.37 |
| Mediana | 49.57 |
| Máximo | 93.46 |
| Desv. estándar | 12.34 |
| Umbral IEC alto (P75) | 57.76 |

La distribución es considerablemente más simétrica que en la Metodología I. En aquella, el tiempo concentraba el 35–40% del peso y dominaba la separación entre trámites. Con pesos derivados de eigenvalores el rango es 11.5%–29.4%, lo que produce un índice más balanceado donde ninguna variable domina unilateralmente.

### Trámites con mayor IEC

| # | Trámite | Secretaría | IEC |
|:-:|---------|-----------|:---:|
| 1 | Autorización para Prestar Servicios de Seguridad Privada | Seguridad Pública | **93.46** |
| 2-3 | Concesiones y Permisos de Servicios Auxiliares / Transferencia por Designación | Movilidad y Transporte | 78.60 |
| 4 | Titulación en la UICEH | Educación Pública | 77.75 |
| 5-7 | Otorgamiento o Renovación de Concesiones y Permisos | Movilidad y Transporte | 76.78 |
| 8-9 | Autenticación y Registro de Certificados de Estudios | Educación Pública | 75.83 |

### Secretarías con mayor concentración de trámites complejos

El índice de concentración mide la sobre-representación de una secretaría en el grupo de IEC alto respecto a su peso en el universo total. Un valor > 1 indica sobre-representación.

| Secretaría | Total | IEC alto | % IEC alto | Índice concentración |
|-----------|:-----:|:--------:|:----------:|:--------------------:|
| Movilidad y Transporte | 29 | 15 | 51.7% | **2.07** |
| Salud | 23 | 9 | 39.1% | 1.78 |
| Medio Ambiente y RN | 16 | 6 | 37.5% | 1.71 |
| Desarrollo Económico | 17 | 6 | 35.3% | 1.61 |
| Gobierno | 108 | 36 | 33.3% | 1.52 |
| Seguridad Pública | 6 | 2 | 33.3% | **5.35** |

La **Secretaría de Movilidad y Transporte** lidera por porcentaje (51.7%), mientras que la **Secretaría de Seguridad Pública**, con solo 6 trámites, tiene el índice de concentración más alto: una fracción desproporcionada de sus trámites cae en el grupo de alta complejidad.

### Hallazgo transversal: trámites recurrentes

Al comparar con la Metodología I, cuatro trámites aparecen en el top de ambos índices independientemente de los pesos utilizados:

| Trámite | IEC — Metodología I | IEC — Metodología II |
|:--------|:-------------------:|:--------------------:|
| Autorización para Prestar Servicios de Seguridad Privada | 89.7 | **93.46** |
| Opinión favorable para actividades con armas, municiones y explosivos | 75.6 | 71.68 |
| Dictaminación de Avalúos (Catastral) | 70.7 | 72.85 |
| Convocatoria del Instituto de Formación Profesional | 71.5 | 71.68 |

La coincidencia sugiere que la alta complejidad de estos trámites es una característica estructural y no un artefacto de la ponderación.

---

## Limitaciones

**Piso por digitalización:** cualquier trámite presencial obtiene un mínimo de ~29.42 puntos por el sub-índice de digitalización, independientemente de sus demás características. Dado que la mediana de `nivel_digitalizacion` es 1 (presencial), más de la mitad del catálogo parte de ese piso, lo que comprime la discriminación en el rango medio.

**Imputación de tiempo:** los trámites con tiempo imputado con la media global (~22,864 minutos ≈ 16 días) reciben $I_{Tiempo} \approx 71.8$, lo que puede inflar el IEC de trámites que en la realidad son simples. Una imputación agrupada por secretaría o tipo de trámite representaría una mejora relevante.

---

## Archivos

| Archivo | Descripción |
|---------|-------------|
| `metodologia2_eigenvalores.ipynb` | Notebook principal: carga, sub-índices, IEC, distribución y análisis de congruencia |
| `df_exp2.csv` | DataFrame base (665 trámites preprocesados, heredado de `PCA-and-correlation-analysis`) |
