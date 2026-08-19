# 02 · Flow Engineering

> Descomposición de un problema complejo en un flujo de fases verificables: analizar, generar, probar, refinar.

[← Prompt Engineering](01-prompt-engineering.md) · [Índice](../README.md) · [Context Engineering →](03-context-engineering.md)

---

## Definición

Flow Engineering reemplaza la pretensión de que un modelo resuelva un problema complejo en una sola pasada por un pipeline explícito de fases discretas, cada una con su propio criterio de verificación. En vez de una llamada gigante tipo System 1 (intuitiva, frágil), el sistema opera en modo System 2: analiza, genera candidatos, los prueba contra evaluadores deterministas, refina, y solo entonces avanza a la fase siguiente.

El caso formalizador de la disciplina es AlphaCodium: pasar de generación directa de código (muestreo masivo, ~1M invocaciones, Pass@5 del 19%) a un flujo estructurado de ~100 invocaciones con Pass@5 del 44% — más del doble de precisión con dos órdenes de magnitud menos de cómputo. El punto no es "más prompt", es mejor arquitectura de proceso.

## Cuándo usarla / cuándo no

**Usar cuando:** el problema tiene fases lógicamente separables con un evaluador determinista en al menos una de ellas (tests que pasan/fallan, un compilador, un linter, un esquema que valida) y el costo de un error en cascada es alto.

**No conviene cuando:** la tarea es genuinamente de un paso — envolver una clasificación simple en 4 fases de flujo es sobre-ingeniería y agrega latencia sin agregar precisión. Tampoco es la herramienta correcta para *estado que persiste entre sesiones* (eso es Context Engineering) ni para *bucles autónomos sin límite de pasos fijo* (eso es Loop Engineering) — Flow Engineering asume un número de fases conocido de antemano.

## El Estándar

- [ ] **Cada fase tiene un contrato de entrada/salida explícito**, no un handoff implícito de texto libre entre pasos.
- [ ] **Al menos una fase tiene un evaluador determinista** (test, compilador, validador de esquema) — si todas las fases se validan con "otro LLM opina", no hay control de calidad real, hay teatro de rigor.
- [ ] **Generación de datos de prueba sintéticos antes de generar la solución**, no después — el orden importa: definir qué es "correcto" antes de intentar serlo.
- [ ] **Autorreflexión acotada por checkpoint**, no libre — el modelo reflexiona sobre un artefacto concreto de la fase anterior, con un límite de iteraciones definido.
- [ ] **Fallos en cascada cortados en la fuente**: si la fase 2 falla su verificación, no avanza a la fase 3 con datos corruptos; vuelve a la fase 2 o escala a intervención humana.
- [ ] **Trazabilidad de fase**: cada corrida deja registro de en qué fase estuvo, cuántos intentos tomó cada una, y con qué evaluador se validó.

## Indicadores de Madurez (0-4)

| Nivel | Nombre | Qué se observa |
|---|---|---|
| **0** | Ad-hoc | Una sola llamada intenta resolver todo; cuando falla, se reintenta con el mismo prompt esperando resultado distinto. |
| **1** | Repetible | Flujo de fases fijo, pero validado manualmente por un humano entre pasos. |
| **2** | Definido | Fases con contratos de entrada/salida explícitos y al menos un evaluador determinista automatizado. |
| **3** | Gestionado cuantitativamente | Métricas de Pass@k (o equivalente del dominio) medidas por fase, con umbrales que bloquean el avance si no se cumplen. |
| **4** | Optimización continua | El flujo mismo se optimiza con datos de producción — qué fases fallan más, dónde vale la pena invertir más cómputo, ruteo dinámico de complejidad. |

**KPI ancla:** Pass@k (o la métrica de éxito equivalente al dominio) del flujo completo, comparado contra la misma tarea resuelta con una sola invocación directa — el delta es la justificación económica del flujo.

## Modos de Falla / Anti-patrones

- **Rigidez ante tareas no estructuradas**: forzar un flujo de fases fijas sobre un problema abierto que en realidad necesita exploración libre.
- **Evaluador de juguete**: la fase de "test" es otro LLM preguntándose a sí mismo si el resultado le gusta, sin criterio determinista real.
- **Explosión combinatoria de reintentos**: sin límite de iteraciones por fase, un flujo puede consumir cómputo sin cota ante un caso patológico.
- **Contratos de fase implícitos**: la fase 2 asume una forma de dato que la fase 1 nunca garantizó explícitamente — rompe en producción, nunca en el ejemplo de demo.

## Recursos Curados

**Paper fundacional**
- [Code Generation with AlphaCodium: From Prompt Engineering to Flow Engineering](https://arxiv.org/abs/2401.08500) (arXiv:2401.08500) — el paper que formaliza la disciplina con datos de Pass@5.
- [codium-ai/alphacodium](https://github.com/codium-ai/alphacodium) — implementación de referencia.

**Frameworks de orquestación de flujo**
- [DSPy](https://dspy.ai/) — compila pipelines declarativos de LLM en flujos auto-optimizables.
- [LangGraph](https://www.langchain.com/langgraph) — cuando el flujo necesita bifurcaciones condicionales, ver también [07 · Graph Engineering](07-graph-engineering.md).

**Lectura**
- [Flow Engineering vs Prompt Engineering — M Accelerator](https://maccelerator.la/en/blog/entrepreneurship/flow-engineering-vs-prompt-engineering-complex-tasks/)
- [The AI shift from prompt engineering to flow engineering — Techzine](https://www.techzine.eu/blogs/applications/118176/the-ai-shift-from-prompt-engineering-to-flow-engineering/)

## Ver también

- [01 · Prompt Engineering](01-prompt-engineering.md) — la unidad atómica que Flow Engineering orquesta en secuencia.
- [07 · Graph Engineering](07-graph-engineering.md) — cuando las fases dejan de ser lineales y necesitan bifurcaciones/ciclos.
- [08 · Eval Engineering](08-eval-engineering.md) — cómo construir el evaluador determinista que cada fase necesita.
