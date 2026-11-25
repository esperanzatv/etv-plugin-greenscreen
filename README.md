# Esperanza TV · Equipo de edición

Plugin para editar cada tema del programa de Esperanza TV (pastor en chroma, 3 cámaras, set virtual con TV) con un equipo de agentes.

## Cómo usarlo
Pide, por ejemplo: "Edita el Tema 6". El coordinador te hará las preguntas iniciales (clips de cada cámara, set, referencia de color o estilos, imágenes de la TV, audio) y repartirá el trabajo.

## Componentes
- Skill `editar-tema`: el coordinador y el método completo (referencia `metodo-estudio-virtual.md`).
- Agentes: `sincronia-cortes`, `guion-visual`, `generador-imagenes`, `fondos-perspectiva`, `montaje-premiere` (solo uno a la vez), `control-calidad`.

## Requisitos
- Disco "Extreme Pro" con la carpeta AMAYA conectada.
- Adobe Premiere Pro 2027 y control del computador habilitado en la app de escritorio.
- Plugin Claude Banana con la clave de Gemini: variable de entorno `GEMINI_API_KEY` (recomendado) o el archivo `gemini-image-gen.local.md` en formato frontmatter.

## Publicar una versión nueva
1. Hacer el cambio en una rama y abrir un pull request.
2. Subir el número `version` en `.claude-plugin/plugin.json` (sin esto no llega la actualización).
3. Revisar y fusionar en `main`.

## Instalar
En Claude: Customize → Plugins → Add → Add marketplace, escribir `usuario/esperanza-tv-plugin`, instalar **esperanza-tv** y activar la sincronización automática.
