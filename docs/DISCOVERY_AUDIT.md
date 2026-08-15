# Auditoría — Sistema de descubrimiento e inteligencia

**Fecha:** 15-ago-2026
**Alcance:** cómo funciona hoy Descubrí (recomendaciones) y "Dónde verla en Argentina" (disponibilidad por plataforma). Solo lectura de código — no se implementó nada.
**Por qué ahora:** antes de seguir sumando UX, Diego quiere entender el mecanismo real detrás de la sección más "inteligente" de la app, para decidir con evidencia si vale la pena invertir ahí.

---

## 1. Cómo recomienda Archivo hoy

**Películas y series usan un mecanismo; libros usan otro completamente distinto.** No es la misma "inteligencia" aplicada a los tres tipos — vale la pena tenerlo claro antes de cualquier decisión.

### Películas / series

1. Toma tus hasta 20 títulos de ese tipo con **tu calificación ≥ 7**, ordenados de mayor a menor.
2. **Solo usa los primeros 12 de esos 20** para pedir recomendaciones (el resto se calculan pero se descartan sin usar — ver "qué ignora" abajo).
3. Por cada uno de esos 12, pide a TMDB `GET /movie|tv/{id}/recommendations` — el endpoint de "a la gente que vio esto también le gustó esto" **que calcula TMDB internamente**. Archivo no tiene ningún algoritmo propio de similitud de contenido — todo el trabajo de "qué se parece a qué" lo hace TMDB, no nosotros.
4. Junta los resultados de las 12 consultas en un mapa, sumando un puntaje por cada vez que un título aparece recomendado: `puntaje += tu_calificación / 10` por cada fuente que lo recomienda.
5. Descarta lo que ya tenés guardado y lo que no tiene póster. **Eso es todo el filtrado que se aplica** — no hay filtro de popularidad, cantidad de votos, ni año.
6. Divide el resultado en 3 baldes según el puntaje:
   - **"Más recomendadas para vos"** (puntaje ≥ 1.2): en la práctica, esto solo lo cruza un título si **al menos 2 de tus favoritos lo recomendaron por separado** (un solo favorito, aunque lo hayas calificado 10/10, aporta como máximo 1.0). Es el balde con más corroboración real.
   - **"Por tus géneros favoritos"**: el nombre es engañoso — **no filtra por género en absoluto**. Son simplemente los títulos que quedaron afuera del balde anterior (mismo puntaje, mismo origen). El texto "Drama · Acción · Comedia" que aparece debajo del título de la sección se calcula aparte, solo para mostrarlo — no interviene en qué títulos aparecen.
   - **"Más opciones"** (puntaje < 1.2): la cola larga — alcanza con que **una sola fuente débil** (ej. algo que calificaste justo 7) lo haya recomendado y tenga póster. Es el balde con menos corroboración, y el más probable candidato a sorpresas raras.

### Libros

Mecanismo distinto y, en este caso, sí hace lo que dice:
1. Toma tus 2-10 libros mejor calificados (≥7).
2. Extrae tus autores y géneros más frecuentes entre esos libros.
3. Busca de verdad por autor (`inauthor:"Nombre"`) y por género (`subject:"Género"`) en Open Library/Google Books.
4. Separa resultados en "De tus autores favoritos" y "Por tus géneros" — acá las dos secciones sí corresponden exactamente a lo que dicen.

### Qué datos usa

- Tu calificación personal (`my_rating`) de cada ítem — es el único insumo real de personalización.
- El propio algoritmo de "recomendaciones similares" de TMDB (para película/serie) o coincidencia literal de autor/género (para libros).
- Nada de tu historial de Biblioteca completo entra al cálculo salvo lo que calificaste con nota — un ítem que viste y no calificaste no aporta ninguna señal.

### Qué datos ignora

- **Vote count y popularidad de TMDB** (ambos vienen en la misma respuesta que ya se pide, no hace falta un llamado nuevo) — no se usan para nada hoy.
- **Tus ítems calificados por debajo de 7** — no descartan candidatos ni informan qué evitar, solo no suman puntos a favor. Si algo te gustó poco, Archivo no lo usa como señal negativa.
- **Tus títulos 13-20 mejor calificados** (los que superan el corte de 12 antes de consultar TMDB) — se calculan y se descartan, ni siquiera se usan para peso adicional.
- **El "Estado"** (visto/viendo/pendiente) — solo se usa para excluir lo ya guardado, no como señal de preferencia.

### Por qué aparecen películas muy antiguas o extremadamente desconocidas

Dos causas concretas, no una casualidad:
1. El algoritmo de recomendaciones de TMDB no prioriza por popularidad ni actualidad — puede perfectamente recomendar un clásico de los 60 o una película con 3 votos si la considera temáticamente cercana a algo que calificaste alto.
2. Archivo no agrega ningún filtro propio encima (paso 5 arriba) — nada evita que ese resultado llegue hasta la pantalla, sobre todo en "Más opciones", donde alcanza una sola corroboración débil.

### Cómo podríamos mejorar la calidad sin perder diversidad (opciones, sin implementar)

- **Piso de `vote_count`** (TMDB ya lo devuelve en la misma respuesta, sin llamado nuevo): descartar candidatos con muy pocos votos totales. Esto no penaliza películas viejas per se — un clásico bien votado pasa igual — solo saca el ruido de títulos casi sin data detrás.
- **Ponderar por popularidad en el puntaje**, no excluir: un título obscuro necesitaría más corroboración (más fuentes) para llegar a "Más recomendadas", pero seguiría pudiendo aparecer en "Más opciones" — ahí es donde vive hoy la diversidad real de la sección, y tiene sentido preservarla como espacio explícito para lo inesperado, no eliminarla.
- **No tocaría un filtro de año/década** — la propia identidad de Archivo (ADN, arquetipos como "El arqueólogo cultural", gráfico de décadas) celebra la diversidad temporal. El problema no es "viejo", es "sin corroborar" — mezclar esos dos conceptos sería resolver el síntoma equivocado.
- **Usar los 20 en vez de 12** para sembrar recomendaciones (hoy se calculan 8 de más y se tiran) sería casi gratis y ampliaría la base de corroboración sin sumar llamados nuevos por título ya considerado — aunque si tenés muchos títulos calificados ≥7, 20 llamados en paralelo a TMDB en vez de 12 sí pesa más en tiempo de carga; habría que medirlo, no asumirlo.
- **Renombrar o rehacer "Por tus géneros favoritos"** es una decisión de producto aparte: o se cambia el nombre para que sea honesto con lo que realmente muestra, o se implementa una consulta real por género (ej. `/discover/movie` con `with_genres`) — que sí sería una sección genuinamente distinta de "Más recomendadas" en vez de una segunda vista del mismo balde.

---

## 2. Disponibilidad por plataformas en Argentina

### ¿Tiempo real o almacenado?

**100% en tiempo real, nada se guarda.** `loadWatchProviders()` llama a `GET /movie|tv/{id}/watch/providers` de TMDB **cada vez que abrís la ficha de un título** — no hay caché, ni siquiera en memoria durante la misma sesión: si cerrás y volvés a abrir la misma ficha dos veces seguidas, es una consulta nueva las dos veces. No se guarda nada en `watchlist` ni en ningún otro lado.

### ¿Cada cuánto se actualiza?

No aplica "cada cuánto" en el sentido de que Archivo no cachea nada que necesite refrescarse — cada apertura de ficha ya trae el dato más reciente que TMDB tiene en ese momento. La pregunta real es cada cuánto actualiza **TMDB** su propio dato, y ahí Archivo depende enteramente de un tercero: TMDB no genera esta información — la obtiene de JustWatch (un agregador de disponibilidad de streaming a nivel industria). No hay una cifra de frecuencia publicada que pueda confirmar desde el código — es información externa a Archivo, no algo que controlemos ni podamos verificar con certeza.

### ¿Qué tan confiable es?

Con la evidencia disponible desde acá: es un dato de terceros (JustWatch vía TMDB), consistente con lo que la industria trata como "razonablemente al día pero no en tiempo real estricto" — cambios de catálogo recientes (algo que sale o entra a una plataforma esta semana) pueden tardar en reflejarse. Solo se lee la región `AR` de la respuesta, que es lo correcto para tu caso. El manejo de errores es silencioso: si la consulta falla o no hay datos, la sección directamente desaparece sin ningún aviso — no hay forma de distinguir desde la UI "no está en ninguna plataforma" de "no se pudo consultar".

### ¿Se puede filtrar recomendaciones por plataformas disponibles en Argentina sin afectar el rendimiento?

Con evidencia, no por intuición: hoy Descubrí ya hace 12 llamados en paralelo a TMDB para armar las recomendaciones. Sumar disponibilidad por plataforma requeriría **una consulta más por cada candidato ya recomendado** (hasta ~30 títulos distintos entre los tres baldes) antes de poder decidir qué mostrar — no es un llamado extra, son hasta 30 llamados extra, la mayoría redundantes entre sí si varios candidatos se repiten. Eso casi triplica la cantidad de llamados de red de la sección y alargaría notablemente el tiempo de carga que ya existe hoy.

Alternativa técnica: el endpoint `/discover` de TMDB sí acepta un filtro nativo de plataforma+región (`with_watch_providers` + `watch_region`) sin costo extra de llamados — pero es un endpoint distinto al que se usa hoy (`/recommendations`), pensado para "explorar por criterios" en vez de "similar a este título puntual". Cambiar a esa fuente sería una decisión de arquitectura más grande, no un filtro que se agrega arriba de lo que ya existe — perdería la personalización actual (basada en tus títulos calificados) a cambio de poder filtrar por plataforma de forma barata.

---

## Resumen para decidir el próximo bloque

- El motor de películas/series **no tiene inteligencia propia** — delega 100% en TMDB y solo agrega una capa de puntaje por corroboración. Es mejorable con datos que ya vienen en la misma respuesta (vote_count), sin llamados nuevos.
- "Por tus géneros favoritos" (película/serie) es **el hallazgo más concreto**: el nombre no corresponde a lo que hace. Libros sí está bien etiquetado.
- La disponibilidad por plataforma es 100% en vivo, sin caché, dependiente de un tercero (JustWatch vía TMDB) cuya frecuencia de actualización no podemos verificar desde acá.
- Filtrar recomendaciones por plataforma es técnicamente posible pero caro con el mecanismo actual (hasta 30 llamados extra) — barato solo si se cambia la fuente de recomendaciones a `/discover`, lo cual es una decisión de arquitectura aparte, no un ajuste chico.
