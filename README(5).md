# 🦜 Asistente de Análisis de Datos con IA

**Asistente conversacional híbrido** para explorar, validar, limpiar, consultar y visualizar **datasets CSV tabulares genéricos**.

Combina un agente **ReAct** basado en LangChain y Groq con **cálculo determinista mediante Pandas**, **Quality Gate**, **Human-in-the-Loop (HITL)**, **ejecución restringida de código para gráficos** y **trazabilidad mediante LangSmith**.

> **Principio de diseño:** el LLM no tiene que hacerlo todo. Cada responsabilidad se delega al componente más adecuado.

---

[![Python](https://img.shields.io/badge/Python-3.13+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.28+-FF4B4B?logo=streamlit&logoColor=white)](https://streamlit.io/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0+-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Groq](https://img.shields.io/badge/Groq-openai/gpt--oss--120b-00A67E)](https://groq.com/)
[![LangSmith](https://img.shields.io/badge/LangSmith-Observability-green)](https://smith.langchain.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---


## 🚀 Aplicación desplegada

La aplicación está publicada en **Streamlit Community Cloud**.

🔗 **Demo:** `https://tu-app.streamlit.app/`

Durante el desarrollo y las pruebas se utiliza **Google Colab**. El túnel se utiliza únicamente para exponer temporalmente la aplicación durante la experimentación; no forma parte del deployment final.

---

## 🧠 ¿Qué es y qué no es?

| ✅ Sí | ❌ No |
|---|---|
| Sistema analítico conversacional híbrido | Agente autónomo con planificación libre |
| Enrutamiento de consultas mediante ReAct | LLM utilizado como fuente de verdad numérica |
| Cálculo determinista con Pandas | Cálculos numéricos delegados al modelo |
| Herramientas especializadas según la tarea | Un único flujo que hace todo mediante LLM |
| Quality Gate y supervisión HITL | Aceptación ciega de entradas o salidas |
| Validación AST y ejecución restringida | Ejecución arbitraria de código |
| Trazabilidad mediante LangSmith | Procesos sin observabilidad |

### En una frase

> **El LLM interpreta y enruta, Pandas calcula, los componentes especializados ejecutan y LangSmith permite rastrear lo ocurrido.**

---

## 🎯 Objetivo

El proyecto busca resolver un problema común: permitir que un usuario trabaje con un CSV mediante lenguaje natural sin convertir al LLM en responsable de los cálculos ni de todas las decisiones del sistema.

La arquitectura separa:

```text
Interpretación
      ↓
Enrutamiento
      ↓
Herramienta especializada
      ↓
Cálculo / ejecución controlada
      ↓
Resultado
      ↓
Presentación
      ↓
Trazabilidad
```

---

## ✨ Características principales

| Área | Implementación |
| :--- | :--- |
| **Quality Gate** | Score 0–100, problemas bloqueantes y advertencias |
| **HITL** | Estados explícitos para supervisión y manejo de excepciones |
| **Limpieza** | Duplicados, nan → NaN, coordenadas (0,0), fechas e imputación conservadora |
| **Auditoría** | Información general y estadísticas ejecutadas directamente |
| **Análisis** | Promedios, sumas, conteos, máximos, mínimos, correlaciones, outliers, filtros y agrupaciones |
| **Cálculo** | Pandas como fuente de verdad numérica |
| **Case‑insensitive** | Resolución de preguntas independientemente de mayúsculas/minúsculas |
| **Unidades** | No se inventan unidades cuando no existe evidencia |
| **Gráficos** | LLM → código Python → AST → ejecución restringida |
| **Soft‑fail** | Errores de operación tratados sin derribar la sesión |
| **Reintentos** | Hasta 3 intentos, con backoff ante rate limits |
| **Observabilidad** | LangSmith obligatorio para trazabilidad |
| **Exportación** | CSV limpio, bitácora, reportes y gráficos en ZIP |

---

## 🛡️ Quality Gate

Antes de procesar el dataset se realiza una validación de calidad.

Se consideran, entre otros:

- filas y columnas mínimas;
- porcentaje global de nulos;
- duplicados;
- columnas completamente vacías;
- columnas con alta proporción de nulos.

El resultado produce un score de 0 a 100 y distingue entre:

```text
Dataset bloqueante
      ↓
STOP
      ↓
HITL
      ↓
Usuario corrige o sustituye el archivo
```

y:

```text
Advertencias
      ↓
La aplicación continúa
      ↓
HITL
      ↓
El usuario decide
```

Después de la limpieza se ejecuta nuevamente el Quality Gate para obtener un score post‑limpieza.

---

## 👤 Human‑in‑the‑Loop

La aplicación utiliza estados explícitos:

```text
READY
QUALITY_WARNING
QUALITY_BLOCKED
EXECUTING
RETRYING
FAILED
SUCCESS
```

El principio es:

> La máquina ejecuta y avisa; el humano decide cuando corresponde.

El HITL no pretende convertir cada operación en una aprobación manual. Se utiliza principalmente para gestionar bloqueos, advertencias y fallos donde continuar automáticamente no es apropiado.

---

## 🔢 Pandas como fuente de verdad

El LLM no realiza los cálculos numéricos finales.

El flujo de análisis es:

```text
Pregunta del usuario
        ↓
LLM interpreta intención
        ↓
Plan estructurado
        ↓
Pandas ejecuta
        ↓
Resultado determinista
        ↓
LLM presenta el resultado
```

Ejemplos de operaciones:

- promedio;
- suma;
- conteo;
- máximo;
- mínimo;
- correlación;
- detección de outliers;
- descripción estadística;
- filtros;
- agrupaciones.

Esta separación reduce el riesgo de que una respuesta lingüística sustituya al cálculo real.

---

## 🔐 Ejecución restringida de gráficos

Los gráficos utilizan un flujo diferente porque el LLM genera código Python.

```text
Solicitud del usuario
        ↓
LLM genera código
        ↓
Limpieza de salida
        ↓
Validación AST
        ↓
Bloqueo de módulos / funciones peligrosas
        ↓
Built‑ins mínimos
        ↓
UN solo exec restringido
        ↓
Figura
        ↓
PNG
```

El sistema bloquea, entre otros, usos asociados con:

- `eval`
- `exec`
- `compile`
- `open`
- `__import__`
- `os`
- `sys`
- `subprocess`
- `socket`
- `requests`
- `urllib`

### Nota de seguridad

La implementación debe describirse como:

> Validación AST + ejecución restringida con built‑ins mínimos.

No se presenta como un sandbox de aislamiento absoluto. La validación reduce la superficie de riesgo, pero no sustituye un aislamiento de proceso o contenedor cuando ese nivel de seguridad sea un requisito.

---

## 📊 Auditoría directa

Las funciones de auditoría se ejecutan directamente y no dependen del agente ReAct.

### Información general

Incluye:

- tipos de datos;
- valores nulos;
- duplicados;
- completitud;
- estado de cada variable.

### Estadísticas

Incluye:

- mínimo;
- media;
- mediana;
- máximo;
- desviación estándar;
- outliers mediante IQR.

Esto reduce la dependencia del LLM en operaciones básicas de diagnóstico.

---

## 🤖 Enrutamiento del agente

El agente ReAct dispone de herramientas especializadas:

- `Información DF`
- `Resumen Estadístico`
- `Analizar Datos`
- `Generar Gráfico`
- `Limpiar Datos`

Ejemplo de política de routing:

```text
información / nulos / duplicados
        → Información DF

estadísticas globales
        → Resumen Estadístico

promedio / suma / conteo / correlación / outliers
        → Analizar Datos

gráfico / barras / heatmap / visualización
        → Generar Gráfico

limpieza
        → Limpiar Datos
```

La auditoría mediante botones de la interfaz sigue una ruta directa y, deliberadamente, no pasa por ReAct.

---

## 📏 Política de unidades

La aplicación evita asumir unidades que el dataset no especifica.

```text
Nombre de columna:
TIEMPO_ENTREGA
        ↓
sin unidad explícita
        ↓
no inventar "minutos"

Nombre de columna:
TIEMPO_ENTREGA_MIN
        ↓
evidencia suficiente
        ↓
puede utilizar "min"
```

También se respeta una unidad indicada explícitamente por el usuario.

El principio es:

> Sin evidencia, no se inventa la unidad.

---

## 🔁 Soft‑fail, reintentos y backoff

Los fallos parciales no deberían convertir una operación fallida en una sesión inutilizable.

El sistema utiliza:

```text
Intento 1
   ↓
fallo
   ↓
Intento 2
   ↓
fallo
   ↓
Intento 3
   ↓
FAILED
```

Los rate limits reciben backoff antes del siguiente intento.

Después de los reintentos agotados:

```text
FAILED
   ↓
La aplicación permanece disponible
   ↓
HITL
   ↓
Usuario decide el siguiente paso
```

---

## 🔎 Observabilidad con LangSmith

LangSmith es obligatorio para la aplicación.

La trazabilidad permite revisar:

- ejecución del AgentExecutor;
- herramienta seleccionada;
- llamadas al modelo;
- entradas y salidas;
- errores;
- reintentos;
- tiempos;
- metadatos de ejecución.

La interfaz mantiene una presentación limpia y el detalle técnico queda disponible en LangSmith.

> Principio: la UI muestra el resultado; LangSmith permite investigar cómo se obtuvo.

---

## 📦 Exportación

La aplicación permite descargar:

- CSV limpio
- Bitácora de limpieza
- Reporte de información
- Reporte estadístico
- Gráficos

como archivos individuales o dentro de un ZIP.

---

## 🧩 Módulos de la aplicación

| Pestaña | Función |
| :--- | :--- |
| **📁 Datos** | Exploración, limpieza, validación post‑limpieza y bitácora |
| **🔍 Auditoría** | Integridad, estadísticas y outliers |
| **🔎 Análisis** | Consultas en lenguaje natural mediante ReAct |
| **📊 Gráficos** | Visualizaciones generadas a partir de lenguaje natural |
| **📚 Historial** | Preguntas y respuestas de la sesión |

---

## 🏗️ Arquitectura

```text
                         Usuario
                            │
                            ▼
                    ┌───────────────┐
                    │   Streamlit   │
                    │      UI       │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ Quality Gate  │
                    └───────┬───────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
            Bloqueante            OK / Warning
                 │                     │
                 ▼                     ▼
               STOP               Agente ReAct
                 │                     │
                HITL          ┌────────┼─────────┐
                               │        │         │
                               ▼        ▼         ▼
                           Analizar  Gráfico   Limpiar
                             Datos               Datos
                               │        │
                               ▼        ▼
                            Pandas   LLM → código
                            cálculo      │
                           determinista  AST
                                        │
                                  exec restringido

                               │
                               ▼
                           Resultado
                               │
                               ▼
                             HITL
                               │
                               ▼
                          LangSmith
```

---

## 🧱 Responsabilidades por componente

| Componente | Responsabilidad |
| :--- | :--- |
| **LLM / Groq** | Interpretación, routing, generación de código de gráficos y presentación |
| **ReAct** | Enrutamiento hacia la herramienta adecuada |
| **Pandas** | Cálculo determinista |
| **AST** | Validación del código generado |
| **Quality Gate** | Validación de entrada |
| **HITL** | Supervisión y decisión humana ante excepciones |
| **Streamlit** | Interfaz y experiencia de usuario |
| **LangSmith** | Observabilidad y trazabilidad |

---

## 💬 Ejemplo de conversación

**Usuario**

> Promedio de TIEMPO_ENTREGA_MIN

**Asistente**

> El promedio de TIEMPO_ENTREGA_MIN es 37.98 min.

---

**Usuario**

> Promedio de TIEMPO_ENTREGA

**Asistente**

> El promedio de TIEMPO_ENTREGA es 37.98. La unidad no está especificada en el nombre de la columna.

---

**Usuario**

> Genera un gráfico de barras del tiempo promedio por clima.

**Asistente**

> Se genera el gráfico solicitado y se ofrece la descarga en formato PNG.

---

**Usuario**

> Promedio de columna_que_no_existe

**Asistente**

> ⚠️ No se indicó una columna válida para el cálculo. Columnas disponibles: [...]

La aplicación continúa disponible.

---

## 🧪 Pruebas y validación

La batería completa está documentada en:

```text
docs/BATTERY_TEST.md
```

### Resultados actuales

| Métrica | Resultado |
| :--- | :--- |
| Batería funcional | **31 / 31 — 100 %** |
| Tool selection | **6 / 6** casos correctos en muestra manual de LangSmith |

La batería cubre:

- Quality Gate;
- HITL;
- limpieza;
- auditoría;
- cálculos;
- agrupaciones;
- filtros;
- correlación;
- manejo de columnas inexistentes;
- case‑insensitive;
- política de unidades;
- gráficos;
- exportación.

> **Importante:** el resultado 6/6 corresponde a una muestra manual de trazas, no a una accuracy global del agente.

---

## 🧪 Evaluation Suite — próxima evolución

La siguiente fase del proyecto es convertir las pruebas manuales en una evaluación reproducible.

La suite no dependerá exclusivamente de un único dataset.

Se utilizarán fixtures CSV genéricos:

```text
evals/
├── fixtures/
│   ├── limpio_min.csv
│   ├── mayusculas.csv
│   ├── con_nulos.csv
│   └── vacio_casi.csv
│
├── casos.json
└── run_eval.py
```

La evaluación separará:

```text
A. Tool selection
   → ¿eligió la herramienta correcta?

B. Cálculo
   → ¿coincide con el oráculo Pandas?

C. Error handling
   → ¿maneja correctamente entradas inválidas?

D. Quality Gate
   → ¿clasifica correctamente el dataset?
```

### Métricas previstas

```text
tool_selection_accuracy
calculation_accuracy
error_handling_rate
pass_rate
retry_rate
```

El objetivo es evitar valores hardcodeados de un único dataset y medir el comportamiento del sistema sobre distintos esquemas tabulares.

---

## 📸 Capturas

Pendiente incorporar capturas reales del proyecto.

Se propone:

```text
docs/images/
├── auditoria.png
├── analisis.png
├── graficos.png
└── historial.png
```

---

## ⚙️ Configuración

### Variables obligatorias

```bash
GROQ_API_KEY=tu_clave_groq
LANGCHAIN_API_KEY=tu_clave_langsmith
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=Asistente-Colab
```

También pueden configurarse mediante:

```text
.streamlit/secrets.toml
```

No subir este archivo al repositorio.

---

## 💻 Instalación local

```bash
git clone https://github.com/tu-usuario/tu-repo.git
cd tu-repo

python -m venv .venv
```

**Linux / macOS**

```bash
source .venv/bin/activate
```

**Windows**

```powershell
.venv\Scripts\activate
```

Instalar dependencias:

```bash
pip install -r requirements.txt
```

Ejecutar:

```bash
streamlit run app.py
```

---

## 🚀 Deployment en Streamlit Community Cloud

1. Subir el repositorio a GitHub.
2. Crear una nueva aplicación en Streamlit Community Cloud.
3. Seleccionar el repositorio.
4. Configurar:
   - **Main file:** `app.py`
5. Añadir los secrets:

```toml
GROQ_API_KEY = "gsk_..."
LANGCHAIN_API_KEY = "lsv2_..."
LANGCHAIN_TRACING_V2 = "true"
LANGCHAIN_PROJECT = "Asistente-Colab"
```

6. Desplegar.

Cada actualización enviada al repositorio puede activar una nueva versión de la aplicación.

---

## 📁 Estructura del repositorio

```text
/
├── app.py
├── herramientas.py
├── requirements.txt
├── README.md
├── .gitignore
│
├── docs/
│   └── BATTERY_TEST.md
│
├── data/
│   └── ejemplo_mini_mayus.csv
│
└── evals/
    ├── fixtures/
    ├── casos.json
    └── run_eval.py
```

La carpeta `evals/` representa la siguiente fase de desarrollo y puede completarse progresivamente.

---

## 🔒 Seguridad y privacidad

### Código generado

- Validación AST.
- Bloqueo de módulos y funciones peligrosas.
- Built‑ins mínimos.
- Un único exec restringido.

### Filtros

Las expresiones para `df.query()` también se validan mediante AST.

### Credenciales

Nunca subir:

```text
.env
.streamlit/secrets.toml
API keys
tokens
```

### Datos

Los datasets cargados se mantienen durante la sesión de Streamlit y no forman parte del repositorio.

---

## ⚠️ Limitaciones conocidas

1. El agente utiliza ReAct clásico, por lo que preguntas extremadamente vagas pueden producir un routing subóptimo.
2. La validación AST reduce la superficie de riesgo, pero no proporciona aislamiento de proceso.
3. Algunas etiquetas generadas en gráficos pueden ser más agresivas con las unidades que la respuesta textual.
4. La evaluación actual es principalmente manual.
5. La suite automática y su integración con CI forman parte de la siguiente evolución.
6. El sistema no debe interpretarse como un agente autónomo con planificación libre o memoria de trabajo a largo plazo.

Estas limitaciones forman parte de la definición real del sistema y se documentan deliberadamente.

---

## 📈 Aprendizajes clave

Este proyecto permitió consolidar varios principios de diseño de sistemas de IA.

### 1. Separación de responsabilidades

El LLM no tiene por qué resolver cada tarea.

```text
LLM
→ interpretar

Pandas
→ calcular

AST
→ validar

Herramientas
→ ejecutar

LangSmith
→ observar

Humano
→ decidir cuando corresponde
```

### 2. Gobernanza antes que autonomía

No todo lo que un modelo puede generar debe ejecutarse automáticamente.

### 3. El cálculo debe ser determinista cuando sea posible

Los modelos de lenguaje son útiles para interpretar lenguaje, pero las operaciones numéricas deben delegarse a componentes especializados.

### 4. La trazabilidad forma parte del diseño

Un sistema no solo debe responder; debe permitir investigar qué ocurrió cuando la respuesta o el proceso no fueron los esperados.

### 5. Los límites también son parte del producto

Definir explícitamente lo que el sistema puede hacer y lo que no puede hacer evita expectativas incorrectas y facilita su evolución.

---

## 📄 Licencia

MIT License — ver LICENSE.

---

## 🙏 Créditos

- **Stack:** Streamlit · Pandas · NumPy · LangChain · Groq · LangSmith · Matplotlib · Seaborn
- **Dataset de ejemplo:** datos de demostración
- **© 2026** — Asistente de Análisis de Datos con IA

---

## Estado del proyecto

```text
Estado actual: validación manual cerrada y aprobada.

31/31 pruebas funcionales
        +
6/6 tool selection en muestra manual
        ↓
Versión funcional estable
        ↓
Evaluation Suite automática
        ↓
Evolución futura hacia orquestación más estructurada
```
