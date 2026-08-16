# Auditoría UX/UI profunda — la capa de interacción vs. la lógica que construimos

**Fecha:** 16-ago-2026
**Por qué esta auditoría:** la lógica interna de Archivo evolucionó mucho (ADN 2.0, ranking propio, personas) pero la capa de interacción no se tocó en el mismo período. Diego pidió evaluarla con criterio de producto — no una ronda de bugs — usando la metodología de la skill `ui-ux-pro-max` (checklist de accesibilidad/touch/performance/layout/navegación) sobre mobile Y desktop por igual, porque el problema de fondo es lenguaje de producto, no solo responsive.
**Pregunta guía:** ¿Archivo se siente como una app premium de conocimiento cultural personal, o como una web con elementos peleando por espacio?
**Método:** cada hallazgo de abajo está verificado con evidencia real (capturas de Diego + inspección en vivo de esta sesión — código, DOM, computed styles), no supuestos. Donde algo parecía un bug y resultó ser un artefacto de la herramienta de testing, se lo descarta explícitamente en vez de reportarlo.

---

## Hallazgos, priorizados por impacto en la experiencia — no por facilidad técnica

### 1. El hero de Inicio no integra el texto con la imagen — "cartel", no editorial (ALTO, mobile + desktop)

El bloque "Buenas tardes, Pipo — Última entrada · N° 2181" es una caja con fondo `rgba(0,0,0,.6)` flotando sobre la imagen, no un degradado que nace de la propia foto. Técnicamente el contraste pasa (60% negro está dentro del rango recomendado), pero visualmente es una caja aislada sobre una foto de alto contraste — exactamente el "cartel sobre imagen" que señalaste, confirmado en ambas plataformas, no un problema de mobile. Un tratamiento de scrim de degradado a pantalla completa (oscurece gradualmente desde donde arranca el texto hasta el borde inferior de la imagen) es el patrón que usan Netflix/Apple TV/Letterboxd para esto — el texto pasa a sentirse parte de la imagen, no pegado encima.

### 2. El layout se rompe con pinch-zoom — pero la solución no es bloquear el zoom (ALTO, mobile)

Confirmado con tu captura: al hacer zoom, aparece scroll horizontal y texto cortado ("...NTES" en vez de "ENTRADAS RECIENTES"). Verifiqué el código: el `viewport meta` de Archivo **ya está bien configurado** (`width=device-width, initial-scale=1, viewport-fit=cover`, sin bloquear el zoom) — y así debe quedar: bloquear el zoom viola una guía de accesibilidad explícita (WCAG exige que el contenido pueda ampliarse hasta 200% para usuarios con baja visión) y es justamente lo que la propia metodología usada en esta auditoría marca como "nunca hacer". El problema real no es que se pueda hacer zoom — es que el layout no reacomoda bien cuando se hace. La solución correcta es más grande de lo que parece: revisar qué contenedores tienen anchos fijos en vez de fluidos, y probar el layout completo con zoom real, no solo en viewport normal.

### 3. Puntitos de estado: prácticamente invisibles y sin alternativa accesible (MEDIO-ALTO, toda la app)

Verificado en código y en vivo: el punto de estado en cada card mide **7×7px**, dos de los tres estados usan grises muy parecidos entre sí (`--paper2` para "Visto", `--paper4` para "Pendiente" — nada que los distinga salvo tono), no tiene `aria-label`, y el único texto explicativo es un atributo `title` que **no funciona en touch** (no hay hover en mobile) y que la mayoría de lectores de pantalla no anuncia. Es exactamente el patrón que la auditoría marca como "transmitir información solo por color, sin texto/ícono" — vos mismo dijiste que "sabés técnicamente qué son pero no aportan visualmente", y ahora hay una causa concreta: tamaño + contraste de color + falta de alternativa textual, los tres a la vez.

### 4. Barra de Safari + header + bottom nav compiten por espacio (MEDIO, mobile, solo Safari sin instalar)

Confirmado con tu captura: en Safari (no en PWA instalada), la barra de direcciones flotante se superpone al contenido y al bottom nav de Archivo. Esto es comportamiento nativo de Safari sobre el que Archivo no tiene control total, pero sí se puede mejorar cuánto "pisa" mediante el `safe-area-inset-bottom` — ya se usa parcialmente (Bloque O), vale la pena revisar si alcanza para este caso puntual una vez que se audite con el dispositivo real.

### 5. Notas y Reacciones — dos formas de decir "qué te pareció", sin conexión visual (MEDIO, producto)

La ficha de detalle tiene un campo "Notas" (textarea libre, uso real: 1 de 2181 ítems) y, por separado, el prompt de "reacciones" (Bloque S, chips como "Me sorprendió") que aparece condicionalmente después de calificar. Son dos mecanismos con el mismo propósito de fondo (capturar qué pensaste) que el usuario nunca ve relacionados entre sí — uno vive en el formulario, el otro aparece como un toast aparte. No proponemos unificarlos sin evidencia de qué funciona mejor, pero vale la pena que sepas que hoy conviven sin que la interfaz explique por qué hay dos.

### 6. "0 Pendientes" ocupa el mismo espacio visual que datos reales (BAJO-MEDIO, Inicio, mobile + desktop)

La tarjeta "Pendientes" en "En números" pesa igual que Películas/Series/Libros/Este año, pero con tu historial 100% "Visto" hasta hace muy poco, hoy y por un tiempo va a mostrar 0 — un dato que no dice nada todavía. No es un error, es una oportunidad de simplificar: capaz no necesita el mismo peso visual que los otros mientras no tenga algo que contar.

### 7. "No es esta" no se entiende fuera de contexto (BAJO, ficha de edición)

El botón que te deja corregir un match equivocado de búsqueda se llama "No es esta" sin ningún ícono o texto de apoyo — para alguien que nunca tuvo un error de búsqueda, no queda claro qué hace. Caso de uso poco frecuente, pero fácil de aclarar.

### 8. Densidad de las explicaciones en Descubrí — observación, no un error (BAJO, a monitorear)

Las explicaciones de Bloque X ("Por tu afinidad con X — solés puntuar alto Y y te funcionan las películas de Z") son exactamente lo que pediste, y funcionan bien en la card destacada. En las filas compactas de la lista, esa misma frase completa compite por espacio con título/año/rating/botón de agregar — no es un problema hoy, pero si en algún momento se siente pesado de leer al scrollear rápido, ya sabemos por qué.

---

## Verificado y descartado explícitamente (para que no se pierda como duda)

- **Poster faltante en la card destacada de Descubrí (mobile):** una captura mostró un espacio vacío grande donde debía estar el póster. Verificado en el DOM: la imagen cargó bien (`complete:true`, tamaño correcto, visible, opacidad 1). Es un artefacto de la herramienta de testing de esta sesión (ya documentado varias veces en este proyecto), no un bug real de Archivo.
- **Viewport meta / z-index / safe-area:** revisados contra el checklist de la skill — z-index tiene una escala razonablemente ordenada, `safe-area-inset` ya se usa en header/bottomnav/toast, no hay patrón `100vh` (el bug que sí afectó a FitCoach no está presente acá).

---

## Propuesta de bloques de implementación (sin implementar todavía)

**Bloque Y — Hero y scrim editorial** (hallazgo #1): rediseñar el tratamiento de texto-sobre-imagen del hero de Inicio con un degradado real, no una caja aislada. Afecta estructura visual central — diseñar antes de tocar código, como pediste.

**Bloque Z — Estabilidad bajo zoom** (hallazgo #2): auditar qué contenedores no reflowan bien, corregir sin tocar el viewport meta (que ya está bien). Necesita probarse con zoom real, no solo con viewport normal — puede requerir acceso a tu dispositivo o una sesión de validación conjunta.

**Bloque AA — Estados legibles** (hallazgo #3): rediseñar el indicador de estado — más grande, con más contraste entre variantes, y una alternativa que no dependa solo de color (ícono o texto corto). Bajo riesgo, no toca datos.

Los hallazgos #4-8 son más chicos — se pueden resolver dentro de los bloques de arriba o como ajustes puntuales, sin ameritar bloques propios.

**Antes de abrir cualquiera de estos tres bloques, quiero tu confirmación sobre el orden y el alcance — no avanzo solo, como pediste para todo lo que afecte estructura.**
