# Plan de diseño — Archivo

**Fecha:** 2026-07-30
**Estado de este documento:** plan de diseño vigente, no un backlog — cada bloque tiene objetivo, respaldo de auditoría y estado real de avance. Se actualiza a medida que el proyecto avanza (no es una foto fija del día que se creó).
**Documento anterior en la cadena:** [MASTER_AUDIT.md](MASTER_AUDIT.md) — este plan nace directamente de sus hallazgos y pendientes.

## Cómo leer este documento

El orden de los bloques es una **propuesta inicial de trabajo, no una secuencia fija**. Puede cambiar durante el proyecto si aparece evidencia nueva durante el diseño de un bloque, si el Bloque 0 (visión de producto) redefine prioridades, o si una validación pendiente (ver MASTER_AUDIT § 5) cambia el panorama de un bloque puntual. Cuando el orden cambie, se actualiza acá con la razón — no se pierde el rastro de por qué se reordenó.

Cada bloque lista sus dependencias explícitas. Un bloque sin dependencias declaradas no espera a nada ni a nadie — puede empezar cuando haya lugar, sin esperar a que otros bloques (aunque parezcan "más importantes") se resuelvan primero.

Estados posibles: **Pendiente** / **En diseño** / **Implementando** / **Validando** / **Finalizado**.

---

## Bloque 0 — Visión de producto a 12 meses

- **Objetivo:** responder qué querés que sea Archivo en un horizonte de 12 meses.
- **Problema que resuelve:** varios bloques (D, F, I, y la urgencia real de B/C) no se pueden diseñar bien sin saber si Archivo sigue siendo estrictamente personal + círculo cerrado, o si hay ambición de que otros lo usen, y si Grupo es un feature real o quedó sin adopción.
- **Valor:** Alto — calibra la prioridad real de todo lo demás.
- **Riesgo:** Bajo — no es un riesgo de implementación (no hay código todavía); el riesgo real es el de saltearlo y diseñar seguridad/grupos sobre supuestos no explicitados.
- **Esfuerzo:** Bajo — es una conversación y un documento corto, no una implementación.
- **Dependencias:** ninguna. Es el bloque fundacional.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 5; MASTER_AUDIT.md § 5.
- **Estado:** Finalizado (2026-07-30).

### Visión definida por Diego (2026-07-30)

- **Para quién es Archivo:** archivo personal, ante todo. Sin ambición de red social ni plataforma abierta. Compartir con un círculo muy cerrado (amigos, pareja, familia) es un interés a futuro, pero siempre bajo lógica de invitación y confianza — nunca una app pública con registro y descubrimiento de usuarios.
- **Grupo:** no se usa hoy en la práctica. Se construyó como exploración de un concepto, no como parte del uso habitual. Decisión explícita: **no debe condicionar las prioridades del proyecto.**
- **Escala esperada:** cientos de ítems hoy, probablemente algunos miles con el tiempo (no volumen de aplicación masiva). Intención de consolidar películas, series y libros — y eventualmente otras fuentes — en un solo lugar.
- **Offline:** no es prioridad. Archivo se diseña asumiendo conexión; no se diseña el producto alrededor de ese requisito.
- **Horizonte a 12 meses:** "el archivo cultural definitivo" — un único lugar para registrar todo lo que ve/lee, con recomendaciones basadas en su propio historial y mejor entendimiento de sus gustos con el tiempo. No busca competir con Letterboxd/Goodreads/IMDb. Si otras personas lo usan en el futuro, debe ser porque la experiencia individual ya es excelente — no porque el objetivo sea crecer como plataforma social.

### Impacto en el resto del plan

- **Confirma y refuerza B y C:** `watchlist` y `profiles` son el núcleo mismo de "el archivo cultural definitivo" — su protección no depende de nada de esto, se reafirma como prioridad más alta del plan.
- **Resuelve el auto-registro abierto (antes "Bloque J"):** la visión de "invitación y confianza, nunca una app pública con registro abierto" es una posición clara en contra del `signup` sin restricciones actual. Queda como insumo directo para el diseño de B/C — no es un bloque aparte, pero ya no es una pregunta sin responder.
- **Baja la prioridad de D e I:** Grupo no se usa y no debe condicionar el proyecto — ambos bloques bajan su urgencia real, aunque el riesgo de seguridad documentado en D (SEC-3) sigue siendo válido en teoría.
- **Sube el valor de F (exportación):** si Archivo va a ser el repositorio definitivo de la historia cultural de Diego, la ausencia total de backup pesa más de lo estimado originalmente.
- **G (consolidar fuentes) gana algo de contexto** por la intención explícita de "eventualmente otras fuentes", sin volverse urgente.
- **Confirma que PROD-8 (offline) y PROD-9 (paginación) siguen diferidos** — offline explícitamente descartado como prioridad; escala de "algunos miles" no amerita paginación todavía.

---

## Bloque A — Housekeeping de repo

- **Objetivo:** dejar el repositorio en un estado consistente y trazable.
- **Problema que resuelve:** el tracking local de la rama `main` apunta al repo stale (`origin`/`cinelog.git`); quedan `.bak` redundantes sin limpiar.
- **Valor:** Medio — necesario, no transformador.
- **Riesgo:** Bajo — recablear el tracking no toca el working tree (ya coincide con `root`); borrar `.bak` ya fue verificado como seguro.
- **Esfuerzo:** Bajo.
- **Dependencias:** ninguna para el recableo de tracking y el borrado de `.bak`. La limpieza completa (dejar de usar `origin` del todo) depende de tu decisión sobre el destino de `cinelog.git` (MASTER_AUDIT § 5).
- **Documentos de auditoría que lo respaldan:** MASTER_AUDIT.md § 2 (EP-3, EP-4), § 5.
- **Estado:** Finalizado (2026-07-30), con alcance acotado explícitamente por Diego.

### Ejecutado

- `main` reseteado a `root/main` (`c83d62c`) y tracking configurado a `root` — el repo ya no arrastra la historia stale de `origin`/`cinelog.git`.
- `.gitignore` agregado (backups, los 3 scripts con credenciales, `.DS_Store`).
- `index.html.bak` e `index_old_backup.html` eliminados.
- `docs/` (las 4 auditorías/plan) versionado por primera vez.
- Working tree confirmado idéntico a lo publicado en `https://diepizaga.github.io/` después de todo el proceso.
- 2 commits locales, **no pusheados** — queda pendiente de autorización explícita cuando corresponda.

### Diferido explícitamente (no bloquea el resto del proyecto)

El destino de `cinelog.git`/`origin` (deprecar Pages / mantener sincronizado / borrar) queda como decisión administrativa pendiente, sin fecha. No condiciona la implementación de B/C ni ningún otro bloque.

---

## Bloque B — Habilitar RLS en `profiles`, `groups`, `group_members`

- **Objetivo:** cerrar el acceso público sin restricción a esas tres tablas.
- **Problema que resuelve:** cualquiera con el `anon key` puede leer/escribir ahí hoy, sin precondiciones.
- **Valor:** Alto — cierra la exposición más severa documentada.
- **Riesgo:** Medio — si el texto de las políticas existentes es más laxo o más estricto de lo esperado, activar RLS puede no corregir nada o romper un flujo real (ej. la creación automática de perfil al loguearse).
- **Esfuerzo:** Bajo-Medio — las políticas ya existen, falta revisarlas, activarlas y validar con uso real.
- **Dependencias:** ninguna abierta — el texto SQL exacto de las 9 políticas ya se obtuvo y se revisó (ver "Diseño aprobado" abajo). Este campo estaba desactualizado hasta 2026-07-30.
- **Documentos de auditoría que lo respaldan:** SECURITY_AUDIT_RLS.md § Riesgo 1.
- **Estado:** Backend implementado y validado (2026-07-30). Falta el cambio de frontend (`owner_id` → `created_by`) como paso independiente.

### Ejecutado — parte backend

1. Aplicado en Supabase (SQL Editor): `profiles_read` reescrita, RLS habilitado en las 3 tablas.
2. **Incidente no anticipado, encontrado en la validación**: `HTTP 500`, `42P17 infinite recursion detected in policy for relation "group_members"`. Causa: `gm_read` (política preexistente, no tocada por el diseño original) se autoconsulta contra `group_members` — mientras esa tabla no tenía RLS, la autoconsulta no pasaba por ninguna política; al activarla, Postgres entra en un ciclo al re-evaluar `gm_read` para resolver la subconsulta interna. Afectaba a las tres tablas (`profiles_read` y `groups_read` consultan `group_members` por dentro).
3. **Corrección aplicada**: función `public.my_group_ids()` (`SECURITY DEFINER`) que lee `group_members` sin pasar por RLS, evitando el ciclo. `gm_read`, `groups_read` y `profiles_read` reescritas para usarla en vez de autoconsultarse. Es el patrón estándar documentado por Supabase para este caso — no es una improvisación.
4. **Validado:** las 3 tablas vuelven a `200 []` con el `anon key` (antes daban `500`). `watchlist.group_read` (no tocada, pertenece al Bloque C) se revisó por las dudas — nunca se rompió, y se beneficia del mismo arreglo al dejar de haber ciclo en `group_members`.

### Validación del backend — completada (2026-07-30)
- [x] Confirmado en Table Editor: `profiles`/`groups`/`group_members` ya no muestran "UNRESTRICTED".
- [x] Probado con cuenta real: perfil, nombre y funcionamiento general correctos, sin regresión.

### Pendiente — frontend (paso independiente, no arrancado)
- [ ] `createGroup()` ([index.html:3115](../index.html#L3115)): cambiar `owner_id` → `created_by` en el body del POST. Único cambio de este paso — sin mezclar otras mejoras.

### Diseño aprobado

- `groups_insert` referenciaba `created_by`, columna real en el schema; el frontend mandaba `owner_id`, que no existe — confirmado con evidencia (error `42703` en `owner_id`, `[]` en `created_by`) y con la historia completa de git (ningún commit tocó nunca `created_by`; `owner_id` aparece una sola vez, en el commit que originó el feature, 06-jun-2026). Sin evidencia de una migración que haya causado el desajuste — se sigue la regla por defecto de Diego: **el esquema es la fuente de verdad, se adapta el código** (`createGroup()` pasa a mandar `created_by`).
- `profiles_read` (`using(true)`) se reescribe acotada a "perfil propio + perfiles de gente con quien compartís un grupo" — auditoría puntual de los 3 consumidores reales (`loadProfile`, `changeDisplayName`, `renderGroupMembers`/`renderGroupCompareList`) confirmó que ese scope cubre todos los casos sin restringir de más.
- Resto de las 6 políticas (`profiles_update`, `profiles_write`, `groups_read`, `gm_read`, `gm_insert`, `gm_delete`) se activan sin cambios — confirmadas correctamente scoped.
- Rollback progresivo: (1) revertir código, (2) revertir/ajustar la política puntual, (3) desactivar RLS como último recurso.
- Criterio de éxito: RLS habilitado en las 3 tablas + test de lectura anónima sigue vacío + `createGroup()` puebla `created_by` + sin regresión en login/perfil + cero errores nuevos en consola/red.

### Observación registrada durante el diseño (fuera de alcance de este bloque)

`profiles` tiene 0 filas hoy pese a que `loadProfile()` intenta crear el perfil en cada login (`index.html:2718`). Causa probable: el `fetch(...).catch(()=>{})` de esa creación solo atrapa fallas de red, no errores HTTP — un fallo silencioso ahí nunca se habría notado. **Decisión explícita de Diego: esto no amplía el alcance de Bloque B.** Se valida como parte del checklist de implementación (crear un perfil real y confirmar que la fila se crea) pero no se diagnostica ni se corrige acá. Si la implementación confirma que la creación automática falla de verdad, se abre un bloque nuevo e independiente para ese problema. Si funciona correctamente, el hallazgo se cierra como validado sin generar trabajo adicional.

**Actualización (30-jul-2026):** se confirmó que falla de verdad — ver **Bloque K** más abajo. De paso, la auditoría de columnas de `profiles` hecha en este mismo bloque B (que daba por existentes `display_name` y `email`) resultó **incorrecta** — corregida y explicada en Bloque K, sección "Corrección de auditoría".

---

## Bloque C — Corregir el bypass `user_id IS NULL` en `watchlist`

- **Objetivo (revisado):** eliminar por completo la posibilidad de que existan filas huérfanas en `watchlist`, en vez de construir un mecanismo para "reclamarlas".
- **Problema que resuelve:** cualquier cuenta autenticada puede leer o apropiarse de filas con `user_id NULL`.
- **Valor:** Alto — cierra el segundo hallazgo crítico, sobre la tabla con el dato personal más sensible.
- **Riesgo:** Medio — bajó respecto de la estimación inicial: no hay lógica de negocio de "claim" que preservar (se retira), el cambio real es más chico de lo previsto.
- **Esfuerzo:** Medio.
- **Dependencias:** ninguna. Confirmado sin filas huérfanas existentes (ver validación abajo), así que tampoco depende de una limpieza manual previa.
- **Documentos de auditoría que lo respaldan:** SECURITY_AUDIT_RLS.md § Riesgo 2 y § Riesgo 4.
- **Estado:** Diseño aprobado (2026-07-30) — pendiente de implementación.

### Diseño aprobado — cambio de dirección respecto de la propuesta original

Diagnóstico: no existe ninguna señal de identidad en una fila con `user_id = NULL` (se revisó `localStorage`, y se probaron 6 nombres de columna candidatos — ninguno existe) — un mecanismo de "claim", aunque viva en el servidor, seguiría siendo "quien pida primero, se la lleva", no una verificación real de propiedad. Se descarta la idea original de mover `claimNullItems()` a una función RPC del lado del servidor.

**Dirección aprobada — prevenir en vez de reclamar:**
1. **Frontend:** cerrar la única ventana real de creación de huérfanas — `#app` es interactivo desde el primer render, antes de que `initAuth()` resuelva su `await fetch(...)` inicial. Gatear las acciones de alta (`openSearchModal`, `runImport`) hasta que `initAuth()` termine. Confirmado con evidencia que `currentUser` solo se pone en `null` en dos lugares de todo el código (arranque de `initAuth()` y `signOut()`), ambos ya acompañados de la pantalla de auth — no hay otro camino de creación de huérfanas.
2. **Retirar `claimNullItems()`** y su invocación en `onSignedIn()` — código muerto una vez que (1) esté en producción.
3. **Sacar `OR (user_id IS NULL)` de `own_data` y `group_read`, sin reemplazo.**
4. **Agregar constraint `NOT NULL` en `watchlist.user_id`** — guardrail permanente a nivel de base de datos.

**Validación de precondición (30-jul-2026):** confirmado en el dashboard (Table Editor, filtro `user_id is null`, rol `postgres` — bypassea RLS, ve la tabla completa) que **0 de 100 filas** de `watchlist` tienen `user_id NULL` hoy. No hace falta ningún paso de limpieza manual antes de aplicar el `NOT NULL`.

**Criterio de éxito:**
- [ ] Agregar un ítem (o correr Importar CSV) en el instante de carga, antes de que `initAuth()` resuelva, ya no es posible — la acción queda gateada hasta que la sesión se resuelva.
- [ ] Un intento de insertar en `watchlist` sin `user_id` es rechazado por la base de datos (constraint `NOT NULL` activo).
- [ ] El texto SQL de `own_data` y `group_read` ya no contiene `OR (user_id IS NULL)`.
- [ ] `claimNullItems()` y su invocación en `onSignedIn()` fueron removidos del código.
- [ ] Flujo normal (login, agregar, editar, ver ítems) sin regresión, probado con tu cuenta real.
- [ ] Cero errores nuevos en consola/red durante uso normal.

**Rollback progresivo:**
1. Primero: revertir el gating del frontend si genera un problema de UX (ej. demora perceptible al abrir la app).
2. Segundo: revertir el cambio de políticas (`own_data`/`group_read`) si algo depende de un caso no anticipado.
3. Último recurso: quitar el constraint `NOT NULL` si bloquea algún flujo no previsto.

---

## Bloque K — Corrección de la creación automática de perfil

- **Objetivo:** que `loadProfile()` cree correctamente la fila de perfil de cada usuario en `profiles` — hoy falla siempre, silenciosamente.
- **Problema que resuelve:** `profiles` tiene 0 filas pese a que la función corre en cada login desde que existe. Salió a la luz al validar el Bloque B: `createGroup()` (ya corregido para mandar `created_by`) falla porque `groups.created_by` tiene una foreign key hacia `profiles.id`, y Diego no tiene fila ahí.
- **Origen:** no es una regresión de esta sesión ni del Bloque B — está roto desde el primer commit que introdujo el feature (`1953096`, 06-jun-2026). Ver más abajo, "Corrección de auditoría".
- **Cómo se descubrió (cronología completa, para trazabilidad):**
  1. Validando Bloque B, crear un grupo de prueba dio `403` con `Prefer: return=representation`.
  2. Se descartaron metódicamente: texto de la política `groups_insert`, tipo de dato de `created_by`, JWT/identidad, mecanismo de `auth.uid()` (confirmado funcionando via lectura real de `watchlist`), triggers en `groups`, y rol/comando de la política (`polcmd`/`polroles` verificados por SQL de solo lectura) — todos descartados con evidencia.
  3. Sacar `Prefer: return=representation` reveló el error real: `409`, `23503`, foreign key `groups_created_by_fkey` — *"Key is not present in table profiles"*.
  4. Con permiso explícito de Diego, se reprodujo el POST real de `loadProfile()` — reveló `PGRST204: "Could not find the 'display_name' column of 'profiles' in the schema cache"`.
  5. Sondeo columna por columna (mismo método que `owner_id`/`created_by`) confirmó que `display_name` y `email` no existen.
- **Estado:** Finalizado (30-jul-2026). Auditoría → modelo de identidad → diseño por 5 flujos → implementación → validación, completo. Ver "Implementación y validación" más abajo.

### Corrección de auditoría (transparencia, 30-jul-2026)

Durante el diseño de **Bloque B**, la auditoría de columnas de `profiles` afirmó que `display_name` y `email` existían, citando "consultas de datos" como evidencia. Eso era **incorrecto**: lo que realmente se hizo en ese momento fue un `grep` sobre el código fuente (confirmando que el código *usa* esos nombres), no una consulta contra la base real — se redactó como si fueran equivalentes, y no lo eran. No se detectó en su momento porque el diseño de `profiles_read` de Bloque B se apoya en `id`, no en `display_name`/`email`, así que nada de lo implementado después ejerció esa suposición hasta que, en este bloque, se probó el INSERT real.

**Esquema real de `profiles`, confirmado columna por columna contra la base (fuente de verdad — no el código):**

| Columna | ¿Existe? |
|---|---|
| `id` | ✅ |
| `username` | ✅ (existe, el código nunca la usa hoy) |
| `created_at` | ✅ |
| `display_name`, `email`, `full_name`, `nickname`, `avatar_url`, `updated_at`, `name`, `nombre`, `bio` | ❌ |

**Regla general que deja esto para el resto del proyecto:** el esquema real de la base (verificado con `select=<columna>` contra el endpoint de datos, nunca contra el endpoint OpenAPI que requiere `service_role`) es la única fuente de verdad sobre qué columnas existen — las referencias del código son una hipótesis a verificar, no evidencia por sí solas.

### Modelo de identidad (diseño, 30-jul-2026)

Antes de tocar código se resolvió el modelo, no las columnas puntuales:

```
auth.users (Supabase Auth — fuente de verdad de la cuenta)
    id · email · password · session

              ↓ (FK profiles_id_fkey, ON DELETE CASCADE)

profiles (Archivo — solo lo que auth.users no puede exponer entre usuarios)
    id · username · created_at
```

- Archivo solo necesita un dato de identidad propio: un **nombre visible**, mostrado a vos mismo y a compañeros de un grupo compartido.
- `email`/contraseña/sesión son responsabilidad exclusiva de `auth.users` — no se duplican en `profiles`. `currentUser.email` ya está disponible en memoria sin consulta extra.
- **Decisión: sin cambio de esquema.** Se reutiliza la columna `username` ya existente (confirmado sin `UNIQUE`, solo `PRIMARY KEY(id)` y la FK a `auth.users`). **En Archivo, `username` representa el nombre visible del usuario — no un handle público ni único.** Es la única semántica que tiene esa columna en este producto.
- Crear una columna nueva (`display_name`) se descartó: la única ventaja real hubiera sido evitar esta nota de documentación, a cambio de una migración innecesaria sobre una tabla con 0 filas. Las políticas de RLS no se ven afectadas por la elección (están scopeadas por `id`, no por nombre de columna).

### Diseño por flujos funcionales (aprobado 30-jul-2026)

**Flujo 1 — Alta automática del perfil**
- Objetivo: que el perfil se cree realmente al loguearse o registrarse, sin fallar en silencio.
- Estado actual: `loadProfile()` (`index.html:2718`) y `signInWithPassword()` (`index.html:2809`) escriben `{display_name, email}` — ambas columnas inexistentes, ambos POST fallan. `.catch(()=>{})` solo atrapa fallas de red; el resultado de `signInWithPassword()` tampoco se chequea. El bug queda invisible porque `loadProfile()` igual devuelve un `fallback` armado en JS.
- Estado esperado: ambos POST mandan `{id, username}`. **Ningún error de persistencia puede quedar silencioso, sea cual sea su origen** (HTTP, red, o cualquier otro) — no alcanza con chequear `r.ok`; el manejo de errores debe cubrir cualquier forma en que la escritura pueda fallar.
- Riesgos: bajo — es agregar verificación, no cambiar lógica de negocio.
- Alcance: `loadProfile()`, `signInWithPassword()`.
- Validaciones: login real → fila creada en `profiles` con `username` poblado; signup de prueba → mismo resultado por el segundo camino.
- Rollback: revertir el cambio de código; no toca esquema ni políticas.

**Flujo 2 — Edición del nombre visible**
- Objetivo: que "Cambiar mi nombre" persista de verdad.
- Estado actual: `changeDisplayName()` (`index.html:2842`) hace `PATCH {display_name}` — falla sin chequeo, pero actualiza `currentUser._profile` en memoria igual, mostrando éxito falso hasta la próxima sesión.
- Estado esperado: `PATCH {username}`, con el mismo principio del Flujo 1 (ningún fallo de persistencia queda silencioso).
- Riesgos: mínimo.
- Alcance: `changeDisplayName()`.
- Validaciones: cambiar nombre → recargar/re-loguear → confirmar que persiste, no solo que se ve bien antes de recargar.
- Rollback: revertir el cambio de código.

**Flujo 3 — Lectura del perfil propio**
- Objetivo: que la app lea el nombre real persistido, no un valor derivado en memoria.
- Estado actual: `updateUserAvatar()` (`index.html:2727`) lee `profile.display_name`; siempre recibe el `fallback` en memoria porque nunca existió una fila real.
- Estado esperado: `updateUserAvatar()` lee `profile.username`. **`profiles.username` pasa a ser la única fuente de verdad del nombre visible.** El objeto `fallback` de `loadProfile()` existe únicamente para mantener operativa la sesión si la persistencia falla en ese momento — no es una segunda fuente de verdad, es una degradación temporal mientras dura la sesión.
- Riesgos: bajo.
- Alcance: `updateUserAvatar()`, forma del objeto `fallback`.
- Validaciones: con el Flujo 1 validado, confirmar que el nombre mostrado viene de la fila real, no del prefijo del email.
- Rollback: revertir el cambio de código.

**Flujo 4 — Visualización de miembros de un grupo**
- Objetivo: que el nombre de cada miembro se resuelva desde el dato real.
- Estado actual: `renderGroupMembers()`/`renderGroupCompareList()` (`index.html:2926`, `:2954`) piden `profiles(display_name,email)` — columnas inexistentes; fallaría directamente si Grupo tuviera datos reales.
- Estado esperado: joins piden `profiles(username)`; resolución de nombre pasa a `p.username || 'Anónimo'`. **Sin fallback a email** — si no hay `username`, el resultado esperado es "Anónimo", coherente con el modelo de identidad (email no pertenece a este dato ni se expone entre usuarios).
- Riesgos: bajo. Cambio de comportamiento menor y deliberado: un miembro sin `username` se ve como "Anónimo" en vez de con su email.
- Alcance: las dos queries + las dos líneas de resolución de `name`.
- Validaciones: Grupo no tiene uso real hoy — validar con cuenta de prueba adicional en un grupo compartido, o diferir la validación funcional completa sin que eso bloquee el resto.
- Rollback: revertir el cambio de código.

**Flujo 5 — Compatibilidad y comparación entre miembros**
- Objetivo: ninguno — sin cambios.
- Estado actual/esperado: `compareWithMember()` recibe el nombre ya resuelto como parámetro del Flujo 4; solo consulta `watchlist`, nunca `profiles`.
- Riesgos/Alcance/Validaciones/Rollback: n/a — hereda el resultado del Flujo 4.

### Implementación y validación (30-jul-2026)

- **Flujo 1:** implementado (`loadProfile()`, `signInWithPassword()`). Validado con datos reales: la fila de perfil se crea de verdad (`{id, username: "die.zaga", created_at, avatar_color}` — `avatar_color` es una columna con default propio, no usada por el código, anotada sin acción). Sin errores en consola.
- **Flujo 2:** implementado (`changeDisplayName()`). Primera validación en el navegador de vista previa automatizado falló por una limitación de esa herramienta (`window.prompt()` no soportado ahí — ver nota abajo), no del código. Revalidado en Safari real: cambio de nombre persiste después de recargar. **Validado.**
- **Flujo 3:** implementado (`updateUserAvatar()`). Validado junto con el Flujo 1 — el nombre mostrado viene de la fila real.
- **Flujo 4:** implementado (los dos joins + resolución de nombre). **Validación funcional diferida** tal como estaba previsto en el diseño — Grupo no tiene datos reales hoy, no hay forma de probarlo con un segundo miembro real. No bloquea el cierre del bloque.
- **Flujo 5:** sin cambios, nada que validar.

**Nota sobre herramientas de prueba:** el navegador automatizado usado para preview tiene `window.prompt()` explícitamente deshabilitado (`Error: prompt() is not supported`) — no soporta diálogos nativos bloqueantes. No afecta a usuarios reales en ningún navegador normal. Cualquier flujo que dependa de `prompt()`/`confirm()`/`alert()` nativos necesita validarse en un navegador real, no en ese panel.

**Estado final: Bloque K finalizado (30-jul-2026).** Flujos 1, 2 y 3 implementados y validados con evidencia real. Flujo 4 implementado, validación funcional diferida (sin datos de Grupo reales). Flujo 5 sin cambios.

---

## Bloque L — Grupo: alta y unión sin ampliar la superficie de lectura

- **Objetivo:** que crear un grupo y unirse a un grupo funcionen, sin que ninguna operación previa a la membresía exponga más que lo indispensable.
- **Problema que resolvía:** `createGroup()` usa `Prefer: return=representation` para leer el grupo recién creado — esa lectura pasaba por `groups_read`, que exigía ser miembro (todavía no lo sos en el instante del insert). Postgres trata `INSERT ... RETURNING` como atómico: si la lectura de vuelta falla por RLS, se revierte el insert completo. `joinGroup()` tenía el mismo problema de fondo: para buscar un grupo por código de invitación necesitaba `groups_read`, que también exige membresía — con lo cual nunca podía encontrar ningún grupo, ni con el código correcto.
- **Cómo se descubrió:** al revalidar Bloque B después de cerrar Bloque K. Confirmado con evidencia: el mismo insert con `return=minimal` daba `201` limpio; con `return=representation` daba `403` con el mismo mensaje genérico de RLS que ya conocíamos.
- **Modelo de dominio definido antes del diseño técnico:** quién puede leer un grupo antes de ser miembro (nadie, en general — solo el creador ve el suyo), quién puede crear uno (cualquier usuario autenticado), quién puede descubrir uno por código (nadie por lectura general — solo mediante una operación puntual de "resolver invitación"), cuándo se pasa a ser miembro (al crear, o al unirse con código válido), y qué operaciones necesitan membresía previa (todo excepto crear y resolver una invitación).
- **Comparación explícita antes de decidir la implementación:** para crear grupo, ajustar `groups_read` alcanza y no cede nada (Opción A). Para unirse, RLS no puede expresar "mostrale este grupo a un no-miembro solo si conoce el código correcto" — cualquier política lo bastante permisiva para eso también permite listar todos los grupos a cualquier cuenta. Se optó por una función `SECURITY DEFINER` mínima, de solo lectura, únicamente para esa pieza puntual (Opción B acotada) — no se reemplazó todo el dominio por funciones.
- **Diseño aprobado, con estos criterios explícitos:** la función representa el concepto **"resolver una invitación"**, no un wrapper de `SELECT ... WHERE invite_code`. Hoy la validez se reduce a "existe el código"; el día que haya expiración o límite de usos, la condición se amplía adentro de la función sin que el llamador cambie. La función **no debe acumular lógica de negocio ajena** (cupos, permisos, auditoría) — si aparece esa necesidad, se evalúa en ese momento si sigue perteneciendo acá o amerita otra abstracción.
- **Implementado:**
  ```sql
  -- groups_read: TO authenticated (no public), agrega OR created_by = auth.uid()
  -- resolve_invitation(p_code text) RETURNS TABLE(id uuid, name text), SECURITY DEFINER,
  --   REVOKE de PUBLIC, GRANT solo a authenticated
  ```
  `joinGroup()` cambia su primera consulta para usar `rpc/resolve_invitation` en vez del `GET` directo. `createGroup()` no necesitó ningún cambio de código.
- **Validado con evidencia real:**
  - `groups` INSERT + `return=representation` → `201`, fila completa (el problema de RETURNING está resuelto).
  - `resolve_invitation` con código inexistente → `[]`. Con código real → `{id, name}` exacto, nada más.
  - `invite_code` confirmado `UNIQUE` a nivel de schema (`groups_invite_code_key`) — no hace falta `LIMIT 1`, la garantía es más fuerte que eso.
  - `GET groups?select=*` sin filtro, sin membresías → sigue vacío (no se amplió la superficie de lectura hacia otros usuarios).
- **Rollback:** revertir `joinGroup()` al GET directo; `DROP FUNCTION resolve_invitation`; revertir `groups_read` a su forma original. Este último paso **solo reintroduce el bug funcional de `createGroup()` — no reabre ningún hallazgo de seguridad (SEC-1/SEC-3)**, porque `OR created_by = auth.uid()` nunca amplió visibilidad hacia otros usuarios.
- **Estado:** Finalizado (30-jul-2026).

### Hallazgo relacionado, NO resuelto acá (mismo criterio: un problema, un bloque)

Al completar la validación (agregar la membresía del creador), apareció un tercer caso del mismo patrón que `owner_id`/`display_name`: **`group_members.role` no existe.** Las únicas columnas reales de `group_members` son `group_id` y `user_id` — sin `role`, sin `id`, sin `created_at`. El código asume `role` en 3 lugares (`createGroup()` escribe `'owner'`, `joinGroup()` escribe `'member'`, `renderGroupMembers()` muestra un badge "admin" si `role === 'owner'`).

**Pregunta de auditoría respondida antes de cerrar este hallazgo (pedido explícito de Diego): ¿el producto necesita distinguir owner/member como dato propio, o es una suposición de código nunca implementada?**

Revisando el código completo: `role` se **lee en un solo lugar** de toda la app (el badge "admin"). Nada más depende de esa distinción — ningún permiso, ninguna función bloqueada a "solo el owner puede...". Y **`groups.created_by` ya captura exactamente esa misma información** (quién creó el grupo). No hace falta ningún dato nuevo: "¿es esta persona el owner?" se responde comparando `member.user_id === group.created_by`, sin duplicar nada.

**Conclusión de la auditoría: `role` no es un concepto que el producto necesite como dato almacenado — es una suposición de código que nunca se implementó en el esquema, y lo único que sustentaba (el badge) ya es derivable de un dato que existe.** Queda como hallazgo pendiente, sin bloque ni alcance técnico asignado todavía — esa decisión (sacar `role` del código vs. agregar la columna) se toma en su propia auditoría/diseño, no acá.

**Housekeeping pendiente (Diego, vía dashboard):** quedaron grupos de prueba sin limpiar ("Prueba Bloque K 3", "Validacion Grupo L") — no se pueden borrar vía API (sin política de DELETE en `groups`, hallazgo ya registrado aparte).

---

## Bloque D — Aislamiento de `group_members` vía invitación validada server-side

- **Objetivo:** que solo pueda unirse a un grupo quien tenga el código de invitación correcto, validado del lado del servidor.
- **Problema que resuelve:** hoy cualquiera puede insertarse en cualquier grupo sin RLS ni validación real.
- **Valor:** Bajo (revisado 2026-07-30) — Bloque 0 confirmó que Grupo no se usa hoy y no debe condicionar prioridades. El riesgo de seguridad (SEC-3) sigue siendo válido en teoría, pero sin datos reales en juego.
- **Riesgo:** Bajo-Medio — toca la única vía de alta a `group_members`.
- **Esfuerzo:** Medio.
- **Dependencias:** requiere que el Bloque B esté hecho (RLS en `group_members` es prerequisito lógico). Ya no depende de Bloque 0 — resuelto: Grupo no se usa, se difiere.
- **Documentos de auditoría que lo respaldan:** SECURITY_AUDIT_RLS.md § Riesgo 3; MASTER_AUDIT.md § 5.
- **Estado:** Pendiente, prioridad baja — no se diseña hasta que Grupo pase a usarse de verdad o cambie el contexto.

---

## Bloque E — "Directores favoritos": poblar o remover

- **Objetivo:** que la sección deje de estar muerta.
- **Problema que resuelve:** la UI de ADN promete una sección que nunca tiene datos para películas/series.
- **Valor:** Bajo-Medio — coherencia de producto, no crítico.
- **Riesgo:** Bajo — es aditivo (traer créditos de TMDB) o sustractivo (ocultar sección); ninguno toca datos existentes.
- **Esfuerzo:** Bajo.
- **Dependencias:** ninguna.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 6.
- **Estado:** Finalizado (30-jul-2026).

### Reverificación antes de implementar

Confirmado con evidencia fresca que el hallazgo seguía vigente (ninguno de los bloques B/K/L/F tocó este código): `director` no se escribe en ningún alta, `tmdbDetail()` nunca pide créditos a TMDB, y la columna `director` no existe en `watchlist` (`42703`).

### Decisión: remover, no poblar

Costo de poblarla (ampliar esquema, cambiar el flujo de alta, migración del histórico) desproporcionado frente al valor de una sola sección de ADN. Mismo criterio que `role` en Bloque L: antes de ampliar el modelo, preguntar si el producto lo necesita — acá la respuesta fue no, por ahora. Si en el futuro se decide profundizar el análisis cultural del ADN (directores, actores, compositores), se reabre como una ampliación deliberada del modelo, no como este arreglo puntual.

### Hallazgo adicional, resuelto en el mismo cambio (no ameritó bloque aparte)

"Autores favoritos" (la sección equivalente para libros, que comparte contenedor DOM con "Directores") **tampoco estaba funcionando** — el contenedor solo se insertaba en el HTML si `topDirs.length > 0`, algo que nunca pasaba, así que el elemento nunca existía y `adnSwitchType()` no podía encontrarlo ni para libros. Como es la misma pieza de código que había que tocar para sacar "Directores favoritos", se resolvió en el mismo cambio: el contenedor ahora está siempre presente en el HTML (oculto por defecto), y `adnSwitchType()` lo muestra/completa correctamente al filtrar por Libros.

**Validado en el navegador:** sección oculta en "Todo"/"Películas"/"Series"; al filtrar "Libros" muestra datos reales (Anna Todd, Stephenie Meyer, John Katzenbach, Timur Vermes, J. K. Rowling) — funcionando por primera vez. Sin errores de consola.

---

## Bloque F — Exportación de datos

- **Objetivo:** que puedas sacar una copia de tu archivo.
- **Problema que resuelve:** hoy no existe ninguna forma de backup fuera de Supabase.
- **Valor:** Alto (revisado 2026-07-30) — Bloque 0 confirmó la intención de que Archivo sea "el archivo cultural definitivo" para consolidar toda su historia cultural. Sin backup, esa promesa depende enteramente de la disponibilidad de Supabase — el valor de mitigar esa pérdida sube en consecuencia.
- **Riesgo:** Bajo — operación de solo lectura, no modifica datos existentes.
- **Esfuerzo:** Bajo-Medio.
- **Dependencias:** ninguna.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 6.
- **Estado:** Finalizado (30-jul-2026).

### Diseño e implementación

Solución mínima: `items` ya está cargado en memoria en cada sesión — sin fetch nuevo, sin backend. `exportData()` serializa a JSON con estructura versionada (`{version: 1, exported_at, items}`, no solo el array crudo) para poder evolucionar el formato sin romper compatibilidad hacia atrás, y dispara una descarga vía Blob + `<a download>`. Nombre de archivo con fecha: `archivo-export-YYYY-MM-DD.json`. Botón "Exportar datos" en el menú de cuenta, junto a "Importar CSV".

**Validado:** con datos reales (1000 ítems), estructura del payload correcta, sin errores en consola al ejecutar la función completa (Blob + descarga).

---

## Bloque G — CSV: soporte de libros + fix del parser

- **Objetivo:** que Importar CSV cubra los tres tipos de ítems y no rompa con comillas escapadas.
- **Problema que resuelve:** libros quedan afuera de la única vía de carga masiva; el parser propio tiene un caso borde sin cubrir.
- **Valor:** Bajo-Medio — consistencia de producto, libros ya son de primera clase en todo lo demás. Bloque 0 confirmó intención de "eventualmente otras fuentes", lo que da algo más de contexto sin volverlo urgente.
- **Riesgo:** Bajo — extiende un flujo existente, no lo rediseña.
- **Esfuerzo:** Bajo-Medio — el fix del parser es casi trivial y puede resolverse solo, sin esperar el soporte de libros.
- **Dependencias:** ninguna.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 3, § 4.
- **Estado:** Finalizado (30-jul-2026).

### Reverificación e implementación

Confirmado con evidencia fresca que las dos limitaciones seguían vigentes antes de tocar código (nada las había resuelto indirectamente). `parseCsvLine()` ahora hace lookahead sobre `""` para preservar comillas literales en vez de perderlas en silencio. `runImport()` detecta `type: libro/book` y recorre `booksSearch → booksDetail → buildBookItem`, en paralelo a la rama existente de películas/series.

**Validado:** parser probado con comillas escapadas y coma dentro de campo — preserva ambos correctamente. Rama de libros probada de punta a punta hasta la construcción del ítem final (detección → búsqueda real → detalle real → objeto limpio, con `type:'book'`, autor, género y rating de la fila) — sin ejecutar el `POST` final, porque el mecanismo de persistencia ya está cubierto por el flujo de películas/series existente y no cambió en este bloque. Decisión explícita de Diego: no escribir un dato de prueba en su biblioteca real solo para repetir una validación ya cubierta.

## Bloque H — `tmdb_rating` con semántica doble

- **Objetivo:** que la columna deje de tener un nombre que no describe su contenido real para libros.
- **Problema que resuelve:** deuda de modelo de datos, ya reconocida en un comentario del propio código.
- **Valor:** Bajo — mantenibilidad futura, hoy no rompe nada.
- **Riesgo:** Medio — toca una columna con datos históricos reales, requiere migración, no solo cambio de código.
- **Esfuerzo:** Medio.
- **Dependencias:** ninguna decisión pendiente, pero el costo de migración es alto en relación al beneficio.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 3, § 4.
- **Estado:** Reclasificado (30-jul-2026) — **deuda técnica planificada, no bloque activo del roadmap.** No rompe funcionalidad hoy; el costo de migración no se justifica todavía. Se retoma si aparece una razón concreta (ej. un bug real ligado a la ambigüedad, o una necesidad de producto que la vuelva relevante), no por calendario.

---

## Bloque M — Estabilización de la aplicación

**Cambio de metodología (30-jul-2026), vigente a partir de acá:** este bloque opera distinto a todos los anteriores. No se documenta un diseño previo por cada bug — se identifica la causa, se corrige, se valida y se sigue, todo dentro de este mismo bloque, sin abrir uno nuevo por cada hallazgo. Solo se frena a diseñar aparte si un bug revela un problema estructural serio (modelo de datos, seguridad, o una funcionalidad importante) — el resto se resuelve directamente.

- **Objetivo:** corregir los problemas visibles que afectan el uso diario — no agregar features ni seguir documentando deuda técnica.
- **Orden de trabajo:**
  1. Bugs funcionales (pantallas que no responden, loaders infinitos, errores de flujo).
  2. Bugs de UI/UX (layout roto, tarjetas superpuestas, problemas de PWA).
  3. Rendimiento.
  4. Deuda técnica pendiente, solo si aporta valor real — al final, no antes.
- **Estado:** Cerrado y **desplegado** (30-jul-2026). Regresión completa en verde sobre todos los flujos principales; `index.html` publicado a producción (subida manual de Diego) y verificado en vivo. Con esto se cierra la etapa de estabilización técnica de Archivo. Log de lo corregido más abajo. **Próxima etapa: auditoría de producto/UX (foco fuera del backend).**

### Log de correcciones

- **30-jul-2026 — Condición de carrera en Descubrí (`renderDisc`/`renderBookDisc`).** Cambiar rápido entre Películas/Series/Libros disparaba llamadas async superpuestas sin protección; la que terminaba último pisaba el contenido, sin importar qué pestaña estuviera activa — reproducido con evidencia (pestaña "series" activa, contenido de "películas" en pantalla). Fix: snapshot de `discType` al entrar, se descarta la respuesta si el usuario ya cambió de pestaña antes de que termine.
- **30-jul-2026 — Misma condición de carrera en el buscador (`doSearch`).** Tipear rápido podía dejar en pantalla resultados de una búsqueda anterior más lenta, pisando los de la búsqueda actual. Reproducido y corregido con el mismo patrón (se descarta la respuesta si el input ya no coincide con la query que la originó).
- **30-jul-2026 — Selects de Biblioteca cortados en mobile.** En el layout de 3 columnas por fila, "Estado: todos" y "Década: todas" no entraban en el ancho disponible y se veían truncados ("Estado: tod"), mientras "Género" (ya con etiqueta corta) se veía bien. Reproducido visualmente en viewport mobile (375px) y corregido acortando las etiquetas a "Estado"/"Década", consistente con el patrón que ya usaba "Género". Sin cambio de `value`, el filtro sigue funcionando igual.
- **30-jul-2026 — El modal de detalle "crecía" 103px después de abierto.** `loadWatchProviders()` corre sin esperar y recién agrega la sección "Dónde verla" cuando responde TMDB — todo lo de abajo (calificación, notas, botones) se corría de golpe. Medido con evidencia (1010px→1113px al abrir). Fix: reservar el espacio con un placeholder desde el inicio; el salto bajó a 33px (mucho menos perceptible) y se colapsa limpio si no hay datos.
- **30-jul-2026 — El scroll de fondo se filtraba detrás de los modales (iOS).** `openModal()`/`closeModal()` no bloqueaban el `body` — en iOS Safari esto permite que la página de atrás siga scrolleando mientras un modal está abierto, y al cerrarlo aparece "saltado" a otra posición. Fix: fijar el body en su scroll actual mientras hay un modal abierto (técnica estándar para iOS) y restaurar la posición exacta al cerrar. Validado: intento de scroll de fondo mientras el modal está abierto queda bloqueado, posición se restaura exacta al cerrar.
- **30-jul-2026 — Investigado, sin confirmar:** un patrón de "pantalla desactualizada" apareció en capturas tomadas justo después de `switchTab()` durante las pruebas, pero el estado del DOM ya era correcto en ese momento (verificado por separado) y no hay ninguna transición CSS en `.pane` que explique una demora real. Es más probable que sea un artefacto de la herramienta de testing (screenshot y ejecución de JS son llamadas separadas) que un bug real de la app — no se aplicó ningún cambio para esto, queda anotado por transparencia, no como corregido.
- **30-jul-2026 — CRÍTICO: la app solo cargaba 1000 de 1883 filas (causa raíz de "no me deja guardar películas" + contadores incorrectos).** `loadItems()` traía la watchlist sin paginar y PostgREST corta en 1000 filas por respuesta. Las ~883 filas no cargadas (muchas del import de películas) no entraban a `items`, así que: (a) al re-buscarlas, `selectResult` no las encontraba, la app las trataba como nuevas, e intentaba insertar → chocaba contra `UNIQUE (tmdb_id, type)` → "Error al guardar"; (b) los contadores se calculaban sobre 1000, no sobre el total real. Evidencia: 1883 filas en base (un solo dueño) vs ~1000 en la app. Fix: `loadItems()` pagina (limit/offset, páginas de 1000) hasta traer todo, robusto a cualquier tamaño. **Validado por Diego con datos reales: total 1883, películas 1589, series 266, libros 28; películas del import (Ant-Man, etc.) ya se guardan y aparecen; contadores consistentes.** Una sola causa raíz resolvió Prioridad 1 y Prioridad 2.

---

## Etapa 2 — UX y producto

**Inicio:** 15-ago-2026. Cierra la etapa de estabilización técnica (Bloque M). A partir de acá el criterio de prioridad no es severidad técnica sino impacto en los flujos de uso diario, y el mandato incluye cuestionar el producto en el camino (simplificar/eliminar lo que no aporte valor, evaluar funcionalidad nueva solo si mejora la experiencia real) — no solo corregir. Nace de [UX_AUDIT.md](UX_AUDIT.md). Orden de trabajo definido por Diego: (1) flujo de buscar y agregar, (2) sensación de movimiento de la interfaz, (3) filtro de género, (4) navegación de Biblioteca a escala, (5) teclado/layouts/detalles de mobile.

**Principio activo durante toda la etapa (confirmado por Diego, 15-ago-2026):** cuestionar permanentemente si cada pantalla, animación, transición, modal o interacción aporta valor real — no mantener algo solo porque "siempre estuvo ahí". Si algo genera fricción o movimiento innecesario, se simplifica aunque no esté escrito en un documento. Toda propuesta de cambio (o de no cambiar algo) se mide con una pregunta única: **¿hace que usar Archivo todos los días sea más rápido o más agradable? Si la respuesta es no, no es prioridad ahora.** Esto aplica a oportunidades que aparezcan durante la implementación, no solo a lo ya listado en UX_AUDIT.md.

**Cambio metodológico confirmado por Diego (15-ago-2026, tras validar el Bloque O): los bloques de esta etapa se enmarcan por dimensión de calidad percibida, no solo por área funcional.** Hasta Bloque N el criterio de agrupación era "qué parte de la app" (buscar/agregar, biblioteca, grupo). Desde Bloque O, el criterio pasa a ser también "qué sensación se está construyendo o eliminando" — ejemplos que dio Diego: Bloque O = eliminar sensación de movimiento; un futuro bloque de cantidad de clics; uno de velocidad para agregar contenido; uno de consistencia visual; uno de pulido desktop; uno de pulido mobile. La idea explícita: "ya no estás arreglando funciones, estás construyendo una sensación" — el objetivo no es que cada pantalla funcione, sino que la app pase de "está buena" a "qué bien hecha está esta aplicación". **Cómo se aplica:** cada bloque nuevo de esta etapa debe poder nombrarse por la sensación que ataca (no solo por la función que toca), aunque el criterio de PRIORIDAD sigue siendo el orden ya fijado por Diego (buscar/agregar → movimiento → género → biblioteca → mobile) — esto no reemplaza ese orden, es una forma de enmarcar y nombrar cada bloque a medida que se abre.

---

## Bloque N — Flujo de buscar y agregar contenido

- **Objetivo:** reducir la fricción del flujo que más se usa a diario — buscar algo y guardarlo en el archivo.
- **Problema que resuelve:** UX_AUDIT.md hallazgos #3b (ruido en resultados), #4 (sin vuelta atrás), #6-parcial (idioma inconsistente en la ficha), #7 (estado por defecto).
- **Valor:** Alto — es la Prioridad 1 explícita de la nueva etapa.
- **Riesgo:** Bajo — cambios de UI/flujo y de un criterio de selección de datos ya presentes en la respuesta de TMDB; no tocan `watchlist` ni requieren backend.
- **Esfuerzo:** Bajo-Medio, variable por sub-problema (ver abajo).
- **Dependencias:** ninguna.
- **Documentos que lo respaldan:** UX_AUDIT.md.
- **Estado:** Finalizado e implementado (15-ago-2026), commit `b14b3a9`. Pendiente solo el deploy manual de Diego (no bloquea el resto de la Etapa 2). Alcance confirmado: **no incluye** buscar por actor/director/autor (ver más abajo).

### Implementado

- **Sub-problema 1 (ranking):** `tmdbSearch()` ahora ordena por `popularity` descendente antes de devolver los primeros 10. Validado con datos reales: "Inception" pasó de un orden sin criterio (Origen, Bikini Inception, Inception: The Cobol Job...) a estrictamente descendente por popularidad (57.09, 5.09, 1.67, 1.35...).
- **Sub-problema 2 (sin vuelta atrás):** nueva función `cancelAddFlow()` — "Cancelar" en la ficha de un ítem **nuevo** (no en edición) vuelve al modal de búsqueda con la misma query y el mismo filtro de tipo, y re-ejecuta la búsqueda. En edición, "Cancelar" sigue cerrando todo como antes (no aplica "volver a resultados" porque no hay a dónde volver). Validado con datos reales: buscar "Poor Things" → seleccionar resultado → Cancelar → vuelve al modal con "Poor Things" y los 7 resultados de nuevo.
- **Sub-problema 3 (estado por defecto):** los 4 puntos donde se arma un ítem nuevo para agregar (búsqueda de películas/series, búsqueda de libros, quick-add de Descubrí para ambos) fuerzan `status:'watchlist'` después de construir el ítem. **No se tocó** el default de `buildTmdbItem`/`buildBookItem` en sí — esas funciones las usa también el import de CSV, donde "watched" sigue siendo el default correcto (películas ya vistas del historial). Validado con datos reales: ítem nuevo no existente en biblioteca abre con Estado="Ver después"; ítem ya existente (ej. Interstellar) sigue abriendo con su estado real guardado ("Visto"), sin cambios.
- **Sub-problema 4 (idioma):** causa raíz confirmada y corregida — `tmdbDetail()` ahora prueba título AR → ES → EN en cascada (antes saltaba directo de AR a EN, ignorando España) y arma `genres`/`overview` con la misma cascada explícita en vez de depender del orden de un spread de objetos. Validado con datos reales: "Dune: La profecía" (id TMDB 90228) ahora abre la ficha con ese mismo título en vez de saltar a "Dune: Prophecy" en inglés. **Límite real, no corregible desde acá:** el género y la sinopsis siguen en inglés para este título puntual porque TMDB directamente no tiene traducción al español cargada para ese dato (confirmado pidiendo `es-AR`/`es-ES` por separado: ambos devuelven los géneros en inglés) — la cascada ya elige lo mejor disponible, pero no puede inventar una traducción que TMDB no tiene.

### Pendiente

- Deploy manual (Diego sube `index.html` a `diepizaga.github.io` como siempre).
- Validación en producción real después del deploy.

### Sub-problema 1 — Ruido en resultados de búsqueda (#3b)

`tmdbSearch()` (index.html:1146) pide `search/multi`/`search/movie`/`search/tv` y devuelve los primeros 10 resultados tal cual los entrega TMDB, sin reordenar. TMDB ya incluye un campo `popularity` en cada resultado — no está siendo usado.
- **Opción A (recomendada):** ordenar los resultados por `popularity` descendente antes de mostrarlos. No requiere ningún llamado nuevo a la API, el dato ya viene en la respuesta.
- **Opción B:** filtrar directamente resultados por debajo de un umbral de popularidad. Más agresivo, riesgo de esconder un título real pero poco popular (ej. algo indie que sí tenés).
- Con A alcanza para resolver el caso observado (Inception arriba, "Bikini Inception" abajo en vez de mezclado) sin arriesgar ocultar nada.

### Sub-problema 2 — Sin vuelta atrás desde la ficha de confirmación (#4)

Hoy "Cancelar" en la ficha de confirmación cierra todo el flujo (`closeModal`) en vez de volver a la lista de resultados.
- **Opción A (recomendada):** que "Cancelar" desde la ficha vuelva a los resultados de búsqueda (conservando la query y el scroll), no cierre todo. Es más intuitivo — "cancelar" debería cancelar el paso actual, no el flujo entero.
- **Opción B:** agregar un botón separado "← Volver" además de "Cancelar" (que sí cerraría todo). Más explícito pero suma un botón más a una ficha que ya tiene varios.
- Recomiendo A por ser más simple y no agregar superficie nueva.

### Sub-problema 3 — Estado por defecto siempre "Visto" (#7)

`buildTmdbItem()` (index.html:1193) fija `status:'watched'` de forma incondicional para cualquier alta nueva.
- **Opción A (recomendada):** default a `'watchlist'` ("Ver después") — probablemente el caso más común al agregar algo recién descubierto.
- **Opción B:** sin default, forzar elección explícita en cada alta.
- Recomiendo A: menos clics en el caso común, sin agregar un paso obligatorio nuevo.

### Sub-problema 4 — Idioma inconsistente entre resultados y ficha (#6, parcial)

Causa identificada en `tmdbDetail()` (index.html:1158-1171): la función pide el detalle en `es-AR`, `es-ES` y `en-US` en paralelo, pero el título final solo elige entre **AR o EN** (`bestTitle = isLatam ? titleAR : titleEN`) — nunca contempla `titleES` (España) como intermedio. Si un título no tiene traducción específica de AR pero sí tiene una de ES, el código la ignora y cae directo a inglés. Los géneros tienen un riesgo similar: se arman con `{...resEN, ...resAR}`, así que si el pedido a `es-AR` no devuelve `genres` por algún motivo, quedan los de `resEN` (inglés) sin que se note. Esto último es una hipótesis fundada en cómo está escrito el código, no confirmada todavía con una prueba puntual — la confirmo antes de tocar nada si avanzamos con este sub-problema.
- **Opción A (recomendada):** agregar `titleES` como paso intermedio en la cascada (`AR → ES → EN`) para título y, explícitamente (no por spread implícito), para género también.
- Alcance acotado a la ficha de agregar/editar. El buscador interno de Biblioteca (que también sufre este problema, hallazgo #6) es Prioridad 4 — no se toca en este bloque.

### Pregunta de producto resuelta acá: ¿buscar por actor/director/autor?

Hallazgo técnico relevante: `tmdbSearch()` ya recibe resultados de tipo persona desde `search/multi` (TMDB los devuelve mezclados con películas/series) y el código los descarta explícitamente (`x.media_type!=='person'`, index.html:1152). Es decir, la búsqueda por actor/director en TMDB **ya está a un filtro de distancia**, no es una integración nueva — el costo real no es "conectar con una API nueva" sino diseñar qué pasa cuando el resultado elegido es una persona (ej. mostrar su filmografía vía `/person/{id}/combined_credits` y dejar elegir un título desde ahí). Para libros, Open Library/Google Books ya traen `author_name` en los resultados de búsqueda (index.html:1204), así que ahí el dato también está disponible, falta decidir si se expone como filtro.
**Respuesta de Diego (15-ago-2026):** tiene mucho valor real, no es un "nice to have" — pero no entra en el Bloque N. Primero el flujo de buscar y agregar tiene que quedar sólido; después se arma un bloque específico de exploración (actor/director/autor, filmografías). Queda anotado como **próximo bloque de la Etapa 2 una vez cerrado N** (sin nombre de letra asignado todavía, sin diseño — el hallazgo técnico de arriba, `media_type!=='person'` descartado en `tmdbSearch()`, es el punto de partida cuando llegue el turno).

---

## Bloque O — Reducir la sensación de movimiento de la interfaz

- **Objetivo:** que la interfaz deje de sentirse en movimiento constante — Prioridad 2 de la Etapa 2.
- **Problema que resuelve:** UX_AUDIT.md hallazgo #3 (ticker), más regla de producto vigente desde acá para toda la etapa (ver abajo).
- **Valor:** Alto — Prioridad 2 explícita.
- **Riesgo:** Bajo — cambios de UI/CSS aislados, no tocan datos ni flujos.
- **Esfuerzo:** Bajo-Medio.
- **Dependencias:** ninguna.
- **Documentos que lo respaldan:** UX_AUDIT.md.
- **Estado:** Finalizado (15-ago-2026), commit `bfb3bc2`. Confirmado por Diego: "no se limitó al ticker y atacó varias causas reales de la sensación de inestabilidad... es el enfoque correcto para esta etapa". Pendiente deploy manual. Única validación que queda abierta (no bloquea el cierre del bloque): confirmar en el celular real de Diego el comportamiento con el teclado — si aparece algún caso raro, se ajusta sobre este mismo bloque, no como uno nuevo.

**Regla de producto fijada por Diego para toda la Etapa 2 (no solo este bloque):** cualquier movimiento permanente que no aporte información imprescindible se elimina — no se reemplaza una animación por otra solución cosmética. Ampliación de alcance explícita: no limitarse al hallazgo ya escrito, identificar y eliminar cualquier causa de sensación de inestabilidad (reflows, cambios de altura, scrolls inesperados, modales que empujan contenido, overlays que mueven el fondo, layouts que cambian con el teclado, loaders que mueven la posición de la pantalla). **Nota de alcance:** "layouts que cambian con el teclado" se solapa con el hallazgo #2 de UX_AUDIT.md, que la prioridad original (Prioridad 5) dejaba para el final — por indicación directa de Diego en este bloque, se adelantó y resolvió acá; cuando llegue el turno de Prioridad 5 puede que quede menos pendiente de lo previsto.

### Investigación antes de proponer (para no limitar el bloque solo al hallazgo ya escrito)

Revisé el resto de fuentes de movimiento de la app antes de acotar el alcance a la barra: las 39 transiciones CSS del archivo son casi todas de foco/hover (`.15s`-`.4s` en color/borde/transform de tarjetas y botones), disparadas por interacción del usuario, no movimiento ambiental — no encontré evidencia de que generen la sensación de inestabilidad que describiste. `switchTab()` ya resetea el scroll de forma instantánea (`behavior:'instant'`), sin salto animado. El único movimiento verdaderamente constante e independiente de la interacción del usuario es el ticker — por eso el bloque queda acotado a él.

### El ticker, mirado de cerca

`updateTicker()` (index.html:1627) arma el texto con: cantidad de películas, series, libros, promedio de calificación, pendientes, y el arquetipo ("El explorador ecléctico"). Casi todo esto **ya está mostrado en otro lado, sin movimiento**: películas/series/libros/promedio están en el bloque "En números" de Inicio; el arquetipo tiene su propia sección completa en ADN. Lo único que no vi repetido en ningún otro lado es el conteo de "pendientes".

- **Opción A (recomendada):** eliminar el ticker por completo. Es información duplicada, envuelta en el único movimiento perpetuo de toda la app — exactamente lo que señalaste. Si "pendientes" es un dato que querés seguir viendo siempre visible, se puede sumar como quinta tarjeta en "En números" (Inicio), sin animación.
- **Opción B:** mantener el texto pero sacarle la animación (`animation: tick 60s linear infinite` → estático). Conserva el detalle editorial del masthead sin el movimiento, a costa de mantener información duplicada.
- **Opción C:** dejar la barra pero pausada por defecto (`animation-play-state: paused`), sin quitarla del todo.

Mi lectura con el litmus test que fijaste ("¿hace que usar Archivo todos los días sea más rápido o más agradable?"): es información redundante en movimiento constante, no aporta nada que Inicio y ADN no den ya sin moverse — Opción A. Antes de tocar código quiero tu confirmación puntual sobre esto porque es sacar un elemento visual identitario del diseño (masthead editorial), no solo un fix.

**Confirmado por Diego (15-ago-2026): Opción A, eliminar por completo, sin dejarlo estático.**

### Implementado

- **Ticker eliminado del todo:** markup (`.ticker-wrap`/`#mh-ticker`), CSS (incluida `@keyframes tick`) y la función `updateTicker()` con sus 3 llamadas removidos. El masthead queda estático.
- **"Pendientes" reubicado:** nueva quinta tarjeta en "En números" (Inicio), estática, sin animación — grid pasa de `repeat(4,1fr)` a `repeat(5,1fr)` en desktop; en mobile se mantiene `repeat(2,1fr)` y la quinta tarjeta ocupa el ancho completo de su fila (`grid-column:1/-1`). Validado visualmente en desktop y mobile: 5 tarjetas parejas en desktop, 2+2+1 en mobile, sin overflow ni recorte.
- **Reflow de Descubrí al cambiar de pestaña (Películas/Series/Libros):** `switchDiscType()` reemplazaba todo el contenido por un loader chico mientras cargaba, sin resetear el scroll — si el usuario estaba scrolleado más abajo, quedaba mirando espacio vacío. Fix: `window.scrollTo({top:0, behavior:'instant'})` al cambiar de tipo, mismo patrón que ya usa `switchTab()`. Validado: scroll baja a 800px, se cambia de tipo, scroll vuelve a 0 antes de que el loader aparezca.
- **Fondo corriéndose al abrir un modal (desktop):** el bloqueo de scroll del body (`position:fixed`, agregado en Bloque M para iOS) hace desaparecer la scrollbar del documento sin compensar su ancho — el header y el resto del contenido se corrían ~4px a la derecha al abrir cualquier modal, y volvían a su lugar al cerrar. Fix: medir el ancho de la scrollbar antes de fijar el body y compensarlo con `padding-right` mientras el modal está abierto. Validado con evidencia real: posición del header idéntica antes/durante/después de abrir un modal (1436px en los tres momentos, antes saltaba a 1440px al abrir).
- **Teclado tapando contenido en mobile (hallazgo #2 de UX_AUDIT.md, adelantado):** no había ningún manejo de `visualViewport` en toda la app. Fix de dos partes: (1) `window.visualViewport.addEventListener('resize', ...)` mantiene una variable CSS (`--kb-vh`) con el alto real del viewport visual, y el modal en mobile usa `max-height: min(92svh, calc(var(--kb-vh)*.92))` en vez de solo `92svh` — así la hoja se achica junto con el teclado en vez de quedar tapada. (2) al enfocar cualquier campo dentro de un modal abierto, se lo trae a la vista con `scrollIntoView` después de un breve delay (tiempo de animación del teclado). Validado: simulando un viewport visual de 400px, el modal se ajusta a 368px (92%) correctamente; enfocar "Notas" en el modal real hace que el campo y los botones de acción queden visibles en pantalla.

### Investigado y descartado (sin evidencia de problema real)

Las 39 transiciones CSS del archivo son de foco/hover (disparadas por el usuario, no ambientales) — no se tocaron. Biblioteca (grilla/lista) y ADN renderizan de forma síncrona desde datos ya en memoria, sin el patrón "loader chico → contenido grande" que sí tenía Descubrí — no aplica el mismo fix ahí. El parpadeo de pósters de Biblioteca (hallazgo #5) es carga de imágenes, no reflow de layout — sigue siendo Prioridad 4, no se tocó acá.

---

## Bloque P — Confiabilidad del filtro de género

*Dimensión de calidad que ataca: que un filtro que existe y se ve prolijo realmente funcione — no "está buena la lista de géneros" sino "el filtro hace lo que promete".*

- **Objetivo:** que el filtro "Género" de Biblioteca sea usable — Prioridad 3 de la Etapa 2.
- **Problema que resuelve:** UX_AUDIT.md hallazgo #1 (~110 opciones sin curar, español/inglés duplicado, tags de libros que no son géneros).
- **Valor:** Alto — Prioridad 3 explícita, es el filtro más visible de la sección más usada.
- **Riesgo:** Bajo — es normalización de datos ya presentes, no un cambio de esquema ni de origen de datos.
- **Esfuerzo:** Bajo-Medio.
- **Dependencias:** ninguna.
- **Documentos que lo respaldan:** UX_AUDIT.md.
- **Estado:** Finalizado (15-ago-2026), pendiente commit y deploy manual. Ver corrección de diseño durante la implementación más abajo — el enfoque final no fue el primero que probé.

### Investigación antes de proponer

- **Los libros ya tienen una función de limpieza (`cleanBookGenre()`, index.html:1639) que hoy no se usa donde hace falta.** Filtra tags con blocklist (`series:`, relaciones familiares como tag suelto, "crimes against", strings no-ASCII de 4+ caracteres — captura alemán/francés/etc., años sueltos, "Autor") y traduce algunos BISAC conocidos vía `BISAC_DISPLAY`. Se usa hoy en `renderBibInsights()` y en una sección de ADN — **pero no en `updateGenreFilter()`**, que es exactamente la función que puebla el combo roto. Ahí está la causa raíz concreta: no falta crear nada, falta invocar lo que ya existe en el lugar correcto.
- **Para películas/series el problema es distinto: no hay ninguna función de canonicalización EN→ES.** Confirmé con una llamada real a TMDB (durante el diseño de Bloque N) que el propio `/tv/{id}` a veces devuelve géneros en inglés aunque se pida en español — no es then algo que podamos arreglar pidiéndole mejor el dato a TMDB, hay que traducir nosotros con un mapa fijo. La lista de géneros de TMDB (película + serie) es corta y estable (~27 valores en total, documentada), así que un mapa manual es exacto y no necesita mantenimiento futuro salvo que TMDB agregue categorías nuevas (no ha pasado en años).
- **`getBibFiltered()` hoy compara con `.includes(fGenre)` (index.html:1488), exacto contra el string crudo guardado.** Cualquier normalización tiene que aplicarse tanto al armar las opciones del filtro como al comparar, si no el filtro se ve prolijo pero deja de encontrar resultados.

### Propuesta

- **Opción A (recomendada):** función `canonGenre(type, g)` — para libros, delega en `cleanBookGenre()` ya existente; para película/serie, traduce vía un mapa fijo EN→ES (`Action`→`Acción`, `Science Fiction`→`Ciencia ficción`, etc.), con fallback al valor original si no está en el mapa. Se usa en dos lugares: `updateGenreFilter()` arma las opciones a partir de valores canonicalizados y deduplicados (en vez del `[...new Set(...)]` crudo actual), y `getBibFiltered()` compara `canonGenre(i.type, g) === fGenre` en vez de comparar el string crudo.
- **Casos combinados de TV sin equivalente exacto en película** (`Action & Adventure`, `Sci-Fi & Fantasy`, `War & Politics`): en vez de forzarlos dentro de una categoría de película (perdiendo la mitad del significado — ej. una serie de solo-fantasía quedaría mal etiquetada como "Ciencia ficción"), se traducen como su propia etiqueta compuesta en español ("Acción y aventura", "Ciencia ficción y fantasía", "Guerra y política"). No se mezclan con las categorías de película que sí están separadas.
- **Límite que no se soluciona con esto (lo dejo anotado, no es parte de este bloque):** un typo real en la fuente de datos ("College stdents" en vez de "College students") no lo agarra ningún blocklist por regex — seguiría apareciendo como su propia opción suelta. Solucionarlo de raíz requeriría una lista blanca curada de géneros válidos en vez de una lista negra de ruido, que es un cambio de enfoque más grande y no se justifica por un solo typo.
- **Opción B (descartada):** curar a mano la lista completa de géneros existentes hoy en la base, como constante estática. Más simple de escribir, pero se desactualiza solo con la próxima película que TMDB categorice distinto o el próximo libro importado — la Opción A se mantiene sola.
- **Estado:** Finalizado (15-ago-2026), pendiente commit y deploy manual.

### Corrección durante la implementación (transparencia, no un detalle menor)

La primera versión de este fix reusaba `cleanBookGenre()` tal cual estaba escrita (lista negra angosta: `series:`, palabras de parentesco sueltas, "crimes against", no-ASCII, años, "Autor"). Validado con datos reales, el combo bajó de 110 a **102** opciones — prácticamente nada, porque la lista negra nunca bloqueaba el grueso del ruido real ("adolescence", "Booksellers and bookselling", "dementors", "fiction"/"Fiction"/"FICTION" como tres entradas separadas, etc.). El lado de película/serie sí funcionó bien desde el primer intento (fusionó Action→Acción, Sci-Fi & Fantasy→Ciencia ficción y fantasía, etc.).
Corregido rediseñando `cleanBookGenre()` de lista negra a **lista blanca**: `BOOK_GENRE_MAP` (index.html) mapea explícitamente las variantes reales observadas (fiction/ficción/FICTION, fantasy/fantasía, suspense/thriller/thrillers, family/families, biography, juvenile/children's/young adult, etc.) a un género final en español; cualquier tag que no está en el mapa se descarta directamente en vez de mostrarse. Es el mismo principio que en Bloque O: no una mejora parcial de lo que ya había, sino sacar la categoría completa del problema.
**Resultado validado con datos reales:** 110 → **27 opciones**, todas géneros reales en español, sin duplicados de idioma, sin tags de tema/trama sueltos. Verificado también que el filtrado sigue funcionando (no solo la lista se ve limpia): "Fantasía" devuelve 290 ítems incluyendo 3 libros de Harry Potter + 287 películas/series; "Terror" devuelve 129 ítems.
**Efecto colateral positivo, no buscado:** `cleanBookGenre()` la usan también `renderBibInsights()` y una sección de ADN — se benefician del mismo arreglo automáticamente, sin tocarlas.
**Límite conocido, no resuelto acá:** un tag que no está en `BOOK_GENRE_MAP` simplemente no aparece como género filtrable (aunque el libro sigue teniendo ese tag guardado en `genres` para otros usos). Es la contracara esperada de pasar a lista blanca — cubre todo lo observado hoy, pero un libro futuro con un subject completamente nuevo no tendrá género hasta que se agregue al mapa. Aceptable: es preferible a que vuelva a inflarse con ruido.

---

## Bloques diferidos — sin diseño formal por ahora

Estos hallazgos están registrados y no se pierden, pero no tienen suficiente definición o urgencia para ser un bloque activo de diseño ahora mismo.

| Hallazgo | Por qué está diferido |
|---|---|
| Modelo de grupos múltiples (antes Bloque I) — PROD-2 | Resuelto por Bloque 0 (2026-07-30): Grupo no se usa hoy y no debe condicionar prioridades. Se retoma si Grupo pasa a usarse de verdad. |
| PWA sin service worker (PROD-8) | Resuelto por Bloque 0: offline explícitamente no es prioridad hoy. |
| Sin paginación en `loadItems()` (PROD-9) | Resuelto por Bloque 0: escala esperada (algunos miles) no lo amerita todavía. |
| `discCache` se invalida por completo en cada escritura (PROD-10) | Optimización interna menor, sin apuro ni dependencia externa. |
| Alias de CSS legacy (PROD-12) | Cosmético interno, sin valor de usuario directo. |
| Scripts `.py` con credenciales (PROD-13 / EP-2) | Depende de tu decisión simple (ignorar vs. sanear) ya registrada en MASTER_AUDIT § 5 — si es "ignorar", no genera bloque. |

---

## Historial de cambios de este plan

- **2026-07-30:** creación del plan a partir de MASTER_AUDIT.md. Todos los bloques en estado Pendiente.
- **2026-07-30:** Bloque 0 finalizado. Reprioritización derivada de la visión de producto: D baja a prioridad baja (Grupo no se usa, no debe condicionar el proyecto); I se mueve a diferidos por la misma razón; F sube a valor Alto (Archivo como "archivo cultural definitivo" sin backup hoy); B y C se reafirman como la prioridad más alta del plan.
- **2026-07-30:** Orden de trabajo ajustado por Diego: Bloque A pasa a ejecutarse justo antes de empezar la implementación (es preparación, no diseño de producto), no antes del diseño de B. Flujo acordado: (1) diseño de B, (2) diseño de C, (3) housekeeping de A, (4) implementación por etapas. Mantiene la separación explícita entre etapa de diseño y etapa de implementación.
- **2026-07-30:** Diseño de Bloque B aprobado. Regla de disciplina de alcance confirmada explícitamente: un hallazgo detectado *durante* el diseño o la implementación de un bloque (ej. `profiles` en 0 filas) no amplía el alcance de ese bloque — se valida/observa dentro de él, y si resulta ser un problema real, se abre un bloque nuevo e independiente. Arranca el diseño de Bloque C.
- **2026-07-30:** Diseño de Bloque C aprobado, con un cambio de dirección respecto de la propuesta inicial: se descarta el mecanismo de "claim" server-side (no hay forma de demostrar propiedad de una fila huérfana) a favor de prevenir su creación de raíz + constraint `NOT NULL`. Confirmado sin filas huérfanas existentes en producción. Con B y C diseñados, sigue el housekeeping de Bloque A, y después implementación por etapas.
- **2026-07-30:** Checklist de pre-implementación corrido sobre B y C antes de ejecutar Bloque A. Encontrados y corregidos dos huecos: dependencia de B desactualizada (ya resuelta, seguía marcada pendiente) y ausencia total de criterio de éxito/rollback en C (agregados). A partir de acá, el ciclo de ejecución es Implementación → Validación → Commit, por bloque, sin mezclar etapas.
- **2026-07-30:** Bloque A ejecutado y finalizado, con alcance acotado explícitamente: tracking + `.gitignore` + borrado de `.bak` + versionado de `docs/`. Destino de `cinelog.git` diferido sin bloquear el proyecto. **Hallazgo nuevo durante la ejecución (ver MASTER_AUDIT § 2, EP-7): `import_movies.py` contiene la `service_role` key de Supabase** (no la `anon key`) — severidad crítica, distinta e independiente de todo lo demás, pendiente de que Diego la regenere. No bloquea el inicio de la implementación de Bloque B.
- **15-ago-2026:** cerrada la etapa de estabilización técnica (Bloque M, desplegado). Arranca la Etapa 2 — UX y producto, con [UX_AUDIT.md](UX_AUDIT.md) como insumo. Diego fija el orden de trabajo (buscar/agregar → movimiento de interfaz → filtro de género → navegación de Biblioteca → mobile) y el mandato explícito de cuestionar el producto en el camino, no solo corregir. Arranca el diseño de Bloque N (flujo de buscar y agregar).
- **15-ago-2026:** diseño de Bloque N aprobado sin cambios de alcance. Buscar por actor/director/autor confirmado como valioso por Diego pero explícitamente fuera de este bloque — queda como próximo bloque de la etapa, sin diseñar todavía. Diego agrega un principio activo para toda la Etapa 2: cuestionar permanentemente el valor de cada pantalla/animación/modal, litmus test único = "¿más rápido o más agradable de usar todos los días?" (ver [[feedback-archivo-ux-stage-litmus-test]] en memoria). Bloque N implementado (los 4 sub-problemas) y validado con datos reales sobre el archivo local — pendiente el deploy manual de Diego.
