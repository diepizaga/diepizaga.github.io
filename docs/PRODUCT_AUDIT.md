# Auditoría de producto — Archivo

**Fecha:** 2026-07-30
**Alcance:** `index.html` completo (3.140 líneas), cruzado con los 43 commits de `root/main`.
**Estado:** auditoría descriptiva del estado actual. No propone soluciones ni prioriza cambios — eso corresponde a la etapa de diseño posterior.
**Documento relacionado:** [SECURITY_AUDIT_RLS.md](SECURITY_AUDIT_RLS.md) — los hallazgos de RLS no se repiten acá, solo se referencian donde tocan al producto.

---

## 1. Qué hace Archivo hoy

**Autenticación** — email + contraseña vía Supabase Auth. Alta de cuenta, login, "olvidé mi contraseña" con flujo de reset por link. Sesión persistida en `localStorage` con reintento automático vía refresh-token cuando expira.

**Inicio** — hero con la última entrada agregada al catálogo; carrusel "Seguís con" (ítems en curso); fila de métricas (películas / series / libros / entradas de este año); grilla de entradas recientes; tarjeta de "Identidad" (mini-resumen del ADN) que linkea a la pestaña completa.

**Biblioteca** — catálogo completo con pills por tipo (Todo / Películas / Series / Libros), filtros de estado / género / década / orden, buscador de texto, toggle de vista grilla/lista, y un panel de estadísticas por tipo (géneros más vistos, top calificados, distribución de notas, promedio).

**Descubrí** — recomendaciones generadas a partir de tu propio archivo, no genéricas. Para películas/series: usa tus ítems calificados ≥7 como semilla contra el endpoint de recomendaciones de TMDB, pondera por tu nota y agrupa resultados con la razón ("porque te gustó X"). Para libros: busca por tus autores/géneros favoritos en Google Books.

**ADN** — perfil cultural: arquetipo (heurística sobre género/década/promedio dominantes), estadísticas generales, géneros y décadas dominantes, directores/autores favoritos, comparación de tu criterio contra IMDb/TMDB/Rotten Tomatoes con narrativa generada, tus mayores diferencias de nota contra IMDb, y comparación contra Goodreads para libros.

**Grupo** — crear un grupo (código de invitación de 6 caracteres) o unirse a uno existente. Ver miembros. "Compatibilidad": comparación entre tu biblioteca y la de otro miembro (porcentaje de géneros en común, porcentaje de coincidencia de notas, score compuesto, recomendaciones del otro miembro que no tenés, ítems que ambos tienen con la diferencia de nota).

**Importar CSV** — carga masiva de títulos (columnas `title` + opcionales `year`/`rating`/`type`), matcheado contra búsqueda de TMDB, evita duplicados.

**PWA** — instalable en el celular (manifest + íconos + meta tags de standalone). No tiene service worker registrado.

---

## 2. Fortalezas del producto

- **Estructura clara pese a ser un solo archivo.** Config → helpers → capa de datos (`sbFetch`/`saveItem`) → módulos por API externa (TMDB, OMDB, Open Library, Google Books) → render por pestaña → modales → auth → grupos. Se navega razonablemente bien para 3.140 líneas en un archivo.
- **Manejo de sesión correcto.** `sbFetch` reintenta automáticamente con refresh-token ante un 401 — un patrón que muchos proyectos personales resuelven mal.
- **Heurística de idioma de títulos (es-AR/es-ES/en-US) pensada específicamente para vos** — no es un detalle genérico, resuelve un problema real de TMDB para un usuario argentino.
- **"Criterio vs IMDb/TMDB/RT" y "Compatibilidad" son ideas de producto genuinamente creativas**, bien ejecutadas sin necesidad de backend propio — todo el cálculo ocurre en el cliente a partir de datos ya disponibles.
- **El rediseño v3 quedó limpio.** Confirmado en la auditoría de estado previa: sin CSS muerto del tema revertido ("Midnight Editorial"), sin hacks de cache, sin `console.log`/`debugger` residuales.
- **Manejo cuidadoso de fuentes de libros duales** (Open Library + Google Books), deduplicando resultados por título+autor antes de mostrarlos.

---

## 3. Deuda técnica

- **`migrate_titles.py`, `migrate_books_rating.py`, `import_movies.py` con credenciales hardcodeadas** (email, contraseña de la cuenta, claves de Supabase/TMDB en texto plano). Ya identificado en el saneamiento inicial, pendiente de decisión.
- **Parser de CSV propio (`parseCsvLine`) no maneja comillas escapadas (`""`)** dentro de un campo — un CSV exportado de Excel/Sheets con ese caso rompe silenciosamente.
- **Sin paginación en `loadItems()`** — trae el catálogo completo en cada carga, sin límite. No es un problema al volumen actual (cientos de ítems), se degradaría en el orden de miles.
- **PWA sin service worker** — es instalable (ícono, pantalla completa) pero no tiene ninguna capacidad offline real pese a presentarse como PWA.
- **Caché de recomendaciones (`discCache`) se invalida por completo en cada escritura** — agregar, editar o borrar cualquier ítem fuerza a recalcular las recomendaciones de las tres categorías (películas/series/libros) la próxima vez que se visite Descubrí, sin persistencia entre recargas de página.
- **Alias de CSS legacy** (`--bg-2`, `--t-1`, `--gold`, etc.) mapeados sobre la paleta nueva del rediseño v3, mantenidos porque el JS todavía los usa en estilos inline — dos sistemas de nombres de color coexistiendo de forma permanente.
- **Sin suite de tests** — consistente con ser un proyecto personal de un solo archivo, se anota como hecho descriptivo, no como juicio.

---

## 4. Contradicciones entre producto, arquitectura y modelo de datos

- **El modelo de datos permite pertenecer a varios grupos; la UI asume uno solo.** `group_members` es una tabla de unión (muchos-a-muchos), pero `loadCurrentGroup()` toma el primer grupo que devuelve la consulta (`d[0].groups`) y la interfaz entera opera sobre ese único grupo, sin indicación de que puedan existir otros.
- **Los libros son ciudadanos de primera clase en toda la app excepto en Importar CSV**, que solo busca contra TMDB (películas/series) y no tiene ningún camino para libros.
- **`tmdb_rating` almacena semánticas distintas según el tipo de ítem** — rating de TMDB para películas/series, rating de Google Books (escala 1-5) para libros. Reconocido en un comentario del propio código, pero es una columna cuyo nombre no describe su contenido real para la mitad de los tipos que la usan.
- **El auto-registro está completamente abierto** (sin invitación, sin aprobación, sin confirmación de email aparente) en una app cuyo diseño de producto — un archivo personal con un círculo cerrado de "Grupo" — sugiere un uso acotado a vos y gente conocida. La superficie de autenticación no refleja esa intención de producto. (Detalle técnico y cadena de explotación documentados en `SECURITY_AUDIT_RLS.md`, Riesgos 2 y 3.)
- **El feature de Grupo depende de que `group_members`/`groups` tengan aislamiento correcto**, pero hoy no tienen RLS habilitado — la funcionalidad social de "comparar con un miembro" da por sentado un aislamiento de datos que actualmente no existe a nivel de base.

---

## 5. Preguntas abiertas para el roadmap

- ¿Archivo va a seguir siendo estrictamente personal + un grupo cerrado de gente conocida, o hay ambición de que lo use gente fuera de ese círculo? Esto incide sobre si el auto-registro abierto es una decisión consciente o un descuido, y sobre si vale la pena invertir en exportación de datos.
- ¿El feature de Grupo se usa hoy, o se construyó y quedó sin adopción real? Cambia el peso relativo de invertir en su capa de seguridad y UX.
- ¿Hay expectativa de escala (miles de ítems) o el archivo se va a mantener en el orden de cientos? Incide sobre si la falta de paginación es relevante pronto o no.
- ¿Interesa que la PWA tenga capacidad offline real, o alcanza con que sea instalable como acceso directo?

---

## 6. Funcionalidades incompletas o "muertas"

- **"Directores favoritos" (ADN) está completamente muerta.** El render filtra por `item.director`, pero ningún camino de alta (`buildTmdbItem()` / `tmdbDetail()`) completa ese campo — TMDB nunca se consulta por créditos/director. Esa sección nunca se muestra para películas ni series. (El equivalente para libros, "Autores favoritos", sí funciona porque `author` se completa en el alta.)
- **Sin exportación de datos.** Existe importación CSV pero ninguna forma de exportar el archivo — hoy la única copia de los datos fuera de Supabase es inexistente desde la perspectiva del usuario.
- **PWA sin capacidad offline** — instalable, pero funcionalmente no distinta de un acceso directo a la web; no cumple la promesa implícita de "app" cuando no hay red.
