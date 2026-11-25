# Esperanza TV · Estudio virtual en Premiere

Flujo para editar cada "Tema" del programa del pastor grabado en chroma con 3 cámaras, compuesto sobre un set virtual (pared de ladrillo, TV, ventana con ciudad de noche). Nació de la edición de Tema 5 (23-nov-2026) y toma lo mejor de la skill "Eddie" (notas de estilo, valores de audio, orden de color, checklist de export, reglas de seguridad).

Responde siempre en **español**.

## Notas de estilo (mandan sobre todo lo demás)

Cuando el usuario corrija o pida algo de forma estable, propón añadirlo aquí como una línea fechada.

- Idioma de respuesta: español.
- Multicámara: **70 % Ángulo 1 (general), 20 % Ángulo 2 (medio frontal), 10 % Ángulo 3 (3/4)**. Ángulo 1 abre y cierra; los otros entran como inserts cortos y naturales.
- Cortes siempre en pausas naturales del pastor, nunca a mitad de frase.
- Ángulo 3: el pastor **justificado a la izquierda**; puede tapar un poco la TV.
- Color: **siempre preguntar antes de colorizar** cuál clip o ángulo es la referencia de color, o si se colorea todo sin referencia. Nunca asumir la referencia de un tema anterior. (En Tema 5 la referencia fue el Ángulo 3.)
- Si el usuario no indica clip de referencia: proponer **dos estilos**, cada uno mostrado en un **fotograma de muestra**, y esperar a que elija antes de aplicar. (23-nov-2026)
- El pastor debe verse integrado a la luz cálida del set (ladrillo, apliques, ambiente nocturno).
- Exportaciones: `<carpeta del proyecto>/EXPORTADOS`.
- Nunca borrar ni sobrescribir material, proyectos o exports: versiones nuevas (v1, v2, v3).

## Dónde están las cosas

- No se requiere ningún disco ni nombre de carpeta en particular. Al empezar, pregunta cuál es la **carpeta del proyecto** (`<carpeta del proyecto>`) y pide acceso a ella con device_request_folder_access.
- Lista su contenido y confirma con el usuario dónde están los clips de las 3 cámaras (en Tema 5: 4K, 59.94 fps; Cam 2 en 4:2:2 10-bit), el proyecto `.prproj`, las imágenes del video de la TV y los fondos.
- `Fondo2.PNG` es el set base; `Fondo2_CAM2.png` y `Fondo2_CAM3.png` son los fondos por perspectiva. Si no aparecen, pídeselos al usuario.
- Exporta a `<carpeta del proyecto>/EXPORTADOS` (créala si no existe).
- Por cada tema pregunta: qué clip de cada cámara corresponde y si el set y las posiciones de cámara son los mismos de Tema 5.

## Flujo por tema

1. **Inventario**: lista los clips del tema, `ffprobe` (resolución, fps, duración, timecode).
1b. **Pregunta de color (obligatoria)**: antes de tocar el color pregunta: «¿Qué clip o ángulo usamos como referencia de color, o lo coloreo completo sin referencia?». Ver la sección «Referencia de color y estilos».
2. **Sincronía**: correlación del audio (envolvente a 100 Hz–1 kHz) de Cam 2 y Cam 3 contra Cam 1. En Tema 5: Cam 2 = Cam 1 − 1.499 s; Cam 3 = Cam 1. Verifica siempre en dos tramos distintos del clip.
3. **Pausas**: `silencedetect=noise=-32dB:d=0.35` sobre el audio de Cam 1 → candidatos de corte = fin de pausa − 0.12 s, redondeado a 24 fps.
4. **Plan de cortes**: bloques de Ángulo 1 de 11–27 s; inserts de Ángulo 2 de 6–11 s y de Ángulo 3 de 5–9 s; nunca tres inserts iguales seguidos; Ángulo 2 y 3 aparecen ya en los primeros 3–4 min. Busca la semilla aleatoria que más se acerque a 70/20/10 y reporta el % real.
5. **Secuencia**: genera un FCP XML (secuencia 1920x1080, 23.976/24 fps) con 3 secuencias anidadas (un ángulo cada una: V1 fondo, V2 video TV, V3 cámara) + secuencia principal con los cortes. Impórtalo con Cmd+I.
6. **Efectos**: **el XML pierde Ultra Key, Corner Pin y Gaussian Blur** (solo conserva Motion y Lumetri). Por eso:
   - Mejor camino: usar las anidadas de un tema anterior como plantilla y cambiar solo el material con **Replace Footage**.
   - Si hace falta copiar: en la anidada origen **bloquea V2**, Cmd+A, Cmd+C, desbloquea; en la destino bloquea V2, Cmd+A, Supr, Inicio, Cmd+V, desbloquea. Luego aplica el Corner Pin y el blur al clip de la TV.
7. **Verifica** fotogramas en el monitor de programa en 3–4 tiempos (al menos uno de cada ángulo) y guarda con Cmd+S.

## Valores de Tema 5 (punto de partida)

**Ángulo 1 – General (Cam 1)**: cámara Scale 64, Position 1390/568, Crop Left 24 %. Fondo2 escala 114.833. Corner Pin TV: UL 198/151 · UR 1055/227 · LL 198/658 · LR 1055/645.

**Ángulo 2 – Medio frontal (Cam 2)**: Scale 65, Position 1085/620. Fondo `Fondo2_CAM2.png` (acercamiento ×1.6 + blur 3 de lente larga). Corner Pin TV: UL −588/146 · UR 777/268 · LL −588/953 · LR 777/933 · Gaussian Blur 3.

**Ángulo 3 – 3/4 (Cam 3)**: Scale 50, Position 767/540 (pastor a la izquierda). Fondo `Fondo2_CAM3.png` (homografía de 15°, zoom 1.25, columna de ladrillo extendida, blur 3.5). Corner Pin TV: UL 403/144 · UR 1417/170 · LL 403/712 · LR 1417/718 · Gaussian Blur 3.

**Key (Ultra Key, igual en los 3)**: Key Color con gotero sobre el verde; Setting Custom; Transparency 45, Highlight 10, Shadow 45, Tolerance 90, Pedestal 100; Matte Cleanup Choke 12, Soften 25. Revisa en Output = Alpha Channel antes de dejarlo en Composite.

**Grade común (Lumetri, después del key)**: Temperature +22, Tint +6, Saturation 92, Exposure −0.4, Contrast +12, Highlights −18, Shadows −8, Whites −10, Blacks −6.

**Igualación Cam 1 → referencia Cam 3** (Lumetri antes del grade): Exposure +0.65, Contrast +15, Highlights +22, Shadows +8, Whites +8, Blacks −4, Saturation 130, Tint −6; ruedas: sombras un poco a azul, luces un poco a naranja. Objetivo medido en la piel del rostro (Lab): L ≈ 52, a ≈ 26, b ≈ 28; ΔE < 2.

**Fondos por perspectiva**: se derivan de Fondo2 sin cambiar la escenografía (homografía y zoom con OpenCV). Si el nuevo encuadre deja ver zonas fuera de Fondo2, rellena con textura existente (p. ej. la columna de ladrillo), nunca con objetos inventados.

## Video de la TV

- Referencia: la secuencia `Gemini_Generated_Image_…` del proyecto. Imágenes `1.jpeg`…`85.jpeg` de ~20 s cada una, con transiciones "1. Simple Left To Right" en dos pistas encima, y el audio de Cam 3 (C8526) con In en 10.75 s. Se exporta como MP4 y se usa en V2 de cada ángulo.
- **Sincronía exacta**: tiempo del video TV = tiempo de Cam 3 − 10.75 s. Con el pastor recortado a In 19.1667 s, el In del video TV debe ser **8.4167 s (202 fr a 24 fps)**. Calcúlalo así en cada tema en lugar de estimarlo a ojo.
- Meta deseada: que cada imagen corresponda a lo que el pastor dice. Flujo: transcribir Cam 1 → dividir por ideas → un prompt por idea → generar imágenes (cuando el usuario habilite el generador) → montar con cortes en los cambios de tema y las mismas transiciones.
- Transcripción: faster-whisper se instala con pip, pero el modelo viene de Hugging Face; si el proxy lo bloquea, usa Speech to Text de Premiere (panel Text) y exporta la transcripción.

## Audio (de Eddie)

- Orden: limpiar → nivelar → ecualizar/comprimir → mezclar → medir.
- Voz entre −12 y −6 dB (picos ≈ −6). Música bajo la voz −22 a −18 dB. Final ≈ **−14 LUFS** para YouTube/redes, true peak −1 dB.
- Premiere: Essential Sound → etiquetar Dialogue/Music; Repair con valores bajos (≤4); Ducking para la música. Crossfade corto en cortes con clic.
- El video de la TV no aporta audio útil (en Tema 5 estaba en silencio): silencia su pista.

## Referencia de color y estilos

1. Pregunta siempre (con AskUserQuestion si está disponible) cuál es la referencia: un ángulo concreto (1, 2 o 3), otro clip, o «colorizar sin referencia».
2. **Con referencia**: iguala las demás cámaras a ella con un Lumetri de igualación antes del grade común. Mide la piel del rostro en un mismo instante (Lab: L, a, b) y ajusta hasta ΔE < 2; reporta antes/después/referencia.
3. **Sin referencia** (o si el usuario no la indica): propone **dos estilos distintos**, cada uno en un fotograma:
   - Toma el mismo instante del plano general ya compuesto (pastor + fondo + TV).
   - Aplica a cada versión un look diferente y coherente con el set, por ejemplo: A) «Cálido nocturno»: piel cálida, ladrillo saturado, negros profundos, contraste medio-alto; B) «Natural limpio»: blancos neutros, piel natural, saturación moderada, contraste suave.
   - Entrega los dos fotogramas (lado a lado o como dos archivos) con el nombre del estilo y los valores Lumetri principales de cada uno.
   - Espera la elección; después aplícalo a los 3 ángulos e iguala las cámaras al estilo elegido.
4. Guarda en «Notas de estilo» la referencia o el estilo elegido para ese tema.

## Color (de Eddie)

- Corregir → igualar → look. La piel es la prueba en cada paso. Confía en las scopes (Waveform, RGB Parade, Vectorscope y su línea de piel), no en el ojo.
- Puntos pendientes de integración del pastor que ayudan mucho: contraluz frío del lado de la ventana, luz suave del lado de la TV, caída de luz hacia abajo, light wrap.

## Export (de Eddie)

- H.264 en MP4, 1920x1080, mismos fps que la secuencia, 10–16 Mbps, AAC 320 kbps 48 kHz, a `<carpeta del proyecto>/EXPORTADOS` con nombre `Tema N - vX.mp4`.
- Antes de exportar: sin medios offline, audio definido (qué pistas suenan), duración del fondo y la TV ≥ la del pastor.
- Después: verifica el archivo con `ffprobe`/`mdls` (resolución, duración, códec) y reporta.

## Lecciones al operar Premiere

- Preferir herramientas en segundo plano; para la línea de tiempo suele hacer falta control de pantalla completa.
- Si los clics no llegan a Premiere pero el teclado y los menús sí, una ventana invisible del sistema (p. ej. com.apple.talagent) está encima: pide al usuario guardar y **reiniciar Premiere**.
- Los atajos de "slip" no funcionaron; corrige desfases del video TV en el XML (campo `in`), no deslizando el clip.
- Escribir en un campo de búsqueda sin foco manda las letras a la línea de tiempo como atajos: haz clic en el campo y confírmalo antes de teclear.
- Tras cambios grandes revisa el panel History; Cmd+Z puede deshacer también una importación.
- Copia un efecto desde Effect Controls seleccionando su encabezado y Cmd+C; el Shift+clic para varios efectos no siempre funciona: verifica qué se pegó.

## Nunca

- Borrar, mover o sobrescribir material, proyectos o exports (los temporales van a una carpeta `_to_delete`).
- Publicar o subir videos.
- Afirmar que algo "se ve bien" sin haber revisado un fotograma o una medición real.