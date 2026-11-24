# Detección de fraude con tarjetas de crédito · XGBoost

Proyecto de clasificación supervisada para una competencia de Kaggle del curso **Métodos Supervisados** (Ciencia de Datos). El objetivo es asignar a cada cliente una probabilidad de fraude y la métrica es **ROC-AUC**.

**Mejor resultado público en Kaggle: 0.95465 ROC-AUC** (XGBoost + bits e intervalos del ID, ensamble de 5 folds).

---

## Problema

- **Train:** 848.548 registros · **Test:** 200.027 registros
- **Objetivo:** `Fraude` (binaria). Prevalencia: **5,985 %**, así que las clases están desbalanceadas.
- **Variables:** número de tarjetas, saldo, total de transacciones, transacciones internacionales, estado, género y tipo de tarjeta. El detalle está en [`data/README.md`](data/README.md).

## Enfoque

1. **Análisis exploratorio:** faltantes, boxplots por categoría y matriz de correlaciones.
2. **Ingeniería de variables (fase de clase):** cuadrados, interacciones cuantitativas, razones, interacciones categóricas filtradas con Kruskal-Wallis e interacciones cuantitativa × categórica. Se seleccionaron por importancia de XGBoost usando solo la partición de entrenamiento, para no filtrar información de validación.
3. **Variables de comportamiento:** tasas de transacciones internacionales y nacionales, saldo por tarjeta, saldo por transacción, transformaciones `log1p` e indicadores (sin transacciones internacionales, saldo cero, etc.).
4. **Comparación de modelos** sobre la misma partición estratificada 80/20:

   | Modelo | AUC train | AUC validación |
   |---|---|---|
   | Ridge | 0.9169 | 0.9166 |
   | Elastic Net | 0.9169 | 0.9165 |
   | Lasso | 0.9169 | 0.9165 |
   | LightGBM | 0.9460 | 0.9338 |
   | CatBoost (early stopping) | 0.9397 | 0.9342 |
   | XGBoost base | 0.9389 | 0.9342 |
   | XGBoost ajustado | 0.9419 | 0.9343 |
   | **XGBoost ajustado + ID** | **0.9570** | **0.9512** |

5. **Representación del ID:** el identificador sí aportaba señal, así que se probaron distintas codificaciones:
   - los 20 bits del `ID`;
   - un intervalo categórico del `ID` (512 o 256 grupos), codificado con one-hot dentro de cada fold.
6. **Validación cruzada estratificada (3 y 5 folds):** se calculó el AUC fuera de fold (OOF) y las predicciones de test se promediaron entre los modelos de cada fold.

## Resultados

| Envío | AUC OOF local | Score público Kaggle |
|---|---|---|
| `submission_roc_auc_cv.csv`: XGBoost con ID, mezcla de 2 configuraciones, 3 folds | 0.950763 | 0.95068 |
| `submission_roc_auc_idbits.csv`: + 20 bits del ID, 3 folds | 0.954088 | 0.95417 |
| `submission_roc_auc_idbits_512bins.csv`: + intervalo de 512 IDs, 3 folds | 0.954621 | 0.95438 |
| **`submission_roc_auc_idbits_512bins_5fold.csv`: + intervalo de 512 IDs, 5 folds** | **0.955099** | **0.95465** |
| `submission_roc_auc_idbits_256bins_5fold.csv`: intervalo de 256 IDs, 5 folds | 0.955219 | menor que 0.95465 |

**Lecciones:**

- Pasar de modelos lineales regularizados a árboles de boosting subió el AUC de validación de 0.917 a 0.934. XGBoost, LightGBM y CatBoost quedaron prácticamente empatados sin el ID.
- La mayor ganancia vino del `ID`: su posición en la secuencia guarda relación con el fraude (+0.017 AUC).
- Con 256 intervalos el AUC OOF local fue un poco mejor, pero el score público fue menor. Una mejora local de ~0.0001 está dentro del ruido, así que se mantuvo la versión de 512 intervalos.

> **Nota para un uso real:** la señal del `ID` es un artefacto de cómo se construyó el conjunto de datos. Sirve para la competencia, pero no se generalizaría a transacciones nuevas en producción. Un modelo para producción tendría que basarse en las variables de comportamiento y elegir un umbral según el costo de las falsas alertas y del fraude no detectado.

## Estructura del repositorio

```
├── data/                  # train.csv, test.csv, sample_submission.csv + diccionario
├── notebooks/
│   └── deteccion_fraude_xgboost.ipynb   # notebook completo, ejecutado de principio a fin
├── submissions/
│   └── submission_roc_auc_idbits_512bins_5fold.csv   # mejor envío (0.95465)
├── presentacion/
│   └── Analisis_Deteccion_Fraude_Entrega.pptx        # presentación entregada
└── requirements.txt
```

## Cómo reproducirlo

```bash
git clone https://github.com/adrianfelipe812/deteccion-fraude-xgboost.git
cd deteccion-fraude-xgboost
python -m venv .venv
.venv\Scripts\activate          # Windows  (Linux/Mac: source .venv/bin/activate)
pip install -r requirements.txt
cd notebooks
jupyter notebook deteccion_fraude_xgboost.ipynb
```

El notebook lee los datos de `../data` y guarda los envíos en `../submissions`. Una ejecución completa tarda unos 25 minutos en un portátil, porque entrena varios ensambles sobre unas 850 mil filas. LightGBM y CatBoost son opcionales: si no están instalados, esas celdas se omiten.

La ejecución publicada reproduce exactamente los AUC originales, y el CSV de la mejor submission es idéntico byte a byte al que se envió a Kaggle.

## Tecnologías

Python · pandas · NumPy · SciPy · scikit-learn (Pipeline, ColumnTransformer, GridSearchCV, StratifiedKFold) · XGBoost · Matplotlib · Seaborn

## Equipo

Proyecto desarrollado en equipo para la competencia de Kaggle del curso de Métodos Supervisados:

- **Adrian Felipe Grateron**: [github.com/adrianfelipe812](https://github.com/adrianfelipe812)
- **Jhonatan Smith Doncel**
- **Jorge Emmanuel Marin**
