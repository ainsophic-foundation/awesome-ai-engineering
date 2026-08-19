# Matriz de Madurez de Ingenierías de IA

[← Índice](../README.md)

Estándar transversal de medición para las ocho disciplinas de este repositorio. Adapta la lógica de un modelo de madurez tipo CMM (Capability Maturity Model) — cinco niveles, de Ad-hoc a Optimización continua — a cada disciplina específica, y define un **KPI ancla**: la métrica cuantitativa individual que mejor resume si esa disciplina está funcionando en producción.

No es una certificación externa ni un framework de cumplimiento. Es una vara propia para que un equipo pueda responder, con datos y no con intuición, dos preguntas: *¿en qué nivel estamos hoy, por disciplina?* y *¿qué es lo próximo que hay que construir para subir un nivel?*

## Cómo usar esta matriz

1. Autoevaluar cada disciplina de forma independiente — no asumir que un equipo maduro en RAG Engineering automáticamente lo es en Eval Engineering.
2. El nivel de una disciplina es el nivel más bajo que cumple **todos** sus criterios, no un promedio — si 4 de 5 criterios del Nivel 3 están cumplidos pero uno no, el nivel real es 2.
3. No hay obligación de llegar a Nivel 4 en todo — un prototipo interno de bajo riesgo puede vivir perfectamente en Nivel 1-2; el objetivo es que el nivel sea una decisión consciente, no un accidente.
4. Revisar la matriz por disciplina en cada release mayor, no solo una vez al arrancar el proyecto — la madurez se degrada si no se sostiene.

## La matriz completa

| Disciplina | Nivel 0 — Ad-hoc | Nivel 1 — Repetible | Nivel 2 — Definido | Nivel 3 — Gestionado cuantitativamente | Nivel 4 — Optimización continua | KPI ancla |
|---|---|---|---|---|---|---|
| **[01 · Prompt](../docs/01-prompt-engineering.md)** | Editado en playground, sin versión | Versionado en git, sin test de regresión | Template estándar + golden set mínimo | Golden set automatizado en CI antes de cada merge | Optimización sistemática (DSPy/MIPROv2) reemplaza tuneo manual | Tasa de adherencia al formato de salida contratado |
| **[02 · Flow](../docs/02-flow-engineering.md)** | Una sola llamada intenta todo | Fases fijas validadas manualmente | Contratos de fase + evaluador determinista automatizado | Pass@k medido por fase con umbral de bloqueo | Ruteo dinámico de complejidad entre fases según datos de producción | Pass@k del flujo completo vs. invocación directa |
| **[03 · Context](../docs/03-context-engineering.md)** | Contexto crece sin límite hasta fallar | Compactación manual reactiva | Compactación automática por umbral + offloading a filesystem | *Context rot* medido explícitamente, dispara compactación | Gestión delegada al agente (patrón Generator/Reflector/Curator) | Tokens efectivos por unidad de trabajo útil completada |
| **[04 · RAG](../docs/04-rag-engineering.md)** | Chunking fijo, top-k directo sin reranking | Pipeline reproducible sin métricas medidas | Recuperación híbrida + reranking + golden set mínimo | Métricas RAGAS en CI con umbral de bloqueo | GraphRAG o recuperación agéntica adaptativa | Faithfulness (RAGAS) sobre golden set |
| **[05 · Harness](../docs/05-harness-engineering.md)** | Ejecución directa contra el host, sin sandbox | Sandbox básico + logging manual | ≥4/7 capas ETCLOVG cubiertas explícitamente | 7/7 capas ETCLOVG; tasa de intervención humana medida | Harness-Bench interno para aislar el efecto de cada componente | Tasa de finalización de tareas sin intervención humana |
| **[06 · Loop](../docs/06-loop-engineering.md)** | Reintento manual del mismo prompt | Loop scripteado, sin detección de estancamiento | Estado persistido externo + detección de "stuck" | Ruteo de complejidad por gateway de cómputo | El loop ajusta estrategia según historial de patrones de fallo | Tasa de finalización autónoma / iteraciones hasta éxito |
| **[07 · Graph](../docs/07-graph-engineering.md)** | Ramificación if/else escondida en prompts | Grafo básico sin checkpoints | Checkpoints persistentes + aprobación humana explícita | Precisión multi-hop medida vs. baseline vectorial | Recorrido agéntico dinámico + ontología mantenida activamente | Precisión en consultas multi-hop vs. búsqueda vectorial pura |
| **[08 · Eval](../docs/08-eval-engineering.md)** | Evaluación a ojo, sin golden set | Golden set revisado manualmente | LLM-as-judge con evaluador separado del generador | Mitigación activa de sesgos + bloqueo de CI por regresión | Harness-Bench + evaluación continua sobre tráfico real | Tasa de acuerdo juez-humano |

## Lectura del perfil de madurez

Un equipo no tiene "un" nivel de madurez — tiene un **perfil**: un vector de ocho valores, uno por disciplina. Dos perfiles típicos y lo que indican:

- **Perfil "punta de iceberg"** — Prompt en Nivel 3-4, todo lo demás en Nivel 0-1. Típico de un producto que arrancó como demo de chatbot y nunca formalizó lo que rodea al modelo. El riesgo no está en el prompt, está en todo lo que no tiene evaluador, no tiene sandbox y no tiene golden set.
- **Perfil "arnés sin instrumentación"** — Harness en Nivel 2-3, Eval en Nivel 0-1. El sistema ejecuta acciones reales de forma relativamente segura, pero nadie puede demostrar con datos si esas acciones son *correctas*, solo que no rompieron nada. Es el perfil más peligroso en dominios regulados porque la ausencia de incidentes se confunde con calidad.

## KPIs cuantitativos — hoja de referencia rápida

| Disciplina | KPI ancla | Frecuencia de medición sugerida |
|---|---|---|
| Prompt Engineering | % adherencia de formato sin reintentos | Por cada cambio de prompt (CI) |
| Flow Engineering | Pass@k del flujo vs. invocación directa | Por release |
| Context Engineering | Tokens efectivos / trabajo útil completado | Continuo (dashboard) |
| RAG Engineering | Faithfulness (RAGAS) | Por cada cambio al pipeline de retrieval (CI) |
| Harness Engineering | % tareas completadas sin intervención humana | Semanal, sobre set representativo |
| Loop Engineering | % finalización autónoma / iteraciones promedio | Por cada tarea lanzada |
| Graph Engineering | Precisión multi-hop vs. baseline vectorial | Por cambio de ontología o estrategia de fusión |
| Eval Engineering | Tasa de acuerdo juez-humano | Mensual, sobre muestra |

## Ver también

- [00 · Panorama Sistémico](../docs/00-overview.md) — cómo se relacionan las ocho disciplinas entre sí.
- [resources/glossary.md](../resources/glossary.md) — definiciones de los términos usados en esta matriz.
