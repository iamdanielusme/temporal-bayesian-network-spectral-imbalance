# temporal-bayesian-network-spectral-imbalance
Temporal Bayesian Network for spectral imbalance detection in sound systems, using MLE and Monte Carlo methods for tracking under uncertainty.


# Red Bayesiana Temporal para la Detección de Desbalance Espectral

## Descripción

Este proyecto propone el desarrollo de una **Red Bayesiana Temporal (RBT)** para la detección de desbalance espectral en sistemas de sonido, considerando la incertidumbre presente en las mediciones de respuesta en frecuencia.

La propuesta busca modelar un escenario de calibración en el que un micrófono realiza mediciones sucesivas de un sistema de reproducción de audio. Debido al ruido, la posición del micrófono y las incertidumbres propias de la medición, el estado espectral real del sistema no puede observarse directamente.

Por esta razón, se plantea un modelo probabilístico en el que el estado espectral es una **variable latente** y las mediciones obtenidas mediante el micrófono constituyen las **observaciones** del sistema.

El proyecto integra:

- Redes Bayesianas Temporales.
- Modelos ocultos de Markov (HMM) como estructura equivalente.
- Máxima Verosimilitud (MLE) para estimación de parámetros.
- Métodos de Monte Carlo para inferencia bajo incertidumbre.
- Procesamiento de señales y análisis espectral.
- Aplicación potencial a sistemas de sonido en estudio y sonido en vivo.

---

## 1. Planteamiento del problema

En un sistema de sonido, la respuesta en frecuencia medida en un recinto depende tanto del sistema de reproducción como de las características acústicas del espacio.

Por lo tanto, una misma configuración puede presentar diferentes respuestas dependiendo del lugar y de las condiciones de medición.

Durante un proceso de calibración, se realizan mediciones sucesivas mediante un micrófono de referencia para determinar si el sistema requiere algún tipo de corrección mediante procesamiento digital de señales (DSP).

El problema principal es que el **estado espectral real del sistema no es observable directamente**.

Lo que se obtiene es una medición indirecta y ruidosa de la respuesta en frecuencia.

Se plantea entonces la siguiente estructura temporal:

\[
X_{t-1} \rightarrow X_t \rightarrow D_t
\]

donde:

- \(X_t\): estado espectral latente del sistema en el instante \(t\).
- \(D_t\): observación obtenida a partir de la medición espectral.
- \(t\): instante de medición.

El objetivo es estimar:

\[
P(X_t \mid D_{1:t})
\]

es decir, la probabilidad del estado espectral actual teniendo en cuenta todas las observaciones realizadas hasta el instante \(t\).

---

## 2. Estados del sistema

Inicialmente se consideran dos estados posibles:

\[
X_t \in
\{
\text{Adecuada},
\text{Desbalance}
\}
\]

### Adecuada

El sistema presenta una respuesta espectral suficientemente cercana a la respuesta objetivo.

### Desbalance

El sistema presenta una desviación espectral significativa respecto a la respuesta objetivo.

El estado \(X_t\) es una variable **latente**, por lo que no se observa directamente.

---

## 3. Variable de observación

La observación se construye a partir de la respuesta en frecuencia medida.

Se define el error espectral promedio como:

\[
E_t =
\frac{1}{N}
\sum_{k=1}^{N}
\left|
20\log_{10}|H_t(f_k)|
-
20\log_{10}|H_{\mathrm{obj}}(f_k)|
\right|
\]

donde:

- \(H_t(f_k)\): respuesta en frecuencia medida en el instante \(t\).
- \(H_{\mathrm{obj}}(f_k)\): respuesta en frecuencia objetivo.
- \(f_k\): frecuencias evaluadas.
- \(N\): número de frecuencias utilizadas.
- \(E_t\): desviación espectral promedio en dB.

A partir de un umbral \(\tau\), se obtiene una observación discreta:

\[
D_t =
\begin{cases}
\text{Baja}, & E_t \leq \tau \\
\text{Alta}, & E_t > \tau
\end{cases}
\]

Por lo tanto:

\[
D_t \in
\{
\text{Baja},
\text{Alta}
\}
\]

La observación es considerada ruidosa porque una misma condición física puede generar diferentes mediciones debido a las incertidumbres del proceso de medición.

---

## 4. Modelo probabilístico

El modelo se basa en tres hipótesis principales.

### H1. Markov de primer orden

El estado actual depende del pasado únicamente a través del estado anterior:

\[
P(X_t \mid X_{0:t-1},D_{1:t-1})
=
P(X_t\mid X_{t-1})
\]

### H2. Independencia de la observación

La observación actual depende únicamente del estado actual:

\[
P(D_t\mid X_{0:t},D_{1:t-1})
=
P(D_t\mid X_t)
\]

### H3. Estacionariedad

Las probabilidades de transición y observación se consideran constantes a lo largo del tiempo:

\[
P(X_t\mid X_{t-1})
\]

y

\[
P(D_t\mid X_t)
\]

son iguales para todos los instantes \(t\).

---

## 5. DAG

Para dos instantes consecutivos, el modelo puede representarse como:

```text
D(t-1)                  D(t)
   ↑                       ↑
   │                       │
 X(t-1) ────────────────> X(t)
