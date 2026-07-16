# Forecasting de Presión Máxima en Cavidad durante Moldeo por Inyección

**Autores:** 
* Juan Miguel Martínez Muñoz
* [Nombre de tu compañero/a de grupo]

## 1. Descripción del problema
En la industria manufacturera de plásticos, las variaciones anómalas de presión durante el proceso de moldeo por inyección pueden generar defectos irreversibles en las piezas (como rebabas o *warpage*) y comprometer la integridad estructural de las matrices metálicas. Validar estas métricas mediante simuladores tradicionales basados en el Método de Elementos Finitos (FEM) resulta computacionalmente costoso y lento. 

Este proyecto propone implementar un modelo de Inteligencia Artificial como "modelo sustituto" (Surrogate Model) para realizar un *forecasting* temprano. El objetivo es predecir la presión máxima de la cavidad de la herramienta (`MEAS_KM_Max WkzDr1 [bar]`)[cite: 1] utilizando únicamente los datos secuenciales de los primeros segundos de inyección, permitiendo anticipar daños en el molde o defectos en el material antes de que el ciclo termine.

## 2. Dataset utilizado
Los datos provienen del **"Injection Molding Dataset"** publicado por el German Plastics Center (SKZ)[cite: 1]. 
* **Descripción:** Contiene registros de 68 experimentos de moldeo por inyección utilizando material ABS[cite: 1].
* **Archivos principales:** 
  * `viscometer_pressure_data.parquet`: Contiene la serie temporal de alta resolución (presión dependiente del tiempo medida en el viscosímetro)[cite: 1].
  * `quality_table_data.parquet`: Contiene los parámetros medidos por los sensores internos de la máquina al finalizar el ciclo[cite: 1].
* **Variables empleadas:** 
  * *Features (X):* Serie temporal de presión frontal y trasera (`MEAS_pressure_frontsensor_bar`, `MEAS_pressure_backsensor_bar`) a lo largo del tiempo (`MEAS_time_s`)[cite: 1].
  * *Target (Y):* Presión máxima de la cavidad de la herramienta (`MEAS_KM_Max WkzDr1 [bar]`)[cite: 1].
* **Unificación:** Las series temporales y los resultados finales se enlazan mediante los identificadores únicos `META_experiment` y `META_run`[cite: 1].

## 3. Metodología aplicada
El flujo de trabajo se dividió en tres fases principales:
1. **Preprocesamiento:** Carga de datos en formato `.parquet` para optimizar el uso de memoria. Se agruparon las trayectorias temporales por ciclo de inyección y se unificaron con la variable objetivo. Se aplicó truncamiento temporal (utilizando solo los primeros segmentos de la curva) y escalado de características (StandardScaler/MinMaxScaler) para facilitar la convergencia de las redes neuronales.
2. **Modelado:** Entrenamiento de modelos de predicción de series temporales.
3. **Evaluación:** Comparación del rendimiento de los modelos frente a datos no vistos (conjunto de test) utilizando métricas de error estándar.

## 4. Modelos implementados
Para este análisis, se implementaron modelos de dos categorías distintas según los requisitos del proyecto:
* **Modelo 1 (Machine Learning Clásico):** [Ej: Random Forest / XGBoost Regressor]. Se extrajeron características estadísticas de la serie temporal inicial para predecir el pico de presión.
* **Modelo 2 (Deep Learning):** [Ej: Long Short-Term Memory (LSTM) / Temporal Convolutional Network (TCN)]. Se introdujo la secuencia temporal pura para capturar dependencias a largo plazo en la curva de inyección.

## 5. Resultados y métricas
*A completar tras la ejecución de los notebooks.*

| Modelo | RMSE | MAE | MAPE |
| :--- | :--- | :--- | :--- |
| **[Modelo ML]** | [...] | [...] | [...] |
| **[Modelo DL]** | [...] | [...] | [...] |

## 6. Visualizaciones
*A completar con imágenes de la carpeta `/results`.*

1. **Serie temporal original:** Curvas de presión del viscosímetro de ciclos de inyección representativos.
2. **Predicciones vs Valores Reales:** Comparativa gráfica entre la presión máxima real y la pronosticada por los modelos.
3. **Análisis de Residuales:** Dispersión de los errores de predicción para evaluar sesgos en el modelo.

## 7. Conclusiones
* [A completar: ¿Qué modelo funcionó mejor para capturar la dinámica del plástico?]
* [A completar: ¿Es factible utilizar este modelo en la industria para prevenir daños en las matrices?]

***
