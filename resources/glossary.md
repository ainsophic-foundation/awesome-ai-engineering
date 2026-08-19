# Glosario

[← Índice](../README.md)

Términos usados de forma consistente en las ocho páginas de disciplina. Cuando un término tiene una definición formal en una disciplina específica, se linkea ahí en vez de duplicarla.

| Término | Definición breve | Disciplina de origen |
|---|---|---|
| **Context rot** | Degradación del razonamiento del modelo cuando la ventana de contexto se satura o se contamina con información irrelevante. | [Context Engineering](../docs/03-context-engineering.md) |
| **God Prompt** | Instrucción única sobrecargada que intenta cubrir contexto, herramientas y razonamiento multi-paso a la vez; degrada el seguimiento de instrucciones. | [Prompt Engineering](../docs/01-prompt-engineering.md) |
| **Golden set** | Conjunto de referencia fijo y versionado con entrada y salida esperada conocida, usado para medir regresiones. | [Eval Engineering](../docs/08-eval-engineering.md) |
| **Harness-Bench** | Diseño experimental que fija tarea, presupuesto y evaluador, y varía solo la configuración del arnés, para aislar el efecto del arnés del efecto del modelo. | [Eval Engineering](../docs/08-eval-engineering.md) / [Harness Engineering](../docs/05-harness-engineering.md) |
| **In-Context Learning** | Capacidad del modelo de adaptar su comportamiento a partir de lo que está presente en el prompt, sin actualizar sus pesos. | [Prompt Engineering](../docs/01-prompt-engineering.md) |
| **LLM-as-judge** | Uso de un modelo de lenguaje como evaluador automatizado de la salida de otro modelo (o de sí mismo), sujeto a sesgos conocidos y medibles. | [Eval Engineering](../docs/08-eval-engineering.md) |
| **Multi-hop (consulta)** | Pregunta cuya respuesta requiere relacionar información de más de una fuente/entidad — no resoluble con una sola recuperación semántica directa. | [RAG Engineering](../docs/04-rag-engineering.md) / [Graph Engineering](../docs/07-graph-engineering.md) |
| **Pass@k** | Probabilidad de que al menos una de *k* muestras generadas por el modelo pase el criterio de éxito; métrica estándar para tareas de generación de código. | [Flow Engineering](../docs/02-flow-engineering.md) |
| **Ralph Loop** | Patrón de loop autónomo donde el mismo prompt se reinyecta iteración tras iteración, con el progreso persistido en filesystem/git en vez de en la ventana de contexto. | [Loop Engineering](../docs/06-loop-engineering.md) |
| **Reciprocal Rank Fusion (RRF)** | Algoritmo para combinar rankings de múltiples métodos de recuperación (denso + disperso) en un único orden final. | [RAG Engineering](../docs/04-rag-engineering.md) |
| **Sandbox** | Entorno de ejecución aislado del host, donde un agente puede correr código o comandos sin riesgo de efectos secundarios no controlados. | [Harness Engineering](../docs/05-harness-engineering.md) |
| **System 1 / System 2** | Metáfora tomada de psicología cognitiva: System 1 es respuesta directa e intuitiva (una invocación); System 2 es razonamiento deliberado y verificado por etapas. | [Flow Engineering](../docs/02-flow-engineering.md) |
| **Taxonomía ETCLOVG** | Las siete capas de responsabilidad de un arnés: Execution, Tool Interface, Context & Memory, Lifecycle, Observability, Verification, Governance. | [Harness Engineering](../docs/05-harness-engineering.md) |
