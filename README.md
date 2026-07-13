# ✈️ Turbofan Engine Degradation Predictive Maintenance (NASA CMAPSS)

El objetivo de este proyecto es **predecir la Vida Útil Restante (RUL - Remaining Useful Life)** de motores turbofán de aeronaves en base a telemetría de sensores, optimizando las intervenciones operativas bajo una perspectiva de costo mínimo para la flota.

---

## 📊 Dataset

📦 **Fuente:** [Turbofan Engine Degradation Simulation Data Set - NASA Ames Prognostics Data Repository (CMAPSS)]  
El estudio se concentra en el sub-dataset **FD001**, el cual simula una condición operativa estándar a nivel del mar con un único modo de falla (degradación por fricción en el compresor de alta presión). Consta de **20.631 registros de entrenamiento** y **100 motores de prueba**.

**Estructura y Variables:**
* `unit_nr` → Identificador único del motor (Activo).
* `time_cycles` → Ciclo de vuelo actual en el que se toma la medición.
* `s_x` (Variables de Telemetría) → 14 sensores críticos seleccionados que capturan variaciones de temperatura, presión y velocidades de rotación (ej. `s_2`, `s_3`, `s_4`, `s_7`, `s_8`, `s_9`, `s_11`, `s_12`, `s_13`, `s_14`, `s_15`, `s_17`, `s_20`, `s_21`).

---

## 🧹 Limpieza de datos y Feature Engineering

* 🗑️ **Reducción de dimensionalidad:** Se eliminaron las variables de configuración (`setting_1, 2, 3`) y 7 sensores específicos (`s_1`, `s_5`, `s_6`, `s_10`, `s_16`, `s_18`, `s_19`) debido a que presentaban varianza nula o constante en el entorno FD001, actuando como ruido para el modelo.
* ⚙️ **RUL Clipping (Piecewise Linear Target):** Se aplicó un tope artificial al RUL objetivo a un máximo de **125 ciclos**. Los motores no se degradan linealmente desde el primer día; mantener un RUL real muy alto al inicio sesga el entrenamiento en la fase estable del activo.
* 📈 **Variables de Degradación Temporal:** * `sensor_roll_mean` → Medias móviles de 5 ciclos para suavizar el ruido de alta frecuencia de la telemetría.
  * `sensor_slope` → Pendiente de cambio (`diff`) calculada sobre la media móvil para capturar la aceleración de la degradación del activo.

---

## 🔍 Comportamiento del Error y Trade-offs

<img width="800" alt="Distribución del Error de Predicción" src="https://github.com/user-attachments/assets/tu-link-a-grafico-error-aqui" />

## 🔍 Insights Principales

* 📉 **El costo de la asimetría:** En mantenimiento aeronáutico, errar por optimismo (estimar más vida útil de la real) genera una rotura catastrófica. Errar por pesimismo genera un costo logístico menor por mantenimiento anticipado. El pipeline asume esta realidad penalizando los errores positivos **5 veces más** que los negativos.
* 🎯 **Curva de Compensación de Costos:** Mediante simulación operativa, se descubrió que restar un "colchón de seguridad" crudo a las predicciones del modelo reduce drásticamente el costo total de la flota.

<img width="800" alt="Curva de Compensación de Costos: Fiabilidad vs Logística" src="https://github.com/user-attachments/assets/tu-link-a-grafico-curva-costos-aqui" />

---

## 🤖 Modelado Predictivo

Se utilizó un algoritmo de ensamble basado en gradiente potenciado (`XGBoost Regressor`) parametrizado con una función de pérdida personalizada (**Asymmetric MSE Loss**) en su Gradiente y Hessiano.

🏆 **Estrategia Seleccionada:** `XGBoost con Colchón Óptimo (13 ciclos)`. Al desplazar la predicción original de forma pesimista pero controlada, el modelo absorbe la incertidumbre tecnológica y blinda la operación.

### 📊 Tabla de Performance (Evolución del Enfoque)

| Configuración del Modelo | MAE Técnico | Bias (Sesgo) Operacional | Fallas Catastróficas Evitadas | Costo Total de la Flota |
| :--- | :---: | :---: | :---: | :---: |
| XGBoost Asimétrico (Base) | **16.33 ciclos** | +6.14 ciclos *(Optimista / Peligroso)* | Temerario | N/D |
| **XGBoost Conservador + Colchón Óptimo** | 22.76 ciclos | **-19.60 ciclos *(Seguro / Conservador)*** | **90.0%** | **$1,867,866.71 USD** |

*Nota: Aunque el MAE técnico aumenta, el comportamiento logístico es infinitamente superior y financieramente viable.*

---

## 💼 Caso de Negocio (Business Impact)

La optimización basada en algoritmos tradicionales persigue métricas abstractas. Este proyecto introduce un **Modelo Financiero Vectorizado** que mapea el error del modelo con el impacto en dólares de la flota:

* **Estrategia:** Encontrar el punto exacto donde el costo de desperdiciar vida útil residual de manera prematura es menor que el riesgo de pagar por fallas críticas en vuelo (AOG - Aircraft on Ground).
* **Parámetros de Estrés:** Costo preventivo programado de \$10,000 USD vs. Costo catastrófico no planificado de \$50,000 USD más lucro cesante por ociosidad de la aeronave (\$300 USD/ciclo).
* **Resultado:** La búsqueda del colchón logístico ideal detectó de forma automatizada un punto de equilibrio a los **13 ciclos**, logrando un presupuesto mínimo optimizado para la flota de **\$1,867,866.71 USD**, mitigando el 90% de los incidentes críticos de la compañía.

> 💡 **Conclusión estratégica:** La Ingeniería de Variables y la customización de las funciones de pérdida no buscan un modelo matemáticamente perfecto, sino un modelo que hable el idioma financiero del negocio: mitigación de riesgo y optimización de capital en activos de alto valor.

---

## 🧰 Tecnologías utilizadas

* **Lenguaje:** Python
* **Bibliotecas:** `pandas`, `numpy`, `xgboost`, `matplotlib`, `seaborn`, `scikit-learn`
* **Técnicas aplicadas:**
  * Modelado de degradación segmentado (Piecewise Linear Target Transformation).
  * Custom Objective Functions para optimización de funciones no simétricas en frameworks de Gradient Boosting.
  * Análisis de Trade-off Logístico mediante Investigación Operativa y simulación matricial rápida en NumPy.
