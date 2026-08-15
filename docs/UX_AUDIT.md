# Auditoría de experiencia de uso — Archivo

**Fecha:** 15-ago-2026
**Alcance:** experiencia de producto y de interacción, desktop y mobile. No es una auditoría técnica — no se buscaron bugs de código, aunque algunos hallazgos tienen respaldo de código cuando fue relevante para entender por qué se siente como se siente.
**Método:** recorrido libre de la app con datos reales (sesión real de Diego, 2181 entradas), como usuario intensivo, sin guion previo. Se probaron: alta de película/serie, biblioteca (grilla y lista, filtros, buscador interno), Descubrí, ADN, Grupo, en desktop (1440px) y mobile (375px).
**Qué falta todavía:** no se resolvió nada de esta lista todavía — pasa a diseño bloque por bloque, siguiendo la metodología habitual del proyecto.

**Prioridad de trabajo (ajustada por Diego, 15-ago-2026):** el criterio no es severidad aislada sino impacto en los flujos que más se usan a diario. Orden vigente:
1. Flujo completo de buscar y agregar contenido → hallazgos #3b, #4, #6 (parte del flujo de alta), #7.
2. Sensación general de movimiento de la interfaz (ticker, scrolls, transiciones, cambios de layout) → hallazgo #3, más cualquier otro salto de layout que aparezca al diseñar este bloque.
3. Curar el filtro de género reusando la clasificación que ya calcula ADN → hallazgo #1.
4. Navegación de Biblioteca para llegar rápido a cualquier título con 2000+ elementos → hallazgos #5, #6 (buscador interno), #8.
5. Teclado, layouts y detalles de mobile → hallazgo #2, #9.

Esta es la vigente. La numeración de hallazgos de abajo quedó como se descubrió (por sección recorrida), no reordenada — la prioridad real es la lista de arriba.

**Además, mandato explícito para toda esta etapa (no solo para corregir):** cuestionar el producto en el camino. Si una funcionalidad no aporta valor, se simplifica o se elimina en vez de pulirla. Si una mejora nueva (ej. buscar por actor/director/autor) realmente mejora el uso diario, se evalúa como parte del diseño del bloque correspondiente — no se agrega funcionalidad nueva por agregar, y tampoco se descarta sin evaluarla primero.

**Nota sobre cobertura:** a mitad del pase mobile el click automatizado del navegador de prueba empezó a fallar de forma consistente (no relacionado con Archivo). El resto del recorrido mobile se hizo disparando las mismas funciones que los botones invocan y observando el resultado, no tocando literalmente cada botón. Esto no afecta la validez de los hallazgos de layout/estructura, pero el hallazgo de teclado (#2) se apoya en estructura + código, no en haber visto un teclado real tapar contenido — vale la pena confirmarlo en tu celular antes de darlo por cerrado, como hicimos otras veces en esta auditoría.

---

## Hallazgos priorizados por impacto en la experiencia

### 1. El filtro "Género" de Biblioteca está, tal como está, roto como filtro
**Dónde:** Biblioteca → combo "Género".
**Qué pasa:** el combo tiene ~110 opciones en una lista plana sin agrupar, mezclando cosas muy distintas:
- Géneros reales en español (Acción, Comedia, Drama...).
- Los mismos géneros duplicados en inglés (Action & Adventure, Sci-Fi & Fantasy...) para títulos sin metadata en español — doble estándar dentro del mismo filtro.
- Tags de libros que no son géneros: "adolescence", "dementors", "the Elder Wand", "hieros gamos", "Priory of Sion", "series:Harry_Potter", palabras sueltas en alemán/francés ("Wiederbelebung", "magie"), un typo visible ("College stdents" duplicado con "College students"), y tags que directamente spoilean trama.

**Por qué importa:** es el filtro más visible de la sección más usada de la app (Biblioteca, 2181 ítems) y nadie va a scrollear 110 opciones para encontrar "Comedia". Es evidencia concreta, no interpretación — de todo lo encontrado, es probablemente lo que más grita "a medio terminar".
**Dato a favor de que es corregible sin gran esfuerzo:** ADN ya muestra una lista de género limpia y con conteos (12 categorías: Comedia, Acción, Drama, Aventura, Ciencia ficción, Suspense, Familia, Fantasía, Romance, Crimen, Animación, Terror). La curaduría ya existe en algún lado de la app — el filtro de Biblioteca simplemente no la reusa.
**Conecta con tu pregunta de producto** sobre buscar por actor/director/autor: antes de sumar más ejes de búsqueda, el filtro que ya existe necesita curaduría.

### 2. El teclado probablemente tapa el formulario de notas en mobile
**Dónde:** ficha de detalle/edición (Estado/Fecha/Notas), mobile.
**Qué pasa:** el campo "Notas" —el que más tiempo tiene el teclado abierto— queda pegado justo arriba de los botones de acción (Eliminar/No es esta/Cancelar/Guardar), al final del sheet. Se confirmó por código que no existe ningún manejo de `visualViewport`, listener de resize por teclado, ni `scrollIntoView` en todo el archivo.
**Por qué importa:** es exactamente la queja que nombraste primero ("el teclado termina tapando resultados o cambiando el layout"). La estructura del layout + la ausencia confirmada de cualquier mecanismo para acomodar el teclado hacen que sea muy probable que, al escribir una nota, el teclado tape el campo que estás escribiendo y/o el botón Guardar.
**Pendiente de confirmar:** en tu celular real, antes de priorizarlo en firme (ver nota de cobertura arriba).

### 3. Hay una barra en movimiento constante en todas las pantallas
**Dónde:** el ticker "X películas – Y series – Z libros – promedio – El explorador ecléctico" debajo del header.
**Qué pasa:** scrollea infinito, sin pausa, en Inicio/Biblioteca/Descubrí/ADN — el 100% del tiempo que la app está abierta. En mobile ocupa una fila completa, cortada de los dos lados.
**Por qué importa:** es un candidato directo y muy concreto para la sensación de "se mueve demasiado" que describís, más allá del buscador puntualmente. Es movimiento que nunca se detiene, en cada pantalla, incluso cuando el usuario no está interactuando con nada.

### 3b. Los resultados de búsqueda mezclan el título correcto con ruido irrelevante
**Dónde:** modal de búsqueda, lista de resultados.
**Qué pasa:** al buscar "Inception", el primer resultado es el correcto ("Origen", 2010), pero debajo aparecen sin ningún criterio de relevancia visible títulos oscuros como "Bikini Inception", "WWA The Inception", "The Kawayoku Inception". No hay ninguna señal de popularidad/relevancia que ayude a distinguir el título real de ruido.
**Por qué importa:** es parte directa del flujo de buscar y agregar — cuantos más resultados irrelevantes hay, más fácil es cliquear el equivocado (ver #4), y más "ruidosa" se siente la experiencia de algo tan simple como agregar una película.

### 4. El flujo de buscar y agregar no tiene vuelta atrás
**Dónde:** modal de búsqueda → ficha de confirmación.
**Qué pasa:** una vez que entrás a la ficha de un resultado, "Cancelar" no vuelve a la lista de resultados — cierra todo el flujo y te devuelve a Inicio. Si clickeaste el resultado equivocado (fácil, dado el ruido de resultados — ver #6 y el caso real "Dune: Parte dos/tres/La profecía" durante este recorrido), hay que rehacer la búsqueda entera desde cero.
**Por qué importa:** es fricción de flujo real, no un detalle — coincide con tu percepción de que "el flujo de buscar y guardar se siente incómodo".

### 5. La Biblioteca aparece vacía por un par de segundos antes de poblarse
**Dónde:** Biblioteca, grilla y lista.
**Qué pasa:** la primera fila de pósters carga al instante, pero todas las filas siguientes (dentro del viewport visible, sin necesidad de scrollear) quedan como cajas negras vacías durante ~2 segundos antes de poblarse de golpe. No hay skeleton ni ningún indicio de que algo está cargando.
**Por qué importa:** refuerza directamente la sensación de "inestable"/"a medio cargar", incluso siendo simplemente latencia de imágenes — es lo primero que ves al entrar a la sección más usada de la app.

### 6. Los títulos y géneros mezclan español e inglés sin aviso, y eso rompe el buscador interno
**Dónde:** ficha de detalle (título/género) y buscador interno de Biblioteca.
**Qué pasa:** en la búsqueda de "Dune", la lista de resultados mostraba "Dune: La profecía" (español), pero al abrir la ficha el título cambiaba a "Dune: Prophecy" (inglés) y los géneros pasaban a inglés también ("SCI-FI & FANTASY"), mientras que en otra película los géneros sí estaban en español. Consecuencia directa: el buscador interno de Biblioteca (que hace matching literal por substring) puede devolver "0 resultados" para una búsqueda razonable — ejemplo real: "planeta" no encuentra nada aunque hay 4 películas de "Planet of the Apes" en la Biblioteca, tituladas en inglés; "planet" sí las encuentra.
**Por qué importa:** el usuario no tiene forma de saber que el problema es de idioma — puede pensar que el buscador está roto. Conecta con la duda que ya tenías sobre el buscador de libros: el problema de idioma no es solo de libros, aparece también en películas y series.

### 7. El estado por defecto al agregar algo nuevo es siempre "Visto"
**Dónde:** ficha de confirmación al agregar.
**Qué pasa:** toda alta nueva arranca en Estado = "Visto", incluso para una serie que recién sale. Fuerza una corrección manual en, probablemente, el caso más común: agregar algo que todavía no viste.
**Por qué importa:** es fricción menor pero repetida en cada alta — no es grave, pero es gratuita.

### 8. En mobile, Biblioteca apila mucho antes de mostrar contenido
**Dónde:** Biblioteca, mobile.
**Qué pasa:** antes del primer póster hay que pasar: header + ticker + título "Biblioteca" + 4 chips de tipo + 3 selects en fila (Estado/Género/Década) + 1 select más (Recientes) + buscador propio. Todo lo que en desktop es una sola barra compacta se vuelve ~4 filas apiladas en mobile.
**Por qué importa:** es justo el tipo de inconsistencia desktop/mobile que pediste identificar — la misma información ocupa proporcionalmente mucho más espacio vertical en mobile antes de llegar al contenido real.

### 9. El botón "+" de Descubrí promete un quick-add pero abre el formulario completo
**Dónde:** Descubrí, filas de recomendaciones.
**Qué pasa:** cada fila de recomendación tiene un botón "+" chico que visualmente sugiere un agregado de un toque, pero abre la misma ficha completa (estado/fecha/notas/calificación) que el flujo de búsqueda.
**Por qué importa:** no es necesariamente malo, pero es el lugar de la app donde un quick-add real ("agregar como Pendiente", sin abrir modal) tendría más sentido — estás navegando recomendaciones, no completando una ficha.

---

## Evaluación de producto

### Lo que ya funciona y tiene "alma" (base para abrir la app todos los días)
**Descubrí** y **ADN** son, con diferencia, las secciones con más personalidad de la app hoy. Descubrí no solo recomienda — explica el porqué ("Por tu afinidad con The Empire Strikes Back y Star Wars", "Porque te gustó Interstellar"), que es exactamente el tipo de detalle que da ganas de volver sin tener algo puntual para agregar. ADN arma un resumen tipo "personalidad" con datos reales (décadas, criterio vs. IMDb, autor top) que se siente hecho a medida. Antes de agregar funcionalidades nuevas para lograr "algo que abra todos los días", tiene sentido terminar de pulir y visibilizar estas dos secciones — ya están haciendo ese trabajo, solo que compiten con las fricciones de arriba.

### Grupo: sin cambios respecto de lo ya decidido
Verificado en tu cuenta real durante este recorrido: hoy no estás en ningún grupo ("No estás en ningún grupo todavía"). Esto no es un hallazgo nuevo — coincide con lo que ya habías definido en el Bloque 0 de `DESIGN_MAP.md` (30-jul-2026): *"Grupo: no se usa hoy en la práctica... no debe condicionar las prioridades del proyecto."* Lo dejo anotado como confirmación de campo, no como pregunta abierta — no vuelvo a plantearlo salvo que decidas retomarlo vos.

### Compartir recomendaciones/listas/gustos
No encontré ningún rastro de esto en la app actual. Grupo hoy solo compara "compatibilidad" entre miembros — no vi forma de compartir una lista puntual, recomendar un título específico a alguien del grupo, ni ver qué está viendo/leyendo otra persona más allá de un puntaje de afinidad general. Si esta idea seguía en pie, hoy no tiene ningún punto de apoyo construido — sería empezar de cero, no completar algo a medio hacer.

### Buscar por actor, director o autor
No llegué a probar específicamente si el buscador ya soporta esto (no until ahora encontré ningún campo ni resultado que sugiera que sí) — lo marco como pendiente de verificar en el próximo paso, no como confirmado. Antes de sumarlo, igual, el filtro de Género (#1) necesita curaduría — es la misma familia de problema (cómo se navega/filtra la biblioteca) y ya está roto en su versión más básica.

### Qué sacaría
No encontré ninguna función completamente muerta en este recorrido (a diferencia de "Directores favoritos", que ya se sacó en un bloque anterior). Lo más cercano a "esto no aporta como está" es el propio filtro de Género: en su estado actual (110 opciones sin curar) probablemente genera más frustración que valor, así que "arreglarlo o sacarlo" es una decisión real, no solo "arreglarlo".

### Qué agregaría
Ninguna propuesta nueva todavía — por diseño de esta etapa. Mi lectura después del recorrido es que el mayor "agregado" de valor a corto plazo no es una función nueva, sino terminar de conectar lo que ya existe: la curaduría de género que ADN ya calcula, aplicada al filtro de Biblioteca; y un quick-add real desde Descubrí. Eso ya cambiaría cómo se siente usar la app sin sumar superficie nueva.

---

## Próximo paso
Esta lista está armada por impacto percibido, no por esfuerzo de implementación — antes de tocar código quiero que la revises y me digas con qué orden armamos el trabajo (podés reordenar, sacar algo, o pedirme que agrupe varios en un mismo bloque). El punto #2 (teclado) conviene confirmarlo en tu celular real antes de darlo por prioritario.
