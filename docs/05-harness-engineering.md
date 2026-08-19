# 05 · Harness Engineering

> La infraestructura completa que envuelve al modelo para convertirlo en un sistema capaz de trabajo autónomo real.

[← RAG Engineering](04-rag-engineering.md) · [Índice](../README.md) · [Loop Engineering →](06-loop-engineering.md)

---

## Definición

Harness Engineering es la disciplina que diseña el "cuerpo" alrededor del "cerebro". El modelo aporta razonamiento e inferencia; el arnés aporta herramientas, gestión de estado persistente, entornos de ejecución aislados (sandboxes), barreras de seguridad y retroalimentación continua. La premisa operativa central de la disciplina, sostenida por evidencia empírica repetida en 2026: **un modelo de capacidad moderada con un arnés excelente supera de forma sistemática a un modelo avanzado con un arnés deficiente.**

Un arnés envuelve al modelo en un bucle cibernético tipo ReAct (*Reason, Act, Observe*). Por encima de ese bucle hay controles de gobernanza que se dividen en dos direcciones — *feedforward* (guía antes de la acción: reglas de arquitectura, plantillas) y *feedback* (sensor después de la acción: linters, tests, evaluadores) — cruzadas con la naturaleza del cómputo — *computational* (determinista, CPU, rápido) e *inferential* (probabilístico, GPU/NPU, lento).

## La Taxonomía ETCLOVG

Siete capas de responsabilidad que todo arnés serio debe cubrir, explícita o implícitamente:

| Capa | Responsabilidad | Ejemplos concretos |
|---|---|---|
| **E** — Execution Environment | Entornos de ejecución seguros y aislados | Contenedores Docker, navegadores headless, sandboxes de ejecución de código |
| **T** — Tool Interface & Protocol | Contratos de herramientas y protocolos | Esquemas de entrada/salida, MCP (Model Context Protocol) |
| **C** — Context & Memory Management | Persistencia de estado y limpieza de contexto | Integración con git, archivos `AGENTS.md`/`CLAUDE.md` — ver [03 · Context Engineering](03-context-engineering.md) |
| **L** — Lifecycle & Orchestration | Control del bucle de razonamiento | Intercepción de señales de salida, coordinación de subagentes |
| **O** — Observability & Operations | Trazas, latencia, telemetría, presupuesto | Logging de ejecución, control de costo por token |
| **V** — Verification & Evaluation | Retroalimentación determinista | Ver [08 · Eval Engineering](08-eval-engineering.md) |
| **G** — Governance & Security | Permisos, listas de comandos, aprobación humana | Aislamiento de red, allowlists de comandos, human-in-the-loop |

Fundamento cibernético: por la Ley de Requisito de Variedad de Ashby, un sistema de control solo puede gobernar un objetivo si posee al menos tanta variedad como el sistema que regula. El arnés reduce la variedad descontrolada de las respuestas probabilísticas del modelo mediante controles *computational* e *inferential* combinados, haciendo viable la gobernanza automatizada.

## Cuándo usarla / cuándo no

**Usar cuando:** el sistema necesita ejecutar acciones reales sobre un entorno (filesystem, terminal, APIs, navegador) de forma autónoma o semi-autónoma, más allá de generar texto.

**No hace falta cuando:** la interacción es puramente conversacional sin efectos secundarios sobre ningún sistema externo — ahí el costo de construir un arnés completo (sandboxing, governance, observability) es sobre-ingeniería.

## El Estándar

- [ ] **Las siete capas ETCLOVG están cubiertas explícitamente**, no implícitas — si no se puede señalar dónde vive cada una en el diseño, falta diseño, no falta código.
- [ ] **Sandboxing por default, no por excepción.** Ejecución de código y comandos aislada del host; el modo "acceso completo" es opt-in explícito, nunca el default.
- [ ] **Allowlist de comandos/herramientas**, no blocklist — es la única postura que no depende de anticipar todo lo que puede salir mal.
- [ ] **Human-in-the-loop en acciones irreversibles.** Cualquier acción que no se pueda deshacer (borrar, enviar, pagar, publicar) pasa por un gate de aprobación explícito.
- [ ] **Observabilidad desde el día uno**: trazas completas de cada paso del bucle ReAct, no solo el resultado final — sin esto, depurar un fallo en producción es arqueología.
- [ ] **Controles feedforward y feedback documentados por separado**, cruzados con computational/inferential — la matriz 2x2 completa, no solo la mitad más fácil de implementar.
- [ ] **Presupuesto de tokens y de tiempo con corte duro**, no solo advertencia — un arnés sin límite de recursos es un bucle infinito en potencia.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | El modelo ejecuta comandos directo contra el host, sin sandbox, sin logging, sin límites. |
| **1** | Repetible | Sandbox básico + logging manual revisado a posteriori. |
| **2** | Definido | Al menos 4 de las 7 capas ETCLOVG cubiertas explícitamente, con allowlist de herramientas y observabilidad estructurada. |
| **3** | Gestionado cuantitativamente | Las 7 capas cubiertas; tasa de intervención humana requerida medida y usada como métrica de calidad del arnés. |
| **4** | Optimización continua | Harness-Bench interno: se miden variantes del arnés (mismo modelo, mismo presupuesto) para aislar qué componente del arnés mueve la aguja del resultado — ver [08 · Eval Engineering](08-eval-engineering.md). |

**KPI ancla:** tasa de finalización de tareas sin intervención humana, medida en un set fijo de tareas representativas del dominio — es la métrica que más directamente valida la premisa central de la disciplina (arnés bueno > modelo bueno).

## Modos de Falla / Anti-patrones

- **Acoplamiento complejo entre capas**: cambiar el sandbox rompe la observabilidad porque nunca se diseñaron como capas independientes.
- **Governance como ocurrencia tardía**: se agregan permisos y aprobaciones después de un incidente, no como parte del diseño original.
- **Sandbox de cartón**: aislamiento que se puede eludir fácilmente porque nunca se probó de forma adversarial.
- **Confundir arnés con modelo**: invertir en el modelo más caro disponible mientras el arnés que lo rodea sigue en Nivel 0-1 — la premisa central de la disciplina dice exactamente al revés dónde está el ROI.

## Recursos Curados

**Frameworks de arnés / runtime de agentes**
- [LangGraph](https://www.langchain.com/langgraph) — orquestación stateful con checkpoints, fuerte en trabajo regulado/auditable.
- [Microsoft Agent Framework](https://github.com/microsoft) — unificación de Semantic Kernel y AutoGen, orquestación por grafos y middleware de intercepción.
- [Google ADK (Agent Development Kit)](https://google.github.io/adk-docs/) — framework code-first con orquestación multi-agente y pipeline de evals integrado.
- [strands-agents](https://github.com/strands-agents) — SDK de AWS con loop, tool-binding y guardrails como primitivas de primera clase.
- [AutoGen/AG2](https://github.com/microsoft/autogen) — sandboxing nativo con Docker, fuerte en ejecución de código y debate multi-agente.

**Listas curadas**
- [ai-boost/awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering) — herramientas, patrones, memoria, MCP, permisos y observabilidad.
- [danielrosehill/AI-Harnesses](https://github.com/danielrosehill/AI-Harnesses) — snapshot de proyectos que se autodenominan "arnés" (abril 2026).

**Lectura de referencia**
- [Harness Engineering for Coding Agent Users — Martin Fowler](https://martinfowler.com/articles/harness-engineering.html)
- [The Anatomy of an Agent Harness — LangChain](https://www.langchain.com/blog/the-anatomy-of-an-agent-harness)
- [What Is an AI Agent Harness? — Databricks](https://www.databricks.com/blog/ai-harness)

## Ver también

- [06 · Loop Engineering](06-loop-engineering.md) — la capa L (Lifecycle & Orchestration) del arnés, en detalle.
- [08 · Eval Engineering](08-eval-engineering.md) — la capa V (Verification & Evaluation) del arnés, en detalle.
- [03 · Context Engineering](03-context-engineering.md) — la capa C (Context & Memory Management) del arnés, en detalle.
