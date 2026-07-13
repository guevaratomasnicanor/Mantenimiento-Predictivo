# ✈️ Turbofan Engine Degradation Predictive Maintenance (NASA CMAPSS)

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-Advanced-gradient.svg)](https://xgboost.readthedocs.io/)
[![Business-Impact](https://img.shields.io/badge/Focus-Operations%20Research%20%26%20ROI-green.svg)]()

El objetivo de este proyecto es **predecir la Vida Útil Restante (RUL - Remaining Useful Life)** de motores turbofán de aeronaves a partir de telemetría de sensores, optimizando las intervenciones de mantenimiento bajo una perspectiva de **costo mínimo para la flota** y mitigando el riesgo operativo y financiero.

---

# 📊 Dataset

📦 **Fuente:** [Turbofan Engine Degradation Simulation Data Set - NASA Ames Prognostics Data Repository (CMAPSS)](https://www.nasa.gov/content/prognostics-center-of-excellence-data-set-repository)

El pipeline fue diseñado de forma **modular** para procesar automáticamente los cuatro subconjuntos del benchmark:

- **FD001**
- **FD002**
- **FD003**
- **FD004**

Estos representan distintos escenarios operativos:

- Un único modo de falla.
- Múltiples modos de falla.
- Desde una única condición operacional hasta **6 condiciones de vuelo** (altitud, velocidad, carga, etc.).

## Variables principales

| Variable | Descripción |
|-----------|-------------|
| `unit_nr` | Identificador único del motor. |
| `time_cycles` | Ciclo de vuelo actual. |
| `s_x` | Sensores que registran temperatura, presión, velocidades de rotación y comportamiento interno del motor. |

---

# 🧹 Limpieza de datos y Feature Engineering

## 🗑️ Filtro de Varianza (Anti-Leakage)

Se automatizó la detección de sensores sin información útil utilizando únicamente el conjunto de entrenamiento:

```python std < 0.001 ```

De esta forma:

- En **FD001** y **FD003** elimina automáticamente hasta **6 sensores** prácticamente constantes.
- En **FD002** y **FD004** conserva los **21 sensores**, ya que las múltiples condiciones operativas generan variabilidad relevante.

---

## ⚙️ RUL Clipping Adaptativo

Los motores no comienzan a degradarse linealmente desde el primer ciclo.

Para evitar introducir un target artificial se utilizó un clipping distinto para cada flota:

```python
RUL_CLIP_DICT = {
    "FD001": 125,
    "FD002": 135,
    "FD003": 125,
    "FD004": 140
}
```

---

## 📈 Variables Temporales

Se incorporaron variables derivadas para capturar la evolución del desgaste:

- **Rolling Mean (5 ciclos)** para reducir ruido de alta frecuencia.
- **Primera derivada (Slope)** para estimar la velocidad de degradación.
- **Lag-1** para incorporar el estado mecánico inmediatamente anterior.
- **Segunda derivada (Acceleration)** para detectar cambios bruscos antes de la falla.

Variables generadas:

- `sensor_roll_mean`
- `sensor_slope`
- `sensor_lag_1`
- `sensor_acceleration`

---

# 🔍 Comportamiento del Error y Trade-offs

En mantenimiento aeronáutico los errores no tienen el mismo costo.

## Overestimation (Optimismo)

Predecir que un motor durará más de lo real puede producir:

- Falla en servicio.
- Aircraft On Ground (AOG).
- Interrupción operativa.

Costo estimado:

**USD 50.000**

---

## Underestimation (Pesimismo)

Predecir menos vida útil provoca:

- Mantenimiento anticipado.
- Desperdicio de ciclos útiles.
- Menor utilización de la aeronave.

Costo base:

- USD 10.000
- más USD 300 por ciclo desperdiciado.

El objetivo del proyecto consiste en encontrar automáticamente el punto donde ambos costos se equilibran.

---

# 🤖 Modelado Predictivo

Se entrenó un **XGBoost Regressor** utilizando tres estrategias.

## 1. Modelo Tradicional

Optimización clásica mediante:

```python objective="reg:squarederror" ``` : Minimiza únicamente el error cuadrático medio (MSE).

---

## 2. Modelo Asimétrico

Se implementó una **función de pérdida personalizada**, penalizando cinco veces más el optimismo que el pesimismo.

---

## 3. Investigación Operativa (Modelo Final)

Sobre la predicción obtenida se ejecutó una simulación vectorizada en NumPy para calcular automáticamente un **colchón logístico óptimo**, desplazando la predicción hacia un punto económicamente más conveniente.

---

# 🛡️ Blindaje Metodológico

Para evitar **data leakage** se utilizó validación mediante **Group Split**, separando motores completos.

De esta manera:

- ningún motor aparece simultáneamente en entrenamiento y validación;
- se preserva la estructura temporal;
- el Early Stopping se evalúa sobre motores completamente desconocidos.

---

# 🏆 Benchmark Financiero (Test Oficial NASA)

La validación final se realizó utilizando el conjunto de test oficial del benchmark NASA CMAPSS, comparando las predicciones con los archivos reales:

```
RUL_FD001.txt
RUL_FD002.txt
RUL_FD003.txt
RUL_FD004.txt
```

## Resultados

| Flota | Costo Tradicional | Costo Optimizado | Ahorro (USD) | Ahorro % | Reducción AOG | Empeoramiento MAE | Bias |
|-------|------------------:|-----------------:|-------------:|---------:|--------------:|------------------:|------:|
| FD001 | $3,377,978.56 | $2,057,971.83 | $1,320,006.73 | 39.08% | -40.0% | +44.6% | -18.74 |
| FD002 | $8,064,858.94 | $5,641,315.86 | $2,423,543.08 | 30.05% | -37.8% | +67.1% | -28.69 |
| FD003 | $3,405,388.32 | $2,281,832.25 | $1,123,556.07 | 32.99% | -35.0% | +27.3% | -19.27 |
| FD004 | $8,177,969.69 | $5,858,326.88 | $2,319,642.81 | 28.36% | -33.1% | +32.4% | -22.27 |
| **TOTAL** | **$23,026,195.51** | **$15,839,446.82** | **$7,186,748.69** | **31.21%** | **-36.1%** | — | — |

---

# 🔍 Insights

## La paradoja del MAE

En **FD002**, el MAE empeoró un **67.1%**.

Sin embargo:

- el sesgo operativo pasó a **−28.69 ciclos**;
- las fallas críticas disminuyeron un **37.8%**;
- el ahorro económico fue de **USD 2.42 millones**.

Esto demuestra que minimizar exclusivamente el MAE no necesariamente minimiza el costo del negocio.

---

## Impacto Global

Costo utilizando MSE clásico:

**USD 23.03 millones**

Costo utilizando optimización económica:

**USD 15.84 millones**

### Ahorro total

**USD 7.19 millones**

equivalente a una reducción del

**31.21%**

---

# 💼 Caso de Negocio

En mantenimiento aeronáutico el objetivo no consiste únicamente en mejorar métricas estadísticas, sino en minimizar el costo total de operación.

El modelo financiero desarrollado considera:

- costo de mantenimiento preventivo;
- costo de falla crítica (AOG);
- costo por pérdida de utilización de la aeronave.

Parámetros utilizados:

| Concepto | Valor |
|----------|-------:|
| Mantenimiento preventivo | USD 10.000 |
| Falla crítica (AOG) | USD 50.000 |
| Lucro cesante | USD 300 por ciclo |

La simulación encontró automáticamente colchones logísticos entre **17 y 27 ciclos**, dependiendo de la complejidad operativa de cada flota.

---

# 💡 Conclusión

Este proyecto demuestra que un modelo de Machine Learning aplicado a mantenimiento predictivo no debería optimizar únicamente métricas tradicionales como MAE o RMSE.

Incorporar funciones de pérdida asimétricas junto con un modelo explícito de costos permite trasladar el problema desde una optimización estadística hacia una optimización económica.

El resultado fue una reducción estimada de **USD 7.19 millones**, disminuyendo significativamente el riesgo de fallas críticas sin modificar la arquitectura base del modelo.

---

# 🧰 Tecnologías Utilizadas

## Lenguaje

- Python

## Bibliotecas

- pandas
- numpy
- scikit-learn
- xgboost
- matplotlib
- seaborn
- json

---

# 📚 Técnicas Aplicadas

- Piecewise Linear Target Transformation.
- Feature Engineering para series temporales.
- Rolling Statistics.
- Derivadas temporales de sensores.
- Group-Based Validation.
- Early Stopping.
- Asymmetric Loss Functions.
- XGBoost Regressor.
- Investigación Operativa aplicada al mantenimiento predictivo.
- Simulación vectorizada mediante NumPy.
- Exportación automática de modelos y metadatos para entornos MLOps.
