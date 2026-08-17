# Auditoría — Archivo como PWA real en iPhone

**Fecha:** 16-ago-2026
**Por qué esta auditoría:** hasta ahora, toda la validación de Bloques Y/AA/Z/AB/AC se hizo con datos reales pero en un entorno de pruebas automatizado (servidor local + navegador embebido, sin Safari real ni gestos táctiles reales). Diego probó Bloque AC en su iPhone real y encontró varios problemas que ese entorno no podía mostrar — no porque el código estuviera mal auditado, sino porque hay una categoría entera de comportamiento (chrome dinámico de Safari, pinch-zoom real, teclado nativo, momentum scroll) que un DOM inspeccionado en frío no reproduce. Su frase guía para esta auditoría: **"No quiero corregir síntomas aislados; quiero que auditemos Archivo como PWA real en iPhone."**
**Encuadre importante, textual de Diego:** esto no es "está todo mal" — la app avanzó muchísimo. Es el primer feedback real de "persona usando la app como app", después de varias etapas validadas en escritorio/simulador. Ahí aparecen cosas que ningún DOM auditado muestra.
**Evidencia:** 7 capturas reales de Diego en iPhone/Safari (`capturas/IMG_5728.PNG` a `IMG_5734.PNG`), analizadas una por una, cruzadas con el código fuente para diagnóstico de causa — no solo descripción de síntoma.

---

## 1. Navegación y chrome — el problema más grave, y es sistémico

### Lo que muestran las capturas

- **`IMG_5728`:** pinch-zoom real activo. El buscador de Biblioteca queda cortado ("...scar título, actor, director, aut..."), las cards se salen del borde derecho de la pantalla con scroll horizontal visible, y el pill flotante de Safari ("diepizaga.github.io") queda superpuesto sobre las cards de abajo.
- **`IMG_5729`:** scroll normal (sin zoom). El header semitransparente (`rgba(...,.92)`) queda pegado arriba, con las cards de la fila superior asomando débilmente detrás — comportamiento esperado de `position:sticky`, pero confirma que el header siempre "come" esa franja superior.
- **`IMG_5730`:** el hallazgo más revelador. Durante un scroll rápido (momentum), el header aparece **a mitad de pantalla**, con cards reales arriba y abajo de él — no pegado al tope como debería estar un `position:sticky`. Al mismo tiempo, la barra completa de Safari (atrás/pestañas/URL/refresh/menú) está desplegada abajo, **y el bottom nav de Archivo también está visible**, ambos superpuestos sobre la fila de cards inferior. Tres capas de chrome (header, bottom nav, Safari) compitiendo por el mismo espacio, dos de ellas fuera de su posición esperada.
- **`IMG_5732`, `IMG_5733`, `IMG_5734` (ADN):** el bottom nav se superpone directamente sobre el texto del insight 04, cortándolo, con el pill de Safari encima de todo.

### Diagnóstico de causa (no solo síntoma)

Hay dos problemas distintos mezclados, que Diego identificó como una sola sensación de "encajonada" pero tienen orígenes diferentes:

**(a) `position:sticky`/`fixed` no está perfectamente sincronizado durante scroll rápido en Safari real.** Esto es un comportamiento documentado de WebKit: durante momentum/inertial scrolling, el compositor de Safari puede renderizar frames donde un elemento `sticky`/`fixed` queda temporalmente desfasado de su posición real, porque el hilo de scroll y el hilo de compositing no siempre están perfectamente sincronizados cuadro a cuadro. `IMG_5730` es exactamente ese frame intermedio. No es un bug de nuestro CSS — es una limitación conocida del motor, la misma familia de problema que ya identificamos en Bloque Z (ahí era `backdrop-filter` + `fixed` durante pinch-zoom; acá es `sticky`/`fixed` durante momentum scroll, sin backdrop-filter de por medio esta vez).

**(b) El pinch-zoom sigue rompiendo la composición, y el diagnóstico anterior (Bloque Z) fue incompleto.** Bloque Z auditó correctamente que no había overflow horizontal real en reposo (medido a 320px) y sacó `backdrop-filter` de los 3 elementos `fixed`/`sticky` como la causa más probable del glitch. `IMG_5728` confirma que el problema persiste **incluso sin backdrop-filter** — así que la causa no era (solo) esa. La causa real y más profunda es más simple y más difícil de resolver con CSS: **el pinch-zoom en iOS es un zoom del *viewport visual*, no un re-layout del documento** — todo lo que está en pantalla se escala como una imagen, incluidos los elementos `fixed`/`sticky`, que dejan de estar anclados a los bordes reales de la pantalla mientras dura el gesto porque el "viewport" al que estaban anclados ahora es más chico que la pantalla física. Esto no se arregla con una regla CSS puntual — es el comportamiento nativo del navegador ante un layout con elementos fijos. Coincidimos en no bloquear zoom (accesibilidad), así que la pregunta real no es "cómo evito que esto se vea raro durante el gesto" sino **"cuántos elementos fijos tiene Archivo, y cuáles de ellos son realmente necesarios todo el tiempo"** — cuantos menos elementos fijos, menos superficie para este problema.

**(c) Chrome de Safari + chrome de Archivo no están coordinados.** Bottom nav de Archivo y toolbar de Safari aparecen/desaparecen por heurísticas independientes (la nuestra: dirección de scroll; la de Safari: la suya propia, no documentada públicamente y no controlable desde JS). `IMG_5730` muestra el peor caso: los dos visibles a la vez. No hay una API pública para sincronizar esto — es una limitación real del entorno, no algo que se resuelva con más código.

### La pregunta de fondo que trajo Diego

*"¿Realmente necesitamos header fijo siempre? ¿Tiene sentido que desaparezca también durante scroll largo? ¿Podemos tener un comportamiento más cercano a apps nativas donde el contenido gana protagonismo?"*

Esto es correcto y va a la raíz: Bloque AC resolvió el bottom nav pero dejó el header intacto — hoy Archivo tiene **dos elementos fijos activos en simultáneo** (header sticky + bottom nav contextual), cada uno con su propia superficie de riesgo frente a Safari/zoom/momentum scroll. Reducir a **un solo elemento fijo real** (o coordinar ambos bajo la misma lógica de auto-hide) reduce la superficie del problema (a) y (c) directamente, no solo la sensación de "encajonada".

---

## 2. Buscador + teclado — fricción real, causa probable identificada

### Lo que muestra la captura

`IMG_5731`: modal de búsqueda con teclado abierto. Dos cosas visibles a la vez: el título "Buscar y agregar" aparece con un desenfoque de movimiento (no un blur de CSS — no hay ningún `filter`/`backdrop-filter` en `.modal`, confirmado en el código), y el pill de Safari ("diepizaga.github.io" + botón "✕" propio de Safari) está superpuesto directamente sobre la zona donde debería estar el campo de búsqueda.

### Diagnóstico — hipótesis fundada, no confirmada al 100%

El texto desenfocado es consistente con una captura tomada a mitad de una transición/repintado (el propio `.modal` desliza con `transform: translateY()` al abrir) — puede ser un artefacto de la captura en un frame intermedio, o puede ser jank real (frames perdidos) durante la apertura del teclado. No se puede distinguir con certeza desde una imagen estática.

Lo que sí es más accionable: **el pill de Safari se reposiciona sobre el modal en el mismo momento en que el teclado está activo.** Esto encaja con el patrón que describe Diego (escribís, borrás, tocás de nuevo, el teclado se cierra) — la hipótesis con más sustento es que el toolbar dinámico de Safari, al reaparecer/reposicionarse mientras el teclado está abierto, dispara un evento de `resize` en `visualViewport` que Archivo ya escucha (para `--kb-vh` y para el auto-hide del Bloque AC) — si ese resize hace que algún elemento se mueva de un modo que el sistema operativo interpreta como "se tocó fuera del campo de texto", el teclado se cierra. Esto no está confirmado línea por línea porque no se puede reproducir el gesto real en este entorno — es la hipótesis más consistente con la evidencia, no un diagnóstico cerrado.

### Lo que sí se puede afirmar con el código en la mano

- El campo de búsqueda no tiene ningún `blur()` explícito en el código propio — nada en `doSearch()`/el listener de `input` llama a `.blur()` ni reemplaza el nodo del input (solo reemplaza `#ms-results`, un hermano).
- Existe un `setTimeout(() => field.scrollIntoView(...), 300)` que corre en cada `focusin` dentro de un modal — se dispara una sola vez al enfocar, no en cada tecla, así que no debería ser la causa de que se cierre *mientras* se escribe.
- No hay ningún listener que cierre el modal ante scroll o resize del `visualViewport`.

Esto acota el problema: no es algo que Archivo esté haciendo activamente mal en su propio JS de búsqueda — el sospechoso principal es la interacción entre el teclado nativo y el chrome dinámico de Safari, que necesita reproducirse a mano en el dispositivo para confirmar la causa exacta antes de intentar un fix.

---

## 3. ADN — confirmado: mismo patrón repetido, sin jerarquía

### Lo que muestran las capturas

`IMG_5732`, `IMG_5733`, `IMG_5734`: la sección "Lo que dicen tus datos" es una lista de filas idénticas — número monoespaciado chico + un párrafo en itálica serif, separadas por una línea fina. Confirmado en el código (`index.html:2288`): `.adn-insight-row` es un `flex` con dos hijos, sin fondo, sin acento de color, sin ícono, sin separar "el dato" de "la explicación" — cada insight pesa visualmente lo mismo sin importar si es el hallazgo más fuerte o el más débil de los 6.

### Por qué importa

Esto no es un problema de contenido (los insights en sí son buenos, ya validados con datos reales en Bloque T) — es un problema de **presentación mobile específicamente**: en una columna angosta, 6 párrafos de itálica corrida sin ningún quiebre visual se leen como un bloque continuo de texto, y el ojo no tiene ningún punto de entrada rápido para "qué es lo más interesante acá". Es exactamente lo que Diego describe: "el contenido importante queda enterrado."

---

## Marco para lo que sigue (auditoría, no diseño todavía)

Los tres hallazgos comparten una raíz distinta a la de las auditorías anteriores: no son bugs de CSS puntuales, son **decisiones de arquitectura de chrome mobile** (cuántos elementos fijos, cómo coexisten con Safari, cómo se comunican con el teclado nativo) que hoy se resolvieron una por una, en capas, sin una decisión unificada. Coincido con el orden que propusiste:

1. ✅ Esta auditoría.
2. Navegación/chrome — el de mayor impacto y el que probablemente reduce también la superficie del problema de zoom.
3. Teclado/búsqueda.
4. Lectura de ADN en mobile.
5. `watch_date`, recién después.

No voy a diseñar ninguno de estos todavía — como en cada bloque anterior, corresponde tu confirmación de alcance y orden antes de pasar a diseño.
