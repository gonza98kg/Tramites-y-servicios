[[💻 COEMERE]] [[Modelado de un Índice de Esfuerzo Ciudadano]]n
<h4 style="color:#0E2A47; font-family:'Montserrat', sans-serif; font-size:22px; font-weight:600; margin-top:15px; margin-bottom:8px;">

Estimación del esfuerzo en trámites y servicios estatales.

</h4>
Se establece una metodología cuantitativa para evaluar la eficiencia administrativa mediante la integración y depuración de registros gubernamentales utilizando técnicas de fuzzy matching. A través de un análisis estadístico avanzado —empleando ECDF, KDE y Boxplots—, se determinaron umbrales objetivos para normalizar la heterogeneidad de los datos, permitiendo la creación de subíndices robustos de tiempo, requisitos, formatos y costos. Como resultado, se consolidó el Índice de Esfuerzo Ciudadano, una métrica compuesta que jerarquiza la carga burocrática desde la perspectiva del usuario, ponderando variables críticas como tiempo (35%), digitalización (20%), requisitos (20%), formatos (15%) y costo (10%). Finalmente, la implementación del algoritmo de agrupamiento K-Means permitió segmentar los trámites en cuatro perfiles operativos: Burocracia Pesada, Recaudación Eficiente, Deficiencia Digital y Servicios Gratuitos Funcionales, proporcionando una hoja de ruta estratégica para focalizar los esfuerzos de simplificación y transformación digital donde mayor es su impacto.


<h4 style="color:#1F4E6B; font-family:'Montserrat', sans-serif; font-size:20px; font-weight:600; margin-top:15px; margin-bottom:8px;">
Normalización Multidimensional de Trámites
</h4>

<h4 style="color:#9FB8C9; font-family:'Montserrat', sans-serif; font-size:16px; font-weight:600; margin-top:15px; margin-bottom:8px;">
Sub-índice de Tiempo
</h4>

Los tiempos registrados en los trámites se encontraban en unidades diferentes, tales como minutos, horas, días y meses. Para que el análisis fuera matemáticamente coherente, se transformaron todos los datos a la unidad base de minutos. Debido al rango extremo de la variable del tiempo, se utilizó una escala logarítmica para representar la ECDF de esta variable, permitiendo un mejor manejo en la información. Se aplicó un KDE para estimar la función densidad de probabilidad de dicha variable, encontrando una distribución bimodal, donde la población A fue catalogada de primera instancia como trámites rápidos y la población B como trámites lentos. Se utilizó la topología de la *pdf* para mediante el cálculo de máximos y mínimos se definieran los siguientes umbrales para categorizar a la población, así como los valores para su normalización

1. *Rápidos:* $0<t<295\; \rm{min.}$ Un aproximado de entre 0 a 4.9 hrs. Puntaje de 0 a 25

2. *Lentos:* $295<t<9357 \; \rm{min.}$ Un aproximado de entre 4.9 hrs a 6.5 días. Puntaje de 26 a 70

3. *Críticos:* $9357<t<238115\;\rm{min.}$ Un aproximado de entre 6.5 días a 165 días. Puntaje de 71 a 100

4. *Outliers:* $t>238115\;\rm{min.}$ Mayores a 165 días. (Correspondientes al 2.7% de la población). Puntaje de 100

Para asignar los valores de normalización se utiliza la siguiente función a tramos $T(t)$

  

$$

T(t)=

\begin{cases}

\frac{t}{295}\times 25&: 0 < t < 295 \\

\frac{t-295}{9357-295}\times(70-25)+25&: 295 < t < 9357\\

\frac{t-9357}{238115-9357}+(100-70)+70&: 9357 < t < 238115\\

100&: t> 238115

\end{cases}

$$

Esta ponderación se basa en intentar definir que tan rapido sube el puntaje por cada unidad de tiempo agregada, tomando como referencia la ecuación $y=mt+b$ donde la pendiente $m=\frac{1}{x_{\rm{max}}-x_{\rm{min}}}$

**¿Por qué no se usa un $z$-score?**

Ya que el $z$-score se define en función del promedio y la varianza tal que $z=\frac{t-\mu}{\sigma}$, debido a la presencia de la cola correspondiente al $2.7\%$ de trámites que duran más de 3.5 meses y de la naturaleza bimodal de la *pdf*, el promedio podría no proporcionar datos correctos y por consecuencia la desviación estandar. El $z$-score funciona muy bien en distribuciones normales, lo cual no es el caso del presente análisis.

Se empleó el siguiente código para leer los datos de tiempo de la base *Effor_index*, aplicar la lógica matemática descrita anteriormente y crear la columna $\texttt{I\_Tiempo}$ en la misma base de datos.

<h4 style="color:#9FB8C9; font-family:'Montserrat', sans-serif; font-size:16px; font-weight:600; margin-top:15px; margin-bottom:8px;">

Sub-índice de Requisitos

</h4>

Dado la limpieza previa para distinguir entre requisitos, condicionantes, ruido y documentos de acreditación, se ajustó el umbral de requisitos mediante el análisis de boxplots, separando las siguientes poblaciones.

  

1. 3 requisitos (Bajo)

2. 6 requisitos (Medio)

3. 11 requisitos o más (Alto).

  

Donde se propone puntar de la siguiente manera.

  

$$

  

R=\rm{min}\left(100, \frac{r}{11}\times 100\right)

  

$$

  

En resumen, se toma el número de trámites y coloca un tope en 11, es decir, la mayor puntuación será de 100 a partir de 11 requisitos en adelante, cualquier número menor se escalará sobre 11. Cada requisito agregado aporta un valor de $\frac{1}{11}\approx9.09$. Por ejemplo

  

1. 3 requisitos $\approx$ 27 puntos.

2. 6 requisitos $\approx$ 54 puntos

3. 11 requisitos o más $\approx$ 100 puntos.

  

Se extablece la lógica matemática en este código y se crea la columna $\texttt{I\_Requisitos}$

<h4 style="color:#9FB8C9; font-family:'Montserrat', sans-serif; font-size:16px; font-weight:600; margin-top:15px; margin-bottom:8px;">

Sub-índice de Formatos

</h4>

Para el caso de los formatos se aplicó una metodología similar empleando boxplots para estimar el umbral, este se encontró alrededor de 2.5 trámites, ya que se está trabajando con números enteros se redondea a 3 trámites. Si se tienen 3 trámites o más, se penaliza de una forma mayor. Por lo que también se aborda una función a trozos.

  

$$

F=

\begin{cases}

f\times10&: f\leq 3\\

30+(f-3)\times20&: f>3

\end{cases}

$$

  

Así de esta forma

1. $f=1\Rightarrow$ 10 puntos

2. $f=3\Rightarrow$ 30 puntos

3. $f=4\Rightarrow$ 50 puntos

4. $f=6\Rightarrow$ 90 puntos

  

Se crea la columna $\texttt{I\_Formatos}$

<h4 style="color:#9FB8C9; font-family:'Montserrat', sans-serif; font-size:16px; font-weight:600; margin-top:15px; margin-bottom:8px;">
Sub-índice de Digitalización
</h4>

Para este caso, se cuenta, de manera previa, con una escala de tipo granular con 14 niveles que van de 1, 2, 3.1, 3.2, 3.3,..., 4.3 según las acciones que puedas hacer o no de manera digital, ver [Tab.1](#tabla_niveles), por lo que intentar cuantificar el esfuerzo para este caso se aborda de manera diferente. Se propone definir 4 variables binarias derivadas de situaciones que pueda vivir el ciudadano y que cuantifiquen el esfuerzo.

<a id="tabla_niveles"></a>
**Tabla 1:** Niveles de Digitalización

| NIVEL   | DESCRIPCIÓN                                                                                                                                          |
| :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| **4.3** | Resolución en línea inmediata.                                                                                                                       |
| **4.2** | Firma electrónica para solicitudes y resoluciones del trámite o servicio.                                                                            |
| **4.1** | Emitir resoluciones oficiales en línea.                                                                                                              |
| **3.9** | Llenar formatos en línea, en su caso.                                                                                                                |
| **3.8** | Agendar citas en línea.                                                                                                                              |
| **3.7** | Pago de derechos en línea.                                                                                                                           |
| **3.6** | Que el trámite o servicio presente intercambio de información con otras dependencias de manera electrónica.                                          |
| **3.5** | Que el trámite o servicio pueda mostrar a los ciudadanos el estatus en el que se encuentra (atendido/en revisión/rechazado) por medios electrónicos. |
| **3.4** | Notificación electrónica de vencimiento de plazo de respuesta.                                                                                       |
| **3.3** | Notificación electrónica de plazos de prevención.                                                                                                    |
| **3.2** | Notificación electrónica de información faltante.                                                                                                    |
| **3.1** | Que el trámite o servicio pueda recibir solicitudes por medios electrónicos con los correspondientes acuses de recepción de datos y documentos.      |
| **2**   | Posibilidad de descargar formatos, en su caso.                                                                                                       |
| **1**   | Información del trámite o servicio público a través de medios electrónicos (Inscrito en el Registro de Trámites y Servicios).                        |

1. V = ¿El ciudadano debe trasladarse físicamente aunque tenga cita o haya pagado? 0/1      : 40%
2. A = ¿El ciudadano deber asistir a la oficina a conseguir formatos o requisitos=? 0/1     : 35%
3. I = ¿El ciudadano carece de medidas digitales para saber el estatus de sus trámites? 0/1 : 15%
4. O = ¿El ciudadano carece de herramientas de gestión de archivos? 0/1                     : 10%

De esta forma y con los niveles proporcionados se tiene.

<a id="tabla_calculo"></a>
**Tabla de Cálculo de Puntajes**

| Grupo         | Componentes   | Cálculo           | Puntaje |
| :------------ | :------------ | :---------------- | :------ |
| **1.0**       | V + O + I + A | 40 + 35 + 15 + 10 | **100** |
| **2.0**       | V + O + I     | 40 + 35 + 15      | **90**  |
| **3.1 - 3.6** | V + O         | 40 + 35           | **75**  |
| **3.7 - 3.9** | V             | 40                | **40**  |
| **4.1 - 4.3** | —             | —                 | **0**   |

Así, mediante esta lógica se crea la columna $\texttt{I\_Digital}$

<h4 style="color:#9FB8C9; font-family:'Montserrat', sans-serif; font-size:16px; font-weight:600; margin-top:15px; margin-bottom:8px;">
Sub-índice de Digitalización
</h4>

Como no se puede diferenciar entre un trámite barato o caro puesto que solo se cuenta con información sobre el trámite/servicio se cobra o no (por ahora), se toma una penalización por existencia y se le asignará un peso bajo.

$$
C=
\begin{cases}
0&:\rm{FALSO}\\
1&:\rm{VERDADERO}
\end{cases}
$$
<h4 style="color:#1F4E6B; font-family:'Montserrat', sans-serif; font-size:20px; font-weight:600; margin-top:15px; margin-bottom:8px;">
Índice de Esfuerzo ciudadano.
</h4>

Se proponen dos índices de esfuerzo ciudadano con las siguientes ponderaciones.

**IEC_A** = $(T\times.40)+ (R\times.25)+(F\times.10)+(D\times.20)+(C\times.05)$ 
1. Tiempo: 40%
2. Requisitos: 25%
3. Formatos: 10%
4. Digitalización: 20%
5. Costo: 05%

**IEC_B** = $(T\times.35)+ (R\times.20)+(F\times.15)+(D\times.20)+(C\times.10)$ 
1. Tiempo: 35%
2. Requisitos: 20%
3. Formatos: 15%
4. Digitalización: 20%
5. Costo: 10%

Estas operaciones matemáticas se realizan en el presente código; se crean además las columnas $\texttt{IEC\_1}$ y $\texttt{IEC\_2}$ con la información adquirida. Es de mencionarse que algunos trámites y servicios contienen vacías alguna o algunas de las variables tomadas calcular este índice; se propuso en un principio asumir el peor escenario/puntuación posible para cada subíndice, sin embargo, se optó por no tomarlos en cuenta, por lo que el número de trámites analizados se redujo.

Comparación entre índices

![[Pasted image 20260423100810.png]]
Se observa que la Propuesta B tiende a correrse un poco más a la derecha de la propuesta A, lo que indica que la primera tiene una forma de más estricta de calificar o ponderar respecto a la segunda, es por ello que de ahora en adelante se trabajará con la propuesta B.

Los trámites del tipo "titulaciones" pertenecientes a la Secretaría de Educación Pública fueron catalogados con un índice alto de acuerdo a la forma de calificar del mismo. Es de recalcar que esta Secretaría abarca casi un 50% de los datos presentados en la base. 

Dentro de las principales observaciones en esta metodología resalta la falta del tratamiento, depuración e imputación de los datos, así como un análisis exploratorio de los mismos. 