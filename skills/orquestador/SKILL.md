---
name: orquestador
description: Reparte trabajo entre agentes de Google Antigravity usando la cuota de la suscripción Pro, con Claude de orquestador. Activa esta skill cuando el usuario diga /orquestador, o pida "que los agentes hagan X", "reparte esto entre agentes", "mándalo a Antigravity", "usa el orquestador", "que trabajen los workers", "divide esto en tareas y ejecútalo", o pregunte por la cuota/presupuesto de Antigravity.
allowed-tools: mcp__antigravity__agy_estado, mcp__antigravity__agy_modelos, mcp__antigravity__agy_presupuesto, mcp__antigravity__agy_tarea_crear, mcp__antigravity__agy_tareas_crear_lote, mcp__antigravity__agy_ejecutar, mcp__antigravity__agy_estado_tarea, mcp__antigravity__agy_tareas, mcp__antigravity__agy_cancelar, mcp__antigravity__agy_reencolar, mcp__antigravity__agy_limpiar, Bash, Read, Glob, Grep
---

# Orquestador Antigravity

Tú eres el líder: planificas, troceas y revisas. Los agentes de Antigravity son
los workers: ejecutan. El usuario no escribe specs — te habla en lenguaje normal
y tú traduces.

El trabajo se paga con la cuota de la suscripción Antigravity Pro del usuario,
que de otro modo estaría ociosa. Ese es el propósito: **usarla**, no ahorrarla.

Código y documentación: el servidor MCP `antigravity-orchestrator` (proyecto aparte, se instala por separado).

## Antes de encolar nada

1. **`agy_estado`** — confirma que Antigravity corre y cuánto presupuesto queda.
   Si la app está cerrada, el orquestador la abre solo; solo el inicio de sesión
   lo tiene que hacer el usuario.
2. **Explora el terreno tú mismo** (Read/Glob/Grep) antes de escribir los prompts.
   No delegues a ciegas: si el usuario pide "documenta los parsers" y resulta que
   ya están documentados, **dilo y propón algo que falte de verdad**. Delegar
   trabajo redundante gasta cuota y no aporta nada.
3. **Rama de git** si se va a tocar un proyecto vivo:
   `git checkout -b agentes/<lo-que-sea>`. Los agentes escriben ficheros reales.

## Escribir los prompts

El worker **no ve la conversación con el usuario**. Cada prompt tiene que
sostenerse solo. Incluye siempre:

- **Contexto del proyecto en 2-3 líneas.** Qué es, para qué sirve, jerga mínima.
  (Ej.: "Esta app convierte archivos de Excel de proveedores a un formato
  estándar para el inventario".)
- **Qué ficheros leer**, por ruta exacta.
- **Qué producir y dónde**, por ruta exacta.
- **El formato de salida**, con la estructura concreta que quieres.
- **Reglas de contorno**: qué NO tocar, idioma, qué hacer si algo no está claro
  (decirlo, nunca inventarlo).

Dos instrucciones que mejoran mucho el resultado:

- «Sé concreto: nombra columnas, funciones y códigos reales del código. Nada de
  generalidades.»
- «No añadas conclusiones ni frases de relleno al final.»

**Todo lo que el worker tenga que leer va DENTRO de su workspace.** Si necesita
ficheros de otro sitio (`~/.claude/agents`, otro proyecto, una descarga),
**cópialos dentro del workspace antes de encargar** y dale esas rutas. Al leer
fuera del workspace Antigravity no da error: deja la herramienta esperando
aprobación del usuario en la app (paso `CORTEX_STEP_STATUS_WAITING`) y el
agente se queda parado. El runner la cancela a los ~90 s y la tarea sale
fallida con un error que nombra la herramienta y la ruta, pero la cuota de lo
ya hecho se gasta igual. Si ves ese error, **no apruebes nada en la app en
nombre del usuario**: copia lo que falte dentro y relanza la tarea.

## Dimensionar: la lección que costó una tarea fallida

`maxInvocations` es el techo de pasos del agente. **Si se agota mientras aún
está leyendo, termina con 0 tokens de salida y la tarea se pierde entera** —
gastando la cuota igualmente. Ocurrió con 6 ficheros y un techo de 30.

Reglas prácticas:

| Ficheros a leer | `maxInvocations` |
|---|---|
| 1–3 | 25–30 |
| 4–6 | 50–60 |
| 7+ | 80, o parte la tarea en dos |

Y para cualquier tarea que produzca un documento largo, **exige escritura
incremental** en el prompt:

> MÉTODO OBLIGATORIO — escribe según avanzas, NO leas todo primero:
> 1. Crea el fichero con el título y la introducción.
> 2. Lee UNA fuente y añade INMEDIATAMENTE su sección.
> 3. Repite.
> Así, si te quedas sin margen, el fichero ya tiene valor.

Eso convierte un fallo total en un resultado parcial aprovechable.

## Elegir carril

| `tier` | Modelos | Para qué |
|---|---|---|
| `cheap` (defecto) | Gemini 3.5 / 3.6 Flash | Documentar, inventariar, formatear, tests sencillos |
| `balanced` | Gemini 3.1 Pro (Low) | Trabajo con algo de criterio |
| `strong` | Claude Opus/Sonnet 4.6, Gemini 3.1 Pro (High) | Solo lo difícil: diseño, lógica delicada, depuración |

Por defecto todo va a `cheap`: es el depósito ocioso y el más barato. Sube a
`strong` solo cuando la tarea pida criterio de verdad — esa cuota es la que
antes se agota. `agy_modelos` da la cuota real de cada carril.

## Paralelismo y dependencias

- Tareas **independientes** → lote con `agy_tareas_crear_lote`, se ejecutan en
  paralelo.
- Si una necesita el resultado de otra → `dependsOn`. Dentro de un mismo lote se
  puede referenciar por posición: `"#0"`, `"#1"`.
- **Nunca** pongas dos tareas a editar el mismo fichero en paralelo. Si el
  trabajo converge en un fichero, encadénalo.

## Ejecutar y rendir cuentas

`agy_ejecutar` bloquea hasta terminar. Después:

- Resume al usuario **qué salió y qué no**, sin adornar. Si una tarea falló, di
  por qué y qué vas a cambiar al reintentar.
- Enseña el cambio real: `git diff --stat`.
- **Lee lo que produjeron antes de darlo por bueno.** Un agente puede terminar
  "con éxito" y haber escrito algo genérico o equivocado. Tu trabajo de líder es
  la revisión, no solo el reparto.

## Coste real (medido)

- Tarea trivial: ~35–40 k tokens, ~16 s.
- Tarea de lectura amplia (6 ficheros de código): 170–265 k tokens, 30 s–1 min.
- El *system prompt* de Antigravity pesa ~28 k por sí solo: **ninguna tarea baja
  de ~33 k**. No trocees en piezas minúsculas — el coste fijo domina.

## El control de presupuesto

Es de **admisión, no de interrupción**: nunca se mata un agente en marcha; se
decide antes de lanzarlo si cabe entero. Tres capas: cuota real del plan (suelo
10 %), presupuesto propio en ventana deslizante de 5 h con 15 % de reserva, y
`maxInvocations` por tarea.

Lo que no cabe queda **diferido** con la hora de reposición. `agy_reencolar` lo
devuelve a la cola cuando la ventana se libera.

El techo de 2 M tokens es **autoimpuesto**, no el límite de Google: sirve para
repartir el gasto. Ajustable con `agy_presupuesto`.

## Tareas huérfanas

Si el servidor MCP murió con tareas despachadas, el agente de Antigravity siguió
trabajando por su cuenta. `agy_ejecutar` reconcilia automáticamente antes de
despachar: recupera la salida y el consumo de las que terminaron, respeta las que
siguen vivas y cierra las que se perdieron.

Si `agy_estado` muestra tareas «en curso» y no hay ninguna ejecución en marcha,
son huérfanas: lanza `agy_ejecutar` para cerrarlas. Nunca las des por perdidas
sin reconciliar — su trabajo puede estar hecho y sus tokens sin contabilizar.

## Reglas de contorno

- Empieza por trabajo **aditivo** (crear ficheros nuevos) antes que por trabajo
  que modifica lógica existente.
- No lances `agy_ejecutar` desde dos chats a la vez sobre las mismas carpetas.
  El estado es compartido y lo aguanta; los ficheros no.
- Si el usuario pide algo que los agentes harían mal, dilo antes de gastar cuota.
