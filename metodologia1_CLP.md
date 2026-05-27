# Índice de Esfuerzo Ciudadano
## Metodología I: Combinación Lineal Ponderada

Se establece una metodología cuantitativa para evaluar la eficiencia administrativa mediante la integración y depuración de registros gubernamentales utilizando técnicas de fuzzy matching. A través de un análisis estadístico avanzado —empleando ECDF, KDE y Boxplots—, se determinaron umbrales objetivos para normalizar la heterogeneidad de los datos, permitiendo la creación de subíndices robustos de tiempo, requisitos, formatos, digitalización y costo.

Se proponen dos esquemas de ponderación: la **Propuesta A**, con mayor peso en el tiempo (tiempo 40%, requisitos 25%, digitalización 20%, formatos 10%, costo 5%), y la **Propuesta B**, con una distribución más equilibrada (tiempo 35%, requisitos 20%, digitalización 20%, formatos 15%, costo 10%). Tras comparar su distribución de frecuencias, se adopta la Propuesta B por ser más estricta y discriminar mejor entre trámites. Finalmente, la implementación del algoritmo de agrupamiento K-Means permitió segmentar los trámites en cuatro perfiles operativos: **Burocracia Pesada**, **Recaudación Eficiente**, **Rezago Analógico** y **Servicios Gratuitos Funcionales**, proporcionando una hoja de ruta estratégica para focalizar los esfuerzos de simplificación y transformación digital.

---

## Normalización Multidimensional de Trámites

### Sub-índice de Tiempo

Los tiempos registrados en los trámites se encontraban en unidades diferentes, tales como minutos, horas, días y meses. Para que el análisis fuera matemáticamente coherente, se transformaron todos los datos a la unidad base de minutos. Debido al rango extremo de la variable del tiempo, se utilizó una escala logarítmica para representar la ECDF de esta variable, permitiendo un mejor manejo de la información. Se aplicó un KDE para estimar la función densidad de probabilidad de dicha variable, encontrando una distribución bimodal, donde la población A fue catalogada de primera instancia como trámites rápidos y la población B como trámites lentos. Se utilizó la topología de la *pdf* para mediante el cálculo de máximos y mínimos se definieran los siguientes umbrales para categorizar a la población, así como los valores para su normalización:

1. *Rápidos:* $0 < t < 295\ \rm{min.}$ — Entre 0 y 4.9 hrs. Puntaje de 0 a 25.
2. *Lentos:* $295 < t < 9357\ \rm{min.}$ — Entre 4.9 hrs y 6.5 días. Puntaje de 26 a 70.
3. *Críticos:* $9357 < t < 238115\ \rm{min.}$ — Entre 6.5 días y 165 días. Puntaje de 71 a 100.
4. *Outliers:* $t > 238115\ \rm{min.}$ — Mayores a 165 días (2.7% de la población). Puntaje de 100.

Para asignar los valores de normalización se utiliza la siguiente función a tramos $T(t)$:

$$
T(t)=
\begin{cases}
\frac{t}{295}\times 25 &: 0 < t < 295 \\
\frac{t-295}{9357-295}\times(70-25)+25 &: 295 < t < 9357\\
\frac{t-9357}{238115-9357}+(100-70)+70 &: 9357 < t < 238115\\
100 &: t > 238115
\end{cases}
$$

Esta ponderación busca definir qué tan rápido sube el puntaje por cada unidad de tiempo agregada, tomando como referencia la ecuación $y = mt + b$ donde la pendiente $m = \frac{1}{x_{\rm{max}} - x_{\rm{min}}}$.

**¿Por qué no se usa un $z$-score?**

El $z$-score se define en función del promedio y la varianza tal que $z = \frac{t - \mu}{\sigma}$. Debido a la presencia de la cola correspondiente al 2.7% de trámites que duran más de 3.5 meses y de la naturaleza bimodal de la *pdf*, el promedio podría no proporcionar datos correctos y, por consecuencia, tampoco la desviación estándar. El $z$-score funciona bien en distribuciones normales, lo cual no es el caso del presente análisis.

---

### Sub-índice de Requisitos

Dado el trabajo previo de limpieza para distinguir entre requisitos, condicionantes, ruido y documentos de acreditación, se ajustó el umbral de requisitos mediante el análisis de boxplots, identificando tres poblaciones:

1. Bajo: hasta 3 requisitos.
2. Medio: hasta 6 requisitos.
3. Alto: 11 requisitos o más.

Se propone la siguiente función de puntuación:

$$
R = \min\left(100,\ \frac{r}{11} \times 100\right)
$$

Se toma el número de requisitos y se coloca un tope en 11, de modo que la mayor puntuación es 100 a partir de 11 requisitos. Cada requisito agrega $\frac{1}{11} \approx 9.09$ puntos. Por ejemplo:

- 3 requisitos → ~27 puntos
- 6 requisitos → ~54 puntos
- 11 o más requisitos → 100 puntos

---

### Sub-índice de Formatos

Para los formatos se aplicó una metodología similar mediante boxplots, encontrando un umbral alrededor de 2.5 formatos, redondeado a 3. A partir de 3 formatos se aplica una penalización mayor mediante una función a trozos:

$$
F =
\begin{cases}
f \times 10 &: f \leq 3\\
30 + (f - 3) \times 20 &: f > 3
\end{cases}
$$

Ejemplos:

- $f = 1 \Rightarrow$ 10 puntos
- $f = 3 \Rightarrow$ 30 puntos
- $f = 4 \Rightarrow$ 50 puntos
- $f = 6 \Rightarrow$ 90 puntos

---

### Sub-índice de Digitalización

Se cuenta con una escala granular de 14 niveles (1, 2, 3.1, 3.2, ..., 4.3) que describen las acciones que el ciudadano puede realizar de manera digital:

| Nivel | Descripción |
|:------|:------------|
| **4.3** | Resolución en línea inmediata. |
| **4.2** | Firma electrónica para solicitudes y resoluciones. |
| **4.1** | Emitir resoluciones oficiales en línea. |
| **3.9** | Llenar formatos en línea. |
| **3.8** | Agendar citas en línea. |
| **3.7** | Pago de derechos en línea. |
| **3.6** | Intercambio de información con otras dependencias de manera electrónica. |
| **3.5** | El ciudadano puede consultar el estatus de su trámite por medios electrónicos. |
| **3.4** | Notificación electrónica de vencimiento de plazo de respuesta. |
| **3.3** | Notificación electrónica de plazos de prevención. |
| **3.2** | Notificación electrónica de información faltante. |
| **3.1** | Recepción de solicitudes por medios electrónicos con acuse de recibo. |
| **2** | Posibilidad de descargar formatos. |
| **1** | Información del trámite disponible por medios electrónicos. |

Se definen 4 variables binarias que cuantifican situaciones de fricción que puede vivir el ciudadano:

1. **V** = ¿El ciudadano debe trasladarse físicamente aunque tenga cita o haya pagado? → 40%
2. **A** = ¿El ciudadano debe asistir a la oficina a conseguir formatos o requisitos? → 35%
3. **I** = ¿El ciudadano carece de medidas digitales para saber el estatus de su trámite? → 15%
4. **O** = ¿El ciudadano carece de herramientas de gestión de archivos? → 10%

| Nivel | Componentes activos | Cálculo | Puntaje |
|:------|:--------------------|:--------|:-------:|
| **1** | V + A + I + O | 40 + 35 + 15 + 10 | **100** |
| **2** | V + A + I | 40 + 35 + 15 | **90** |
| **3.1 – 3.6** | V + A | 40 + 35 | **75** |
| **3.7 – 3.9** | V | 40 | **40** |
| **4.1 – 4.3** | — | — | **0** |

---

### Sub-índice de Costo

Como no es posible diferenciar entre un trámite barato o caro —solo se cuenta con información sobre si el trámite cobra o no—, se aplica una penalización binaria por existencia de costo:

$$
C =
\begin{cases}
0 &: \text{FALSO} \\
100 &: \text{VERDADERO}
\end{cases}
$$

Dado el carácter binario de esta variable, se le asignó un peso bajo en el índice compuesto.

---

## Índice de Esfuerzo Ciudadano

Se proponen dos esquemas de ponderación:

**Propuesta A:**

$$\text{IEC}_A = T \times 0.40 + R \times 0.25 + F \times 0.10 + D \times 0.20 + C \times 0.05$$

**Propuesta B:**

$$\text{IEC}_B = T \times 0.35 + R \times 0.20 + F \times 0.15 + D \times 0.20 + C \times 0.10$$

Los trámites con valores nulos en alguna de las variables no fueron imputados sino excluidos del cálculo, lo que redujo el universo evaluado.

Al comparar la distribución de frecuencias de ambas propuestas, la Propuesta B tiende a correrse hacia la derecha, lo que indica que es más estricta al calificar. Por esta razón, se adopta la **Propuesta B** como índice de referencia para el análisis posterior.

---

## Resultados

### Cobertura del índice

De los 666 trámites del catálogo estatal, **422 obtuvieron un IEC calculado** bajo la Propuesta B. Los **244 restantes (~37%)** quedaron fuera por valores nulos en al menos una variable —principalmente `Tiempo_en_minutos`—, lo que representa una limitación metodológica significativa: casi un tercio del catálogo no pudo ser evaluado.

La **Secretaría de Educación Pública concentra 210 trámites** (~50% del universo evaluado), lo que distorsiona cualquier análisis agregado. Al filtrarla, el universo de trabajo se reduce a **412 trámites** distribuidos de forma más homogénea entre dependencias.

### Perfiles K-Means (universo completo, 4 clusters)

| Cluster | Nombre | n | Tiempo | Requisitos | Formatos | Digital | Costo |
|:-------:|:-------|:-:|:------:|:----------:|:--------:|:-------:|:-----:|
| 0 | Burocracia Pesada | 52 | 39.5 | **67.3** | 18.7 | 82.7 | 92.3 |
| 1 | Recaudación Eficiente | 223 | 37.1 | 17.4 | 5.8 | **94.6** | **100** |
| 2 | Rezago Analógico | 57 | 31.4 | 19.8 | 2.6 | **19.7** | 86.0 |
| 3 | Servicios Gratuitos | 90 | 39.2 | 15.3 | 6.7 | 94.4 | **0** |

### Trámites con mayor IEC (sin SEP)

Los trámites con mayor puntuación corresponden principalmente a la **Secretaría de Gobierno** y al **Registro Público de la Propiedad y del Comercio**:

| # | Trámite | IEC |
|:-:|:--------|:---:|
| 1 | Autorización para Prestar Servicios de Seguridad Privada en el Estado | **89.7** |
| 2 | Opinión favorable para actividades con armas, municiones, pólvoras y explosivos | 75.6 |
| 3 | Convocatoria del Instituto de Formación Profesional | 71.5 |
| 4 | Dictaminación de Avalúos (Catastral) | 70.7 |
| 5 | Autorización en materia de impacto ambiental del transporte de residuos industriales | 70.3 |
| 6 | Inscripción de registro de perito valuador (Catastral) | 68.6 |
| 7 | Inscripción registro de perito Topógrafo (Catastral) | 66.8 |
| 8-10 | Inscripciones en el Registro Público de la Propiedad y del Comercio | 64.3 |

### Concentración por secretaría (sin SEP)

| Secretaría | Trámites alto IEC | % del cluster | % del universo | Índice de concentración |
|:-----------|:-----------------:|:-------------:|:--------------:|:-----------------------:|
| Secretaría de Gobierno | 38 | 76.0% | 26.3% | **2.89** |
| Secretaría de Seguridad Pública | 2 | 4.0% | 1.5% | **2.74** |
| Procuraduría General de Justicia | 1 | 2.0% | 1.7% | 1.17 |
| Sec. de Medio Ambiente y RN | 2 | 4.0% | 3.9% | 1.03 |
| Secretaría de Hacienda | 6 | 12.0% | 12.2% | 0.99 |

La Secretaría de Gobierno presenta el índice de concentración más alto (2.89): está sobre-representada en el cluster de trámites más complejos respecto a su peso en el universo total.

---

## Limitaciones

1. **Pesos subjetivos:** Los pesos de la Propuesta B se asignaron por criterio experto sin sustento estadístico. Esto introduce un sesgo del analista que puede no reflejar la estructura real de los datos.

2. **Agrupamiento K-Means no discriminante:** El clustering no separó correctamente los trámites por nivel de esfuerzo global. Un trámite podía quedar en el cluster de "Burocracia Pesada" por tener digitalización baja (nivel 1, presencial), aunque su IEC general fuera bajo (~23 puntos). Un sub-índice dominaba la separación en lugar del índice compuesto.

3. **Sin EDA ni tratamiento formal de nulos:** No se realizó un análisis exploratorio previo ni se documentó la distribución de valores faltantes por variable y secretaría. Esto genera incertidumbre sobre el sesgo introducido por los 244 trámites excluidos.

---

## Motivación para siguientes metodologías

Las limitaciones anteriores motivaron dos líneas de trabajo posteriores:

- **Análisis de Correlación y PCA** (`PCA-and-correlation-analysis`): para entender la estructura de los datos antes de construir el índice y determinar si las variables son independientes o redundantes.
- **Ponderación por Eigenvalores** (`eigenvalue-weighted-index`): para derivar los pesos directamente de la dispersión estadística de los datos, eliminando la subjetividad del analista.
