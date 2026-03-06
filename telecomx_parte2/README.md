# Telecom X — Prediccion de Cancelacion de Clientes (Churn)

## Descripcion General

Telecom X es una empresa de telecomunicaciones que enfrenta una tasa de cancelacion de clientes (churn) de aproximadamente el 26.5% sobre su base activa. Este proyecto aborda ese problema en dos etapas complementarias:

- **Parte 1:** Pipeline ETL completo y Analisis Exploratorio de Datos (EDA) para limpiar, estandarizar y comprender los datos.
- **Parte 2:** Pipeline de modelado predictivo para identificar a los clientes con mayor probabilidad de cancelar sus servicios.

El objetivo final es proveer al equipo de negocio un modelo operativo que permita actuar de forma preventiva sobre los clientes en riesgo antes de que tomen la decision de cancelar.

---

## Estructura del Proyecto

```
telecomx_parte2/
├── telecomx_parte2.ipynb    # Preprocesamiento, modelado y evaluacion
├── datos_tratados.csv       # Dataset procesado generado en la Parte 1
└── README.md                # Este archivo
```

---

## Tecnologias y Herramientas

| Componente | Detalle |
|---|---|
| Lenguaje | Python 3.13.11 |
| Manipulacion de datos | Pandas, NumPy |
| Visualizacion | Matplotlib, Seaborn |
| Machine Learning | Scikit-learn |
| Consumo de API | Requests |
| Entorno de ejecucion | Google Colab / VS Code |
| Control de versiones | Git / GitHub |

---

### Preprocesamiento

Sobre el dataset limpio de la Parte 1 se aplican las siguientes transformaciones previas al modelado:

- Correccion de valores residuales en `Lineas_Multiples` y `Servicio_Internet` heredados del CSV de origen
- One-Hot Encoding (OHE) de variables categoricas: `Servicio_Internet`, `Tipo_Contrato` y `Metodo_Pago`
- Codificacion binaria de `Genero` (F=0, M=1)
- Split estratificado 80/20 para garantizar la misma proporcion de churn en train y test
- Estandarizacion con `StandardScaler` aplicada exclusivamente al modelo sensible a escala

### Modelos entrenados

Se entrenaron dos modelos de clasificacion con estrategias complementarias:

| Modelo | Requiere escalado | Estrategia de desbalance |
|---|---|---|
| Regresion Logistica | Si (StandardScaler) | class_weight='balanced' |
| Random Forest | No | class_weight='balanced' |

La metrica principal de evaluacion es el **Recall de la clase Churn**: detectar la mayor cantidad posible de clientes que se van tiene mayor valor de negocio que evitar falsas alarmas.

El desbalance de clases (73.5% / 26.5%) se maneja con `class_weight='balanced'` en ambos modelos. No se aplica SMOTE porque el desbalance no es severo.

### Evaluacion

Los modelos se evaluan con las siguientes metricas: Recall, Precision, F1-Score, AUC-ROC y Average Precision. Se generan matrices de confusion y curvas ROC para comparacion visual. El analisis de importancia de variables permite identificar los factores de mayor peso en la prediccion.

### Resultado

La **Regresion Logistica** es el modelo recomendado para una primera iteracion en produccion por tres razones: ofrece el mayor Recall entre los modelos evaluados, sus coeficientes son directamente interpretables por equipos no tecnicos, y su tiempo de inferencia es minimo para un pipeline de scoring mensual.

---

## Instalacion y Dependencias

### Google Colab (recomendado)

[Abrir](https://colab.research.google.com/drive/1sAOgswFitnHeIAFb5f9-uVgncMht4Fng?authuser=1) 
 directamente en Google Colab. Las librerias principales estan disponibles en el entorno base sin instalacion adicional.

### Entorno local

Requiere Python 3.10 o superior:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn requests scipy
```

Orden de ejecucion:

1. Ejecutar [telecomx_parte1.ipynb](https://github.com/reddjedet/Telecom-x_parte_1) . Esto genera `datos_tratados.csv`.
2. Colocar `datos_tratados.csv` en el mismo directorio que `telecomx_parte2.ipynb`.
3. Ejecutar `telecomx_parte2.ipynb` completo.

---

## Recomendaciones Estrategicas

**Intervencion en los primeros 12 meses.** Es el periodo de mayor riesgo. Se recomienda onboarding personalizado y contacto proactivo del equipo comercial en el mes 3 y mes 6.

**Migracion de contratos mensuales a anuales.** La diferencia de 40 puntos porcentuales en la tasa de churn entre ambos tipos justifica un descuento del 10-15% en el primer año de contrato anual.

**Migracion del metodo de pago.** Incentivar el uso de debito automatico sobre el cheque electronico mediante beneficios concretos como descuento en factura o puntos de fidelidad.

**Oferta de servicios adicionales en el alta.** Un periodo de prueba gratuito de seguridad online o soporte tecnico aumenta la vinculacion desde el inicio y reduce el riesgo de abandono temprano.

**Investigacion del segmento de fibra optica.** La tasa de churn superior al 40% en ese segmento justifica una encuesta NPS especifica para determinar si el problema es de precio, calidad o expectativas no cumplidas.

---

## Autor

Christian Quidel — https://www.linkedin.com/in/quidelchristian/