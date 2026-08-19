# Awesome AI Engineering

**La taxonomía, el estándar y los indicadores de madurez de las ocho ingenierías detrás de un sistema de IA en producción.**

No es otra lista de links de "herramientas de IA". Es un mapa de las disciplinas de ingeniería que separan un demo de chatbot de un sistema autónomo de producción — con un estándar accionable, una matriz de madurez con KPIs, y recursos curados por disciplina, no mezclados en un balde único.

---

## Por qué existe este repo

La construcción de aplicaciones sobre LLM dejó de ser "escribir un buen prompt". Los modelos probabilísticos no tienen determinismo estricto, su atención está acotada por una ventana de contexto finita, y no pueden actuar sobre el mundo real de forma aislada. De esas tres limitaciones nacieron ocho disciplinas de ingeniería, cada una con su propio vocabulario, su propio estándar y sus propios modos de falla — y la confusión entre ellas es, en la práctica, la causa más común de que un sistema agéntico se rompa en producción después de funcionar perfecto en la demo.

Este repo separa las ocho, define qué significa "hacerlo bien" en cada una, y da una forma de medirlo.

## Las ocho disciplinas

| # | Disciplina | Resuelve | KPI ancla |
|---|---|---|---|
| 01 | [**Prompt Engineering**](docs/01-prompt-engineering.md) | Diseño de una invocación aislada al modelo | % adherencia de formato sin reintentos |
| 02 | [**Flow Engineering**](docs/02-flow-engineering.md) | Descomposición en fases verificables (AlphaCodium: Pass@5 19%→44%) | Pass@k del flujo vs. invocación directa |
| 03 | [**Context Engineering**](docs/03-context-engineering.md) | Gestión de la memoria de trabajo activa, *context rot* | Tokens efectivos / trabajo útil completado |
| 04 | [**RAG Engineering**](docs/04-rag-engineering.md) | Fundamentación en conocimiento externo verificable | Faithfulness (RAGAS) |
| 05 | [**Harness Engineering**](docs/05-harness-engineering.md) | Infraestructura completa de ejecución autónoma (taxonomía ETCLOVG) | % tareas completadas sin intervención humana |
| 06 | [**Loop Engineering**](docs/06-loop-engineering.md) | Bucles autónomos con criterio de parada (Ralph Loop) | % finalización autónoma / iteraciones hasta éxito |
| 07 | [**Graph Engineering**](docs/07-graph-engineering.md) | Orquestación multi-agente y GraphRAG multi-hop | Precisión multi-hop vs. baseline vectorial |
| 08 | [**Eval Engineering**](docs/08-eval-engineering.md) | Medición cuantitativa de todo lo anterior | Tasa de acuerdo juez-humano |

Cada página sigue la misma estructura: **Definición → Cuándo usarla/cuándo no → El Estándar (checklist) → Indicadores de Madurez (0-4) → Modos de Falla → Recursos Curados → Ver también.**

## Empezá acá

- **¿Primera vez en el repo?** → [00 · Panorama Sistémico](docs/00-overview.md) — cómo se relacionan las ocho disciplinas entre sí, y por qué no son ocho listas paralelas sino una jerarquía de inclusión.
- **¿Ya tenés un sistema en producción?** → [Matriz de Madurez](maturity-model/MATURITY-MATRIX.md) — autoevaluá cada disciplina en una escala 0-4 y encontrá dónde está tu cuello de botella real.
- **¿Buscás una herramienta puntual?** → entrá directo a la disciplina correspondiente; la sección "Recursos Curados" de cada página está organizada por categoría, no es un dump de links.
- **¿Te perdiste con un término?** → [Glosario](resources/glossary.md).

## La relación entre disciplinas, en una frase por capa

- **Prompt Engineering** gobierna una invocación individual.
- **Context Engineering** administra el estado dentro de esa invocación o de varias.
- **Flow Engineering** organiza secuencias de razonamiento en fases verificables.
- **RAG Engineering** y **Graph Engineering** proveen fundamentación fáctica y estructura de conocimiento.
- **Loop Engineering** sostiene la continuidad temporal de la ejecución autónoma.
- **Harness Engineering** envuelve a todas las anteriores en infraestructura real: sandboxes, herramientas, observabilidad, gobernanza.
- **Eval Engineering** mide, con rigor, si cualquiera de las anteriores está funcionando de verdad.

Detalle completo, con diagrama, en [docs/00-overview.md](docs/00-overview.md).

## Qué hace a esto un *estándar* y no solo una lista

Cada disciplina tiene tres capas obligatorias, no opcionales:

1. **El Estándar** — un checklist de prácticas concretas, accionables, verificables. No "mejores prácticas" vagas — cosas que se pueden marcar como hechas o no hechas.
2. **Indicadores de Madurez (0-4)** — un modelo tipo CMM adaptado a cada disciplina, con criterios explícitos por nivel y un **KPI ancla** cuantitativo.
3. **Modos de Falla** — los anti-patrones documentados de la disciplina, para reconocerlos antes de que aparezcan en producción propia.

Los recursos curados (papers, frameworks, herramientas) vienen *después* de estas tres capas, no en lugar de ellas.

## Estructura del repositorio

```
awesome-ai-engineering/
├── README.md                          ← estás acá
├── docs/
│   ├── 00-overview.md                 ← panorama sistémico y diagrama de relaciones
│   ├── 01-prompt-engineering.md
│   ├── 02-flow-engineering.md
│   ├── 03-context-engineering.md
│   ├── 04-rag-engineering.md
│   ├── 05-harness-engineering.md
│   ├── 06-loop-engineering.md
│   ├── 07-graph-engineering.md
│   └── 08-eval-engineering.md
├── maturity-model/
│   └── MATURITY-MATRIX.md             ← estándar transversal de indicadores 0-4 + KPIs
├── resources/
│   └── glossary.md
├── CONTRIBUTING.md                    ← criterios de curación
└── LICENSE                            ← CC0
```

## Contribuir

Ver [CONTRIBUTING.md](CONTRIBUTING.md) — hay criterios distintos para agregar un link curado versus proponer un cambio al Estándar o a la Matriz de Madurez. No es un "PRs welcome" genérico: el valor de este repo es la curación, no el volumen.

## Licencia

[CC0 1.0](LICENSE) — dominio público. Los recursos linkeados conservan sus propias licencias originales.
