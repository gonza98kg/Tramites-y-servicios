# Experimentación: Inclusión de `visitas_totales` y `Visitas RUTS` en el RF No Supervisado

## Contexto

En la implementación del Random Forest no supervisado para la construcción del Índice de Esfuerzo Ciudadano (IEC), se planteó inicialmente trabajar con **7 variables**, incorporando `visitas_totales` y `Visitas RUTS` a las 5 variables originales del modelo (`Tiempo_en_minutos`, `CONTEO_NETO`, `N_FORMATOS_FINAL`, `nivel_digitalizacion`, `TraCosto`).

## Hipótesis de inclusión

Se argumentó que ambas variables podrían funcionar como **proxies de complejidad percibida**:

- **`visitas_totales`**: alta demanda podría reflejar que el trámite no es sencillo de resolver en un solo intento.
- **`Visitas RUTS`**: si los ciudadanos buscan información del trámite en el portal oficial antes de realizarlo, podría indicar que el trámite genera dudas o requiere preparación previa, lo cual es una forma de esfuerzo cognitivo.

## Procedimiento

1. Se normalizaron las 7 variables con `MinMaxScaler` → rango [0, 1].
2. Se generaron 665 trámites sintéticos mediante muestreo uniforme dentro del rango observado de cada variable.
3. Se entrenó un `RandomForestClassifier` con 500 árboles (`random_state=42`) para distinguir trámites reales de sintéticos. Accuracy = 1.0.
4. Se extrajo la matriz de proximidad (665×665): proximidad promedio = 0.71.
5. Se aplicó clustering jerárquico aglomerativo (método Ward) sobre la matriz de distancias `1 - proximidad`.
6. Se cortó el dendrograma en 3 clusters.

## Resultados

| Cluster | n trámites | Tiempo (med.) | Digitalización (med.) | Visitas totales (med.) | Visitas RUTS (med.) |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | 44 | 2,160 min | 1 (presencial) | **6,262** | **20,812** |
| 2 | 100 | 7,200 min | **9 (semdigital)** | 609 | 404 |
| 3 | 521 | 4,320 min | 1 (presencial) | 767 | 593 |

## Análisis

La separación entre clusters estuvo determinada principalmente por `visitas_totales` y `Visitas RUTS`, y no por las variables que caracterizan el esfuerzo ciudadano directo:

- **Cluster 1** agrupa trámites de alta demanda y visibilidad (visitas muy elevadas), no necesariamente los más complejos.
- **Cluster 2** agrupa trámites con mayor nivel de digitalización (Niveles 3.7–3.9), separados del resto por esta variable y no por la carga burocrática.
- **Cluster 3** concentra el 78% de los trámites (521 de 665), evidenciando que el modelo no logra discriminar de manera útil la mayor parte del catálogo.

## Decisión

Se decide **eliminar `visitas_totales` y `Visitas RUTS`** del modelo. Ambas variables miden **demanda o popularidad del trámite**, no el esfuerzo que el ciudadano invierte en realizarlo. Su inclusión distorsiona la separación en clusters y desvía el índice de su objetivo original.

El modelo final trabajará con las **5 variables originales**:

| Variable | Dimensión de esfuerzo |
|---|---|
| `Tiempo_en_minutos` | Esfuerzo temporal |
| `CONTEO_NETO` | Carga documental |
| `N_FORMATOS_FINAL` | Carga de formularios |
| `nivel_digitalizacion` | Fricción digital |
| `TraCosto` | Esfuerzo económico |


