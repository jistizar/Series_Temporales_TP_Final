# Forecasting Multi-Output de Parámetros de Máquina en Moldeo por Inyección

**Autor:** 
* Juan Miguel Martínez Muñoz

## 1. Descripción y Formulación Matemática del Problema
En la industria manufacturera de plásticos, monitorear la estabilidad del proceso de moldeo por inyección es crucial para evitar defectos irreversibles en las piezas y asegurar la integridad mecánica del equipo. Históricamente, la validación se realiza tras finalizar el ciclo o mediante simuladores por Elementos Finitos (FEM) de alto costo computacional. 

Desde la perspectiva del Machine Learning, este proyecto no se aborda como un *Forecasting* clásico (Sequence-to-Sequence), sino que se formula formalmente como una **Regresión Extrínseca de Series Temporales (Sequence-to-Scalar)** bajo un enfoque **Multi-Output**. En este paradigma, el modelo asimila una secuencia temporal multivariada (los primeros 3 segundos de la curva de presión) para extraer la dinámica y memoria del fluido. Posteriormente, proyecta ese contexto latente hacia un espacio tabular para predecir de forma anticipada 8 escalares estáticos que representan el estado final de la máquina (tiempos totales, presiones máximas y consumo energético), previniendo fallos antes de que el ciclo concluya.

## 2. Dataset utilizado
Los datos provienen del **"Injection Molding Dataset"** publicado por el German Plastics Center (SKZ). 
* **Descripción:** Registros de experimentos de moldeo por inyección utilizando material ABS.
* **Archivos principales:** 
  * `viscometer_pressure_data.parquet`: Serie temporal de alta resolución de las presiones dependientes del tiempo.
  * `quality_table_data.parquet`: Parámetros finales del ciclo medidos por los sensores internos de la máquina.
* **Variables empleadas:** 
  * **Features (X):** Series temporales de presión frontal (`MEAS_pressure_frontsensor_bar`), trasera (`MEAS_pressure_backsensor_bar`) y su diferencia (`MEAS_pressure_difference_bar`).
  * **Targets (Y - 8 variables):** Tiempo de Ciclo, Tiempo de Plastificación, Presión de Conmutación, Presión Máxima de Masa, Integral de Presión, Potencia Máxima, Contrapresión Máxima y Energía Consumida.

## 3. Metodología aplicada
El flujo de trabajo se estructuró de forma modular para garantizar la integridad científica y aislar el espacio de las secuencias temporales (X) del espacio tabular predictivo (Y), evitando el *data leakage*:

1. **Notebook 1 (Preprocesamiento):** Carga de datos `.parquet`, truncamiento temporal a los primeros 3 segundos (fase activa de inyección que concentra la varianza dinámica antes de la etapa asintótica de enfriamiento), y unificación de las trayectorias con los *targets* escalares finales.
2. **Notebook 2 (Modelado y Optimización):** 
   * **Machine Learning Clásico (Baseline Extrínseco):** Extracción de características estadísticas estáticas (media, desviación estándar, máximo, integral trapezoidal) sobre la ventana de 3 segundos para entrenar ensambles arbóreos (`MultiOutputRegressor` con XGBoost y Random Forest).
   * **Deep Learning Temporal:** Inyección directa de la matriz secuencial pura (3 canales x ~30,000 timesteps) en arquitecturas CNN-LSTM y FCN para extracción automática de características espaciotemporales, adaptando una capa *Fully Connected* en la salida para la regresión escalar.
3. **Notebook 3 (Evaluación):** Inferencia sobre un set de prueba asimétrico y generación de gráficos de dispersión (Scatter Plots) para medir la distancia entre la predicción escalar escalar y el *ground truth* del sensor físico.

## 4. Arquitecturas implementadas
Para este análisis, se contrastaron modelos basados en árboles y arquitecturas de aprendizaje profundo:
* **Machine Learning (MultiOutputRegressor):** Random Forest y XGBoost.
* **Deep Learning (Series Temporales puras):** CNN-LSTM (red híbrida convolucional-recurrente) y FCN (Fully Convolutional Network).

## 5. Resultados y Métricas Comparativas
A continuación se presenta el **coeficiente de determinación (R²)** obtenido por cada modelo en el conjunto de prueba para las 8 variables objetivo. 

| Variable Objetivo | Random Forest | XGBoost | CNN-LSTM | FCN |
| :--- | :---: | :---: | :---: | :---: |
| **Tiempo de Ciclo** | 0.9820 | **0.9994** | 0.9061 | 0.8374 |
| **Tiempo Plastificación** | 0.9803 | **0.9909** | 0.8904 | 0.9326 |
| **Presión Conmutación** | **0.9883** | 0.9727 | 0.9840 | 0.9072 |
| **Presión Máx Masa** | **0.9967** | 0.9586 | 0.8104 | 0.9861 |
| **Integral de Presión** | **0.9790** | 0.9736 | 0.9280 | 0.7929 |
| **Potencia Máxima** | 0.9851 | **0.9969** | 0.7634 | 0.7980 |
| **Contrapresión Máxima** | 0.3202 | 0.3582 | **0.5736** | 0.4572 |
| **Energía Consumida** | **0.5677** | 0.0861 | 0.5212 | 0.2292 |

> *Nota: Valores en negrita indican el modelo con mejor rendimiento para esa variable específica.*

## 6. Visualizaciones
*(Consultar la carpeta `/results` o el Notebook 03 para ver los gráficos interactivos)*

1. **Comparativa R² (Bar Plot):** Rendimiento global agrupado por variable y arquitectura, destacando la superioridad de los modelos basados en árboles.
2. **Dispersión (Real vs Predicho):** Gráficos de correlación que evalúan la alineación de las predicciones de los modelos frente a la línea de predicción perfecta para cada una de las 8 variables.

## 7. Conclusiones Principales
* **Superioridad del ML Clásico:** La extracción de características estadísticas (media, máximo, integral, etc.) combinada con XGBoost y Random Forest demostró ser altamente superior y computacionalmente más eficiente que introducir la secuencia pura a arquitecturas de Deep Learning para este problema específico.
* **Alta Predictibilidad:** Variables críticas como el Tiempo de Ciclo (MAPE 0.02% con XGBoost) y la Presión Máxima de Masa (R² 0.996 con RF) pueden ser pronosticadas casi a la perfección utilizando únicamente la lectura inicial de los viscosímetros.
* **Limitaciones de Datos:** Variables como la *Contrapresión Máxima* y la *Energía Consumida* presentaron puntuaciones R² bajas en todos los modelos. Esto indica que estas métricas dependen de dinámicas de la máquina (ej. temperatura de resistencias, fricción mecánica) que no están representadas exclusivamente en las curvas de presión de la boquilla.
