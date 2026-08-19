# Contribuir

Este repositorio combina dos cosas que normalmente viven separadas: una lista curada estilo *awesome* y un estándar de ingeniería con indicadores de madurez. Las dos partes tienen criterios de aceptación distintos.

## Criterios para agregar un recurso curado

Un link entra a la sección "Recursos Curados" de una disciplina solo si cumple **todos** estos puntos:

- [ ] Es una herramienta, framework, paper o guía que existe y funciona hoy — no un anuncio, no un roadmap, no un "próximamente".
- [ ] Corresponde inequívocamente a la disciplina donde se lo agrega. Si aplica a dos, se linkea desde ambas páginas en vez de forzarlo en una sola.
- [ ] Tiene evidencia de uso real (estrellas de GitHub, adopción documentada, o un paper con implementación de referencia) — no autopromoción sin trayectoria.
- [ ] El link va directo al recurso, no a una landing page de marketing genérica sobre el recurso.

No se aceptan:
- Prompts o "trucos" sin ningún componente de ingeniería reproducible.
- Herramientas de pago sin capa gratuita/open-source funcional, salvo que sean el estándar de facto de la industria sin alternativa comparable.
- Contenido duplicado de otra lista *awesome* sin agregar contexto propio de por qué corresponde a esta disciplina específica.

## Criterios para modificar el Estándar o la Matriz de Madurez

Estos son los documentos de mayor peso del repositorio — cambiar un checklist de "El Estándar" o una fila de la [Matriz de Madurez](maturity-model/MATURITY-MATRIX.md) no es agregar un link, es cambiar el criterio con el que alguien va a auditar su propio sistema. Antes de proponer un cambio:

1. Justificar con evidencia (paper, benchmark, incidente documentado) por qué la práctica actual está incompleta o incorrecta.
2. Proponer la redacción exacta del cambio, no solo la idea general.
3. Verificar que no rompe la relación de inclusión escalar descripta en [00 · Panorama Sistémico](docs/00-overview.md) — un cambio en una disciplina no debería contradecir el estándar de otra.

## Estilo

- Prosa técnica directa, sin relleno. Si una oración no agrega información nueva, se borra.
- Tablas para comparaciones, checklists (`- [ ]`) para estándares accionables, prosa para definiciones y contexto.
- Cada página de disciplina sigue la misma estructura fija (ver cualquier página en `docs/` como plantilla): Definición → Cuándo usarla/cuándo no → El Estándar → Indicadores de Madurez → Modos de Falla → Recursos Curados → Ver también. No romper el orden — es lo que hace que el repo sea navegable como referencia, no solo legible como blog.
- Links verificados antes de mergear — un link roto en un repo de referencia técnica destruye la confianza en todo lo demás.
