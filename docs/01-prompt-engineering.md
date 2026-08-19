# 01 · Prompt Engineering

> Diseño de instrucciones, formatos y plantillas de razonamiento para una invocación aislada de un LLM.

[← Volver al índice](../README.md) · [Ver Loop Engineering →](06-loop-engineering.md)

---

## Definición

Prompt Engineering es la disciplina que estructura instrucciones, ejemplos, roles y formatos de salida para maximizar la calidad de una respuesta de un modelo de lenguaje en una sola invocación (o un puñado de invocaciones aisladas, sin estado compartido entre ellas). Se apoya en *in-context learning*: el modelo no se reentrena, se lo guía con lo que cabe en la ventana de contexto de esa llamada puntual.

Es el nivel de abstracción más bajo de toda la pila. Todo lo demás en este repo —Flow, Context, RAG, Harness, Loop, Graph, Eval— existe porque el Prompt Engineering solo, a secas, se rompe ante tareas de múltiples pasos, estado persistente o ejecución de herramientas.

## Cuándo usarla / cuándo no

**Usar cuando:** la tarea es de un solo paso, con criterio de éxito verificable a simple vista (clasificación, extracción, resumen corto, generación de un fragmento acotado) y no requiere memoria entre llamadas ni herramientas externas.

**No alcanza cuando:** la tarea exige múltiples pasos verificables de forma independiente (→ Flow Engineering), estado que sobrevive más de una invocación (→ Context Engineering), conocimiento externo no paramétrico (→ RAG Engineering) o ejecución de acciones sobre un entorno real (→ Harness Engineering). Forzar estos casos dentro de un único prompt masivo produce lo que se conoce como *God Prompt*: instrucciones sobrecargadas que degradan el seguimiento de instrucciones y disparan el costo por token sin mejorar el resultado.

## El Estándar

Prácticas que separan un prompt de producción de un prompt de demo:

- [ ] **Rol y audiencia explícitos.** El prompt declara quién es el modelo en ese contexto y para quién escribe, no lo deja implícito.
- [ ] **Formato de salida especificado por contrato**, no por ejemplo suelto — JSON Schema, XML tags, o una plantilla literal, según lo que consuma el sistema aguas abajo.
- [ ] **Few-shot con ejemplos negativos y positivos** cuando la tarea tiene ambigüedad de formato, no solo ejemplos de "cómo sí".
- [ ] **Chain-of-Thought solo cuando el problema lo necesita.** Pedir razonamiento explícito en tareas triviales agrega latencia y costo sin ganancia medible; reservarlo para tareas con pasos lógicos genuinos.
- [ ] **Versionado del prompt como artefacto de código** — en git, con changelog, no editado a mano en un playground y perdido.
- [ ] **Test de regresión de prompt** antes de cada cambio: un set fijo de casos (golden set) que corre contra la versión nueva y la vieja.
- [ ] **Presupuesto de tokens declarado**, no descubierto en producción — cuánto contexto, cuánto output esperado, a qué costo.
- [ ] **Sin lógica condicional compleja dentro del texto del prompt** ("si pasa X hacé A, si no B, salvo que C…") — eso es una máquina de estados disfrazada de prosa; si aparece, es señal de que la tarea pertenece a Flow Engineering.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | Prompts escritos y editados directo en el playground/UI, sin versión, sin test, sin registro de qué cambió y por qué. |
| **1** | Repetible | Prompts guardados como archivos versionados en el repo, pero sin test de regresión automatizado. |
| **2** | Definido | Existe un template estándar (rol, formato, few-shot) aplicado de forma consistente y un golden set mínimo por prompt crítico. |
| **3** | Gestionado cuantitativamente | Cada cambio de prompt corre contra el golden set con métricas objetivas (adherencia de formato, tasa de error) antes de mergear. |
| **4** | Optimización continua | Optimización sistemática de prompts (ej. DSPy/MIPROv2 u optimizadores equivalentes) reemplaza el tuneo manual; el prompt se trata como parámetro optimizable, no como prosa artesanal. |

**KPI ancla:** tasa de adherencia al formato de salida contratado (parseabilidad sin reintentos) sobre el golden set. Es el proxy más barato y más honesto de calidad de prompt en producción.

## Modos de Falla / Anti-patrones

- **God Prompt**: un único prompt intenta cubrir instrucciones, contexto, herramientas y razonamiento multi-paso a la vez. Síntoma: el prompt crece cada vez que aparece un caso nuevo, nunca se poda.
- **Fragilidad semántica**: cambios menores de redacción (agregar una coma, reordenar una frase) alteran drásticamente la salida. Señal de que el prompt no tiene contrato de formato real.
- **Few-shot desbalanceado**: todos los ejemplos son casos "felices"; el modelo nunca vio un caso límite y falla exactamente ahí en producción.
- **CoT usado como placebo**: pedir "pensá paso a paso" en tareas triviales, pagando latencia sin ganancia de precisión medible.

## Recursos Curados

**Guías de referencia**
- [Prompt Engineering Guide](https://www.promptingguide.ai/) — guías, papers, notebooks y técnicas, mantenido activamente.
- [Learn Prompting](https://learnprompting.org/) — introducción estructurada de técnicas de prompt engineering.

**Listas curadas (awesome)**
- [natnew/Awesome-Prompt-Engineering](https://github.com/natnew/Awesome-Prompt-Engineering) — lista con secciones de Context Engineering, agentes y glosario extenso.
- [promptslab/Awesome-Prompt-Engineering](https://github.com/promptslab/Awesome-Prompt-Engineering) — curada a mano, foco en GPT/ChatGPT/PaLM.
- [brandonhimpfen/awesome-prompt-engineering](https://github.com/brandonhimpfen/awesome-prompt-engineering) — herramientas, papers y plataformas.

**Herramientas de testing y versionado de prompts**
- [Promptfoo](https://www.promptfoo.dev/) — testing, evaluación y benchmarking de prompts, pensado para CI.
- [PromptLayer](https://www.promptlayer.com/) — versionado y gestión de prompts en producción.
- [DSPy](https://dspy.ai/) — framework de Stanford NLP para optimización sistemática de prompts (y pesos) en vez de tuneo manual; el puente natural hacia el Nivel 4 de madurez.

**Papers**
- Liu et al., *Prompt Programming for Large Language Models* — patrones fundacionales de la disciplina.

## Ver también

- [02 · Flow Engineering](02-flow-engineering.md) — cuando la tarea deja de ser una sola invocación.
- [03 · Context Engineering](03-context-engineering.md) — cuando el estado tiene que sobrevivir más de una llamada.
- [Matriz de Madurez completa](../maturity-model/MATURITY-MATRIX.md)
