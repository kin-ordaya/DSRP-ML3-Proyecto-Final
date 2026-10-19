# Detección de fraude en transacciones (PaySim)

Proyecto final del MLE3: modelo supervisado para detectar fraude en el dataset PaySim, con validación cruzada estratificada, optimización bayesiana de hiperparámetros, interpretabilidad y monitoreo de drift. Los experimentos quedaron registrados en DagsHub.

## Dataset

**PaySim1**: https://www.kaggle.com/datasets/ealaxi/paysim1

- 6,362,620 transacciones, 11 columnas, ~470 MB.
- El CSV **no se sube al repo** (GitHub no acepta archivos de más de 100 MB): se descarga de Kaggle y se coloca en `Datos/` con el nombre original `PS_20174392719_1491204439457_log.csv`.
- Fraude global: 0.1291% (dataset muy desbalanceado).
- El corte es temporal, no aleatorio: `step <= 400` para entrenar (5,787,030 filas, 0.0774% fraude) y `step >= 401` para evaluar como si estuviera en producción (575,590 filas, 0.6491% fraude).

## Estructura

```
Notebooks/   4 notebooks en orden de ejecución
Datos/       CSV (no versionado) + parquet procesados (no versionados)
modelos/     modelo final, métricas y matriz de confusión
Reports/     reportes HTML de Evidently, PSI y métricas por ventana
docs/        estrategia de Git
```

## Cómo correrlo

Con [uv](https://docs.astral.sh/uv/):

```bash
uv sync
uv run jupyter lab
```

Y ejecutar los notebooks en orden:

1. `01_preprocesamiento.ipynb` — carga, chequeo de anomalías de PaySim, feature engineering y split por step.
2. `02_entrenamiento.ipynb` — baseline, Optuna y registro en DagsHub.
3. `03_interpretabilidad.ipynb` — SHAP, PDP y LIME.
4. `04_monitoreo.ipynb` — Evidently, PSI y métricas por ventana.

También se puede con pip: `pip install -r requirements.txt`.

Para registrar en DagsHub se necesitan dos variables de entorno (`DAGSHUB_USERNAME` y `DAGSHUB_TOKEN`); el token nunca se escribe dentro del repo.

## Resultados

Métricas con umbral 0.5 (CV media de 2 folds estratificados; offline = train completo; online = período de producción):

| Métrica | Baseline (CV) | Offline (train) | Online (producción) |
|---|---|---|---|
| Precision | 0.9905 | 0.9942 | 1.0000 |
| Recall | 0.9916 | 0.9931 | 0.9995 |
| F1 | 0.9911 | 0.9936 | 0.9997 |
| AUC-ROC | 0.9994 | 0.9998 | 1.0000 |
| AUC-PR | 0.9933 | 0.9958 | 0.9997 |
| Lift (top 5%) | 19.96 | 19.98 | 20.00 |

Optuna mejoró el F1 del baseline de 0.9911 a 0.9922 en CV. Mejores hiperparámetros: `max_depth=4`, `learning_rate=0.0187`, `n_estimators=145`, `min_child_weight=6`, `subsample=0.77`, `colsample_bytree=0.72`, `reg_lambda=0.28`.

12 features según el Anexo 1 del curso (se descartaron `isFlaggedFraud`, `nameOrig` y `nameDest`). Las más importantes: `errorBalanceOrig` y `ratio_amount` juntas concentran ~74% del gain.

## Experimentos en DagsHub

https://dagshub.com/kin-ordaya/DSRP-ML3-Proyecto-Final

8 runs en el experimento `fraude-paysim`: baseline, 6 trials de Optuna y el modelo final, cada uno con sus params, métricas y artefactos (modelo `modelo_xgb.json`, matriz de confusión en PNG y métricas en JSON).

## Monitoreo

- `Reports/evidently_data_drift.html` y `Reports/evidently_target_drift.html` — reportes de Evidently entre train y producción.
- `Reports/psi_por_variable.csv` — PSI calculado a mano: solo `step` pasa de 0.25 (11.51, esperado porque las ventanas de tiempo son disjuntas); el resto bajo 0.02.
- `Reports/metricas_por_dia.csv` — recall y precision por día en producción (recall mínimo 0.9958, precision 1.0 todos los días).
- `Reports/drift_resumen.json` — resumen de todo lo anterior.

## Estrategia de Git

Todo en `development` con commits por fase, un Pull Request final a `main` y release `v1.0.0`. Detalle en [`docs/estrategia-git.md`](docs/estrategia-git.md).
