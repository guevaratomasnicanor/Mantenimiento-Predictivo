# ✈️ Turbofan Engine Degradation Predictive Maintenance (NASA CMAPSS)

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-Advanced-gradient.svg)](https://xgboost.readthedocs.io/)
[![Business-Impact](https://img.shields.io/badge/Focus-Operations%20Research%20%26%20ROI-green.svg)]()

Proyecto de **Machine Learning para Mantenimiento Predictivo** cuyo objetivo es predecir la **Vida Útil Restante (Remaining Useful Life - RUL)** de motores turbofán utilizando telemetría de sensores. Además de maximizar la precisión, el modelo optimiza las decisiones de mantenimiento minimizando el costo operativo y el riesgo financiero de la flota.

---

# 📊 Dataset

📦 **Fuente:** NASA Ames Prognostics Data Repository (**CMAPSS**).

El pipeline procesa automáticamente los cuatro subconjuntos del benchmark (**FD001–FD004**), contemplando distintos modos de falla y condiciones operativas.

**Variables principales**

| Variable | Descripción |
|-----------|-------------|
| `unit_nr` | Identificador del motor. |
| `time_cycles` | Ciclo de vuelo actual. |
| `s_x` | Sensores de temperatura, presión y velocidades internas del motor. |

---

# 🧹 Preprocesamiento y Feature Engineering

Se desarrolló un pipeline completamente automatizado que incluye:

- Eliminación dinámica de sensores sin información mediante un **filtro de varianza** (anti-leakage).
- **RUL Clipping** adaptativo según cada flota.
- Variables temporales para capturar la evolución del desgaste:
  - Rolling Mean
  - Slope (primera derivada)
  - Lag-1
  - Acceleration (segunda derivada)

---

# 🤖 Modelado

Se compararon tres estrategias utilizando **XGBoost Regressor**:

1. **Modelo Tradicional (MSE)** mediante `reg:squarederror`.
2. **Modelo Asimétrico**, penalizando cinco veces más el optimismo que el pesimismo.
3. **Modelo Final**, incorporando una optimización mediante Investigación Operativa que calcula automáticamente un colchón logístico para minimizar el costo económico de la flota.

La función de pérdida personalizada utilizada fue:

\[
Loss=
\begin{cases}
5(y_{pred}-y_{true})^2 & \text{si } y_{pred}>y_{true}\\
(y_{pred}-y_{true})^2 & \text{si } y_{pred}\le y_{true}
\end{cases}
\]

La validación se realizó mediante **Group Split**, evitando data leakage entre motores y preservando la estructura temporal.

---

# 🏆 Resultados

| Flota | Costo Tradicional | Costo Optimizado | Ahorro (USD) | Ahorro % |
|-------|------------------:|-----------------:|-------------:|---------:|
| FD001 | $3,377,978.56 | $2,057,971.83 | $1,320,006.73 | 39.08% |
| FD002 | $8,064,858.94 | $5,641,315.86 | $2,423,543.08 | 30.05% |
| FD003 | $3,405,388.32 | $2,281,832.25 | $1,123,556.07 | 32.99% |
| FD004 | $8,177,969.69 | $5,858,326.88 | $2,319,642.81 | 28.36% |
| **TOTAL** | **$23,026,195.51** | **$15,839,446.82** | **$7,186,748.69** | **31.21%** |

### Principales resultados

- **USD 7.19 millones** de ahorro estimado respecto al enfoque tradicional.
- Reducción aproximada del **36%** en fallas críticas (AOG).
- El modelo prioriza minimizar el costo total del negocio, incluso cuando ello implica sacrificar parcialmente métricas tradicionales como el MAE.

---

# 💼 Business Impact

Se desarrolló un modelo de costos que transforma el problema de predicción en un problema de optimización económica.

Costos considerados:

- **Mantenimiento preventivo:** USD 10.000
- **Falla crítica (AOG):** USD 50.000
- **Lucro cesante:** USD 300 por ciclo

El sistema determina automáticamente el colchón logístico óptimo para equilibrar el riesgo de falla y el mantenimiento prematuro, reduciendo el costo total de la operación.

---

# 💡 Conclusión

Este proyecto demuestra que los modelos de mantenimiento predictivo pueden optimizarse utilizando métricas de negocio en lugar de depender exclusivamente de indicadores estadísticos tradicionales. Al combinar **Machine Learning**, **funciones de pérdida asimétricas** e **Investigación Operativa**, se obtuvo una reducción estimada del **31.2%** en los costos de mantenimiento de la flota.

---

# 🧰 Tecnologías

- Python
- pandas
- NumPy
- scikit-learn
- XGBoost
- Matplotlib
- Seaborn

---

# 📚 Técnicas Aplicadas

- Predictive Maintenance
- Remaining Useful Life (RUL)
- Feature Engineering para Series Temporales
- XGBoost Regressor
- Asymmetric Loss Function
- Group-Based Validation
- Early Stopping
- Investigación Operativa
- Simulación vectorizada con NumPy
- MLOps Ready
```
