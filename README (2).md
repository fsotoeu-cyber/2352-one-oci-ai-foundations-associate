# Sistema Analítico Financiero CNBS

**Asistente conversacional para el análisis de indicadores financieros del sistema hondureño**, basado en datos oficiales de la Comisión Nacional de Bancos y Seguros (CNBS).

Arquitectura híbrida: **Pandas** realiza los cálculos determinísticos y **Groq (Llama 3.3)** genera explicaciones en lenguaje natural a partir de resultados ya calculados y sujetos a validación.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.28+-FF4B4B?logo=streamlit&logoColor=white)](https://streamlit.io/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0+-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Groq](https://img.shields.io/badge/Groq-Llama%203.3-00A67E)](https://groq.com/)
[![Plotly](https://img.shields.io/badge/Plotly-5.18+-3F4F75?logo=plotly&logoColor=white)](https://plotly.com/)
[![ReportLab](https://img.shields.io/badge/ReportLab-PDF-orange)](https://www.reportlab.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

<!-- Captura principal del dashboard (opcional)
![Dashboard del Sistema Analítico Financiero CNBS](docs/dashboard.png)
-->

---

## Aplicación en producción

La aplicación puede desplegarse en **Streamlit Community Cloud**.

> Actualiza esta sección con tu URL real tras el deploy:
>
> **URL pública:** `https://<tu-app>.streamlit.app/`

Nombre visible del producto: **Sistema Analítico Financiero CNBS**.

---

## Estado actual del proyecto

| Métrica | Valor |
|---|---|
| Fuente | Comisión Nacional de Bancos y Seguros (CNBS) |
| Motor de cálculo | **Pandas** (determinístico) |
| Redacción | **Groq · Llama 3.3 70B** (solo lenguaje natural) |
| Interfaz | **Streamlit** |
| Exportaciones | **PDF · PNG · Excel · CSV** |
| Observabilidad | **LangSmith** (opcional) |

> Ajusta la fila de instituciones, indicadores y fecha del dataset para que coincidan exactamente con tu `indicadores_financieros_CNBS.csv` antes de publicar.

El universo predeterminado para las consultas financieras son los **bancos comerciales**. El usuario puede solicitar de forma explícita otros grupos institucionales cuando el dataset los incluya.

---

## Objetivo

Demostrar una arquitectura práctica para consultar y analizar indicadores financieros oficiales mediante lenguaje natural, manteniendo una separación clara entre:

1. **Interpretación de la consulta**
2. **Cálculo determinístico**
3. **Generación de lenguaje**
4. **Validación de la respuesta**

Principio central:

> **El LLM explica los resultados; Pandas calcula los resultados.**

Esto reduce el riesgo de que un modelo de lenguaje invente cifras, altere rankings o sustituya un cálculo financiero determinístico por una estimación.

---

## Arquitectura híbrida

El sistema **no** delega los cálculos financieros al LLM.

```text
                    ┌───────────────────────────┐
                    │       Streamlit UI        │
                    │ Asistente · Tendencias    │
                    │          · Datos          │
                    └─────────────┬─────────────┘
                                  │
                                  ▼
                       Consulta en lenguaje natural
                                  │
                                  ▼
                       Detección / planificación
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
          Indicadores          Bancos /           Universo
          y sinónimos           años             institucional
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ▼
                         Pandas / DataFrame
                         cálculo determinístico
                                  │
                     ┌────────────┴────────────┐
                     ▼                         ▼
              Respuesta directa          ¿Necesita
              tablas / rankings              LLM?
                                             │
                                             ▼
                                      Groq · Llama 3.3
                                      solo redacción
                                             │
                                             ▼
                                        Validador
                                  ganador · cifras · ranking
                                             │
                                             ▼
                                      Respuesta final
                                             │
                              ┌──────────────┼──────────────┐
                              ▼              ▼              ▼
                             UI             PDF         Exportaciones
```

### Principio de gobernanza

El LLM recibe un contexto estructurado previamente calculado por Pandas.

**No debe:**

- recalcular indicadores
- cambiar rankings
- cambiar el ganador determinado por Pandas
- inventar cifras
- sustituir un indicador por otro
- presentar como disponible un indicador que no existe en el dataset

Cuando una consulta puede resolverse solo con Pandas, **no se invoca** el LLM.

---

## Cómo se reducen las alucinaciones

### 1. Cálculo determinístico

Pandas realiza promedios, rankings, ratios, scores, comparaciones, filtros por institución y por año, y agregaciones.

### 2. Contexto estructurado

El resultado de Pandas se entrega al componente de generación como referencia inmutable (incluyendo ganador, ranking y métricas cuando aplican).

### 3. Redacción controlada

Groq / Llama 3.3 se utiliza únicamente cuando se necesita explicación en lenguaje natural.

### 4. Validación

La salida se comprueba frente al resultado determinístico (institución ganadora, ranking, cifras).

### 5. Corrección

Si la redacción no respeta el resultado de Pandas, el sistema puede activar **un reintento** de corrección.

```text
Pregunta
   │
   ▼
Planificación
   │
   ▼
Pandas ───────────────► Resultado determinístico
                           │
                           ├──► Respuesta directa
                           │
                           └──► Contexto para LLM
                                      │
                                      ▼
                                Llama 3.3
                                redacción
                                      │
                                      ▼
                                  Validador
                                      │
                         ┌────────────┴────────────┐
                         │                         │
                      Correcto                 Discrepancia
                         │                         │
                         ▼                         ▼
                    Respuesta                  Reintento (1×)
```

---

## Resolución de consultas

### Indicadores y sinónimos

El sistema asocia expresiones del usuario con el indicador correspondiente del dataset (por ejemplo ROA, ROE, morosidad, adecuación de capital, spread, liquidez, eficiencia).

Si un indicador no existe en el dataset, el sistema debe **informarlo**, no sustituirlo en silencio.

### Universos institucionales

Cuando el dataset lo permite, se contemplan grupos como:

- Bancos comerciales (universo por defecto)
- Bancos estatales
- Sociedades financieras

El filtro por tipo de institución se aplica **antes** del Top-N.

### Top-N

Se reconocen expresiones como *top 3*, *los 3*, *tres mejores*, etc.

Flujo:

```text
Pregunta → Identificar universo → Filtrar tipo → Calcular ranking → Aplicar Top-N
```

### Contexto de la consulta

Los parámetros **explícitos** de la consulta actual (año, banco, indicador, tipo de institución) tienen prioridad sobre el contexto de consultas anteriores, para evitar arrastrar temas residuales.

---

## Capacidades analíticas

El sistema permite consultar, entre otros:

- ROA y ROE
- Morosidad y cobertura
- Adecuación de capital
- Spread y liquidez
- Gastos de administración / ingresos totales (eficiencia)
- Rankings y comparaciones entre instituciones
- Evolución temporal
- Indicadores agregados del sistema
- Relación rentabilidad–riesgo (ROE / morosidad)
- Equilibrio rentabilidad–solvencia (ROE y capital)

### Scores calculados por Pandas

| Consulta típica | Criterio determinístico |
|---|---|
| Mejor relación rentabilidad–riesgo | Ratio `ROE / Morosidad` (mayor = mejor) |
| Equilibrio rentabilidad–solvencia | Score `ROE × Capital / 100` |
| Equilibrio triple (si la consulta lo pide) | Score `(ROE / Morosidad) × (Capital / 100)` |

Los scores los calcula **Pandas**, no el LLM.

---

## Ejemplos de consultas

```text
Ranking de morosidad en 2025

Top 3 bancos comerciales por adecuación de capital en 2026

Compara el ROA de BAC y Ficohsa en 2025

Compara la eficiencia (Gastos de Administración / Ingresos Totales)
de BAC y Ficohsa en 2025

Compara la evolución del ROA y ROE del sistema entre 2024 y 2025

Analiza el riesgo crediticio del sistema en 2025
(mora, cobertura y tarjetas)

¿Qué banco tiene mejor relación rentabilidad-riesgo en 2025?
Considera ROE y morosidad.

Compara AZTECA, BAC y FICOHSA en 2025 con ROA, ROE, morosidad
y adecuación de capital. ¿Qué banco presenta el perfil más equilibrado?
```

---

## Módulos de la aplicación

| Módulo | Descripción |
|---|---|
| **Asistente** | Consultas financieras en lenguaje natural |
| **Tendencias** | Series temporales, KPIs y análisis exploratorio |
| **Datos** | Explorador del dataset y filtros |
| **Exportaciones** | PDF del informe, PNG del gráfico, Excel y CSV de la vista filtrada |

### Exportaciones por pestaña

| Pestaña | Exporta | Contenido |
|---|---|---|
| Asistente | PDF | Consulta + respuesta + metadatos (no el dataset completo) |
| Tendencias | PNG / PDF del gráfico | Serie y KPIs del indicador filtrado |
| Datos | Excel / CSV | Solo la vista filtrada visible en pantalla |

---

## Trazabilidad y observabilidad

El proyecto puede integrar **LangSmith** para observar las ejecuciones del componente LLM:

- trazas de prompts y respuestas
- latencia y tokens
- reintentos del validador
- metadata (ganador, ranking, motor)

La observabilidad es **opcional**. El cálculo financiero no depende de LangSmith.

---

## Tecnologías

| Capa | Herramientas |
|---|---|
| Lenguaje | Python 3.10+ |
| Interfaz | Streamlit |
| Datos y cálculo | Pandas |
| Visualización | Plotly · Kaleido |
| LLM | Groq · Llama 3.3 70B |
| Orquestación LLM | LangChain |
| Informes | ReportLab |
| Excel | OpenPyXL |
| Observabilidad | LangSmith (opcional) |

---

## Estructura del repositorio

```text
agente-financiero-cnbs/
├── app.py
├── pdf_renderer.py
├── indicadores_financieros_CNBS.csv
├── requirements.txt
├── README.md
├── LICENSE
│
├── docs/                    # capturas opcionales
│   ├── dashboard.png
│   ├── asistente.png
│   ├── tendencias.png
│   ├── datos.png
│   └── informe_pdf.png
│
└── .streamlit/
    └── secrets.toml         # no versionar claves reales
```

> Ajusta esta estructura si tu repo incluye extractor CNBS, `Pipfile` u otros módulos.

---

## Instalación local

```bash
git clone https://github.com/<tu-usuario>/agente-financiero-cnbs.git
cd agente-financiero-cnbs

python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux / macOS
source .venv/bin/activate

pip install -r requirements.txt
streamlit run app.py
```

---

## Configuración de API

### Groq (requerido para redacción LLM)

En Streamlit Secrets o variable de entorno:

```toml
GROQ_API_KEY = "gsk_..."
```

```bash
export GROQ_API_KEY="gsk_..."
```

Las consultas resueltas solo con Pandas funcionan sin invocar al modelo; la API de Groq se usa cuando la ruta de redacción está activa.

### LangSmith (opcional)

```toml
LANGCHAIN_API_KEY = "lsv2_..."
LANGCHAIN_TRACING_V2 = "true"
LANGCHAIN_PROJECT = "Sistema-Analitico-Financiero-CNBS"
```

### Seguridad

**No** subas al repositorio:

- `.streamlit/secrets.toml` con claves reales
- `.env` con secretos
- API keys en el código fuente

---

## Deploy en Streamlit Community Cloud

| Archivo | Obligatorio |
|---|---|
| `app.py` | Sí |
| `pdf_renderer.py` | Sí (si usas informes PDF) |
| `indicadores_financieros_CNBS.csv` | Sí |
| `requirements.txt` | Sí |
| `README.md` | Recomendado |
| `docs/*.png` | Opcional |

1. Publica el repositorio en GitHub.
2. Conéctalo a [Streamlit Community Cloud](https://share.streamlit.io/).
3. Main file path: `app.py`
4. Configura el secret `GROQ_API_KEY`.
5. Despliega.

---

## requirements.txt

```text
streamlit>=1.28.0
pandas>=2.0.0
plotly>=5.18.0
langchain-groq>=0.2.0
langchain-core>=0.3.0
reportlab>=4.0.0
kaleido>=0.2.1
openpyxl>=3.1.0
langsmith>=0.1.0
```

---

## Rendimiento

Los tiempos dependen del entorno y de si interviene el LLM. Orden de magnitud observado en pruebas:

| Tipo de consulta | Orden de magnitud |
|---|---|
| Determinística (solo Pandas) | ~0.02–0.05 s |
| Con redacción LLM | ~1–2 s |

El objetivo es que **los cálculos financieros no dependan** de la latencia ni de la variabilidad del modelo de lenguaje.

---

## Limitaciones

- Analiza únicamente los indicadores disponibles en el dataset de la CNBS.
- No realiza predicciones ni proyecciones financieras.
- No sustituye análisis profesional ni dictámenes regulatorios.
- No constituye asesoría de inversión.
- Si un indicador no está en el dataset, el sistema debe informarlo.
- El dataset se centra en ratios e indicadores relativos; no debe asumirse cobertura de todos los montos absolutos posibles.
- Los resultados dependen de la calidad, cobertura y fecha de actualización del CSV.
- La extracción o actualización del dataset desde el portal CNBS, si existe, es un proceso **independiente y opcional**.

---

## Decisión arquitectónica

| Componente | Responsabilidad |
|---|---|
| **Pandas** | Filtrado, cálculo, rankings, ratios, scores y agregaciones |
| **Planificador** | Interpretación y enrutamiento de la consulta |
| **Groq / Llama 3.3** | Explicación y redacción |
| **Validador** | Coherencia frente al resultado determinístico |
| **Streamlit** | Interfaz y experiencia de usuario |
| **LangSmith** | Observabilidad opcional del LLM |
| **ReportLab / export** | Informes PDF y descargas |

Esta separación prioriza **reproducibilidad, trazabilidad y control del dato** sobre la generación libre de respuestas.

---

## Dataset

Datos publicados por la **Comisión Nacional de Bancos y Seguros (CNBS), Honduras**.

Organización típica del CSV:

- institución
- tipo de institución (si aplica)
- indicador
- fecha / periodo de reporte
- valor del indicador

Archivo principal:

```text
indicadores_financieros_CNBS.csv
```

> Indica en esta sección la fecha de corte real de tu archivo (por ejemplo, última `FechaReporte` del CSV).

---

## Alcance del proyecto

Este repositorio se presenta como:

- demostración técnica y de portafolio
- ejercicio de análisis financiero aplicado
- ejemplo de arquitectura conversacional donde el modelo **no** es responsable del cálculo financiero

Los datos pertenecen a la CNBS. **Este proyecto no es un producto oficial de la CNBS** ni sustituye análisis o dictámenes regulatorios.

---

## Roadmap (opcional)

Mejoras posibles sin cambiar el principio arquitectónico:

- [ ] Actualización asistida del CSV desde fuentes públicas CNBS
- [ ] Más scores y rankings compuestos documentados
- [ ] Centro de ayuda y glosario en la UI
- [ ] Batería de pruebas de regresión automatizada
- [ ] Enriquecimiento de metadata en LangSmith

---

## Licencia

Distribuido bajo **MIT License**. Ver archivo `LICENSE`.

Proyecto con fines educativos, demostración técnica y portafolio profesional.

Los datos pertenecen a la **Comisión Nacional de Bancos y Seguros (CNBS)**. Este proyecto no constituye un producto oficial de la CNBS.

---

## Créditos

**Desarrollado por:** Euraque Analytics  

**Datos:** Comisión Nacional de Bancos y Seguros (CNBS), Honduras  

**Stack principal:** Python · Streamlit · Pandas · Plotly · Groq · Llama 3.3 · LangChain · ReportLab · OpenPyXL · LangSmith  

**Producto:** Sistema Analítico Financiero CNBS  
