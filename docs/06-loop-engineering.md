# 06 · Loop Engineering

> Diseño de bucles de ejecución autónomos con self-prompting, iteración hasta criterio de parada explícito, y gestión propia del error.

[← Harness Engineering](05-harness-engineering.md) · [Índice](../README.md) · [Graph Engineering →](07-graph-engineering.md)

---

## Definición

Loop Engineering diseña el sistema que decide *cuándo* volver a invocar al agente, no el contenido de cada invocación individual. Mientras un bucle ReAct simple se detiene ante el primer fallo sintáctico o respuesta imprecisa, un loop de producción implementa persistencia real: si la verificación no se cumple, el sistema limpia la ventana de contexto pero preserva el estado del trabajo en disco (filesystem, git), y reinyecta la consigna original para forzar al agente a seguir iterando desde un entorno de contexto fresco.

El patrón de referencia es el **Ralph Loop** (Geoffrey Huntley, 2025): en su forma más pura, un bucle de Bash que alimenta el mismo prompt una y otra vez, dejando que el agente vea su propio trabajo previo a través del filesystem y el historial de git — no por retroalimentación de texto, sino por estado externo persistente. La cita que resume el cambio de foco de la disciplina, atribuida a Boris Cherny (Anthropic): "mi trabajo es escribir loops" — el prompt individual dejó de ser el cuello de botella; el diseño del bucle que decide qué prompt mandar y cuándo, sí lo es.

## Cuándo usarla / cuándo no

**Usar cuando:** la tarea requiere trabajo autónomo sostenido más allá de lo que un humano puede supervisar turno por turno — tareas largas con criterio de éxito verificable de forma determinista (tests que pasan, un PRD con items marcables como completos).

**No conviene cuando:** no existe una condición de parada verificable de forma objetiva — un loop sin evaluador determinista claro es un bucle que puede "declarar victoria" prematuramente (el modelo mira alrededor, ve progreso, asume que terminó) o correr indefinidamente sin nunca completar. Tampoco es la herramienta si la tarea es de un paso — ahí Loop Engineering es sobre-ingeniería pura.

## El Estándar

- [ ] **Condición de parada verificable y determinista**, no "el modelo decide que terminó". Tests que pasan, un compilador sin errores, un diff contra spec en cero — nunca la propia opinión del agente sobre su trabajo.
- [ ] **Límite máximo de iteraciones (o de tiempo/costo) siempre presente**, incluso con condición de parada bien definida — es la red de seguridad contra loops que nunca convergen.
- [ ] **Estado persistido fuera de la ventana de contexto** (filesystem, git, base de datos) — el contexto de cada iteración puede reiniciarse limpio; el progreso real vive afuera.
- [ ] **Actualización de memoria de aprendizajes entre iteraciones** (tipo `AGENTS.md`) — cada iteración deja un rastro legible para la siguiente, y para humanos que auditen después.
- [ ] **Detección explícita de "stuck"**: si N iteraciones consecutivas no mueven la métrica de progreso, el loop escala a intervención humana en vez de seguir consumiendo cómputo.
- [ ] **Ruteo de complejidad por gateway de cómputo**: pasos mecánicos van a modelos livianos, errores lógicos complejos escalan a modelos de mayor capacidad — no todo el loop corre al costo del modelo más caro.
- [ ] **Checker separado del generador.** El mismo proceso que genera el trabajo no debería ser el único que decide si está bien hecho — ver [08 · Eval Engineering](08-eval-engineering.md).

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | Un humano reintenta manualmente el mismo prompt cuando el resultado no sirve. |
| **1** | Repetible | Loop scripteado básico (tipo Ralph puro) con condición de éxito y límite de tiempo, sin detección de "stuck". |
| **2** | Definido | Estado persistido en filesystem/git entre iteraciones, actualización de memoria de aprendizajes, detección de estancamiento. |
| **3** | Gestionado cuantitativamente | Ruteo de complejidad por gateway de cómputo (modelos livianos vs. pesados según el tipo de error) y métricas de iteraciones-hasta-éxito medidas por tipo de tarea. |
| **4** | Optimización continua | El propio loop ajusta su estrategia según datos históricos de qué patrones de estancamiento predicen fallos, e interviene antes de agotar el presupuesto de iteraciones. |

**KPI ancla:** tasa de finalización autónoma (tareas completadas sin intervención humana / tareas totales lanzadas) y, como métrica secundaria, iteraciones promedio hasta criterio de éxito — un loop maduro no solo termina, termina en pocos ciclos.

## Modos de Falla / Anti-patrones

- **Salida prematura ("premature exit")**: el agente mira su propio progreso parcial, concluye que terminó, y el loop lo acepta sin verificación determinista real.
- **Loop infinito por condición de parada mal definida**: consume presupuesto completo sin nunca cumplir el criterio, porque el criterio en sí es ambiguo o inalcanzable.
- **Contaminación de contexto entre iteraciones**: en vez de reiniciar limpio y leer el estado desde el filesystem, el loop arrastra la ventana de contexto completa iteración tras iteración — reintroduce exactamente el *context rot* que Loop Engineering debería evitar.
- **Deriva operacional**: sin gateway de cómputo, cada iteración paga el costo del modelo más caro disponible, incluso para pasos triviales.

## Recursos Curados

**Implementaciones de referencia**
- [snarktank/ralph](https://github.com/snarktank/ralph) — implementación del patrón Ralph Loop lista para usar con Claude Code o Amp, con actualización automática de `AGENTS.md` por iteración.

**Lectura fundacional**
- [From ReAct to Ralph Loop — Alibaba Cloud Community](https://www.alibabacloud.com/blog/from-react-to-ralph-loop-a-continuous-iteration-paradigm-for-ai-agents_602799)
- [What Is Loop Engineering? — MindStudio](https://www.mindstudio.ai/blog/what-is-loop-engineering-ai-coding-agents)
- [Stop Orchestrating AI Agents. Start Running Ralph Loops.](https://www.decodingai.com/p/ralph-loops)
- [The Agentic Loop — A Practical Field Guide (DEV Community)](https://dev.to/truongpx396/the-agentic-loop-a-practical-field-guide-mnc)
- [Loop Engineering: A Crash Course — The AI Agent Factory](https://agentfactory.panaversity.org/docs/loop-engineering-crash-course)

## Ver también

- [05 · Harness Engineering](05-harness-engineering.md) — la capa L (Lifecycle & Orchestration) donde vive formalmente el loop.
- [07 · Graph Engineering](07-graph-engineering.md) — cuándo un loop simple deja de alcanzar y hace falta coordinar múltiples nodos/agentes.
- [08 · Eval Engineering](08-eval-engineering.md) — cómo separar generador y verificador de forma rigurosa.
