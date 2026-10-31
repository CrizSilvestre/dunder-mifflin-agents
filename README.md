# Dunder Mifflin × Pied Piper: agentes y skills para Claude Code

Un equipo de 15 subagentes para [Claude Code](https://claude.com/claude-code), cada uno con
una personalidad de *The Office* o *Silicon Valley* y un trabajo concreto, más tres skills
propias. Están escritos en español y pensados para trabajar en equipo: uno diseña, otro
programa, otro prueba, otro revisa y otro cuida lo que se publica.

## Los agentes

| Agente | Para qué |
|---|---|
| `richard` | Arquitecto: diseña la solución y parte el trabajo **antes** de construir algo grande. Entrega un plan, no código. |
| `jim` | Desarrollador pragmático: la solución más simple que funcione, sin sobreingeniería. |
| `dinesh` | Pruebas: escribe y corre tests, reproduce bugs con una prueba que falle. |
| `dwight` | Revisor implacable de código y calidad. Solo lee y ejecuta pruebas. |
| `pam` | Diseño e interfaces, correos HTML compatibles con Outlook, textos de UI. |
| `gilfoyle` | Infraestructura: servidores locales, puertos, despliegues, rendimiento. |
| `jared` | Gestión de proyecto y documentación: READMEs, manuales, pendientes. |
| `monica` | Estrategia y segunda opinión: qué puede salir mal y cuánto cuesta de verdad. |
| `ryan` | Investigación en la web: herramientas, repos y competidores, con fuentes. |
| `oscar` | Análisis de datos y estadística; detecta cuando una cifra no cuadra. |
| `angela` | Guardiana del formato: normas, APA 7, ortografía, consistencia. |
| `erlich` | Marketing y contenido: títulos, descripciones, guiones, posicionamiento. |
| `toby` | Cumplimiento y seguridad antes de publicar: secretos, datos personales, permisos. |
| `kevin` | Tareas sencillas y mecánicas, para gastar poco. |
| `bighead` | Enlace con agentes de Google Antigravity (necesita la skill `orquestador`). |

## Las skills

| Skill | Qué hace |
|---|---|
| `abogado-del-diablo` | Critica una idea, plan o proyecto sin adular: asume que va a fracasar y lo demuestra por ocho ángulos (premisas, mercado, competencia, viabilidad, números, ejecución, pre-mortem y punto ciego), con un veredicto y una lista priorizada de qué arreglar. |
| `neo` | Agente de productos digitales (marca «Papel con Cariño»): investiga qué vender, espía anuncios en la Biblioteca de Meta, fabrica el PDF, hace imágenes y videos y calcula la rentabilidad. Supone un proyecto de trabajo en `~/neo`. |
| `orquestador` | Reparte trabajo entre agentes de Google Antigravity con Claude como orquestador, usando la cuota de una suscripción. Necesita el servidor MCP `antigravity-orchestrator`, que se instala aparte. |

## Instalación

Copia los agentes y las skills a tu carpeta de Claude Code:

```bash
git clone https://github.com/CrizSilvestre/dunder-mifflin-agents.git
cp dunder-mifflin-agents/agents/*.md ~/.claude/agents/
cp -R dunder-mifflin-agents/skills/* ~/.claude/skills/
```

O solo los que quieras, uno por uno. Al abrir Claude Code aparecen en la lista de agentes
(`/agents`) y de skills. Se invocan por su nombre, por ejemplo: «que Dwight revise este cambio»
o `/abogado-del-diablo`.

## Cómo se usan juntos

Un flujo típico para un cambio importante:

1. `richard` propone el diseño y los pasos.
2. `jim` lo implementa; `pam` se encarga de lo visual.
3. `dinesh` escribe las pruebas.
4. `dwight` revisa en busca de fallos.
5. `toby` revisa antes de hacer push o publicar.

`monica` y `abogado-del-diablo` sirven antes de empezar, para decidir si vale la pena.

## Notas

- Algunos agentes mencionan «las reglas de la casa»: son las que cada proyecto define en
  su `CLAUDE.md`.
- `gilfoyle` incluye trucos específicos de macOS (TCC, `sudo`); en otros sistemas se ignoran.
- Los personajes son un homenaje a *The Office* y *Silicon Valley*; no hay afiliación.

## Licencia

MIT. Hecho por Criz Mervin Silvestre Sarmiento (CSilvestre).
