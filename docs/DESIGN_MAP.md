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

## Bloque L — Creación de grupo: `INSERT ... RETURNING` bloqueado por `groups_read`

- **Objetivo:** que `createGroup()` pueda crear un grupo y obtener su `id` de vuelta sin que la falta de membresía (todavía no creada) lo bloquee.
- **Problema que resuelve:** `createGroup()` usa `Prefer: return=representation` para leer el `id` del grupo recién creado. Esa lectura pasa por `groups_read`, que exige ser miembro — y en el instante del insert, todavía no lo sos (la membresía se crea en una segunda request, después). Postgres trata `INSERT ... RETURNING` como una operación atómica: si la lectura de vuelta falla por RLS, se revierte el insert completo. Resultado: crear un grupo falla siempre, en cualquier cuenta, la primera vez.
- **Cómo se descubrió:** al revalidar Bloque B después de cerrar Bloque K (el bug de la FK a `profiles` quedó resuelto, pero crear un grupo seguía fallando). Confirmado con evidencia: el mismo insert con `Prefer: return=minimal` da `201` (éxito limpio); con `return=representation` da `403` con el mismo mensaje genérico de RLS que ya habíamos visto antes (`"new row violates row-level security policy"`) — el mismo mensaje sirve tanto para un `WITH CHECK` fallido como para una lectura de `RETURNING` bloqueada, son indistinguibles por el mensaje solo.
- **Hallazgo adicional relacionado:** `groups` tampoco tiene ninguna política de **DELETE** — ni siquiera el creador de un grupo puede borrarlo vía la API normal. Confirmado al intentar limpiar un grupo de prueba: el `DELETE` devolvió `200` sin afectar ninguna fila (RLS filtra en silencio cuando no hay política aplicable, sin error).
- **No pertenece a Bloque K** (no es de identidad/perfiles) ni se mezcla con el diseño ya cerrado de Bloque B — bloque nuevo e independiente, mismo criterio "un problema, un bloque".
- **Estado:** Auditado (30-jul-2026). Diseño no arrancado.

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
- **Estado:** Pendiente.

---

## Bloque F — Exportación de datos

- **Objetivo:** que puedas sacar una copia de tu archivo.
- **Problema que resuelve:** hoy no existe ninguna forma de backup fuera de Supabase.
- **Valor:** Alto (revisado 2026-07-30) — Bloque 0 confirmó la intención de que Archivo sea "el archivo cultural definitivo" para consolidar toda su historia cultural. Sin backup, esa promesa depende enteramente de la disponibilidad de Supabase — el valor de mitigar esa pérdida sube en consecuencia.
- **Riesgo:** Bajo — operación de solo lectura, no modifica datos existentes.
- **Esfuerzo:** Bajo-Medio.
- **Dependencias:** ninguna.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 6.
- **Estado:** Pendiente.

---

## Bloque G — CSV: soporte de libros + fix del parser

- **Objetivo:** que Importar CSV cubra los tres tipos de ítems y no rompa con comillas escapadas.
- **Problema que resuelve:** libros quedan afuera de la única vía de carga masiva; el parser propio tiene un caso borde sin cubrir.
- **Valor:** Bajo-Medio — consistencia de producto, libros ya son de primera clase en todo lo demás. Bloque 0 confirmó intención de "eventualmente otras fuentes", lo que da algo más de contexto sin volverlo urgente.
- **Riesgo:** Bajo — extiende un flujo existente, no lo rediseña.
- **Esfuerzo:** Bajo-Medio — el fix del parser es casi trivial y puede resolverse solo, sin esperar el soporte de libros.
- **Dependencias:** ninguna.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 3, § 4.
- **Estado:** Pendiente.

---

## Bloque H — `tmdb_rating` con semántica doble

- **Objetivo:** que la columna deje de tener un nombre que no describe su contenido real para libros.
- **Problema que resuelve:** deuda de modelo de datos, ya reconocida en un comentario del propio código.
- **Valor:** Bajo — mantenibilidad futura, hoy no rompe nada.
- **Riesgo:** Medio — toca una columna con datos históricos reales, requiere migración, no solo cambio de código.
- **Esfuerzo:** Medio.
- **Dependencias:** ninguna decisión pendiente, pero el costo de migración es alto en relación al beneficio.
- **Documentos de auditoría que lo respaldan:** PRODUCT_AUDIT.md § 3, § 4.
- **Estado:** Pendiente.

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
