# Exploración — cómo capturar el "por qué", no solo el "cuánto"

**Fecha:** 15-ago-2026
**Encargo de Diego:** el próximo bloque ataca el hallazgo central de PRODUCT_VISION.md (1 nota escrita de 2181 ítems) — pero explícitamente **no** como sistema de reseñas. El objetivo no es que escribas más, es que Archivo entienda mejor por qué te gustan las cosas. Pidió pensarlo desde cero, sin asumir que la solución es un textarea.
**Estado de este documento:** exploración divergente, no una propuesta cerrada. No implementé nada — es material para decidir juntos el rumbo antes de diseñar el bloque en firme.

---

## Un reencuadre antes de las opciones

Mi primera propuesta en PRODUCT_VISION.md ("tags rápidos en vez de textarea") sigue siendo básicamente **un formulario, solo que más corto**. Vale la pena nombrarlo así de directo porque vos pediste ir más allá de eso.

Hay una distinción que cambia todo el espacio de soluciones: **el objetivo declarado es que el sistema entienda, no que quede algo humano-legible para releer.** Si el destino final del dato es alimentar ADN y el motor de recomendaciones (no un diario que vos releas), entonces la solución no tiene por qué producir texto en absoluto — puede producir señal estructurada directamente, sin pasar nunca por "escribir algo". Esa es la idea que organiza las opciones de abajo.

## Un límite real de esquema, para tenerlo sobre la mesa

`watchlist` tiene `UNIQUE (tmdb_id, type)` — un mismo título solo puede existir una vez en tu archivo. Esto significa que "memoria" en el sentido más profundo (cómo cambió tu relación con algo a lo largo de varias veces que lo viste) **no es solo un campo que falta, está bloqueado por el esquema actual.** No lo estoy proponiendo para este bloque — es una decisión más grande, y quiero que la tengas presente al elegir entre las opciones de abajo, porque algunas la rozan y otras no.

---

## Opciones (varias direcciones genuinamente distintas, no variaciones de la misma idea)

### A. Reacciones contextuales en el momento exacto de calificar
En vez de un campo nuevo en algún lado del modal, el gesto de poner la calificación (que ya hacés en el 90% de los casos) se extiende: apenas elegís la nota, aparecen 3-4 chips de una palabra, **que cambian según la nota que pusiste** (no es una lista fija) — con 9-10 aparecen cosas tipo "me obsesionó" / "de las mejores que vi"; con 3-4, "me decepcionó" / "no era lo que esperaba". Un toque, opcional, cero pantallas nuevas. Es la opción de **menor fricción posible que sigue pidiendo algo activo** — pero es lo más parecido a "un tag", así que vale la pena decir explícitamente que sigue siendo una forma (mínima) de formulario.

### B. Señal inferida, sin pedirte nada nuevo
No agregar ningún gesto nuevo — extraer contexto de datos que **ya existen**: la diferencia entre tu nota y la de la crítica (ya la calcula ADN: "tu criterio vs. IMDb") ya es una señal de "por qué" sin que digas una palabra — una nota tuya muy por encima de la crítica es, en sí misma, información sobre tu gusto. Lo mismo con secuencia (agregaste 4 películas del mismo director seguidas → eso es señal de interés real) y con timing (calificaste apenas lo agregaste vs. días después). Fricción cero, pero el techo de lo que puede "entender" es más bajo — infiere patrones, no capta una sensación puntual como "me hizo llorar".

### C. Comparación en vez de reflexión abierta
En vez de preguntar "¿qué te pareció?" (pregunta abierta, cuesta responder), preguntar algo relativo y mucho más fácil de contestar: "¿te gustó más o menos que [algo que ya calificaste parecido]?". Las respuestas comparativas son más rápidas de dar que las abstractas, y generan una señal más rica para el motor de recomendaciones que un número aislado (preferencia relativa, no solo absoluta). Fricción media — depende de que la comparación que se sugiera sea realmente relevante, si no se siente un capricho.

### D. Voz en vez de texto
Un botón para grabar 10-15 segundos hablando, justo después de calificar — sin transcribir necesariamedte a texto para vos (para vos alcanza el audio), pero transcribible para que el sistema lo procese. Cambia el *modo* de expresión, no solo la fricción: para mucha gente hablar es más rápido y más natural que escribir, y el tono de voz en sí ya es señal (entusiasmo vs. tibieza). Es la opción técnicamente más pesada de las cuatro (grabación, storage, y si querés que alimente al sistema, transcripción) — la nombro porque descarta la premisa de "texto" por completo, que es lo que pediste explorar, no porque sea la más viable a corto plazo.

## Cómo se comparan

| | Fricción real | Qué tan rico es el "por qué" que capta | Alimenta directo al motor de recomendaciones | Toca el límite de "una sola vez por título" |
|---|---|---|---|---|
| A. Reacciones contextuales | Muy baja (1 toque opcional) | Medio — capta una emoción puntual, no el matiz | Sí, directo (son categorías) | No |
| B. Señal inferida | Cero (nada nuevo que hacer) | Bajo — patrones, no momentos | Sí, ya derivable hoy | No |
| C. Comparación relativa | Baja-media | Medio-alto — preferencia relativa es rica | Sí, muy directo | No |
| D. Voz | Media (nuevo gesto, nueva modalidad) | Alto — capta tono y matiz real | Solo si se transcribe/procesa | Roza el tema (¿grabás una vez o cada vez que lo revisitás?) |

## Lo que yo haría, si tuviera que arrancar por algo — pero es tu llamado

No son excluyentes. Mi lectura: **B (señal inferida) es la que menos discusión necesita** — no le pide nada nuevo a nadie, así que no hay riesgo de que termine en el mismo 0.05% de uso que las notas. Es low-risk, se puede prototipar ya. **A (reacciones contextuales)** es el complemento natural si querés algo más expresivo sin resignar fricción baja — pero seguí de cerca si termina usándose o si repite el patrón de las notas. **C y D** son direcciones más grandes, con más para diseñar y más riesgo de sobre-construir antes de saber si A/B ya alcanzan.

No elegí por vos. Quiero tu reacción a estas cuatro direcciones (o una combinación, o algo que no esté acá) antes de diseñar el bloque en firme.
