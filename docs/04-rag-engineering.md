# 04 · RAG Engineering

> Arquitecturas que conectan la generación con conocimiento externo no paramétrico, verificable y sin reentrenar el modelo.

[← Context Engineering](03-context-engineering.md) · [Índice](../README.md) · [Harness Engineering →](05-harness-engineering.md)

---

## Definición

RAG Engineering diseña el puente entre un modelo generativo y una fuente de conocimiento externa, con el objetivo de fundamentar la respuesta en evidencia verificable en vez de en lo que el modelo memorizó durante el entrenamiento. Se divide en dos canalizaciones: una *offline* de preparación de datos (chunking, embeddings, indexado vectorial) y una *online* de recuperación y generación en tiempo real (búsqueda densa + dispersa, fusión de rangos, reranking, ensamblado del prompt final).

En 2026 el patrón de producción dominante para stacks serios combina recuperación (LlamaIndex o un vector store dedicado) + orquestación (LangChain/LangGraph) + evaluación (RAGAS o equivalente) — rara vez un solo framework cubre las tres capas bien.

## Cuándo usarla / cuándo no

**Usar cuando:** la respuesta necesita fundamentarse en conocimiento que cambia con el tiempo, es demasiado grande para caber en contexto, o es propietario/privado y no puede vivir en los pesos del modelo.

**No conviene cuando:** el conocimiento requerido es estable y acotado (cabe directo en el prompt sin recuperación) o la tarea necesita que el modelo *actúe* sobre el mundo, no solo consultarlo — eso es Harness Engineering. RAG tampoco reemplaza fine-tuning cuando lo que hace falta es cambiar el *comportamiento* del modelo (tono, formato, vocabulario de dominio) en vez de darle datos nuevos.

## El Estándar

- [ ] **Chunking estructural, no por conteo fijo de tokens.** Segmentar respetando encabezados, tablas o jerarquías del documento original — cortar a ciegas cada N tokens rompe relaciones conceptuales.
- [ ] **Recuperación híbrida por default.** Combinar búsqueda densa (semántica) con búsqueda dispersa tipo BM25 — la densa sola falla sistemáticamente en códigos de producto, nombres propios y terminología técnica exacta.
- [ ] **Reranking con cross-encoder antes de construir el prompt final**, no confiar en el orden crudo del retriever.
- [ ] **Reescritura y descomposición de consultas complejas** en subconsultas antes de recuperar, especialmente para preguntas *multi-hop*.
- [ ] **Golden set de evaluación con métricas de las cuatro dimensiones RAGAS**: context precision, context recall, faithfulness, answer relevancy — medidas antes de cada release, no solo revisadas a ojo.
- [ ] **Compresión post-recuperación**: eliminar oraciones irrelevantes del contexto recuperado antes de pasarlo al generador, para no pagar tokens por ruido.
- [ ] **Versionado y linaje de corpus**: saber de qué fuente, con qué fecha y bajo qué licencia salió cada chunk que termina fundamentando una respuesta.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | Chunking por tamaño fijo, embedding con un solo modelo, top-k directo sin reranking ni evaluación. |
| **1** | Repetible | Pipeline reproducible (ingesta → embedding → índice → retrieval → generación) pero sin métricas de calidad medidas. |
| **2** | Definido | Recuperación híbrida (densa + dispersa) con reranking, y un golden set mínimo de preguntas/respuestas esperadas. |
| **3** | Gestionado cuantitativamente | Métricas RAGAS (faithfulness, context precision/recall) corridas en CI antes de cada cambio al pipeline; umbrales que bloquean regresiones. |
| **4** | Optimización continua | GraphRAG o recuperación agéntica adaptativa: el sistema decide dinámicamente la estrategia de recuperación según la complejidad de la consulta, y el corpus se re-indexa de forma continua ante cambios de la fuente. |

**KPI ancla:** *faithfulness* (¿la respuesta contiene solo afirmaciones sostenidas por el contexto recuperado?) medido con RAGAS o equivalente sobre el golden set — es la métrica que más directamente correlaciona con reducción real de alucinaciones.

## Modos de Falla / Anti-patrones

- **Fragmentación semántica por chunking ciego**: un chunk corta una tabla o una definición a la mitad, y el retriever recupera la mitad inútil.
- **Recall alto, faithfulness bajo**: el sistema recupera los documentos correctos pero el generador igual alucina por encima de ellos — sin medir faithfulness por separado, este fallo queda invisible.
- **Vector store como única fuente de verdad**: perder la capacidad de responder consultas exactas (SKU, ID, nombre propio) porque todo pasa por similitud semántica.
- **Corpus sin linaje**: no poder auditar de dónde salió una afirmación específica cuando un usuario la cuestiona.

## Recursos Curados

**Frameworks de orquestación**
- [LlamaIndex](https://www.llamaindex.ai/) — fuerte en ingesta de datos y parsing de documentos difíciles.
- [LangChain](https://www.langchain.com/) + [LangGraph](https://www.langchain.com/langgraph) — orquestación multi-paso y agéntica.
- [Haystack](https://haystack.deepset.ai/) — pipelines tipados como DAGs, fuerte en evaluación y casos regulados.
- [DSPy](https://dspy.ai/) — optimización sistemática de pipelines RAG en vez de tuneo manual de prompts.
- [RAGFlow](https://github.com/infiniflow/ragflow) — fuerte en documentos escaneados y con tablas.

**Evaluación**
- [RAGAS](https://github.com/explodinggym/ragas) — framework de referencia para las cuatro métricas estándar de calidad RAG.

**Listas curadas**
- [Yigtwxx/awesome-rag-production](https://github.com/Yigtwxx/Awesome-RAG-Production) — foco explícito en producción, cada benchmark etiquetado por fuente (vendor/third-party/autor).

**GraphRAG (frontera del Nivel 4)**
- Ver [07 · Graph Engineering](07-graph-engineering.md) para la integración con grafos de conocimiento en consultas multi-hop.

## Ver también

- [03 · Context Engineering](03-context-engineering.md) — qué hacer con lo recuperado una vez que entra a la ventana de contexto.
- [07 · Graph Engineering](07-graph-engineering.md) — GraphRAG como evolución de RAG puro para razonamiento multi-hop.
- [08 · Eval Engineering](08-eval-engineering.md) — cómo construir el golden set y automatizar las métricas RAGAS en CI.
