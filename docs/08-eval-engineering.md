# 08 · Eval Engineering

> Infraestructura de prueba, golden sets y juzgamiento automatizado para medir cuantitativamente lo que un sistema probabilístico produce.

[← Graph Engineering](07-graph-engineering.md) · [Índice](../README.md) · [Volver al inicio](../README.md)

---

## Definición

Eval Engineering es el mecanismo de medición que hace científicamente auditables a todas las demás disciplinas de este repositorio. Frente al comportamiento no determinista de los LLM, introduce tres componentes: **golden sets** (conjuntos de referencia fijos con respuesta esperada conocida), **separación generador/evaluador escéptico** (el agente que genera contenido nunca es el único que lo juzga, porque los modelos tienden a ser auto-complacientes con su propio trabajo) y **contratos de sprint** (acuerdo formal, antes de empezar, sobre cuál es la condición de éxito medible).

Un patrón específico de 2026 vale la pena nombrar aparte: **Harness-Bench** — bancos de prueba en aislamiento que mantienen fijas la tarea, el presupuesto y los evaluadores, mientras varían deliberadamente la configuración del arnés. Es el único método honesto para responder "¿esta mejora vino del modelo o vino del arnés?" — pregunta que, sin este diseño experimental, casi siempre se responde mal.

## Cuándo usarla / cuándo no

**Usar siempre**, en rigor — cualquier disciplina de este repo sin una capa de Eval Engineering correspondiente está operando a ciegas. La pregunta real no es "¿cuándo evaluar?" sino "¿con qué nivel de rigor?": un prototipo interno necesita un golden set mínimo; un sistema en producción con usuarios reales necesita LLM-as-judge con detección de sesgos, métricas por dimensión y bloqueo de regresiones en CI.

**El error más común no es no evaluar, es evaluar mal**: usar el mismo modelo generador como único juez, sin golden set, sin medir sesgos conocidos del juzgamiento automatizado.

## El Estándar

- [ ] **Golden set versionado y congelado por release**, no un puñado de ejemplos que cambian cada vez que alguien los toca.
- [ ] **Generador y evaluador son procesos separados**, idealmente de familias de modelos distintas — un mismo modelo evaluando su propio output infla resultados por sesgo de auto-favorecimiento (self-enhancement bias, medido en la literatura entre 5-7% de inflación sistemática).
- [ ] **Mitigación explícita de sesgos conocidos de LLM-as-judge:**
  - *Position bias* (hasta ~40% de inconsistencia en algunos modelos): evaluar en ambos órdenes (A,B) y (B,A), contar solo los veredictos consistentes.
  - *Verbosity bias* (~15% de inflación): escalas acotadas (1-4) y premiar concisión explícitamente en el rubric.
  - *Self-enhancement bias*: usar familias de modelo distintas para generar y para juzgar.
  - *Judge drift*: fijar versión del modelo juez; recalibrar cuando cambie.
- [ ] **Contrato de sprint escrito antes de empezar la tarea**, no negociado después de ver el resultado — condición de éxito medible acordada entre generador y evaluador (o entre equipo y stakeholder) de antemano.
- [ ] **Harness-Bench cuando la pregunta es sobre el arnés, no sobre el modelo**: tarea, presupuesto y evaluador fijos; solo varía la configuración del arnés bajo prueba.
- [ ] **Muestreo de tráfico de producción para evaluación continua**, no solo evaluación pre-release — 1-5% de tráfico real evaluado de forma continua detecta drift que el golden set estático nunca va a capturar.
- [ ] **Métricas por dimensión, no un score único agregado** — "calidad 7/10" no dice si falló por precisión factual, formato, tono o seguridad; separar las dimensiones desde el diseño del rubric.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | "Se ve bien" a ojo de quien lo construyó; sin golden set, sin métrica, sin evaluador separado. |
| **1** | Repetible | Golden set fijo revisado manualmente por un humano antes de cada release. |
| **2** | Definido | LLM-as-judge con evaluador separado del generador, corriendo sobre el golden set de forma automatizada. |
| **3** | Gestionado cuantitativamente | Mitigación activa de sesgos de juzgamiento (position/verbosity/self-enhancement), métricas por dimensión, bloqueo de CI ante regresión. |
| **4** | Optimización continua | Harness-Bench institucionalizado + evaluación continua sobre muestra de tráfico real + evaluadores destilados de bajo costo para escalar a 100% del tráfico sin pagar el costo completo de un LLM-judge grande en cada request. |

**KPI ancla:** tasa de acuerdo juez-humano (*judge-human agreement rate*) medida periódicamente sobre una muestra — es el número que valida si el evaluador automatizado sigue siendo un proxy confiable del juicio humano, o si empezó a derivar.

## Modos de Falla / Anti-patrones

- **Evaluación auto-complaciente**: el mismo modelo (o la misma familia) genera y evalúa, inflando sistemáticamente los scores sin que nadie lo note hasta que un usuario real se queja.
- **Golden set contaminado**: los casos de evaluación terminan filtrándose al set de ejemplos few-shot o de fine-tuning, invalidando silenciosamente la métrica.
- **Score único que esconde el problema real**: un promedio agregado de 8/10 puede esconder un 2/10 en seguridad y un 10/10 en todo lo demás.
- **Ausencia de evaluación continua**: el sistema pasó todos los tests pre-release pero nadie mide qué pasa con tráfico real seis semanas después, cuando el mundo (y las consultas de los usuarios) ya cambiaron.

## Recursos Curados

**Frameworks de evaluación**
- [DeepEval](https://deepeval.com/) — framework open-source de evaluación de LLM con soporte extenso de LLM-as-judge (incluyendo G-Eval).
- [RAGAS](https://github.com/explodinggym/ragas) — métricas específicas para pipelines RAG (ver [04 · RAG Engineering](04-rag-engineering.md)).
- [LangSmith](https://www.langchain.com/langsmith) — tracing, evaluación y debugging integrado al ecosistema LangChain.

**Benchmarks de juzgamiento automatizado**
- [MT-Bench / Chatbot Arena](https://lmsys.org/) — evaluación de preferencia humana a gran escala, referencia histórica del campo.
- RewardBench / RewardBench 2 — evaluación de modelos de recompensa y jueces LLM con tripletas prompt-chosen-rejected.
- JudgeBench — evalúa corrección objetiva de jueces LLM en conocimiento, razonamiento, matemática y código.

**Lectura de referencia**
- [LLM-as-a-Judge in 2026: Top Techniques and Best Practices — DeepEval](https://deepeval.com/blog/llm-as-a-judge)
- [LLM as a Judge: A 2026 Guide — Label Your Data](https://labelyourdata.com/articles/llm-as-a-judge) — tabla de sesgos conocidos con magnitudes medidas, usada como base del Estándar de esta página.

## Ver también

- [02 · Flow Engineering](02-flow-engineering.md) — el evaluador determinista que cada fase de un flujo necesita.
- [05 · Harness Engineering](05-harness-engineering.md) — la capa V (Verification & Evaluation) del arnés.
- [06 · Loop Engineering](06-loop-engineering.md) — por qué el checker debe ser un proceso separado del generador dentro del loop.
