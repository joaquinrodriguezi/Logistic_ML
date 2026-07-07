Sistema de alertas de entregas y estimación de días de atraso 🚚📈

“La gracia de hacerlo con datos es que deja de ser una ‘sensación’ y pasa a ser una decisión con respaldo: una probabilidad clara, una regla de alerta entendible y un número de días estimados para dimensionar el problema.”  
Fuente: Informe final del proyecto de logística (archivo adjunto).

Tabla de contenidos
Resumen

Estructura del repositorio

Cómo reproducir el análisis

Outputs operativos y uso

Métricas clave y validación

Buenas prácticas aplicadas

Siguientes pasos sugeridos

Contacto y contribución

Resumen
Objetivo: anticipar si un envío llegará tarde (alerta binaria) y, en caso de atraso, estimar cuántos días se demorará. El proyecto transforma intuición en decisiones con respaldo: probabilidad interpretable, regla de alerta explicable y estimación de magnitud del atraso.
Dataset: DataCoSupplyChainDataset_clean.csv (versión limpia del DataCoSupplyChainDataset).

Estructura del repositorio
Código
.
├─ data/
│  ├─ DataCoSupplyChainDataset_clean.csv
│  ├─ baseline2_group_rates.csv
│  └─ baseline2_predictions_test.csv
├─ notebooks/
│  ├─ Logistica_EDA.ipynb
│  ├─ modelo_alertas.ipynb
│  ├─ validacion_modelo.ipynb
│  ├─ sistema_de_alertas.ipynb
│  └─ modeloB_mejorado.ipynb
├─ outputs/
│  ├─ alertas.csv
│  ├─ scoring_completo.csv
│  ├─ resumen_ejecutivo.txt
│  ├─ paso5_metrics_recomputed.csv
│  ├─ paso5_calibration_deciles_model.csv
│  ├─ modeloB_metricas.csv
│  └─ mejoras_modeloB_metricas.csv
└─ README.md
Archivos clave

Baseline: baseline2_group_rates.csv, baseline2_predictions_test.csv

Validación: paso5_metrics_recomputed.csv, paso5_calibration_deciles_model.csv

Operación: alertas.csv, scoring_completo.csv, resumen_ejecutivo.txt

Modelo B: modeloB_metricas.csv, mejoras_modeloB_metricas.csv

Cómo reproducir el análisis
Requisitos
Python 3.8+

Entorno virtual (venv / conda)

Dependencias principales: pandas, numpy, scikit-learn, xgboost o lightgbm, matplotlib, seaborn, joblib

Instalación rápida
bash
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
Flujo recomendado
EDA y limpieza: ejecutar notebooks/Logistica_EDA.ipynb.

Revisar baseline: abrir baseline2_group_rates.csv.

Modelo de alertas: ejecutar notebooks/modelo_alertas.ipynb.

Validación y calibración: ejecutar notebooks/validacion_modelo.ipynb.

Simulación operativa: ejecutar notebooks/sistema_de_alertas.ipynb → genera alertas.csv y scoring_completo.csv.

Estimación de días: ejecutar notebooks/modeloB_mejorado.ipynb para el enfoque en dos etapas.

Outputs operativos y uso
Alertas operativas: alertas.csv → casos de alto riesgo para acción inmediata (priorizar revisiones, avisos, cambios de envío).

Auditoría: scoring_completo.csv → scoring completo para análisis y trazabilidad.

Estimación de días:

delay_expected = probabilidad de atraso × magnitud estimada.

delay_operativo = magnitud solo si el riesgo supera un umbral; si no, 0.

Simulación de ejemplo: se evaluaron 34,553 envíos y se generaron 6,912 alertas usando una política Top 20%.

Métricas clave y validación
Baseline por segmento: regla explicable por Shipping_Mode × Order_Region.

Modelo 1 (alertas): comparación justa contra baseline; baseline PR-AUC ≈ 0.8368, modelo PR-AUC ≈ 0.9991 tras recalculo y calibración.

Modelo B (días de atraso):

Regresión directa: MAE ≈ 0.99, RMSE ≈ 1.27, R² ≈ 0.28.

Enfoque dos etapas (clasificación + regresión en positivos) tras corregir fugas: PR-AUC ≈ 0.8387; MAE_pos ≈ 0.4817; MAE global ≈ 0.695; R² ≈ 0.2715.

Mejora del MAE: ~0.99 → ~0.70 (expected) y ~0.64 (operativo).

Buenas prácticas aplicadas
Baseline fuerte: usar reglas simples y explicables como referencia antes de aplicar ML.

Control de fugas: detectar variables proxy del target (ej. processing_time) y evitar encoders ajustados con train+test juntos.

Calibración: validar que los scores sean interpretables como probabilidades (calibration por deciles).

Separación de outputs: archivo de alertas para operaciones y scoring completo para auditoría y mejora continua.

Reproducibilidad: notebooks documentados y CSVs con outputs intermedios.

Siguientes pasos sugeridos
Añadir badges reales (CI, notebooks ejecutables, tamaño del dataset).

Crear CONTRIBUTING.md con pautas para contribuir y CHANGELOG.md con hitos.

Implementar tests unitarios para transformaciones críticas.

Automatizar ejecución de notebooks clave en CI (por ejemplo, con nbconvert o papermill).

Desarrollar un dashboard (Streamlit / Dash) para visualizar alertas y métricas en tiempo real.
