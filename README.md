# Telecom X — Analisis de Evasion de Clientes (Churn)

## Descripcion General

Telecom X es una empresa de telecomunicaciones que enfrenta una tasa de cancelacion de clientes (churn) de aproximadamente el 26.5% sobre su base activa. Este nivel de abandono representa un riesgo operativo y financiero significativo, dado que el costo de retener a un cliente existente es sustancialmente menor que el de adquirir uno nuevo.

El presente proyecto aborda ese problema desde el rol de asistente de analisis de datos. El objetivo es recopilar, procesar y analizar los datos disponibles para que el equipo de Data Science pueda desarrollar modelos predictivos sobre un dataset limpio, estandarizado y documentado. El trabajo cubre el ciclo completo de ETL (Extraccion, Transformacion y Carga) seguido de un Analisis Exploratorio de Datos (EDA) orientado a identificar los factores que explican el abandono.

---

## Tecnologias y Herramientas

| Componente | Detalle |
|---|---|
| Lenguaje | Python 3.10+ |
| Manipulacion de datos | Pandas, NumPy |
| Visualizacion | Matplotlib, Seaborn |
| Consumo de API | Requests |
| Entorno de ejecucion | Google Colab |
| Formato de entrega | Jupyter Notebook (.ipynb) |

---

## Estructura del Proyecto

```
telecom-x-churn/
├── telecomx_churn_final.ipynb   # Notebook principal con ETL, EDA e informe
├── datos_tratados.csv           # Dataset procesado exportado al final del pipeline
└── README.md                    # Este archivo
```

---

## Proceso de ETL

### Extraccion

Los datos se obtienen desde una API publica en formato JSON con estructura anidada. Se utiliza `requests.get` con manejo de excepciones de red y `pd.json_normalize` para aplanar la estructura en un DataFrame tabular.

```python
import requests
import pandas as pd

URL_RAW = 'https://raw.githubusercontent.com/...'

try:
    respuesta = requests.get(URL_RAW, timeout=10)
    respuesta.raise_for_status()
    df_raw = pd.json_normalize(json.loads(respuesta.text))
except requests.exceptions.RequestException as e:
    print(f"Error de red: {e}")
```

El dataset crudo contiene 7.267 filas y 21 columnas con informacion de clientes, servicios contratados, tipo de contrato, metodo de pago y facturacion.

### Transformacion

Las transformaciones aplicadas sobre el dataset crudo son las siguientes:

**Renombrado de columnas.** Todas las columnas fueron renombradas al castellano mediante un diccionario de mapeo explicito para mejorar la legibilidad del codigo y la trazabilidad del analisis.

**Limpieza de valores.** Se convirtieron strings vacios a `NaN` mediante expresion regular. La columna `Factura_Total` fue convertida a tipo numerico con `errors='coerce'` para capturar valores no parseables. Se eliminaron filas con nulos en las variables criticas `Cliente_Perdido` y `Adulto_Mayor`.

**Estandarizacion binaria.** Los valores categoricos `Yes/No` fueron convertidos a `1/0`. Los tipos de contrato y metodos de pago fueron traducidos al castellano para uniformidad del informe.

**Tratamiento de servicios adicionales.** Las columnas de servicios contenian los valores `Sin internet` y `Sin telefonia` como indicadores de ausencia de servicio. Ambos fueron reemplazados por `0` para mantener coherencia con la escala binaria.

**Eliminacion de redundancias.** Se elimino `Factura_Total` por su alta correlacion con `Factura_Mensual` (r > 0.82), que introduciria multicolinealidad en futuros modelos lineales. Se elimino `Cliente_ID` por ser un identificador sin valor predictivo.

**Metrica derivada.** Se creo la columna `Factura_Diaria = Factura_Mensual / 30` como indicador de gasto diario promedio, util para comparaciones entre segmentos con distinto nivel de facturacion.

### Resultado del pipeline

El DataFrame resultante contiene **7.043 filas y 16 columnas**, todas con tipos de datos correctos y sin valores faltantes en las variables criticas. Se exporta como `datos_tratados.csv` al final del notebook.

---

## Analisis Exploratorio de Datos (EDA)

El EDA se desarrolla enteramente sobre el DataFrame transformado sin modificarlo, trabajando con copias cuando es necesario. Los subapartados cubren:

**Distribucion de la variable objetivo.** Analisis del desbalance de clases (73.5% sin churn / 26.5% con churn) mediante graficos de barras y proporciones. Este desbalance es un input critico para el equipo de modelado.

**Matriz de correlacion.** Heatmap de variables numericas para identificar correlaciones relevantes y detectar redundancias antes del modelado.

**Distribuciones por grupo de churn.** Histogramas comparativos de `Meses_Cliente_Activo`, `Factura_Mensual` y `Factura_Diaria` entre clientes que abandonan y los que permanecen, acompanados de estadisticas descriptivas (media, mediana, desviacion estandar) por grupo.

**Tasa de churn por variable categorica.** Graficos de barras para ocho variables categoricas (tipo de contrato, metodo de pago, servicio de internet, genero, pareja, dependientes, adulto mayor, servicio telefonico) con linea de referencia del promedio global.

**Analisis de servicios adicionales.** Comparacion de la tasa de churn entre clientes con y sin cada servicio adicional contratado (seguridad online, respaldo, soporte tecnico, streaming).

**Relacion entre Factura_Diaria y churn.** Boxplot por grupo y analisis de tasa de churn por cuartil de facturacion diaria.

---

## Instalacion y Dependencias

### Opcion A: Google Colab (recomendado)

Abrir el archivo `telecomx_churn_final.ipynb` directamente en Google Colab. Las librerias principales (Pandas, NumPy, Matplotlib, Seaborn, Requests) ya estan disponibles en el entorno base de Colab sin instalacion adicional.

### Opcion B: Entorno local

Requiere Python 3.10 o superior. Instalar las dependencias con:

```bash
pip install pandas numpy matplotlib seaborn requests scipy
```

Luego abrir el notebook con:

```bash
jupyter notebook telecomx_churn_final.ipynb
```

### Nota sobre tiempos de carga

La primera celda de importaciones puede tardar varios segundos en ejecutarse. Esto es normal y se debe al tiempo de inicializacion de Matplotlib y Seaborn al arrancar una sesion nueva. Las celdas siguientes corren sin demora una vez que las librerias estan cargadas en memoria.

---

## Insights e Informe Final

Los hallazgos completos se encuentran documentados en el **Bloque 6** del notebook. A continuacion se presenta un resumen ejecutivo.

### Perfil del cliente en riesgo

El churn en Telecom X no es aleatorio. Esta concentrado en un perfil definido: cliente con contrato mensual, menos de 12 meses de antiguedad, factura elevada, ningun servicio adicional contratado y metodo de pago por cheque electronico.

### Principales factores de abandono

| Factor | Hallazgo |
|---|---|
| Tipo de contrato | Contratos mensuales: 43% churn. Bi-anuales: 3% churn. |
| Antiguedad | Clientes con menos de 12 meses concentran la mayor tasa de abandono. |
| Facturacion | Clientes con churn pagan en promedio $74/mes vs $61/mes los que permanecen. |
| Metodo de pago | Cheque electronico: 45% churn, mas del doble que debito automatico. |
| Servicios adicionales | La ausencia de servicios adicionales es un indicador de baja vinculacion. |
| Fibra optica | Tasa de churn superior al 40%, posiblemente por percepcion costo-calidad. |

### Recomendaciones estrategicas

- Implementar un programa de fidelizacion durante los primeros 12 meses de vida del cliente.
- Incentivar la migracion de contratos mensuales a anuales con condiciones atractivas.
- Promover la adopcion de debito automatico como metodo de pago principal.
- Investigar la satisfaccion del segmento de fibra optica con una encuesta especifica.
- Incluir un periodo de prueba gratuito de servicios adicionales en el proceso de alta de nuevos clientes.

---

## Posibles Problemas y Soluciones

### Error de conexion en la extraccion

El notebook implementa manejo de excepciones para los errores de red mas comunes (`ConnectionError`, `Timeout`, `HTTPError`). Si la extraccion falla, verificar la conectividad a internet y que la URL del repositorio siga activa.

### Columnas con tipos mixtos

La columna `Factura_Total` llega como string desde el JSON en algunos registros. El pipeline la convierte con `pd.to_numeric(..., errors='coerce')`, lo que genera `NaN` en los valores no numericos. Esos registros son minoria y se descartan en la etapa de limpieza sin impacto significativo en el dataset.

### Re-ejecucion de celdas fuera de orden

El pipeline esta disenado para ejecutarse de arriba hacia abajo en una sola pasada. Re-ejecutar celdas individuales fuera de orden puede generar errores si el DataFrame fue modificado por celdas posteriores. En ese caso, reiniciar el kernel y ejecutar todas las celdas desde el inicio (`Runtime > Run all` en Colab).

### Desbalance de clases

El dataset tiene un desbalance moderado (26.5% churn). Esto no afecta el EDA pero es un input critico para el equipo de modelado: se recomienda usar `class_weight='balanced'` en modelos lineales y metricas como Recall y AUC-ROC en lugar de Accuracy como criterio de evaluacion.

### Volumen de datos y memoria

El dataset procesado ocupa menos de 5 MB en memoria, por lo que no hay limitaciones de RAM en Colab ni en entornos locales estandar. Si en el futuro el dataset crece significativamente, se recomienda evaluar el uso de `dtype` explicitos en la lectura para reducir el consumo de memoria.

---

## Autor

Christian Quidel — Asistente de Analisis de Datos
