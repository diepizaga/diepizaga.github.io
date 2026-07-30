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
- **Estado:** Pendiente.

---

## Bloque B — Habilitar RLS en `profiles`, `groups`, `group_members`

- **Objetivo:** cerrar el acceso público sin restricción a esas tres tablas.
- **Problema que resuelve:** cualquiera con el `anon key` puede leer/escribir ahí hoy, sin precondiciones.
- **Valor:** Alto — cierra la exposición más severa documentada.
- **Riesgo:** Medio — si el texto de las políticas existentes es más laxo o más estricto de lo esperado, activar RLS puede no corregir nada o romper un flujo real (ej. la creación automática de perfil al loguearse).
- **Esfuerzo:** Bajo-Medio — las políticas ya existen, falta revisarlas, activarlas y validar con uso real.
- **Dependencias:** ninguna abierta — el texto SQL exacto de las 9 políticas ya se obtuvo y se revisó (ver "Diseño aprobado" abajo). Este campo estaba desactualizado hasta 2026-07-30.
- **Documentos de auditoría que lo respaldan:** SECURITY_AUDIT_RLS.md § Riesgo 1.
- **Estado:** Diseño aprobado (2026-07-30) — pendiente de implementación (después de Bloque C, antes housekeeping de Bloque A).

### Diseño aprobado

- `groups_insert` referenciaba `created_by`, columna real en el schema; el frontend mandaba `owner_id`, que no existe — confirmado con evidencia (error `42703` en `owner_id`, `[]` en `created_by`) y con la historia completa de git (ningún commit tocó nunca `created_by`; `owner_id` aparece una sola vez, en el commit que originó el feature, 06-jun-2026). Sin evidencia de una migración que haya causado el desajuste — se sigue la regla por defecto de Diego: **el esquema es la fuente de verdad, se adapta el código** (`createGroup()` pasa a mandar `created_by`).
- `profiles_read` (`using(true)`) se reescribe acotada a "perfil propio + perfiles de gente con quien compartís un grupo" — auditoría puntual de los 3 consumidores reales (`loadProfile`, `changeDisplayName`, `renderGroupMembers`/`renderGroupCompareList`) confirmó que ese scope cubre todos los casos sin restringir de más.
- Resto de las 6 políticas (`profiles_update`, `profiles_write`, `groups_read`, `gm_read`, `gm_insert`, `gm_delete`) se activan sin cambios — confirmadas correctamente scoped.
- Rollback progresivo: (1) revertir código, (2) revertir/ajustar la política puntual, (3) desactivar RLS como último recurso.
- Criterio de éxito: RLS habilitado en las 3 tablas + test de lectura anónima sigue vacío + `createGroup()` puebla `created_by` + sin regresión en login/perfil + cero errores nuevos en consola/red.

### Observación registrada durante el diseño (fuera de alcance de este bloque)

`profiles` tiene 0 filas hoy pese a que `loadProfile()` intenta crear el perfil en cada login (`index.html:2718`). Causa probable: el `fetch(...).catch(()=>{})` de esa creación solo atrapa fallas de red, no errores HTTP — un fallo silencioso ahí nunca se habría notado. **Decisión explícita de Diego: esto no amplía el alcance de Bloque B.** Se valida como parte del checklist de implementación (crear un perfil real y confirmar que la fila se crea) pero no se diagnostica ni se corrige acá. Si la implementación confirma que la creación automática falla de verdad, se abre un bloque nuevo e independiente para ese problema. Si funciona correctamente, el hallazgo se cierra como validado sin generar trabajo adicional.

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
