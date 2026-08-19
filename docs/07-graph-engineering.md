# 07 · Graph Engineering

> Representación de conocimiento y flujos de control como estructuras de grafo — dos campos de aplicación distintos bajo el mismo nombre.

[← Loop Engineering](06-loop-engineering.md) · [Índice](../README.md) · [Eval Engineering →](08-eval-engineering.md)

---

## Definición

Graph Engineering es, en realidad, dos disciplinas hermanas que en 2026 comparten nombre y generan confusión sistemática. Vale la pena separarlas con precisión desde el primer párrafo:

1. **Grafos de orquestación (control-flow graphs):** modelar la ejecución de un sistema multi-agente como una máquina de estados finita — nodos son pasos de cómputo o invocaciones a modelos, aristas condicionales dictan las transiciones según el estado global. Reemplaza cadenas lineales o bucles no restringidos cuando la lógica de negocio necesita bifurcación, paralelismo genuino o puntos de pausa para revisión humana.
2. **Grafos de conocimiento (knowledge graphs / GraphRAG):** modelar el conocimiento del dominio como entidades (nodos) y relaciones tipadas (aristas) que un agente puede recorrer, en vez de documentos planos buscados por similitud. Resuelve consultas *multi-hop* que la búsqueda vectorial pura no puede responder porque la respuesta no está en un chunk, está en la relación entre dos chunks distintos.

Un tercer uso del término, más especulativo y menos accionable en 2026, describe "grafos de loops" — redes de ciclos de auto-mejora que se observan entre sí. Se documenta acá por completitud terminológica, pero no tiene aún un estándar de ingeniería maduro.

## Cuándo usarla / cuándo no

**Usar grafos de orquestación cuando:** el flujo tiene bifurcaciones condicionales genuinas, necesita ejecutar ramas en paralelo, o requiere puntos de pausa/reanudación para aprobación humana en medio de la ejecución.

**Usar GraphRAG cuando:** las consultas requieren relacionar conceptos dispersos en múltiples documentos (multi-hop) y la búsqueda vectorial pura viene fallando en ese tipo de consulta específicamente.

**No conviene ninguna de las dos cuando:** el flujo es una secuencia lineal simple sin bifurcaciones reales, o el corpus de conocimiento no tiene relaciones explícitas que valga la pena modelar — la advertencia más repetida por la propia comunidad en 2026 es que la mayoría de las tareas nunca necesitan un grafo; forzarlo antes de que el trabajo lo exija compra un problema de sistemas distribuidos que antes no existía.

## El Estándar

- [ ] **Distinguir explícitamente cuál de los dos significados se está usando** en cualquier documento de diseño — "graph engineering" sin calificar es ambiguo y genera decisiones de arquitectura equivocadas.
- [ ] **Nodos con contrato de estado explícito**, no nodos que mutan estado global de forma implícita — cada nodo declara qué lee y qué escribe.
- [ ] **Aristas condicionales auditables**: la lógica de ruteo entre nodos vive en código versionado, no en la interpretación libre de un LLM en cada corrida.
- [ ] **Checkpoints persistentes** que permiten pausar, reanudar y bifurcar la ejecución en puntos específicos — sin esto, "grafo" es solo una cadena lineal con nombre pretencioso.
- [ ] **Para GraphRAG: ontología versionada y mantenida**, no generada una vez y abandonada — un grafo de conocimiento que no se actualiza es peor que un vector store, porque además transmite falsa confianza de estructura.
- [ ] **Motor de fusión explícito entre vector search y graph traversal** en implementaciones híbridas — no elegir una u otra por corrida, combinar ambas y declarar cómo se pondera cada una.
- [ ] **Solo se adopta el grafo cuando un loop simple ya demostró no alcanzar** — no como punto de partida por defecto.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | Lógica de ramificación si/entonces escondida dentro de prompts o código imperativo sin estructura de grafo explícita. |
| **1** | Repetible | Grafo de orquestación básico (nodos + aristas fijas) sin checkpoints ni capacidad de pausar/reanudar. |
| **2** | Definido | Checkpoints persistentes, ramas paralelas genuinas, puntos de aprobación humana explícitos en el grafo. |
| **3** | Gestionado cuantitativamente | Para GraphRAG: métricas de precisión en consultas multi-hop medidas contra un baseline de vector search puro, con umbral de mejora que justifica la complejidad agregada. |
| **4** | Optimización continua | Recorrido agéntico y dinámico del grafo (el agente decide en vivo qué saltos tomar) con ontología mantenida activamente y motor de fusión ajustado por tipo de consulta. |

**KPI ancla:** para orquestación, tasa de ejecuciones que requieren intervención manual fuera del flujo diseñado (indicador de que el grafo no cubre casos reales). Para GraphRAG, precisión en preguntas multi-hop comparada contra la misma consulta resuelta con búsqueda vectorial pura — el delta es lo que justifica el costo de mantener la ontología.

## Modos de Falla / Anti-patrones

- **Sobrediseño arquitectónico**: construir un grafo completo para un flujo que en realidad es tres pasos lineales — el anti-patrón más citado en 2026 dentro de esta disciplina específicamente.
- **Grafo de conocimiento abandonado**: la ontología se crea una vez en el kickoff del proyecto y nunca se actualiza; seis meses después describe un mundo que ya no existe.
- **Confundir los dos significados en la misma conversación de diseño**: alguien propone "usar un grafo" pensando en orquestación, otro lo interpreta como GraphRAG, y el proyecto termina construyendo las dos cosas sin que nadie las haya pedido juntas.
- **Aristas condicionales delegadas 100% a interpretación del LLM en cada corrida**: el "grafo" deja de ser auditable porque el camino tomado depende de una decisión probabilística no versionada.

## Recursos Curados

**Orquestación por grafos**
- [LangGraph](https://www.langchain.com/langgraph) — el framework de referencia para grafos de orquestación stateful en 2026.
- [Microsoft AutoGen / AG2](https://github.com/microsoft/autogen) — orquestación multi-agente con soporte de grafo.
- [Google ADK](https://google.github.io/adk-docs/) — patrones Runner/AgentTool para envolver subagentes como herramientas dentro de un grafo mayor.

**GraphRAG y grafos de conocimiento**
- [Microsoft GraphRAG](https://github.com/microsoft/graphrag) — implementación de referencia para consultas multi-hop sobre corpus grandes.
- Bases de grafo comunes en stacks de producción: Neo4j, Memgraph, ArangoDB, FalkorDB.
- [Graphiti](https://github.com/getzep/graphiti) (motor detrás de Zep) — grafo temporal que trackea cuándo un hecho fue verdadero en el mundo y cuándo el sistema lo aprendió, útil para memoria de agente que cambia con el tiempo.

**Lectura**
- [Graph Engineering for AI Agents: A Complete Guide in LangGraph — Analytics Vidhya](https://www.analyticsvidhya.com/blog/2026/07/graph-engineering/)
- [What Is Graph Engineering? A Field Guide for Builders](https://theaioperator.io/p/what-is-graph-engineering-a-field)
- [Knowledge Graph vs RAG: When Each One Wins — Atlan](https://atlan.com/know/knowledge-graphs-vs-rag-for-ai/)

## Ver también

- [04 · RAG Engineering](04-rag-engineering.md) — GraphRAG como evolución sobre RAG vectorial puro.
- [06 · Loop Engineering](06-loop-engineering.md) — el punto de partida que hay que agotar antes de justificar un grafo de orquestación.
