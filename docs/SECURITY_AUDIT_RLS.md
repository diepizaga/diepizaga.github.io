# Auditoría de seguridad — Row Level Security (RLS)

**Fecha:** 2026-07-30
**Corrección post-cierre (2026-07-30):** el Riesgo 2 citaba como "comprobado" que la columna `user_id` admite `NULL`, verificado supuestamente vía el endpoint OpenAPI de PostgREST (`GET /rest/v1/`). Esa consulta en realidad requiere la clave `service_role` — con el `anon key` devuelve `{"message":"Invalid API key"}`, y el parsing interpretó silenciosamente la ausencia de la clave `definitions` en ese error como "columna no listada = nullable". No fue una verificación real; se retiró esa línea de evidencia. La conclusión de fondo (que existen filas con `user_id = NULL` en la práctica) sigue sostenida por evidencia independiente y válida: el código inserta sin `user_id` cuando no hay sesión (`index.html:1120`), `claimNullItems()` opera sobre `user_id=is.null` (`index.html:2723`), y las políticas `own_data`/`group_read` tienen la condición `OR (user_id IS NULL)` en su SQL real — ese SQL solo tiene sentido si la columna admite `NULL`. El resto de las conclusiones del Riesgo 2 no cambia.

**Alcance:** políticas RLS de las tablas `watchlist`, `profiles`, `groups`, `group_members` en el proyecto Supabase `mnstruspburqiyexhtrc`.
**Estado:** auditoría cerrada, sin correcciones implementadas. Este documento es insumo para la etapa de diseño posterior.
**Metodología:** auditoría → diseño → implementación → validación → integración/deploy. Ninguna corrección se aplica hasta que este documento esté completo y las soluciones mínimas de cada hallazgo estén diseñadas y revisadas.

## Cómo leer este documento

Cada hallazgo separa explícitamente:
1. **Comprobado** — verificado con evidencia directa (capturas del dashboard, SQL exacto de políticas, lectura de código, requests reales).
2. **Inferencia** — razonamiento apoyado en lo comprobado, pero no ejecutado/confirmado end-to-end.
3. **Impacto real** — qué pasa si el riesgo se explota.
4. **Reproducción** — pasos concretos (marcados como ejecutados o razonados-no-ejecutados).
5. **Precondiciones** — qué tiene que ser cierto para que el riesgo sea explotable, y qué tan probable es hoy.
6. **Solución mínima** — la corrección más chica que resuelve el riesgo sin rediseñar el modelo de datos completo.

---

## Riesgo 1 — `profiles`, `groups`, `group_members` sin RLS habilitado

### 1. Comprobado
Captura del dashboard de Supabase (Database → Policies), texto literal de la propia UI de Supabase para las tres tablas:
> "This table can be accessed by anyone via the Data API as RLS is disabled."

Políticas existentes en cada tabla (inertes mientras RLS esté deshabilitado, porque Postgres ignora todas las políticas de una tabla sin RLS):
- `profiles`: `profiles_read` (SELECT), `profiles_update` (UPDATE), `profiles_write` (INSERT).
- `groups`: `groups_insert` (INSERT), `groups_read` (SELECT).
- `group_members`: `gm_read` (SELECT), `gm_insert` (INSERT), `gm_delete` (DELETE).

Test de lectura anónima (sin JWT, solo `anon key` público) contra las tres tablas: `HTTP 200`, body `[]`, `Content-Range: */0`.

### 2. Inferencia
Que estas tablas tengan datos reales expuestos *hoy*. El `[]` del test anónimo es consistente con "tablas casi vacías" tanto como con "protegidas" — no se puede distinguir desde afuera. No se confirmó el conteo real de filas (requiere Table Editor con sesión de administrador del proyecto, no disponible en esta auditoría).

### 3. Impacto real
Cualquier fila presente o futura en estas tres tablas (nombre de perfil, grupos creados, membresías) es legible y escribible por cualquiera que tenga el `anon key` — público, visible en el HTML del sitio vía "Ver código fuente".

### 4. Reproducción
Ejecutado: `curl -H "apikey: <ANON_KEY>" https://mnstruspburqiyexhtrc.supabase.co/rest/v1/profiles?select=*` — sin login, sin token de usuario. Repetible igual para `groups` y `group_members`.

### 5. Precondiciones
Ninguna. No requiere cuenta, login, ni nada más que el `anon key`.

### 6. Solución mínima
Habilitar el interruptor "Enable RLS" en las tres tablas. Antes de activarlo, revisar el texto SQL exacto de cada política existente (no revisado todavía para estas tres, solo se revisó el de `watchlist`) — si alguna tiene una condición demasiado laxa (ej. `using (true)`), activar RLS no corrige nada, solo da falsa sensación de protección.

---

## Riesgo 2 — `watchlist`: bypass vía `user_id IS NULL`

### 1. Comprobado
- SQL exacto de la política `own_data`: `using ((user_id = auth.uid()) OR (user_id IS NULL)) with check (mismo)`. Aplica a `ALL` (SELECT/INSERT/UPDATE/DELETE).
- SQL exacto de la política `group_read`: incluye el mismo `OR (user_id IS NULL)`, además de la lógica de grupos compartidos (ver Riesgo 3).
- Código del cliente inserta sin `user_id` cuando no hay sesión activa: `index.html:1120` — `const p = currentUser?.id ? { ...clean, user_id: currentUser.id } : clean;`
- Función `claimNullItems()` (`index.html:2723-2725`): `PATCH watchlist?user_id=is.null` con body `{ user_id: currentUser.id }`.
- `claimNullItems()` se invoca desde `onSignedIn()` (`index.html:2705`), que corre en **cada login, de cualquier cuenta** — no es un caso especial ni una función administrativa.
- Alta de cuenta sin restricciones: `POST /auth/v1/signup` con solo email+password (`index.html:2783`); el código llama a `signInWithPassword()` inmediatamente después del signup exitoso, sin esperar confirmación de email — indicio fuerte (no confirmado en el dashboard) de que "Confirm email" está deshabilitado en el proyecto.

### 2. Inferencia
Que este bypass sea explotado en la práctica contra datos reales. No se ejecutó un ataque de prueba (no se creó cuenta de prueba, no se corrió el `PATCH`/`GET` con `user_id=is.null` autenticado) para no crear ni tocar datos sin autorización explícita. Los pasos de reproducción (punto 4) son razonamiento directo sobre el SQL comprobado, no un exploit verificado end-to-end.

### 3. Impacto real
Cualquier cuenta (creable libremente) puede:
- **Leer** cualquier fila de `watchlist` con `user_id = NULL` en el momento de la consulta.
- **Apropiarse** de esas filas ejecutando el mismo `PATCH` que usa la app legítima.

### 4. Reproducción (razonada, no ejecutada)
1. Atacante crea cuenta vía `/auth/v1/signup` con cualquier email/password.
2. Inicia sesión, obtiene JWT propio.
3. Mientras exista una fila con `user_id IS NULL` (ver precondición): `GET /watchlist?user_id=is.null` con su JWT → permitido por el `OR (user_id IS NULL)` de `own_data`/`group_read`, sin relación con su propio `auth.uid()`.
4. Opcional: `PATCH /watchlist?user_id=is.null` con `{user_id: <su_id>}` → apropiación de la fila.

### 5. Precondiciones
Debe existir, en el momento del intento, al menos una fila con `user_id = NULL` en `watchlist`. Esto solo ocurre si la sesión del usuario legítimo expira (o hay una condición de carrera) justo al agregar un ítem, antes de que `claimNullItems()` la reclame en el próximo login. Es una ventana de tiempo, no un estado permanente. No se verificó si existen filas así hoy. Probabilidad de explotación real: **baja pero no nula**, y depende de un evento (expiración de sesión) fuera del control del atacante — a diferencia del Riesgo 1, que no tiene precondiciones.

### 6. Solución mínima
No eliminar directamente el `OR (user_id IS NULL)` — rompería la función de adopción de ítems huérfanos, que es intencional. Reemplazar el `PATCH` client-side abierto por una función Postgres (`security definer`) invocada vía RPC, con una condición de adopción más estricta que "cualquier cuenta logueada después". Sacar el `OR (user_id IS NULL)` de las políticas de lectura/escritura directas — el claim pasa a ser la única puerta de acceso a filas huérfanas, no una condición abierta en la política.

---

## Riesgo 3 — Cadena `group_members` (sin RLS) → `watchlist.group_read`

### 1. Comprobado
- SQL exacto de `group_read`: self-join de `group_members` por `group_id`, filtrando por usuarios que comparten grupo con `auth.uid()`.
- `group_members` tiene RLS deshabilitado (evidencia del Riesgo 1).

### 2. Inferencia
Que un atacante pueda insertarse exitosamente en `group_members` vía API directa. Inferencia de alta confianza (con RLS deshabilitado, el acceso depende solo de los GRANT de Postgres, y el patrón por defecto de Supabase suele otorgar INSERT al rol `authenticated`), pero no verificada con un INSERT real.

### 3. Impacto real
Si la inferencia es correcta, cualquier cuenta puede unirse a cualquier grupo sin invitación y, vía `group_read`, leer el `watchlist` de todos los miembros reales de ese grupo — no limitado a filas huérfanas.

### 4. Reproducción (razonada, no ejecutada)
1. Atacante crea cuenta (igual que Riesgo 2).
2. Consigue o adivina un `group_id` válido — no necesita el `invite_code` de la UI, porque sin RLS un INSERT directo a la API lo saltea.
3. `POST /group_members` con `{group_id, user_id: <su_id>, role:'member'}`.
4. `GET /watchlist` con su JWT → `group_read` expone los ítems de todos los miembros del grupo.

### 5. Precondiciones
Requiere un `group_id` real existente. Dado que `groups` aparenta estar casi vacía (Riesgo 1), este riesgo es **hoy más teórico que práctico** — se vuelve real en cuanto la función de grupos se empiece a usar.

### 6. Solución mínima
Habilitar RLS en `group_members` con política de INSERT limitada a `user_id = auth.uid()`, y que la membresía solo pueda crearse vía función server-side que valide el `invite_code` correcto — no vía INSERT directo del cliente a la tabla.

---

## Riesgo 4 (menor) — Superposición de políticas en `watchlist`

No es un riesgo nuevo: con el SQL a la vista, `owner_select/insert/update/delete` son estrictas y correctas (`auth.uid() = user_id`, sin atajos); `own_data` es la única política laxa (ver Riesgo 2). Es deuda técnica de claridad — Postgres combina políticas permisivas con OR, así que tener 6 políticas para 4 comandos en una sola tabla dificulta razonar cuál manda. Recomendación: consolidar en una política por comando una vez resuelto el Riesgo 2.

---

## Priorización por explotabilidad hoy

De más a menos inmediato, según precondiciones:

1. **Riesgo 1** — sin precondiciones, explotable ahora mismo si hay o llega a haber cualquier dato en las tres tablas.
2. **Riesgo 2** — requiere una ventana de tiempo específica (sesión vencida al momento de agregar un ítem).
3. **Riesgo 3** — requiere que la función de grupos ya esté en uso con datos reales, que aparenta no ser el caso todavía.

## Pendiente antes de diseñar la corrección

- Confirmar en el dashboard (Table Editor) si `profiles`, `groups`, `group_members` tienen filas reales hoy.
- Confirmar en Authentication → Settings si "Confirm email" está deshabilitado (inferido del código, no confirmado directamente).
- Leer el texto SQL exacto de las políticas de `profiles`, `groups`, `group_members` antes de activar RLS (se leyó el nombre y comando, no el `USING`/`WITH CHECK`).
- Decidir si el auto-registro abierto se mantiene tal cual, dado que es la precondición común a los Riesgos 2 y 3.
