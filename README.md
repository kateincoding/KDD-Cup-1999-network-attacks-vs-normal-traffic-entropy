# Detección de anomalías en tráfico de red por entropía de Shannon

> **Resumen visual del proyecto:** [kateincoding.github.io/KDD-Cup-1999-network-attacks-vs-normal-traffic-entropy](https://kateincoding.github.io/KDD-Cup-1999-network-attacks-vs-normal-traffic-entropy/)

Análisis sobre el dataset **KDD Cup 1999** (494,021 conexiones de red) usando **entropía de Shannon** e **información mutua** para detectar tráfico malicioso. Modelo final de regresión logística binaria con **AUC 0.9921** y **98.80% de accuracy** en test.

**Autoras:** Katherine Soto y Paulina Peralta — Estadística aplicada, La Salle.

---

## Idea central

El tráfico malicioso colapsa o explota la diversidad estadística de los paquetes. Esa firma es medible con entropía **antes** de necesitar un modelo supervisado:

- **DoS flood** → H ≈ 0 (paquetes idénticos)
- **Probe / escaneo** → H máxima (variedad forzada)
- **Normal** → H intermedia estable
- **R2L / U2R** → H camuflada en el perfil base, las más difíciles de detectar

---

## Pipeline

1. **Carga y limpieza** del subset 10% del KDD99 (5 clases agrupadas: normal, dos, probe, r2l, u2r)
2. **EDA** por tipo de ataque: histogramas, boxplots, detección de outliers
3. **Entropía de Shannon** por feature y por clase → heatmap
4. **Información mutua** con 3 métricas: `infogain`, `gainratio`, `symuncert`
5. **Correlación** y filtrado por multicolinealidad
6. **Regresión logística** binaria (normal vs. ataque) con partición 80/20 estratificada
7. **Evaluación** con matriz de confusión, ROC y AUC

---

## Resultados (test, 98,803 conexiones)

| Métrica | Valor |
|:--------|------:|
| Accuracy | **98.80%** |
| Sensitivity | 98.73% |
| Specificity | 99.08% |
| Kappa | 0.9625 |
| AUC | **0.9921** |
| Gap train/test | 0.01% |

Variables finales del modelo: `serror_rate`, `dst_host_serror_rate`, `src_bytes`, `same_srv_rate`, `count`.

---

## Estructura

```
.
├── entropy_network_anomaly_detection.ipynb   # Notebook principal (R kernel)
├── entropy_network_anomaly_detection.pdf     # Versión PDF del análisis
├── docs/                                      # Sitio web del proyecto (GitHub Pages)
│   ├── index.html
│   └── img/                                   # Gráficas exportadas
└── README.md
```

---

## Reproducir

Requiere R + Jupyter kernel `IRkernel`.

```r
install.packages(c(
  "tidyverse", "ggplot2", "gridExtra", "GGally",
  "FSelectorRcpp", "corrplot", "caret", "ROCR", "pROC"
))
```

Dataset: KDD Cup 1999 10% subset — [Kaggle](https://www.kaggle.com/datasets/galaxyh/kdd-cup-1999-data).

---

## Limitaciones

- Clases U2R y R2L subrepresentadas (52 y 1,126 instancias) → métrica global dominada por DoS.
- Colinealidad fuerte entre `serror_rate` y `dst_host_serror_rate` (r ≈ 0.999).
- Regresión logística lineal en logit → R2L y U2R no son linealmente separables; Random Forest / XGBoost capturarían mejor las interacciones.

---

## Extensión natural

Detector de **entropía deslizante en tiempo real**: calcular H(X) sobre ventanas temporales y alertar cuando la entropía cae bajo el baseline de normalidad. No requiere etiquetas ni modelo entrenado por adelantado. Capa de alerta temprana auditable, útil antes de cualquier clasificador supervisado.

---

## Licencia

MIT — ver [LICENSE](LICENSE).
