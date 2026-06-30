# 📊 Análisis de Precios WOM — Cyber Days 2026

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Google Colab](https://img.shields.io/badge/Google%20Colab-F9AB00?style=flat&logo=googlecolab&logoColor=black)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Status](https://img.shields.io/badge/Status-Completado-brightgreen)

> Proyecto de Business Analytics que analiza la estrategia de precios de WOM Chile durante el evento Cyber Days 2026. Se construyó un pipeline completo de extracción vía API, consolidación de períodos, ETL y visualización en Power BI.

---

## Pregunta de negocio

¿WOM realmente baja sus precios durante el Cyber Days? ¿Qué marcas benefician más al consumidor? ¿Cuánto duran los descuentos una vez terminado el evento?

---

## Pipeline del proyecto

```
API interna WOM
      │
      ▼
 Extraccion.ipynb               ← Extracción + limpieza (Google Colab)
 (5 ejecuciones, una        ← Una por período
  por período)
      │
      ▼ 5 archivos CSV
 Consolidacion.ipynb            ← Consolidación en un único dataset
      │
      ▼ wom_total_unido2.csv (602 registros)
 Power Query (Power BI)     ← ETL final: tipos, columnas calculadas
      │
      ▼
 Dashboard Power BI         ← 3 páginas de análisis
```

---

## Metodología de extracción

El script consume directamente la **API interna de WOM** (`store-srv.wom.cl/rest/V1/content/getGraphqlDataFromSkus`) mediante `requests`, sin parseo de HTML. La API requiere una lista explícita de SKUs que fueron **relevados manualmente** desde el catálogo web en cada fecha de extracción.

> Nota: equipos publicados por WOM con posterioridad a la primera extracción (13/05) no están incluidos en el dataset.

Por cada SKU, la API entrega los precios correspondientes a los distintos planes disponibles. WOM ofrece precios diferenciados según el tipo de contrato del cliente:

| Columna en dataset | Plan WOM | Descripción |
|---|---|---|
| `precio_normal` | Sin plan | **Precio base de referencia** sobre el que se aplican los descuentos |
| `precio_prepago` | Prepago | Precio para clientes sin contrato |
| `precio_portabilidad/renovacion` | Portabilidad · Renovación | Precio al portar número o renovar contrato — ambos planes presentaron precios idénticos en todos los períodos, por lo que se consolidaron en una sola columna |
| `precio_linea_nueva` | Línea nueva | Precio para contratación de línea nueva |
| `cuota_arriendo` | Arriendo | Cuota mensual en modalidad de arriendo |

---

## Períodos analizados

| # | Período | Fecha | Registros |
|---|---|---|---|
| 1 | Normal | 13/05/2026 | 122 |
| 2 | Cyber Days | 01/06/2026 | 120 |
| 3 | Cyber Extendido | 07/06/2026 | 120 |
| 4 | Post Cyber | 15/06/2026 | 120 |
| 5 | Normal 2 | 22/06/2026 | 120 |
| | **Total** | | **602** |

---

## Estructura del repositorio

```
wom-cyber-analysis/
│
├── scraping/
│   ├── Extraccion.ipynb              # Extracción vía API + limpieza por período
│   └── Consolidacion.ipynb           # Consolidación de los 5 CSV en un único dataset
│
├── data/
│   └── wom_total_unido2.csv      # Dataset consolidado final (602 registros)
│
├── dashboard/
│   └── WOM_Dashboard.pbix        # Archivo Power BI Desktop
│
└── README.md
```

---

## ETL en Power Query

Antes de construir el dashboard, se realizaron las siguientes transformaciones en Power Query:

**Corrección de tipos numéricos** — algunos precios exportados por el script terminaban en `,00` (ej. `16000,00`), lo que provocaba que Power BI los interpretara como `1.600.000`. Se corrigió el tipo de dato en la carga.

**Columnas calculadas añadidas:**

```


// Segmento de gama (basado en precio_portabilidad/renovacion)
Segmento =
  if [precio_portabilidad/renovacion] < 150000  then "Básico"
  else if [precio_portabilidad/renovacion] < 400000  then "Gama Media"
  else if [precio_portabilidad/renovacion] < 800000  then "Gama Alta"
  else if [precio_portabilidad/renovacion] >= 800000  "Premium"
  else "No hay datos"

// Orden de período: para eje X cronológico en gráficos
Orden_Segmento =
  if [Segmento] = "Básico" then 1
  else if [Segmento] = "Gama Media" then 2
  else if [Segmento] = "Gama Alta" then 3
  else if [Segmento] = "Premium" then 4
  else 5

// Período: mapeo de fecha a nombre legible
Periodo =
  if [fecha_extraccion] = "2026-05-13" then "Normal"
  else if [fecha_extraccion] = "2026-06-01" then "Cyber 1"
  else if [fecha_extraccion] = "2026-06-07" then "Cyber 2"
  else if [fecha_extraccion] = "2026-06-15" then "Post Cyber"
  else if [fecha_extraccion] = "2026-06-22" then "Normal 2"
  else "Error Fecha"

```

---

## Dataset consolidado

- **602 registros** — 5 períodos concatenados
- **122 SKUs únicos** por período
- **8 marcas**: Apple, Samsung, Xiaomi, Motorola, Honor, ZTE, Tecno, Infinix
- **5 tipos de precio** por producto

---

## Hallazgos principales

### 1. El 70% del catálogo no bajó de precio durante el Cyber Days
De 120 productos, **84 (70%) mantuvieron exactamente el mismo precio** de portabilidad/renovación durante el evento. Solo 36 equipos (30%) registraron una reducción real.

### 2. La variación del precio promedio del catálogo fue de solo −1.64%
El precio promedio pasó de **$481K a $474K**. La percepción de "gran Cyber" se construye sobre la brecha permanente entre precio base y precio de portabilidad, que existe los 365 días del año.

### 3. El descuento varía hasta 3× según la marca

| Marca | Descuento Cyber Days |
|---|---|
| Motorola | 49.3% |
| ZTE | 48.0% |
| Tecno | 46.1% |
| Infinix | 45.5% |
| Samsung | 39.6% |
| Honor | 37.5% |
| Xiaomi | 35.7% |
| **Apple** | **13.5%** |

Apple aplica el descuento mínimo del mercado, protegiendo su percepción de valor. Las marcas de gama baja descuentan hasta 3× más para competir por volumen.

### 4. Los precios se recuperaron en 8 días

| Estado post-Cyber | Equipos |
|---|---|
| Restaurados al precio original | 92 (77%) |
| Con descuento residual activo | 14 (11%) |
| Terminaron más caros que antes | 12 (10%) |
| Precio permanente más bajo | 2 (2%) |

### 5. Samsung genera el mayor ahorro absoluto con solo 8 SKUs
Con menos del 7% del catálogo, Samsung concentra los productos con mayor ahorro en CLP durante el evento.

---

## Dashboard Power BI — 3 páginas

**Resumen Ejecutivo** — Jerarquía de precios WOM (Base → Prepago → Línea Nueva → Portabilidad/Renovación), catálogo por marca, distribución por segmento de gama y equipos con descuento activo por período.

**Análisis Cyber Days** — Cuántos equipos bajaron realmente de precio, subsidio comercial por marca, descuento táctico en los 5 períodos, top 5 productos por ahorro absoluto y posicionamiento estratégico de marca (precio base vs. % descuento).

**Post Cyber** — Evolución de los 4 tipos de precio en los 5 períodos, clasificación del resultado post-evento por estado y tabla de trazabilidad producto × período con deltas ΔN→C1, ΔC1→C2, ΔN→PC.

---

## Cómo reproducir el análisis

**1. Extracción — Google Colab**

Abre `scraping/Extraccion.ipynb` en Google Colab y ejecuta todas las celdas. El script genera un CSV por ejecución. Repite el proceso en cada período que quieras analizar.

```python
# Las únicas variables a revisar entre períodos:
FECHA_HOY = date.today().isoformat()  # Se actualiza automáticamente
SKUS = "..."  # Actualizar si WOM incorpora nuevos modelos al catálogo
```

**2. Consolidación — Google Colab**

Abre `scraping/Consolidacion.ipynb`, actualiza los nombres de los archivos CSV a concatenar y ejecuta. El script une todos los períodos en `wom_total_unido2.csv`.

**3. Carga en Power BI**

Importa `wom_consolidado.csv` en Power BI Desktop vía Power Query. Aplica las transformaciones de tipos y columnas calculadas descritas en la sección ETL. Verifica que los precios no presenten el problema del `,00` antes de construir las medidas DAX.

---

## Autor

**Bryan Alarcón**  
Ingeniero Comercial · Business Analytics

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/bryan-alarcon-sepulveda10)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/gdarx)
---

## Licencia

Este proyecto es de uso libre para fines educativos y de portafolio.  
Los datos fueron obtenidos desde la API de WOM Chile únicamente con fines analíticos y académicos.
