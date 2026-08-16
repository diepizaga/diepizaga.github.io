# Visión de producto — Archivo como experiencia completa

**Fecha:** 15-ago-2026
**Por qué este documento:** Diego pidió explícitamente dejar de pensar por módulos aislados y evaluar Archivo como producto completo, poniéndome en el lugar de un usuario intensivo diario. No es una lista de ideas — es una síntesis para decidir el próximo rumbo, priorizada por impacto real en el uso diario, no por facilidad de implementación.
**Método:** antes de escribir esto, miré los datos reales de tu archivo (2181 ítems) para no construir la visión sobre supuestos. Algunos de los números me obligaron a corregir cosas que hubiera asumido distinto.

---

## Lo que los datos reales dicen, antes de opinar nada

- **2181 ítems: 1862 películas, 291 series, 28 libros, 1965 con tu calificación (90%).**
- **`Estado`: 2181 de 2181 son "Visto" — el 100%.** Cero "Viendo", cero "Pendiente" en tu historial real (esto recién empieza a cambiar con el default nuevo de Bloque N).
- **`Notas`: 1 solo ítem de 2181 tiene una nota escrita.** No es "poco uso" — es prácticamente cero.
- **`Fecha de visto` (`watch_date`): 0 de 2181.** No existe un solo registro de "cuándo viste esto" más allá de cuándo lo cargaste en Archivo.
- **Tu historial *en Archivo* (`created_at`) va del 6-jun-2026 al 15-ago-2026 — unas 10 semanas, no "varios meses".** El contenido que catalogaste sí abarca décadas (tu ADN ya lo muestra: de los 30s a los 20s), pero *cuándo lo cargaste* está comprimido en dos meses y medio, casi seguro por la carga inicial masiva del Excel.

Esto último corrige algo importante antes de seguir: **lo que tenés no es un diario temporal de tu consumo — es un catálogo de gustos con profundidad estadística.** Vale la pena decirlo así de directo porque cambia qué tipo de "próxima capa" tiene sentido: una función de "hace un año viste esto" no tiene mucho para pararse todavía (no hay fechas reales de visto, y el historial de carga es muy corto); una función que aprenda de tus 1965 calificaciones sí tiene con qué trabajar, y recién ahora.

---

## ¿Qué hace que Archivo ya sea mejor que Letterboxd, IMDb o Goodreads?

1. **Un solo lugar, una sola identidad, para película/serie/libro.** Ninguno de los tres competidores hace esto — cada uno vive en su propio silo. Vos podés ver "quién sos como consumidor cultural" de punta a punta, no fragmentado en tres apps distintas con tres perfiles distintos.
2. **ADN — una síntesis narrativa real de quién sos, no solo estadísticas.** "El explorador ecléctico", tu criterio vs. la crítica, tu autor top — eso no es un dashboard, es un espejo. Ni Letterboxd (con su "Year in Review") ni Goodreads (con sus "reading challenges") arman un perfil narrativo continuo así.
3. **Cero fricción social/de rendimiento.** No hay contador de seguidores, no hay "likes", no hay presión de mostrar buen gusto. Es coherente con lo que vos mismo definiste desde el Bloque 0 ("archivo personal, sin ambición de plataforma social") — y es, en sí mismo, una ventaja real frente a productos que se volvieron más sobre la audiencia que sobre el usuario.
4. **Recomendaciones que ya empiezan a tener criterio propio** (Bloque R) — Letterboxd e IMDb delegan 100% en TMDB; Goodreads en su propio motor, opaco y ajeno a vos. Archivo está en camino a tener un criterio que es solo tuyo.
5. **Identidad visual propia**, sin comparación real con la estética genérica de IMDb o el feed saturado de Letterboxd.

## ¿Qué hace que todavía no lo sea?

1. **No hay expresión real capturada.** Con 1 nota en 2181 ítems, hoy Archivo sabe *cuánto* te gustó algo pero no sabe *por qué*, ni qué pensaste, ni qué te generó. Letterboxd y Goodreads viven de eso — la reseña, aunque sea de una línea, es el corazón emocional del producto para mucha gente.
2. **No hay dimensión temporal real de tu vida.** No hay "releíste este libro", no hay "lo viste con quién", no hay fecha real de cuándo pasó. Ahora mismo Archivo no puede distinguir "lo vi ayer" de "lo vi hace 10 años y lo cargué la semana pasada".
3. **La búsqueda todavía no cubre actor/director/autor** — ya lo identificamos técnicamente cerca (Bloque N), sigue pendiente.
4. **La disponibilidad por plataforma es fràgil** (auditado en Bloque R): en vivo, sin caché, dependiente de un tercero, sin ningún tipo de fallback.
5. **No hay ningún mecanismo de "esto es para vos" hacia otra persona** — ni liviano ni pesado. Grupo lo intentó con "compatibilidad" y no prendió (0 uso real, confirmado, ver Bloque 0).

## Las tres funcionalidades que más aumentarían el valor percibido (priorizadas por impacto de uso diario, no por costo)

**1. Bajar la fricción de decir algo, no solo calificar algo.**
Con 90% de calificación y 0.05% de notas, el problema no es que no tengas nada para decir — es altamente probable que el campo actual (un textarea en blanco, al fondo de un modal, sin ningún estímulo) no invite a usarlo. No hace falta un sistema de reseñas pesado como Letterboxd: alcanza con bajar el costo de expresar algo a un gesto — tags rápidos ("me sorprendió", "para volver a ver", "no era lo que esperaba"), no un párrafo. Esto no es solo una mejora de UX: es la señal que le falta al motor de recomendaciones para dejar de depender de TMDB (ver Bloque R, visión de largo plazo) — un número del 0 al 10 dice mucho menos que "me sorprendió" o "predecible".

**2. Profundizar ADN con la escala que ya tenés.** 1965 calificaciones reales es, recién ahora, suficiente para que ADN deje de ser "un resumen bonito" y empiece a mostrar patrones que vos mismo no notaste — cómo cambió tu criterio en el tiempo, qué combinaciones de género+década te gustan más de lo que pensás, dónde tu gusto se aleja más de la crítica y por qué. Es la pieza que ya te distingue de los tres competidores — profundizarla es apalancar una ventaja real, no construir una nueva desde cero.

**3. Acelerar el motor de recomendaciones hacia criterio propio** (ya es el rumbo confirmado en Bloque R) — con 1965 calificaciones reales, por primera vez hay suficiente data propia para que el ranking dependa de patrones tuyos y no del score de TMDB. Antes de esta escala, esto hubiera sido prematuro; ahora es viable.

Ninguna de las tres es "fácil". Las tres comparten algo: **todas usan datos que ya existen o que se generan con muy poca fricción nueva** — no son features aisladas, son la misma apuesta (que Archivo entienda mejor a su único usuario) mirada desde tres ángulos distintos.

## ¿Qué agrega complejidad hoy y aporta poco valor?

- **Grupo.** Confirmado con evidencia, no supuesto: 0% de uso real en tu cuenta, y ya lo habías definido vos mismo en el Bloque 0 como algo que "no debe condicionar las prioridades". Llegado este punto de repensar el producto completo, vale la pena una pregunta más directa que "no priorizar": ¿lo sacamos, lo dejamos dormido tal cual está, o lo repensamos completamente bajo otra idea (no "compatibilidad", sino algo más chico, tipo "quiero que fulano vea esto")? Las tres son válidas — la que no me parece sostenible es seguir puliendo una función con cero uso.
- **El eje "Viendo" del Estado.** Hoy 0% de tus 2181 ítems lo usan (aunque recién empieza a existir de verdad con los nuevos altas desde Bloque N) — no propongo sacarlo, pero si en unos meses sigue en 0%, vale la pena revisar si "en curso" es un estado real para tu forma de usar Archivo o una distinción que nadie necesita.
- **Fecha de visto (`watch_date`).** Existe el campo, nadie lo llena (0/2181). O se le baja la fricción a completarlo (ej. default a "hoy" en vez de vacío) o se acepta que no es un dato que vayas a cargar y se deja de tratar como si fuera a alimentar una futura función de diario — construir sobre un campo vacío sería el error que este documento justamente quiere evitar.

## ¿Qué oportunidades reales aparecen ahora, con 2000+ títulos?

No la que yo hubiera asumido antes de mirar los datos (una función de diario/temporal) — la data no la sostiene todavía. Las reales:

- **Suficiente volumen estadístico para un modelo de preferencia propio** (motor de recomendaciones, ya en marcha desde Bloque R).
- **Suficiente volumen para que ADN diga cosas que vos no sabías de vos mismo**, no solo resuma lo obvio.
- **Suficiente catálogo para que la búsqueda por actor/director/autor tenga sentido real** — con 28 libros no cambia mucho, pero con 1862 películas, buscar "qué vi de este director" empieza a ser una pregunta real que te vas a hacer.

---

## Síntesis

Archivo ya tiene la base correcta para ser mejor que sus competidores — el diferencial no es una función que falta, es que las piezas que ya lo distinguen (un solo lugar, ADN, criterio propio) todavía no están explotadas a la escala que la data actual permite. La brecha más grande no es funcional, es de **expresión**: el producto sabe *cuánto* te gustó algo con una precisión altísima (90% calificado) y casi no sabe *nada más* sobre eso (0.05% con una palabra escrita).

Mi recomendación de rumbo, en orden: **primero bajar la fricción de expresión** (es barato, y es el insumo que las otras dos apuestas necesitan) — **después profundizar ADN y acelerar el motor de recomendaciones en paralelo**, ya que ambos se alimentan de la misma data y del mismo cambio de fondo (más señal por ítem, no solo un número).

No decidí un bloque todavía a propósito — es tu llamado, con esta síntesis como base en vez de inercia.

---

## Revisión del roadmap — 16-ago-2026, después de Bloques R-X

Las tres apuestas que este documento proponía (bajar fricción de expresión, profundizar ADN, acelerar el motor de recomendaciones) **ya están implementadas y en producción**: Bloque S (reacciones + señal inferida), Bloques T/V/W (ADN 2.0 + personas, director/reparto separados), Bloque X (ranking propio de Descubrí). Diego pidió una revisión honesta del roadmap completo antes de elegir el próximo movimiento — no seguir por inercia sobre lo que ya viene encadenado.

### 1. Qué sigue teniendo sentido del roadmap original

- **Memoria** (recordar tu historia en el tiempo) — sigue siendo el tercer pilar correcto de la visión (aprender → ayudar a descubrir → recordar). No perdió sentido, está **estructuralmente bloqueada**: `watch_date` sigue en 0 de 2153 ítems. Empezarla hoy sería construir sobre una tabla vacía.
- **Grupos como inteligencia colectiva de gustos** — sigue siendo una dirección real y que a Diego le interesa, no se desarmó con nada de lo construido. Sigue necesitando su propia auditoría (qué hay en código/base, qué datos se pueden cruzar, privacidad) antes de diseñar nada — no se hizo todavía.
- **Auditoría UX móvil/PWA** — sigue en pie, con evidencia real ya recolectada (4 capturas de Diego, 15-ago) sin actuar todavía: superposición de la barra de Safari, posibilidad de pinch-zoom que rompe el layout, puntitos de estado poco legibles.

### 2. Qué perdió prioridad — no por dejar de importar, sino porque ya está resuelto

- **Las tres apuestas de PRODUCT_VISION.md ya no son "el próximo paso"** — están hechas. No perdieron valor, perdieron urgencia porque ya se cobraron.
- **"¿Género o persona pesa más?"** (la pregunta abierta desde Bloque V) — dejó de ser una pregunta pendiente: Bloque X la resolvió con evidencia (afinidad por valor específico, no peso fijo por categoría). Cerrada, no diferida.
- **Buscar por actor/director/autor** (Bloque U) — se planteó originalmente como mejora de navegación ("la voy a usar, pero no cambia la identidad del producto"). Terminó siendo mucho más que eso: es la base de datos que hizo posible Bloque V, W y X. La ambición del bloque creció con el uso que terminó teniendo, no se achicó.

### 3. El próximo bloque de mayor impacto — mi recomendación, con el razonamiento

**La auditoría UX móvil/PWA.** No es la continuación obvia de la secuencia ADN→Descubrí→Memoria, y lo digo a propósito: creo que seguir esa secuencia ahora sería inercia, no la decisión de mayor impacto real. Razones concretas:

- **Memoria no puede arrancar todavía** — no hay con qué (`watch_date` en 0). Empezarla ahora sería repetir el error que ya evitamos una vez con reacciones: construir sobre datos que no existen.
- **Grupos necesita una auditoría propia antes de poder ni siquiera dimensionarse** — no está listo para ser "el próximo bloque", está listo para ser "la próxima auditoría".
- **ADN y Descubrí ya tienen su primera versión real, funcionando y validada** — profundizarlas más hoy tiene rendimientos decrecientes hasta que haya más datos (más reacciones, más uso). No es que no haya más para hacer ahí, es que no es lo más urgente.
- **La UX móvil/PWA, en cambio, ya tiene evidencia real esperando** (no hay que auditar de cero, ya se juntaron las capturas) y **afecta cada sesión, no una sección puntual** — un layout que se rompe con pinch-zoom o una barra de Safari que tapa contenido pesa en cada uso diario, no solo cuando abrís Descubrí o ADN.

Es mi recomendación, no una decisión tomada — la dejo para que la confirmes o la redirijas.

### 4. Deuda técnica y de UX que sigue abierta

- **Duplicación real de lógica:** `computeADNInsights()` (ADN) y `computeAffinityMaps()` (Descubrí, Bloque X) calculan el mismo tipo de cosa — efecto de un valor sobre tu calificación, ponderado por confianza — con dos implementaciones separadas. Funciona hoy, pero es la misma señal calculada dos veces; en algún momento conviene unificarlas en un solo módulo que alimente a ambos.
- **`watch_date` en 0/2153** — ya nombrado como el bloqueo estructural de Memoria, lo dejo también acá como deuda de dato, no solo como "falta implementar una feature".
- **Los 3 puntos concretos de la auditoría UX/PWA** (pinch-zoom, superposición de Safari, puntitos de estado) — diagnosticados, no corregidos.
- **Grupos** — infraestructura de "biblioteca compartida" sin uso real, sin auditoría de qué serviría para "inteligencia colectiva".
