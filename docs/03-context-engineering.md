# 03 · Context Engineering

> Gestión dinámica de qué información vive en la ventana de contexto activa a lo largo de una ejecución extendida.

[← Flow Engineering](02-flow-engineering.md) · [Índice](../README.md) · [RAG Engineering →](04-rag-engineering.md)

---

## Definición

Context Engineering administra la memoria de trabajo del agente: qué entra a la ventana de contexto, en qué orden, cuánto tiempo permanece y cuándo se descarta. Es necesaria porque los LLM no tienen memoria persistente propia y su razonamiento se degrada cuando el contexto se satura o se contamina con información irrelevante — el fenómeno conocido como *context rot*.

La disciplina se organiza en cuatro operaciones, popularizadas por el equipo de LangChain y ya estándar de facto en 2026: **Write** (sacar contexto fuera de la ventana activa — scratchpads, archivos de memoria, planes corriendo en disco), **Compress** (resumir turnos resueltos, podar salidas de herramientas verbosas), **Isolate** (separar el trabajo en ventanas de contexto independientes vía subagentes o sandboxes, para que el ruido de una tarea no contamine otra) y **Select** (elegir con precisión qué recuperar de la memoria externa cuando hace falta).

## Cuándo usarla / cuándo no

**Usar cuando:** la ejecución se extiende por múltiples turnos o pasos de agente, hay llamadas a herramientas con salidas voluminosas (logs, resultados de test, dumps de datos), o la sesión necesita sobrevivir a un reinicio de contexto sin perder el estado del trabajo.

**No hace falta cuando:** la interacción es de una sola invocación acotada (eso es simplemente Prompt Engineering) o el estado completo cabe cómodo dentro del presupuesto de tokens sin riesgo de saturación.

## El Estándar

- [ ] **Compactación por umbral, no por pánico.** Definir un umbral de llenado de contexto (ej. 70-80%) que dispara resumen/independización del historial antiguo antes de llegar al límite duro de la API.
- [ ] **Tool-call offloading obligatorio para salidas masivas.** Cualquier salida de herramienta que exceda un tamaño razonable (ej. cientos de líneas de logs) se trunca a las líneas iniciales/finales en el contexto activo, con el contenido completo derivado al sistema de archivos para inspección bajo demanda.
- [ ] **Divulgación progresiva de herramientas/skills.** Los esquemas de API y definiciones de herramientas se cargan solo cuando la tarea concreta los necesita, no de forma estática al arrancar la sesión.
- [ ] **Archivos de memoria persistente versionados** (tipo `AGENTS.md`/`CLAUDE.md`) leídos e inyectados de forma determinista al iniciar cada sesión — no memoria implícita que vive solo en la cabeza de quien escribió el prompt original.
- [ ] **Aislamiento de contexto entre subagentes.** Un subagente que investiga o ejecuta una subtarea no debería contaminar la ventana del agente orquestador con su proceso interno completo — solo con el resultado sintetizado.
- [ ] **Presupuesto de tokens monitoreado activamente**, no descubierto cuando la API devuelve error de límite excedido.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | El contexto crece sin límite hasta que la sesión falla o el modelo empieza a alucinar por saturación. |
| **1** | Repetible | Compactación manual: alguien nota que la sesión "se puso rara" y reinicia con un resumen escrito a mano. |
| **2** | Definido | Compactación automática por umbral + offloading de salidas de herramientas grandes al filesystem. |
| **3** | Gestionado cuantitativamente | Se mide *context rot* de forma explícita (degradación de precisión en función de tokens usados) y el sistema decide compactar en base a esa métrica, no solo al conteo de tokens. |
| **4** | Optimización continua | Gestión de contexto delegada parcialmente al propio agente (patrón Generator/Reflector/Curator tipo ACE): el sistema decide qué vale la pena recordar, no solo cuánto recortar. |

**KPI ancla:** tokens efectivos por unidad de trabajo útil completada — no tokens totales consumidos, sino la relación entre contexto gastado y progreso real logrado. Complementarlo con tasa de *context rot* (degradación de precisión medida contra la misma tarea a distintos niveles de saturación de ventana).

## Modos de Falla / Anti-patrones

- **Context rot silencioso**: el modelo empieza a ignorar instrucciones tempranas de la sesión sin que nadie lo note hasta que el output ya es incorrecto.
- **Offloading que nadie lee**: se manda todo al filesystem "por las dudas" pero no hay mecanismo real de recuperación bajo demanda — es basura, no memoria.
- **Carga estática de todas las skills/herramientas al inicio**: infla el prompt de sistema con esquemas que la tarea concreta nunca va a usar.
- **Memoria persistente desactualizada**: un `AGENTS.md` que documenta decisiones de hace seis meses y contradice el estado real del código, generando instrucciones activamente engañosas.

## Recursos Curados

**Frameworks y toolkits**
- [Context7](https://context7.com/) — documentación de código actualizada inyectada directamente en el contexto de agentes de coding.
- [LangGraph](https://www.langchain.com/langgraph) — checkpointing y grafos de estado construidos explícitamente sobre las cuatro operaciones (Write/Compress/Isolate/Select).
- [muratcankoylan/Agent-Skills-for-Context-Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) — colección de skills enfocadas en gestión de contexto para agentes de producción, con benchmarks propios de efecto por skill.

**Listas curadas**
- [danielrosehill/Context-Engineering-Resources](https://github.com/danielrosehill/Context-Engineering-Resources) — índice de herramientas y utilidades del espacio.
- [bonigarcia/context-engineering](https://github.com/bonigarcia/context-engineering) — guía open-source con foco en sistemas consistentes y predecibles.

**Papers y research**
- Investigación ACE (*Agentic Context Engineering*, Stanford / SambaNova / UC Berkeley) — arquitectura Generator/Reflector/Curator para gestión de contexto delegada al propio agente.

## Ver también

- [04 · RAG Engineering](04-rag-engineering.md) — la fuente externa que Context Engineering selecciona y recupera.
- [05 · Harness Engineering](05-harness-engineering.md) — la capa E-T-**C**-L-O-V-G donde Context & Memory Management es una de las siete responsabilidades formales.
