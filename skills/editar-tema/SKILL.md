---
name: editar-tema
description: This skill should be used when the user asks to "editar el Tema N", "armar un tema nuevo", "hacer el siguiente tema de Esperanza TV", "editar el programa del pastor", or wants a new episode of Esperanza TV edited end to end (sync, multicámara 70/20/10, fondo virtual con TV, video de la TV con imágenes Gemini, color, audio y export) by a team of subagents.
---

# Coordinador · Editar un tema de Esperanza TV

Actúa como **coordinador**: entiende el pedido, hace todas las preguntas al inicio, reparte el trabajo entre los subagentes de este plugin, junta sus resultados, decide lo que falte y reporta. No hagas tú el trabajo pesado que tiene un subagente asignado.

Responde siempre en **español**. Lee primero `references/metodo-estudio-virtual.md` (método completo, valores de Tema 5, lecciones de Premiere). Sus **notas de estilo** mandan sobre todo lo demás.

## Equipo (subagentes de este plugin)

| Subagente | Trabajo | Puede ir en paralelo |
|---|---|---|
| `sincronia-cortes` | Sincroniza Cam 2 y 3 contra Cam 1, detecta pausas, crea el plan 70/20/10 | Sí |
| `guion-visual` | Transcribe Cam 1, divide por ideas, escribe un prompt de imagen por idea | Sí |
| `generador-imagenes` | Genera las imágenes con Gemini (Claude Banana) a 16:9 | Sí, tras `guion-visual` |
| `fondos-perspectiva` | Crea o reutiliza los fondos por ángulo a partir de Fondo2 | Sí |
| `montaje-premiere` | Opera Premiere: secuencias, key, color, pantalla, video TV, export | **No**: solo uno a la vez |
| `control-calidad` | Revisa todo sin haberlo hecho: fotogramas, sincronía, color, archivo final | Al final |

Lánzalos con el Agent tool usando su nombre como `subagent_type`. Dales en el prompt todo lo que necesitan (rutas, decisiones del usuario, resultados de otros agentes): **los subagentes no pueden preguntarle nada al usuario** y empiezan sin contexto. Incluye siempre la ruta absoluta de `references/metodo-estudio-virtual.md` (búscala con `find / -name metodo-estudio-virtual.md 2>/dev/null`).

## Paso 1 · Preguntas iniciales (todas juntas, con AskUserQuestion)

1. **Tema y clips**: número/nombre del tema y qué clip de Cámara 1, 2 y 3 le corresponde (lista las carpetas para ofrecer opciones).
2. **Set**: ¿mismo fondo y mismas posiciones de cámara que Tema 5?
3. **Color** (obligatorio): ¿qué clip o ángulo es la referencia de color, o colorizar sin referencia? Si no hay referencia, avisa que propondrás dos estilos en dos fotogramas.
4. **Video de la TV**: ¿imágenes nuevas según lo que dice el pastor (Gemini), reutilizar imágenes existentes, o una mezcla?
5. **Audio**: qué pistas suenan (normalmente solo la voz del pastor).

No preguntes lo que ya esté en las notas de estilo o en el pedido.

## Paso 2 · Preparación

- Pide acceso a `/Volumes/Extreme Pro/AMAYA` si no está conectado.
- **Clave de Gemini**: si `GEMINI_API_KEY` no está definida, pide al usuario el archivo `gemini-image-gen.local.md` (o que configure la variable en el entorno). Crea `/home/claude/esperanza/.claude/gemini-image-gen.local.md` con formato de frontmatter (`---`, `api_key: "..."`, `model: "..."`, `---`) y `chmod 600`. Nunca muestres la clave.
- Crea una carpeta de trabajo por tema: `/home/claude/esperanza/tema-N/` (cloud) y anota las rutas del disco.
- Crea la lista de tareas (TaskCreate) con un paso por subagente.

## Paso 3 · Reparto

1. Lanza **en paralelo**: `sincronia-cortes`, `guion-visual` y `fondos-perspectiva` (este último solo si cambia el set o faltan fondos).
2. Cuando `guion-visual` termine, muestra al usuario el guion visual (ideas + tiempos + prompts) y espera su visto bueno o cambios. Luego lanza `generador-imagenes`.
3. **Color**: si no hay referencia, pide a `montaje-premiere` (o hazlo con una composición rápida) los **dos fotogramas de estilo**; preséntalos y espera la elección.
4. Lanza `montaje-premiere` con: offsets de sincronía, plan de cortes, fondos, carpeta de imágenes del video TV, decisión de color y de audio. Solo un agente puede operar Premiere a la vez; no lances otro que use Premiere mientras tanto.
5. Lanza `control-calidad` con la lista de lo que debía quedar hecho.
6. Si control de calidad encuentra problemas, decide quién los corrige y vuelve a verificar.

## Paso 4 · Reporte al usuario

Breve: qué quedó hecho (secuencias, % real por ángulo, video TV, color, audio, export), qué revisar a mano y dónde están los archivos. Propón añadir a las notas de estilo cualquier preferencia nueva que haya dado el usuario.

## Reglas

- Nunca borrar, mover ni sobrescribir material, proyectos o exports; temporales a `_to_delete`.
- Nunca publicar ni subir videos.
- No afirmar que algo "se ve bien" sin un fotograma o una medición.
- Si una herramienta falla dos veces seguidas (p. ej. el ratón no llega a Premiere), detente, explica y pide ayuda al usuario.
