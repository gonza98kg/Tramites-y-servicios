# Índice de Esfuerzo Ciudadano — Hidalgo

**Unidad de Planeación y Prospectiva · Gobierno del Estado de Hidalgo · 2026**

---

## ¿Qué es este proyecto?

El Registro Único de Trámites y Servicios (RUTS) del Estado de Hidalgo publica alrededor de 2,000 trámites y servicios de distintas dependencias. Este repositorio documenta la construcción del **Índice de Esfuerzo Ciudadano (IEC)**: una métrica de complejidad burocrática diseñada como herramienta interna de diagnóstico para identificar trámites candidatos a simplificación administrativa.

Se aplicaron cuatro metodologías independientes sobre 665 trámites con información suficiente. La convergencia de enfoques radicalmente distintos en los mismos resultados es la evidencia más sólida del proyecto.

---

## Hallazgos principales

1. **La complejidad es documental, no temporal.** Requisitos y formatos explican la carga burocrática más que el tiempo de resolución. Reducir tiempos no reduce la fricción que enfrenta el ciudadano.

2. **El sistema fue diseñado sin coordinación.** Las variables del esfuerzo ciudadano son estadísticamente independientes entre sí — reflejo matemático de un sistema construido en silos institucionales sin criterio unificado de diseño (*fragmentación regulatoria*).

3. **Los mismos trámites aparecen siempre.** Con independencia de la metodología utilizada, los mismos trámites figuran consistentemente entre los de mayor carga burocrática. Eso no es coincidencia — es una señal.

---

## Estructura del repositorio

Cada rama contiene el notebook, el código y la documentación de una etapa metodológica:

| Rama | Contenido |
|---|---|
| `combinacion-lineal-ponderada` | Metodología I: pesos por criterio experto + K-Means |
| `PCA-and-correlation-analysis` | Análisis exploratorio: PCA, correlación e independencia estadística |
| `eigenvalue-weighted-index` | Metodología II: pesos derivados de eigenvalores del PCA |
| `entropia` | Metodología III: entropía de Shannon como medida de rareza |
| `RandomForest` | Metodología IV: Random Forest no supervisado + matriz de proximidad |

---

## Entregables

La carpeta [`docs/`](./docs/) contiene los documentos finales del proyecto:

| Documento | Descripción |
|---|---|
| `IEC_Hidalgo_Reporte_Tecnico_2026.pdf` | Reporte técnico completo con metodología, análisis y hallazgos |
| `IEC_Hidalgo_Resumen_Ejecutivo_2026.pdf` | Resumen ejecutivo de una página para tomadores de decisiones |
| `IEC_Hidalgo_Entregable_ServicioSocial_2026.pdf` | Documento presentado como entregable final del servicio social |

---

## Tecnologías

`Python` · `pandas` · `scikit-learn` · `scipy` · `matplotlib` · `seaborn`
