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

**Inicio:** 15-ago-2026. Cierra la etapa de estabilización técnica (Bloque M). A partir de acá el criterio de prioridad no es severidad técnica sino impacto en los flujos de uso diario, y el mandato incluye cuestionar el producto en el camino (simplificar/eliminar lo que no aporte valor, evaluar funcionalidad nueva solo si mejora la experiencia real) — no solo corregir. Nace de [UX_AUDIT.md](UX_AUDIT.md).

**Orden de trabajo (actualizado por Diego, 15-ago-2026, tras cerrar Bloque Q):**
1. ✅ Flujo de buscar y agregar — Bloque N.
2. ✅ Sensación de movimiento de la interfaz — Bloque O.
3. ✅ Filtro de género — Bloque P.
4. ✅ Navegación de Biblioteca a escala — Bloque Q.
5. **Próxima capa (sin bloques asignados todavía), los flujos donde sigue habiendo fricción diaria:**
   - Agregar película/serie/libro: velocidad y sensación premium del flujo *completo*, no solo la búsqueda (que ya se resolvió en N) — el resto del camino hasta guardar.
   - Detalle del ítem: modal, acciones, edición, calificación, notas.
   - Mobile/PWA fino (lo que quedaba de teclado/layout de mobile, adelantado parcialmente en el Bloque O).
   - Recomendaciones/inteligencia real de Archivo.

**Dos oportunidades grandes identificadas, registradas pero sin abrir todavía (no son parte de la Prioridad 5, son su propio frente futuro):**
- **Buscar por actor/director/autor** — ya identificado durante el diseño de Bloque N: técnicamente cerca, TMDB ya devuelve resultados de tipo persona y hoy se descartan explícitamente (`tmdbSearch()`, index.html). Diego confirmó que le interesa en serio, no es un "nice to have".
- **La inteligencia real de Archivo** — recomendaciones basadas en el historial propio, en línea con la visión de "archivo cultural definitivo" ya definida en el Bloque 0. Descubrí y ADN (ver sus respectivas secciones) ya son la base de esto.

Diego prioriza cerrar Q con deploy manual antes de seguir, porque es la pantalla que más se usa.

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
- **Estado:** Finalizado (15-ago-2026), commit `882f9bd`. Pendiente deploy manual. Ver corrección de diseño durante la implementación más abajo — el enfoque final no fue el primero que probé.

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

## Bloque Q — Navegación de Biblioteca a escala

*Dimensión de calidad que ataca: que encontrar un título concreto en una biblioteca de ~2000+ ítems sea rápido y obvio, no una tarea.*

- **Objetivo:** Prioridad 4 de la Etapa 2 — validar y mejorar cómo se navega Biblioteca cuando la colección es grande.
- **Problema que resuelve:** UX_AUDIT.md hallazgos #5 (parpadeo de carga), #6 (buscador interno y el idioma), #8 (mucho "chrome" antes del contenido en mobile).
- **Estado:** Finalizado (15-ago-2026), pendiente commit y deploy manual. Investigación cerrada, virtualización descartada por ahora (deuda técnica documentada), rediseño de la navegación implementado y validado con datos reales — ver detalle más abajo.

### Metodología de esta investigación

Se midió con datos reales (sesión de Diego, 2180 ítems) en vez de asumir: tiempo/clics para llegar a un título, render performance real (`performance.now()`), cuántas cards entran en una pantalla sin scrollear, altura real del "chrome" antes del contenido en mobile, y si buscador+filtros combinan bien. No se implementó nada — es la misma disciplina que auditoría/diseño previos.

### Hallazgos, priorizados por impacto

**1. (Alto) El buscador interno es, de lejos, el camino más rápido a un título concreto — pero pesa menos visualmente que un select más.**
Con 2180 ítems y 14 cards visibles por pantalla en desktop (7 columnas × 2 filas), llegar a un título por scroll puro requeriría del orden de ~150 pantallas de scroll — inviable a esta escala. El buscador interno, en cambio, ya es rápido y correcto: debounce de 120ms, filtra bien combinado con tipo/género/década (validado: película + "the" → 567 resultados, el 100% del tipo correcto). El problema no es el mecanismo, es que **no se distingue del resto de los filtros** — mismo tamaño, mismo peso, última posición de la barra. En mobile esto es más grave: el buscador queda a 383px de scroll en un viewport de 812px, es decir, a mitad de pantalla, por debajo de los 4 chips de tipo y los 4 selects/dropdowns. La herramienta más importante a esta escala es hoy la menos jerarquizada.

**2. (Medio, atacar ahora que es barato) El render de Biblioteca reconstruye todo el DOM en cada interacción, sin virtualización.**
Medido: renderizar la grilla completa (2180 cards) tarda ~81ms y genera ~19.400 nodos DOM; la vista lista, ~51ms. Cada cambio de filtro, orden, tipo o búsqueda dispara un rebuild completo (`innerHTML` desde cero), no hay virtualización ni paginación. Estos números son en hardware de desarrollo — un celular real probablemente sea 2-4x más lento en este tipo de trabajo. Hoy no es un problema grave (por debajo del umbral de lag perceptible en la mayoría de los casos), pero la visión de producto ya definida en el Bloque 0 espera "algunos miles" de ítems — a ese volumen el costo escala linealmente y sí se va a notar. No es optimización prematura: es evidencia real de una tendencia, atacarla ahora que el fix es barato es más barato que esperar a que se sienta lento.

**3. (Bajo hoy, no requiere acción) El filtro "Estado" tiene poco valor discriminante todavía.**
Confirmado con datos reales (visto durante Bloque O): "Pendientes" = 0 en toda la biblioteca hoy. Es historia, no un bug — casi todo el historial de 2180 ítems es anterior al cambio de default a "watchlist" del Bloque N. Coincide con una pista que ya estaba anotada desde la auditoría original ("el estado de cada card es ruido, ~99% Visto"). No propongo tocar nada acá: el filtro va a ganar valor solo, a medida que se agreguen ítems nuevos con el default corregido. Lo dejo documentado para no reabrirlo por error más adelante pensando que es un hallazgo nuevo.

**4. (Confirmado sin problema) Búsqueda y filtros combinan bien; Género/Década/Orden no generan ruido.**
Validado con datos reales, sin hallazgos: la búsqueda interna y los filtros de tipo/género/década se combinan con lógica AND correcta. Década tiene una distribución desigual (2010s domina) pero es información real, no ruido. Género ya se resolvió en Bloque P. Orden ("Recientes", "A–Z", etc.) son todas opciones con uso real posible para una colección de este tamaño.

**5. (Conecta con #1) Mobile necesita un recorrido distinto, no una versión angosta del de desktop.**
En mobile la grilla usa 3 columnas (vs. 7 en desktop) — menos ítems por pantalla todavía, y el "chrome" antes del contenido pesa proporcionalmente más del viewport. La solución de #1 (subir la jerarquía del buscador) importa más en mobile que en desktop porque ahí el costo de no encontrarlo rápido es mayor.

### Decisión de Diego (15-ago-2026) sobre la propuesta original

**Virtualización (punto #2 de la propuesta original): fuera de alcance por ahora, queda como deuda técnica documentada.** 81ms con ~2200 ítems no justifica hoy sumar complejidad de arquitectura de render — se monitorea, no se implementa hasta que el rendimiento real lo exija. Ver tabla de "Bloques diferidos" más abajo.
**Prioridad confirmada: atacar #1 a fondo.** No un cambio cosmético (mover el buscador de lugar) — repensar toda la jerarquía de navegación de Biblioteca asumiendo una escala de miles de ítems, no cientos. Diego pidió cuestionar explícitamente: ubicación y tamaño del buscador, jerarquía visual, filtros, cantidad de controles visibles por defecto, qué queda siempre accesible vs. qué puede esconderse, y si desktop/mobile deberían resolverse distinto.

### Rediseño de la navegación de Biblioteca (propuesta, pendiente de tu confirmación antes de implementar)

**Punto de partida (medido, no supuesto):** hoy, antes de llegar al contenido en mobile, se muestran 9 controles en secuencia — 4 chips de tipo, 4 selects (Estado/Género/Década/Orden) y recién al final el buscador, a 383px de scroll (mitad del viewport). El buscador es la herramienta más rápida y correcta que existe (validado en la investigación), pero es la última en aparecer.

**Respuestas a cada pregunta, pensando en 5000 ítems:**

- **Ubicación:** el buscador pasa a ser el primer control después del título de la sección, antes que cualquier chip o filtro. Es lo primero que se toca al entrar a Biblioteca.
- **Tamaño y jerarquía visual:** deja de tener el mismo peso que un `<select>`. Pasa a ocupar su propia fila completa, más alto, con tipografía más grande y un ícono de lupa — visualmente "la forma de entrar a tu archivo", no un filtro más.
- **Qué queda siempre accesible:** buscador + los 4 chips de tipo (Todo/Películas/Series/Libros). Son las dos formas de uso más frecuentes: "sé lo que busco" (buscador) y "quiero ver todas mis películas/series/libros" (tipo). Ambos son baratos en espacio y de alto valor.
- **Qué puede esconderse:** Estado, Género, Década y Orden se agrupan detrás de un único control "Filtros" (con contador de filtros activos si hay alguno puesto) que despliega un panel — dejan de ocupar espacio permanente. Son refinamientos para sesiones de exploración ("qué me pinta ver"), no para el caso de uso principal a esta escala (encontrar algo puntual).
- **Cantidad de controles visibles por defecto:** baja de 9 a 6 (buscador + 4 chips + botón Filtros), y el más importante pasa de la posición 9 a la posición 1.
- **Desktop vs. mobile — se resuelven distinto, con justificación:** en desktop el espacio horizontal ya alcanza para mostrar los 4 selects en una sola fila compacta sin costo real de scroll (hoy el problema en desktop es solo de jerarquía visual del buscador, no de espacio) — ahí NO escondo los filtros detrás de "Filtros", simplemente agrando y adelanto el buscador. En mobile sí se esconden, porque ahí el costo es real y medido (383px de scroll). No aplico el mismo patrón en los dos porque el problema que resuelven es distinto en cada uno: en desktop es jerarquía, en mobile es espacio vertical.
- **Vista grilla/lista:** sin cambios — es un control barato (dos íconos) que no contribuye al problema medido, no hay evidencia de que haya que tocarlo.

**Esto es un cambio de UX real, no cosmético — por eso lo presento para tu confirmación antes de tocar código, como pediste.**

### Confirmado por Diego (15-ago-2026), con dos principios adicionales para la implementación

1. El buscador tiene que ser **el protagonista**, no solo cambiar de lugar — tiene que quedar visualmente evidente que buscar es la forma principal de navegar Biblioteca.
2. **Ocultar complejidad, no funcionalidad.** Mobile esconde Estado/Género/Década/Orden detrás de "Filtros"; desktop los mantiene siempre visibles (el costo de espacio ahí es despreciable y evita un clic extra para explorar) — confirmó explícitamente no forzar la misma solución en los dos.

Más 3 criterios de implementación: sin saltos de layout al abrir/cerrar "Filtros"; estado abierto/cerrado predecible al rotar o redimensionar; filtros activos visibles aunque el panel esté cerrado (para que nadie piense "faltan películas").

### Implementado

- **Buscador rediseñado como pieza propia (`.bib-search-hero`):** primera fila de la sección, antes que cualquier chip o filtro. Más grande (16px vs. los ~12-13px de antes), ícono de lupa más grande, fondo y borde propios que lo distinguen de los `<select>`, placeholder en serif itálica (mismo lenguaje tipográfico que los títulos editoriales de la app) para reforzar que es "la forma de entrar a tu archivo", no un campo de formulario más.
- **Panel de filtros secundarios (`#bib-filters-panel`: Estado/Género/Década/Orden) separado del buscador**, con visibilidad resuelta por CSS según breakpoint, no por JS: en desktop el panel es `display:flex` incondicional (sin botón "Filtros", ni siquiera existe la posibilidad de ocultarlo); en mobile (`@media max-width:840px`, el breakpoint que ya usaba el resto de la app) el mismo panel colapsa detrás de un botón "Filtros" vía una clase `.open` que anima `max-height` (transición suave, sin salto — se abre/cierra igual que cualquier otro acordeón de la app).
- **Estado predecible entre breakpoints, sin lógica JS de reseteo:** como la visibilidad depende de CSS/media query y no de un estado que el JS reinicie al redimensionar, rotar el teléfono o cambiar de tamaño la ventana nunca puede dejar el panel en un estado inconsistente — validado: se abrió el panel en mobile, se cambió a ancho desktop (el panel se ve, viene del CSS de desktop, no de la clase `.open`), se volvió a mobile (panel sigue abierto, tal como se dejó).
- **Indicador de filtros activos en el botón "Filtros":** cuenta Estado/Género/Década (no Orden, porque reordena, no oculta nada) y muestra un badge numérico + resalta el botón en rojo — validado con datos reales: 2 filtros activos (Estado=Visto, Género=Terror) → badge "2", botón resaltado, **y el badge se mantiene visible con el panel cerrado**, cumpliendo el criterio explícito de que nunca se pierda de vista que se está filtrando. El contador "N con estos filtros" bajo el título también sigue funcionando como respaldo redundante.
- **No se tocó:** vista grilla/lista, virtualización, lógica de filtrado (`getBibFiltered()`) — todo eso queda igual, tal como pidió Diego.

### Validado con datos reales

- Desktop: buscador grande y en primera posición; los 4 selects siempre visibles debajo, sin botón "Filtros" (correctamente ausente).
- Mobile: buscador visible sin scrollear (antes a 383px de scroll, ahora a 218px — 43% más cerca); primera fila de contenido ya visible en el viewport inicial sin scrollear nada (antes hacía falta scrollear para ver cualquier ítem).
- Toggle de filtros: abre/cierra con transición suave (sin salto), badge correcto, estado sobrevive un cambio de viewport mobile→desktop→mobile sin romperse.

### Estado

Finalizado (15-ago-2026), commit `ce2537e`. **Desplegado y verificado en producción** — ver "Incidente de deploy" más abajo: el workflow de deploy de Archivo se reparó en el proceso, este bloque fue el primero en salir a producción con el nuevo flujo automático (push real, no upload manual).

---

## Incidente de deploy (15-ago-2026) — resuelto, cambia el workflow del proyecto

Después de commitear el Bloque Q, Diego notó una contradicción: se le había dicho "commiteado" varias veces durante N/O/P/Q pero él nunca hizo un deploy manual. Investigación con evidencia (no supuestos, como pidió):

- **Local vs GitHub:** `git fetch` + `git branch -vv` confirmaron que `main` estaba 21 commits adelante de `root/main` (el remoto `diepizaga.github.io.git`) — todo el trabajo desde Bloque M en adelante, incluidos N/O/P/Q, había quedado solo en local.
- **Qué había en GitHub Pages:** verificado con `curl` directo a producción + diff byte a byte: exactamente el commit `7b19e0c` ("Add files via upload", de Diego, 10-ago-2026 21:15) — su último upload manual. Confirmado que ese contenido era, a su vez, **idéntico byte a byte** a `84ba78c` (el commit local donde cerré Bloque M) — es decir, el remoto no tenía ningún cambio único, solo estaba desactualizado.
- **Causa raíz de que el push nunca funcionara:** el remoto tenía un token (PAT) embebido en la URL, y ese token devolvía 401 (revocado) al probarlo contra la API de GitHub. Consistente con un incidente de filtración de token de sesiones anteriores de este mismo proyecto — probablemente el mismo token, nunca reemplazado porque el deploy real seguía el camino manual de Diego sin que nadie lo necesitara.
- **Hallazgo adicional durante el diagnóstico:** el comando `git remote -v` usado para diagnosticar volvió a imprimir ese token en texto plano en el output — mismo tipo de incidente que ya había pasado antes en este proyecto. Señalado a Diego de inmediato.

**Decisión de Diego:** unificar el workflow de deploy de Archivo con el de FitCoach y CFO Personal — Implementación → Validación → Commit → Push → Deploy → Verificación en producción, sin que él tenga que subir nada a mano nunca más. Si en algún momento se pierde acceso a push/deploy, frenar y avisar antes de seguir acumulando commits locales. Ver `feedback-unified-deploy-workflow` en memoria — aplica a todos sus proyectos, no solo Archivo.

**Resuelto:** remoto reconfigurado para usar la credencial ya autenticada de `gh` CLI (Keychain de macOS, `gh auth setup-git`) en vez del token muerto embebido en la URL. Historial divergente reconciliado con un merge (`git merge root/main -X ours`, seguro porque se confirmó que el remoto no tenía contenido único que perder). Push probado y funcionando; Bloque Q desplegado con este flujo y verificado con `gh api .../pages/builds/latest` (`status:"built"`) + `curl` a producción con diff byte a byte contra el HEAD local — coinciden exacto.

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
| Virtualización/paginación del render de Biblioteca (Bloque Q) | Medido: 81ms / ~19.400 nodos DOM con 2180 ítems, todavía por debajo del umbral de lag perceptible. Diego decidió (15-ago-2026) no sumar complejidad de arquitectura de render hasta que el rendimiento real lo exija — se retoma si el tamaño de la colección lo justifica, no antes. |
| Scripts `.py` con credenciales (PROD-13 / EP-2) | Depende de tu decisión simple (ignorar vs. sanear) ya registrada en MASTER_AUDIT § 5 — si es "ignorar", no genera bloque. |

---

## Bloque R — Motor de recomendaciones con criterio propio

*Dimensión de calidad que ataca: que Descubrí deje de ser un espejo del algoritmo de TMDB y empiece a reflejar un criterio propio de Archivo sobre el historial real del usuario.*

- **Objetivo:** que cada sección de Descubrí (película/serie) haga lo que su nombre promete, y sumar criterios propios (género, década, rating, piso de calidad) encima del insumo de TMDB — no reemplazarlo.
- **Problema que resuelve:** hallazgos de [DISCOVERY_AUDIT.md](DISCOVERY_AUDIT.md) — "Por tus géneros favoritos" no filtra por género; sin piso de calidad (`vote_count`); corte arbitrario de 12 sobre 20 sin justificación documentada.
- **Valor:** Alto — es la pieza que Diego identificó como el paso de "agregador de TMDB" a "criterio propio", parte de la visión de "archivo cultural definitivo" (Bloque 0).
- **Riesgo:** Bajo — todos los datos nuevos que hacen falta (género, año, `vote_count`, `vote_average`) ya vienen en la respuesta de `/recommendations` que se pide hoy. No requiere llamados nuevos a la API ni cambios de esquema.
- **Esfuerzo:** Medio.
- **Dependencias:** ninguna.
- **Documentos que lo respaldan:** DISCOVERY_AUDIT.md.
- **Estado:** Implementado y validado con datos reales, pendiente commit/push/deploy — ver detalle completo más abajo.
- **Fuera de alcance, confirmado por Diego:** disponibilidad por plataformas — requiere una decisión de arquitectura distinta (cambiar de fuente de recomendaciones o multiplicar llamados), se trata como su propio frente futuro, no parte de este bloque.

### Investigación adicional para este diseño (datos reales, no supuestos)

- **TMDB ya devuelve todo lo necesario sin llamados nuevos:** confirmado con una consulta real — cada candidato de `/recommendations` trae `genre_ids`, `vote_count`, `vote_average` y `release_date`. Verifiqué también en vivo las listas oficiales de género de TMDB (`/genre/movie/list` y `/genre/tv/list`, es-AR): **incluso el endpoint oficial de géneros de TMDB devuelve en inglés las mismas categorías combinadas de serie que ya habíamos detectado en Bloque P** (Action & Adventure, Sci-Fi & Fantasy, War & Politics, Kids, News, Reality, Soap, Talk) — no es un caso puntual de un show, es un límite real y consistente de TMDB. El mapa de traducción de Bloque P (`GENRE_CANON_EN_ES`) se puede reusar tal cual, solo hace falta una versión indexada por ID numérico (los `genre_ids` de recomendaciones son números, no nombres).
- **Costo real de usar 20 fuentes en vez de 12:** medido con tus datos reales (20 llamados en paralelo vs. 12): 411ms vs. 364ms — un 13% más, no un costo lineal (son requests en paralelo). Confirma la sospecha del audit: "casi gratis" era correcto, no una suposición.
- **Distribución real de `vote_count` entre tus candidatos actuales** (20 fuentes, 192 candidatos únicos sin ver): mínimo 42, percentil 25 = 163, mediana = 569. Un piso de 50 excluiría solo 4 candidatos (2%); un piso de 100 excluiría 28 (15%). **Matiz honesto:** con tu perfil de calificaciones actual, la obscuridad extrema por cantidad de votos no es tan dominante como la hipótesis inicial suponía — un piso ayuda, pero probablemente no sea la explicación principal de "películas extremadamente desconocidas". Es más probable que sea una cuestión de **familiaridad** (una película real y con votos, pero que vos nunca escuchaste nombrar) que de obscuridad estadística.
- **Distribución real por década** entre esos mismos 192 candidatos: de 1940s a 2020s, con peso real en décadas viejas (28 candidatos antes de 1980, ~15%). Confirma que la diversidad temporal ya existe en el pool — el diseño no debería filtrarla, solo decidir cómo tratarla.

### Propuesta por cada objetivo de Diego

**1. Que cada sección haga lo que promete**
- "Por tus géneros favoritos" deja de ser "lo que sobró del balde de arriba" y pasa a filtrar de verdad: de todos los candidatos puntuados, se queda con los que comparten género con tus títulos mejor calificados (usando `genre_ids` + el mapa de traducción), ordenados por puntaje dentro de ese subconjunto. El texto de géneros que ya se muestra deja de ser decorativo.
- "Más recomendadas para vos" y "Más opciones" no cambian de criterio (siguen siendo por corroboración/puntaje), pero si se suma vote_count/rating al puntaje (punto 2), automáticamente mejoran también.

**2. Sumar criterios propios de Archivo, no solo TMDB**
- **Piso de calidad:** descartar candidatos con `vote_count` por debajo de un umbral antes de puntuar. Con la distribución real de arriba, propongo **50** como default razonable (excluye poco, saca el peor ruido) — es un número de gusto de producto, no algo que pueda decidir por vos. Lo dejo como parámetro a confirmar.
- **Rating como señal adicional:** sumar una fracción de `vote_average` normalizado al puntaje existente (que hoy solo usa tu calificación de la fuente), para que un candidato bien valorado en general pese un poco más que uno mal valorado, a igual corroboración.
- **Década:** no como filtro (ver más abajo, diversidad), sino como dato mostrado — ej. sumar la década al "por qué" de la recomendación cuando coincide con tus décadas favoritas de ADN, para que se sienta parte del criterio sin excluir nada.
- **Diversidad:** este es el criterio menos concreto de los cinco que nombraste, y no quiero adivinarlo — ¿te referís a que "Más recomendadas" no quede dominado por un solo género/década aunque tu historial esté cargado ahí (ej. tope de cuántos títulos del mismo género entran a un mismo balde), o a otra cosa? Antes de diseñar esta parte puntual necesito que me digas qué es "perder diversidad" en concreto para vos.

**3. Por qué 20 vs. 12, y si simplificar**
No encontré ninguna razón documentada para el corte en 12 — parece haber sido un límite arbitrario (probablemente pensado como resguardo de llamados en paralelo) que quedó desalineado del texto de la UI, que ya dice "basado en tus N mejor calificadas" usando el número real de hasta 20. Con el costo medido (13%, no lineal), propongo **usar los 20 de verdad** en vez de calcular 20 y usar 12 — no es "simplificar restando", es "dejar de calcular algo que se tira". El copy de la sección queda automáticamente correcto sin tocarlo.

### Confirmado por Diego (15-ago-2026)

- **`vote_count` mínimo: 50, fijo, no configurable.** "Prefiero una buena decisión por defecto antes que sumar opciones."
- **Diversidad, definida por Diego:** no es cuota artificial por género — "si mi historial está muy cargado hacia un género, está bien que eso se refleje". El objetivo es evitar que una lista se sienta "la misma película repetida diez veces": diversidad *dentro* del perfil (décadas, estilos, países, directores, niveles de popularidad), sin perder coherencia.
- **Director y país quedan fuera de este bloque:** no vienen en la respuesta de `/recommendations` que ya se usa — requerirían una consulta extra por candidato (mismo problema de costo que la disponibilidad por plataformas, documentado en DISCOVERY_AUDIT.md). Implementé la diversificación con lo que **no cuesta nada nuevo**: género principal, década, y nivel de popularidad (por `vote_count`). Si más adelante Diego quiere sumar director/país, es una extensión concreta y acotada de la misma función, no un rediseño — queda anotado como follow-up, no implementado.

### Implementado

- **`TMDB_GENRE_ID_ES`**: mapa de traducción por ID numérico (no por nombre, porque `/recommendations` trae `genre_ids`), verificado en vivo contra `/genre/movie|tv/list` — mismos nombres finales que `GENRE_CANON_EN_ES` de Bloque P, misma confirmación de que hasta el endpoint oficial de géneros de TMDB devuelve en inglés las categorías combinadas de serie.
- **20 fuentes reales, no 12**: `topItems.slice(0,12)` → `topItems` (ya limitado a 20 más arriba). El texto "Basado en tus N mejor calificadas" queda automáticamente correcto.
- **Piso de calidad (`DISC_MIN_VOTES = 50`)**: aplicado una sola vez, a nivel de todo el pool de candidatos, antes de puntuar — mismo criterio para las tres secciones.
- **Empujón chico por calidad general**: `vote_average` de TMDB multiplica el puntaje hasta +15% (candidato con nota perfecta) sin pisar la corroboración de tus propias calificaciones, que sigue siendo la señal dominante.
- **"Por tus géneros favoritos" genuinamente filtra por género**: candidatos cuyo `genre_ids` (traducido) se cruza con tus géneros mejor calificados — validado con datos reales: **100% de los candidatos mostrados coinciden con el género real**, no "lo que sobró" del balde de arriba.
- **`diversifyPicks(pool, n)`**: selección greedy que evita que género principal, década o nivel de popularidad dominen más del ~40% de una misma lista — sin cuota fija, sin dejar huecos (si no hay suficientes candidatos variados, se completa con lo que mejor puntúa).

### Validado con datos reales

- Piso de calidad: mínimo real observado entre los candidatos, 53 votos — nada por debajo de 50 pasa.
- Género: 100% de los candidatos de "Por tus géneros favoritos" comparten género real con el perfil (antes: 0% de garantía, eran sobras del otro balde).
- Diversidad, con pool grande (157 candidatos, 10 elegidos): ningún género ni década superó el tope de 4 (el 40% de 10) — spread real logrado: géneros 1/2/2/3/2, décadas 1940s/1970s/2000s(4)/2010s/2020s.
- Diversidad, con pool chico (6 candidatos elegibles para 12 lugares): se completó con los 6 igual, sin forzar huecos vacíos por diversidad — el diseño se comporta como se documentó ("la coherencia nunca pierde").
- Costo de 20 fuentes vs. 12: confirmado antes de implementar, ~13% más tiempo, no lineal (medido en la etapa de investigación de este mismo bloque).

### Visión a futuro del motor (documentado, no implementado en este bloque)

Diego fijó el rumbo de largo plazo para esta pieza del producto, con una formulación precisa que vale la pena citar tal cual (15-ago-2026, después de cerrar Bloque R):

> **TMDB deja de ser el motor de recomendación y pasa a ser principalmente una fuente de candidatos. El orden final y la relevancia comienzan a depender progresivamente del conocimiento que Archivo tiene del usuario (historial, evolución de gustos, patrones de consumo y contexto), no solo de la similitud calculada por TMDB.**

La distinción clave, en términos de arquitectura: **sourcing** (de dónde salen los candidatos) sigue siendo TMDB — eso no cambia, es dato que Archivo no tiene forma de generar por sí solo. Lo que se traslada progresivamente a Archivo es el **ranking** (qué orden y qué tan relevante es cada candidato) — hoy ese ranking es prácticamente 100% el score de corroboración de TMDB con ajustes chicos encima (Bloque R: piso de calidad, empujón por rating, diversidad). El rumbo es que ese ranking dependa cada vez más de conocimiento propio de Archivo sobre el usuario, no del algoritmo de caja negra de TMDB.

Este bloque (R) es un primer paso concreto en esa dirección, pero el ranking sigue apoyado en el score de "similares" de TMDB como base. El salto real —que el ranking dependa de un modelo de conocimiento propio del usuario— es un bloque futuro distinto, más grande, sin diseñar todavía. Se anota acá para que cualquier trabajo futuro sobre Descubrí parta de este rumbo ya confirmado, no lo redescubra.

### Estado

Finalizado (15-ago-2026), commit `f05de6e`, pusheado y verificado en vivo en producción (`diepizaga.github.io`).

---

## Bloque S — Capa de conocimiento propio: señal inferida + reacciones universales

*Dimensión de calidad que ataca: que Archivo empiece a describir por qué te gustan las cosas, no solo cuáles — la continuación directa del hallazgo central de PRODUCT_VISION.md.*

- **Objetivo:** dos piezas complementarias, decididas por Diego a partir de [EXPRESSION_DESIGN.md](EXPRESSION_DESIGN.md): (1) una capa permanente de señal inferida de datos que ya existen, sin pedir nada nuevo; (2) reacciones universales opcionales, mostradas solo cuando el sistema decide que vale la pena preguntar. Ambas alimentan tanto el motor de recomendaciones (Bloque R) como ADN.
- **Problema que resuelve:** PRODUCT_VISION.md — 1 nota de 2181 ítems, Archivo sabe cuánto te gustó algo y casi nada de por qué.
- **Valor:** Alto — es la apuesta que Diego priorizó primero de las tres del documento de visión.
- **Riesgo:** Bajo — B no toca UI ni pide nada nuevo; A es un dato nuevo aditivo (columna), sin tocar flujos existentes.
- **Esfuerzo:** Medio.
- **Dependencias:** ninguna abierta.
- **Documentos que lo respaldan:** PRODUCT_VISION.md, EXPRESSION_DESIGN.md.
- **Estado:** Finalizado (15-ago-2026), commit `5973337`, pusheado y verificado en vivo en producción. Columna `reactions` confirmada en Supabase; ciclo completo de guardar/borrar una reacción probado extremo a extremo contra la base real (agregada y removida como prueba, sin dejar datos falsos).

### Parte B — Capa de señal inferida (permanente, sin UI)

Diego fue explícito: esto no es una feature, es una capa del sistema. Señales concretas a computar, todas de datos que ya existen hoy:

- **Delta vs. crítica:** `my_rating` menos el promedio de `tmdb_rating`/`imdb_rating` — ya se calcula puntualmente en ADN ("tu criterio vs. IMDb"), pasa a calcularse y guardarse como señal reutilizable por ítem, no solo como agregado global.
- **Delta vs. tu propio promedio por género:** tu nota a este ítem vs. tu promedio histórico en ese género — más fuerte que el delta vs. crítica para detectar "esto te gustó de una forma distinta a lo habitual", porque se compara contra vos mismo, no contra un tercero.
- **Secuencia:** ¿agregaste esto justo después de otro ítem del mismo director/autor? Señal de interés sostenido, no un dato puntual aislado.
- **Timing:** calificaste apenas lo agregaste vs. días/semanas después — con `watch_date` en 0/2181 hoy, esta señal por ahora solo puede usar `created_at`, con la misma limitación ya anotada en PRODUCT_VISION.md (no es un dato de "cuándo lo viste", es de "cuándo lo cargaste").

**Dónde vive:** una función que corre sobre `items` en memoria (no requiere columna nueva ni cambio de esquema — todo se puede derivar de lo que ya está en la fila) y expone estas señales a quien las necesite: ADN (para narrar patrones) y Descubrí (para pesar candidatos). No se persiste en la base porque es 100% derivable de datos que ya están ahí — persistirlo sería una segunda fuente de verdad para algo que ya se puede calcular al vuelo.

### Parte A — Reacciones universales opcionales

Diego corrigió mi primera propuesta explícitamente: nada de chips que cambian según la nota. Chips universales, centrados en la experiencia, muy pocos. Propuesta de lista final (5, siguiendo los ejemplos que diste):

**Me sorprendió · Me emocionó · Me hizo pensar · Gran actuación/interpretación · Volvería a verla/leerla**

Matiz que quiero confirmar: "gran actuación" no traduce bien a libros — propongo que el copy se adapte por tipo (`interpretación` para película/serie, algo como `muy bien escrita` para libro) manteniendo la misma cantidad y el mismo criterio de selección — esto no es "chips distintos por nota" (lo que rechazaste), es el mismo concepto expresado con la palabra correcta según el medio. Confirmame si te sirve o preferís una sola redacción para los tres tipos.

**Cuándo vale la pena preguntar** (la parte que delegaste al sistema): propongo que se muestren solo cuando el ítem es una **reacción notable**, no una rutinaria — definido con las señales de la Parte B, no arbitrario:
- Tu calificación está entre el 20% más alto o más bajo de tu propia distribución para ese tipo, **o**
- El delta vs. la crítica es grande (`|my_rating − promedio_crítica| ≥ 3`).

Es decir: se pregunta cuando ya hay, por los propios datos, evidencia de que pasó algo distinto de lo habitual — no en cada guardado. Con esto no hace falta además un límite artificial de frecuencia: como estas condiciones no se cumplen en la mayoría de los guardados, el prompt ya va a ser infrecuente por construcción. Si en el uso real resulta que aparece más seguido de lo que gustaría, se ajusta el umbral — no agregaría un límite de sesión por separado desde el arranque, sería resolver con una regla algo que ya resuelve el propio criterio.

**Dónde vive:** columna nueva `reactions` (array de texto) en `watchlist` — aditiva, mismo patrón que `genres`. El prompt aparece como un paso opcional, no bloqueante, después de guardar (toast/chip picker chico, no un modal nuevo) — desaparece solo si no se toca.

### Cómo alimenta a ADN y a Descubrí

- **ADN:** nueva sección, gateada por volumen mínimo de datos (mismo patrón que ya usa ADN: "calificá al menos 5 para ver tu perfil") — con reactions y señal inferida acumulándose desde cero, va a tardar en tener algo real que decir. Se construye la infraestructura ahora; el "por qué" narrado en ADN crece con el uso, no aparece completo el día uno.
- **Descubrí:** las reacciones y la señal inferida se suman como un eje más de `diversifyPicks()`/puntaje (Bloque R) — un candidato que comparte género con ítems que marcaste "me emocionó" en el extremo alto de tu distribución pesa más que uno que solo comparte género en general. No se implementa en este bloque hasta que haya datos reales acumulados — implementarlo antes sería optimizar sobre una tabla vacía.

### Decisiones finales de Diego (15-ago-2026)

- **Chips:** universales de verdad, sin listas distintas por tipo — solo el último chip adapta el texto. Lista final: *Me sorprendió · Me emocionó · Me hizo pensar · No pude soltarla · Volvería a verla/leerla* (reemplaza "Gran actuación", que no traducía bien a libros).
- **Cuándo preguntar:** confirmado el criterio basado en señales reales, con una regla extra — si el ítem ya tiene reacciones registradas, no se vuelve a preguntar (salvo que en el futuro exista el concepto de revisitar una obra — hoy no existe, `UNIQUE(tmdb_id, type)` lo impide estructuralmente, ver arriba).
- **Descubrí:** confirmado que queda para después. Primero ADN construye conocimiento real.
- **Principio de diseño nuevo, explícito:** *"cada chip debería sentirse como una conversación, no como una encuesta — si en algún momento aparece la sensación de estar completando un formulario, el diseño perdió el objetivo."* Aplicado a la implementación: pregunta con tono conversacional ("¿Qué te generó...?", no una etiqueta tipo "Reacción:"), sin botón de "enviar" — cada chip se guarda al toque, sin un paso de confirmación aparte, y desaparece solo si no se toca.

### Implementado

- **Corrección de diseño encontrada validando con datos reales (transparencia, no un detalle menor):** la primera versión de "cuándo preguntar" usaba percentil 20/80 **sobre el valor** de la nota. Validado con tus 1646 películas calificadas, esto marcaba el **100% como "reacción notable"** — no el ~5-10% esperado. Causa: tus notas se concentran fortísimo en 6-7 (72% de tus 1646 películas), así que el percentil 20 y el percentil 80 *por valor* caen exactamente sobre esos dos números dominantes, y casi cualquier nota los toca. Corregido a un método estadístico estándar para esto (outliers por rango intercuartílico, Q1−1.5·IQR / Q3+1.5·IQR) — validado con los mismos datos reales: **70 de 1646 (4.3%)**, más cerca de lo esperado y de lo que "infrecuente por construcción" pedía.
- Columna nueva `reactions text[]` en `watchlist` (aditiva, mismo patrón que `genres`) — **pendiente que Diego la ejecute en Supabase** antes de que el guardado de reacciones funcione en producción (el resto de la app no se rompe si no está: el guardado falla en silencio, es un gesto opcional).
- `saveDetail()` dispara el prompt solo cuando se tocó la calificación en ese guardado (no en cualquier edición).
- Nueva sección en ADN, "Por qué te gustan las cosas" — gateada a ≥5 ítems con reacciones registradas (mismo patrón de gate que el resto de ADN). Con reacciones empezando de cero, va a tardar en aparecer — es esperado, no un bug.
- Validado en vivo: prompt se muestra/oculta correctamente, chips seleccionables, sin salto de layout, sin superposición con la barra de navegación mobile (medido con `getBoundingClientRect()`, no a simple vista — una primera lectura visual sugería superposición y resultó ser un artefacto de timing entre capturas, no un bug real).
- **Pendiente:** que corras el SQL de la columna nueva, y recién ahí commit → push → deploy → verificación en producción.

---

## Bloque T — ADN 2.0: insights, no estadísticas

*Dimensión de calidad que ataca: que ADN deje de ser un resumen y empiece a ser un espejo — patrones que Diego no sabía sobre sus propios gustos.*

- **Objetivo:** primer bloque de la hoja de ruta post-Bloque S. Diego pidió explícitamente NO retomar el ritmo de "documentar para después" — seguir construyendo sin pausas entre bloques salvo cambio estructural importante.
- **Documentos que lo respaldan:** DESIGN_MAP.md § Hoja de ruta, PRODUCT_VISION.md.
- **Estado:** Finalizado (15-ago-2026), commit `5e4427e`, pusheado y verificado en vivo en producción. 4 insights genuinos aparecen hoy (brecha película/serie/libro, duración de película, género polarizante, alineación con fuente de crítica). El insight de reacciones (Bloque S) queda consolidado en el mismo sistema, inactivo hasta acumular datos, como se esperaba.

### Auditoría: qué insights son reales hoy (no supuestos)

Antes de proponer nada, calculé varias correlaciones candidatas contra tus 2181 ítems reales. Algunas confirmaron lo esperado, dos me sorprendieron a mí también:

| Candidato | Resultado real | ¿Sirve hoy? |
|---|---|---|
| Película vs. serie vs. libro | Películas 6.36 (n=1646), series 7.35 (n=291), libros 7.46 (n=28) — **casi un punto completo de diferencia** | **Sí, fuerte** |
| Duración de película vs. nota | <90min: 6.02 · 90-110: 6.13 · 110-130: 6.56 · 130min+: 7.05 — patrón monótono y claro | **Sí, fuerte** |
| Género polarizante (promedio vs. presencia en tus 9-10) | Comedia: promedio 6.33 (uno de los más bajos), pero es el género del 40% (20/50) de tus calificaciones de 9+ | **Sí, matiz real** |
| Alineación con fuentes de crítica | vs. IMDb ≈ 0 · vs. Rotten Tomatoes +0.73 · vs. Metacritic +0.99 (n=852-853) — mucho más cerca de IMDb que de la crítica especializada | **Sí, fuerte** |
| Década vs. nota | 6.43 a 6.75 en todas las décadas con n≥15 — prácticamente plano | **No — se probó y no hay señal real, no se fuerza un insight sin sustento** |
| Delta vs. crítica por género | Rango real: -0.35 (Action & Adventure) a +0.03 (Romance) — existe pero es chico | **Señal secundaria, no un insight por sí solo** |
| Reacciones × género (tus propios ejemplos: "sorprendió → sci-fi") | 0 reacciones registradas todavía | **No todavía — infraestructura lista, se activa sola con uso** |
| Patrones temporales/Memoria | `watch_date` en 0/2181 | **No — ya registrado como Bloque futuro aparte** |

### Diseño: un sistema de insights rankeados, no secciones fijas

En vez de agregar una sección nueva por cada patrón, `computeADNInsights()` calcula **todos** los candidatos con datos suficientes, les asigna una fuerza (tamaño del efecto × log del tamaño de muestra) y muestra los más fuertes primero — el mecanismo generaliza: a medida que las reacciones acumulen datos, esos insights van a competir y aparecer solos, sin tocar código de nuevo.

- **Se consolida el hallazgo de Bloque S** ("Por qué te gustan las cosas") como un candidato más de este mismo sistema, no una tarjeta aparte — evita que ADN tenga dos mecanismos de insight en paralelo.
- **Se mantienen las secciones existentes** (géneros con conteo, décadas, criterio vs. IMDb, coincidencia con la crítica, mayores diferencias) como detalle de apoyo debajo de los insights — no hay evidencia de que sean redundantes o sin uso (a diferencia de "Directores favoritos" en Bloque E, que sí estaba confirmado muerto); se revisan si en el uso real se sienten de más, no se sacan por anticipado.
- **Gate propio, más exigente que el resto de ADN:** 15 calificados mínimo (no 5) — un insight sobre un patrón necesita más base que un resumen simple para no decir algo sin sustento.

---

## Bloque U — Buscar por actor/director/autor

- **Objetivo:** que el buscador de Biblioteca (ya dice "Buscar título, autor…" desde Bloque Q) también encuentre por director/actor — Prioridad 2 de la hoja de ruta post-Bloque S.
- **Estado:** Finalizado (16-ago-2026), commit `c41ac91`, pusheado y verificado en vivo en producción. Backfill completo: 2145 de 2153 películas/series (99.6%) con `people` real. Los 8 restantes confirmados uno por uno como límite real de TMDB (ej. Jujutsu Kaisen: `created_by` y `cast` vacíos en la fuente, no un bug del backfill) — no reality shows/anime, no un patrón que valga la pena perseguir más. Validado con datos reales: "tom hanks" encuentra 23 títulos, "nolan" 8.

### Incidente durante el backfill (transparencia, no un detalle menor)

El primer intento se congeló en silencio a los ~350 ítems — un solo `fetch()` sin resolver bloqueaba para siempre el `Promise.all` del lote, sin ningún error visible. Encontrado inspeccionando `elementFromPoint`-style (comparando el conteo real de `items` contra los logs de consola, no confiando en que "no hay más logs" significara "terminó"). Corregido de raíz con `fetchTimeout()` (10s, `AbortController`) aplicado tanto a `tmdbDetail()` como a `sbFetch()` — **beneficia a toda la app, no solo al backfill**, ya que `sbFetch()` se usa en cada lectura/escritura a Supabase y antes no tenía ninguna protección contra una request colgada.
Segundo intento con batch=40 disparó rate-limiting real de TMDB (429), con hasta 42% de fallos en un momento dado — bajado a batch=10-12 (0 fallos) para el grueso del trabajo. Quedó documentado como referencia: **10-12 en paralelo es el techo seguro para TMDB con esta app**, no hace falta volver a probarlo la próxima vez que haga falta un backfill similar.

### Auditoría: qué existe hoy

- **Libros ya funcionan.** `author` es una columna real, ya se busca (`bib-search` matchea contra `title`/`original_title`/`author`) y ya alimenta "Autores favoritos" en ADN (Bloque E). No hace falta nada nuevo acá.
- **Películas y series: cero datos de director/reparto hoy.** Ni se piden ni se guardan — confirmado revisando `buildTmdbItem()` y el bloque `adnSwitchType()` (el `else` de la sección "Directores ↔ autores" simplemente esconde la sección para película/serie, remanente de cuando existía "Directores favoritos", removida en Bloque E).
- **El dato está disponible sin costo extra para altas nuevas.** Verificado en vivo: agregando `append_to_response=credits` a las llamadas que `tmdbDetail()` ya hace (AR/ES/EN en paralelo, sin llamados nuevos), la respuesta ya trae `credits.crew` (para sacar el director) y `credits.cast` (reparto). Para series, `credits.crew` viene vacío (director no es un concepto estable a nivel serie en TMDB) — el equivalente real es `created_by` (showrunner), que **ya viene en la respuesta que `tmdbDetail()` pide hoy**, sin ningún cambio.
- **El problema real no es técnico, es de volumen: 2153 películas/series ya guardadas no tienen este dato.** Conseguirlo para altas nuevas es gratis; conseguirlo para lo que ya existe requiere **un llamado nuevo por ítem** — hasta ~2153 llamados a TMDB, una sola vez.

### Diseño

- Columna nueva `people text[]` en `watchlist` (aditiva, mismo patrón que `genres`/`reactions`) — para película: `[director, ...top 5 del reparto]`; para serie: `[...creadores, ...top 5 del reparto]`. No se toca `author` (libros), que ya funciona.
- `getBibFiltered()`/`bib-search` suman `people` a los campos que ya matchea (`title`, `original_title`, `author`) — mismo mecanismo, sin UI nueva, sin filtro nuevo: escribís "Nolan" y aparece lo que tenga a Nolan en `people`.
- Altas nuevas: `buildTmdbItem()` arma `people` a partir de la respuesta que `tmdbDetail()` ya trae (con `credits` agregado) — sin llamados extra.

### Decisión pendiente de Diego antes de cerrar el bloque

**El backfill de las 2153 películas/series existentes es la parte que sí toca algo estructural** (volumen de llamados a una API externa, no solo una columna) — prefiero confirmarlo con vos en vez de decidirlo solo:
- **Opción A (recomendada):** backfill en lote, corrido una sola vez desde el navegador con tu sesión — client-side, sin tocar backend, en tandas para no saturar TMDB (ej. 10-20 en paralelo, con pausas). Con 2153 ítems, a un ritmo conservador, son varios minutos, no segundos — lo puedo dejar corriendo y avisarte cuando termine.
- **Opción B:** backfill perezoso — se completa solo cuando abrís la ficha de un ítem viejo (ya se llama a `tmdbDetail`/`loadWatchProviders` al abrir cada ficha) en vez de un lote de una vez. Más lento en cubrir todo el archivo, cero riesgo de saturar la API de una sola vez.
- **Opción C:** no hacer backfill — la búsqueda por persona solo funciona para lo que agregues de acá en adelante. Más simple, dato incompleto por mucho tiempo dado tu ritmo de altas.

Mientras tanto, implemento la columna, el fetch en altas nuevas, y el buscador extendido — no depende de esta decisión.

---

## Bloque V — ADN aprende de personas

- **Objetivo:** antes de pasar a Descubrí con ADN, auditar qué habilita la capa de `people` (Bloque U) para ADN — misma metodología que Bloque T, medir primero, mostrar solo lo sostenible.
- **Estado:** Finalizado (16-ago-2026), commit `315f5bf`, pusheado y verificado en vivo en producción. Nuevo candidato de persona sumado a `computeADNInsights()` (sin distinguir rol, ver decisión pendiente abajo), sección de ADN ahora muestra hasta 6 insights (antes 4). Con datos reales aparecen 5 hoy, incluido el de persona ("Gino Renni, 1.4 puntos más bajo que tu general"). Pendiente: tu respuesta sobre separar director/reparto (no bloquea el cierre de este bloque, es una decisión para un bloque futuro si la querés).

### Auditoría: qué sostiene la evidencia y qué no

- **Promedio de nota por persona, con muestra comparable a la de género (n≥20): spread real de 1.35 puntos** (Tom Hanks 7.30 — Dwayne Johnson 5.95, 16 personas califican). **Prácticamente el mismo orden de magnitud que género (1.29 en Bloque T)** — ni género ni persona explica claramente más que el otro a igual nivel de confianza; los dos son señales reales y comparables, no hay un ganador.
- **Bajando el umbral a n≥8 (200 personas califican) el spread crece a 2.41** (Robert Downey Jr. 7.54 — Gino Renni 5.13) — más rico, pero con menos muestra por persona que en género, así que hay que tratarlo con menos certeza, no como un hallazgo más fuerte que el de género.
- **"Personas que aparecen repetidamente en tus favoritos" (calificación ≥9): descartado, no hay sustento.** De tus 50 títulos con 9+, solo **una persona** (Robert Downey Jr.) aparece en 3 o más — no hay un patrón real ahí, es un solo dato suelto. No se fuerza un insight con esto.
- **Combinaciones (director + actor juntos): descartado tal como se planteó, hallazgo metodológico real.** Calculé 185 pares con muestra ≥4 — el top está dominado por elencos de Avengers (Chris Evans + Robert Downey Jr., Chris Hemsworth + Mark Ruffalo, etc.). No es una señal nueva: es la misma información que "te gustan las pelis de Avengers" contada muchas veces distintas, porque un elenco ensemble de 6 genera 15 pares de una sola película. Mostrar esto como "combinación interesante" sería inventar un insight que en realidad es redundante con el de personas individuales.
- **Género vs. persona, con evidencia:** no hay un ganador claro a igual confianza estadística — ambos son ejes reales y de magnitud similar. Lo que sí cambia es la granularidad: género te dice "te gusta el thriller", persona te dice "te gusta cuando aparece Robert Downey Jr." — dos niveles de explicación distintos, no competidores.

### Decisión de arquitectura pendiente: ¿separar director de reparto?

Hoy `people` es una lista plana — director/creador y reparto mezclados sin etiqueta de rol (fue la decisión de Bloque U, para no multiplicar llamados a TMDB). Esto alcanza para un insight tipo *"cuando participa X, tendés a calificar más alto"* — cierto sea X director o actor, sin necesidad de saber cuál es. **Pero no alcanza** para separar específicamente "tus directores favoritos" de "tus actores favoritos" como pediste en los ejemplos — eso requeriría volver a tocar el esquema (columnas separadas `director`/`cast`) y, como el dato ya viene mezclado en la base, un segundo backfill sobre lo ya guardado (no gratis, mismo volumen que Bloque U). Antes de encarar eso prefiero preguntarte: ¿te alcanza con el insight sin distinguir rol, o te importa lo suficiente la distinción director/actor como para justificar un segundo backfill?

### Propuesta (con la respuesta a la pregunta de arriba, sin distinguir rol)

Sumar como candidatos nuevos a `computeADNInsights()` (Bloque T), mismo mecanismo de fuerza (efecto × log muestra), sin sección aparte:
- Persona con promedio de nota más alto/bajo, gate en n≥8 (documentado como umbral menor que género porque el dato es más granular, no porque sea más confiable).
- Sin insight de "aparece en tus favoritos" ni de "combinaciones" — no hay evidencia real detrás, según la auditoría de arriba.

### Sobre navegación vs. alimentar ADN/recomendaciones

Con evidencia real de un spread comparable al de género, **sí vale la pena que `people` alimente ADN** — ya no es solo una herramienta de navegación. Para Descubrí, Diego ya lo confirmó diferido hasta que haya más señal acumulada (mismo criterio que reacciones, Bloque R/S) — `people` puede sumarse ahí en el mismo momento, no antes.

---

## Bloque W — Separar director de reparto (calidad de dato, no feature visual)

- **Objetivo:** Diego confirmó la separación de rol planteada en Bloque V — no es una mejora visual, es preparar la estructura para que ADN, Descubrí y grupos puedan pesar distinto "quién define una obra" (director) de "quién participa" (reparto), más adelante.
- **Alcance explícito de Diego:** sin complejidad visual nueva; sin forzar insights nuevos hasta que haya muestra suficiente; dejar la estructura lista para uso futuro.
- **Estado:** Finalizado (16-ago-2026), commit `727baaa`, pusheado y verificado en vivo en producción. Segundo backfill completo, 2153/2153 procesados, 0 fallos, mismos 2145/2153 (99.6%) con datos reales que Bloque U (los 8 sin datos son el mismo límite de TMDB ya confirmado, no nuevos). Buscador e insight de ADN validados sin regresión ("nolan" 8, "tom hanks" 23, mismos resultados que antes de la migración). `people` queda en desuso — pendiente confirmación de Diego para el `DROP COLUMN`.

### Diseño

- Columnas nuevas `director text[]` y `cast_names text[]` (no `cast`, palabra reservada) en `watchlist` — mismo patrón aditivo que `people`/`reactions`.
- `extractPeople()` se separa en `extractDirector()` y `extractCast()` — mismos datos ya disponibles hoy (`credits`/`created_by`, sin llamados nuevos a TMDB), solo dejan de mezclarse en un único array.
- **Una sola fuente de verdad, no dos:** `getBibFiltered()`/`bib-search` y el candidato de persona en `computeADNInsights()` pasan a leer `director`+`cast_names` combinados en el momento (no se sigue escribiendo en `people`) — mismo comportamiento visible hoy (el buscador sigue encontrando por cualquier persona, el insight de ADN sigue sin distinguir rol todavía), pero la estructura queda preparada para el día que sí haga falta distinguir.
- **No se agrega insight de "tus directores favoritos" ni "tus actores favoritos" separados todavía** — exactamente lo que pidió Diego: no forzar hasta medir si hay muestra suficiente una vez separado. Queda como paso siguiente, auditado con datos reales antes de mostrarse.
- **Backfill, de nuevo:** los 2145 ítems ya tienen `people`, pero no con el rol separado — hace falta un segundo backfill, mismo volumen que Bloque U, reusando el mecanismo ya calibrado (`fetchTimeout`, batch 10-12, idempotente).
- **`people` queda en desuso después de esto** — no se le sigue escribiendo. Se propone dropear la columna una vez confirmado que el backfill nuevo cerró bien, para no dejar dos fuentes de verdad del mismo concepto en el esquema — se pide confirmación aparte antes de correr un `DROP COLUMN` (operación destructiva).

---

## Hoja de ruta confirmada después de Bloque S (15-ago-2026, sin bloques abiertos todavía)

Diego cerró la sesión de Bloque S con una lectura de conjunto del roadmap: primero la base técnica (Bloque M), después la UX de uso diario (Bloques N-Q), y ahora el arranque de la inteligencia propia de Archivo (Bloques R-S). Definió la secuencia de las próximas cuatro apuestas, en este orden — **ninguna diseñada todavía**, esto es la hoja de ruta, no un bloque en curso:

1. ✅ **ADN 2.0** — Bloque T, finalizado y en producción (15-ago-2026).
2. **Buscar por actor/director/autor.** Mejora de navegación real y valiosa ("la voy a usar"), pero no cambia la identidad del producto como sí lo hace ADN 2.0 — por eso va después, no antes. **En curso.**
3. **Descubrí usando el conocimiento de ADN** — recién cuando haya suficientes reacciones acumuladas (la integración que Bloque R y Bloque S dejaron explícitamente diferida).
4. **Memoria — un tercer frente nuevo, anotado, no abierto.** Distinto de recomendaciones y de estadísticas: reconstruir tu propia historia cultural en el tiempo. Ejemplos de Diego: *"Hace exactamente dos años viste..."*; *"La última vez que un libro te emocionó fue..."*; *"Hace ocho meses que no ves ciencia ficción"*; *"Hace mucho que no calificás algo con un 10"*. Depende de `watch_date` real, que hoy está en 0/2181 (PRODUCT_VISION.md) — no tiene datos suficientes todavía, se anota para cuando los tenga.

Lectura de conjunto de Diego, para no perderla: **primero que Archivo te conozca (ADN 2.0), después que use ese conocimiento para ayudarte a descubrir cosas nuevas (Descubrí con ADN), y finalmente para reconstruir tu propia historia (Memoria)** — una progresión consistente con la visión de producto ya definida, no una lista de features sueltas.

### Dos líneas de contexto adicionales (16-ago-2026), sin abrir — para no tomar decisiones aisladas

Diego las trajo explícitamente como contexto de producto, no como bloques a diseñar ahora. Se documentan con detalle para que, cuando corresponda, no haya que reconstruir el razonamiento desde cero.

**A. Auditoría de UX móvil/PWA — "que se sienta sólido, no solo que funcione".**
Diego usa Archivo activamente desde Safari en iPhone y compartió 4 capturas reales de uso (16-ago-2026). Revisadas una por una, confirman con evidencia (no solo su descripción) varios puntos:
- **Elementos superpuestos sobre contenido:** la barra de estado de Safari (hora/batería) se superpone visualmente al botón "Abrir ficha" en el hero de Inicio.
- **Poco espacio útil real:** el header fijo de Archivo + la bottom nav fija de Archivo + la barra de direcciones flotante de Safari (con el pill "diepizaga.github.io") conviven, y esta última se superpone al contenido de la app, no solo lo empuja.
- **Zoom de interfaz posible — el hallazgo más claro de las 4 capturas:** una de las capturas muestra la página entera pinch-zoomeada, con texto cortado y scroll horizontal roto ("...NTES" en vez de "ENTRADAS RECIENTES") — confirma que hoy nada impide hacer zoom sobre la interfaz, rompiendo el layout premium por completo.
- **Puntitos de estado poco claros visualmente:** el punto pálido en la esquina de cada póster (estado watched/watching/pendiente) es difícil de leer a simple vista — se ve, pero no comunica.
- **Biblioteca en mobile:** en la grilla revisada no se vio un problema tan claro como los anteriores — Diego lo describe como "sensación de que algunos elementos no están perfectamente contenidos" más que un caso puntual reproducible; queda para revisar con más casos cuando se audite.
Diego pidió explícitamente: cuando llegue el momento, una auditoría específica de cómo se *siente* Archivo (no solo que funcione) en iPhone Safari, PWA instalada, desktop, distintos viewports, teclado abierto, scroll, modales y navegación — no arreglar síntomas sueltos sin entender la causa raíz (ej. el zoom y la superposición de barras de Safari son plausiblemente la misma causa: falta de un `viewport meta` que fije `user-scalable=no`/`maximum-scale=1`, pero esto es una hipótesis a confirmar en la auditoría, no un diagnóstico cerrado).

**B. Grupos como inteligencia colectiva de gustos — dirección de producto, no diseño.**
Hoy Grupo es "biblioteca compartida" (confirmado sin uso real, Bloque 0). La idea que Diego quiere explorar es distinta y más ambiciosa: no compartir una lista, sino que Archivo compare gustos entre personas. Ejemplos que dio: ¿qué tenemos en común?, ¿en qué somos diferentes?, ¿qué películas tienen alta probabilidad de gustarnos a todos?, ¿qué vería el grupo esta noche?, ¿qué título funciona como puente entre dos personas de gustos distintos?, ¿qué miembros tienen gustos más parecidos? Contextos que nombró: pareja, amigos, familia, varios usuarios comparando gustos.
Diego mismo definió los tres frentes de auditoría que haría falta antes de diseñar nada: qué existe hoy realmente en código/base (el modelo de Grupo actual — `group_id`/`user_id`, sin roles reales, ver Bloque L/EP-11 — probablemente no alcanza para esto), qué datos se pueden usar (¿cruzar `my_rating`/géneros/reacciones entre usuarios del mismo grupo, dentro de las políticas RLS actuales?), y cómo manejar privacidad/permisos (¿alguien puede ver el detalle de las calificaciones de otro miembro, o solo agregados/comparaciones?). No se audita ni se diseña ahora.

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
