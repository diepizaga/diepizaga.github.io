# Tablero de auditoría — Archivo

**Fecha de cierre de la etapa de auditoría:** 2026-07-30
**Propósito de este documento:** punto de entrada para cualquier sesión futura. No resume el contenido de los otros documentos — indexa el estado del proyecto, apunta al detalle, y deja explícito qué sigue abierto y de quién depende. La etapa de diseño no empieza hasta que los pendientes de decisión de este tablero se resuelvan o se declaren conscientemente diferidos.

---

## 1. Áreas auditadas

| Área | Estado | Documento de detalle |
|---|---|---|
| Estado del proyecto (repo, git, credenciales) | **Cerrada** — hallazgos críticos corregidos; pendientes de config documentados abajo | Este documento (sección 2) — no tiene doc propio, se auditó en conversación |
| Seguridad (RLS, Supabase) | **Cerrada** — documentada, **sin corrección implementada** | [SECURITY_AUDIT_RLS.md](SECURITY_AUDIT_RLS.md) |
| Producto (features, deuda técnica) | **Cerrada** — descriptiva, **sin priorización** | [PRODUCT_AUDIT.md](PRODUCT_AUDIT.md) |

Ninguna de las tres áreas tiene correcciones de código aplicadas todavía, salvo las de credenciales listadas en la sección 2 (rotación de secretos, no cambios de arquitectura).

---

## 2. Estado del proyecto — hallazgos

| ID | Severidad | Hallazgo | Estado |
|---|---|---|---|
| EP-1 | **Crítico** | Token PAT de GitHub embebido en texto plano en `.git/config`; además expuesto en esta conversación por un error mío de redirección de shell. | **Resuelto** — revocado por Diego, confirmado con `401` vía API de GitHub. |
| EP-2 | **Crítico** | Contraseña de Supabase (`die.zaga@gmail.com`) hardcodeada en `migrate_titles.py` y `migrate_books_rating.py`; confirmado que era la contraseña **vigente** de la cuenta (mismo proyecto Supabase que usa la app en producción). | **Resuelto** — cambiada por Diego vía flujo de "olvidé mi contraseña" de la propia app. |
| EP-3 | **Alto** | Confusión de repos: dos remotes locales (`origin`→`cinelog.git`, `root`→`diepizaga.github.io.git`) con historias divergentes (22 commits únicos en uno, 43 en el otro), sirviendo dos URLs públicas distintas con contenido distinto. | **Resuelto** — `main` reseteado a `root/main` y tracking configurado a `root` (Bloque A, 30-jul). Destino de `origin`/`cinelog.git` queda **diferido explícitamente**, sin bloquear el proyecto. |
| EP-4 | **Medio** | `index.html.bak` / `index_old_backup.html`: verificados idénticos entre sí y con su contenido completamente cubierto por HEAD + el v3 actual (comparación por conjunto de funciones, no solo por hash). | **Resuelto** — eliminados en Bloque A (30-jul), `.gitignore` agregado para prevenir backups futuros. |
| EP-5 | **Medio** | `books_rating_backup_2026-06-12.json` vacío (`[]`). | Verificado como comportamiento esperado — el script solo escribe al backup los libros con match exitoso; cero matches ese día, no una migración sin respaldo. No requiere acción. |
| EP-6 | **Bajo** | `titulos_backup_2026-06-12.json` (922 registros) confirma que la migración es-ES→es-AR de títulos **sí corrió** en junio. | Corrige un supuesto anterior ("migración pendiente"); no requiere acción. |
| EP-7 | **Crítico** | `import_movies.py` contiene la clave **`service_role`** de Supabase (no la `anon key`) en texto plano — a diferencia de la `anon key`, esta ignora RLS por completo y da acceso total de lectura/escritura a cualquier tabla. Nunca estuvo en git (script siempre untracked), pero quedó expuesta en esta conversación al leer el archivo durante el Bloque A (30-jul). | **Pendiente** — Diego debe regenerarla desde el dashboard de Supabase (Settings → API → service_role secret). Independiente de EP-1/EP-2, no bloquea el diseño ni la implementación de B/C. |

---

## 3. Seguridad (RLS) — hallazgos

Detalle completo, evidencia y reproducción en [SECURITY_AUDIT_RLS.md](SECURITY_AUDIT_RLS.md).

| ID | Severidad | Hallazgo | Documento |
|---|---|---|---|
| SEC-1 | **Crítico** | `profiles`, `groups`, `group_members` sin RLS habilitado — accesibles por cualquiera con el `anon key` (público, en el HTML). Sin precondiciones. | SECURITY_AUDIT_RLS.md § Riesgo 1 |
| SEC-2 | **Crítico** | `watchlist`: bypass vía `OR (user_id IS NULL)` en las políticas `own_data`/`group_read` — cualquier cuenta auto-registrada puede leer o apropiarse de filas huérfanas. Requiere una ventana de tiempo (sesión vencida al agregar un ítem). | SECURITY_AUDIT_RLS.md § Riesgo 2 |
| SEC-3 | **Alto** | Cadena `group_members` (sin RLS) → `watchlist.group_read`: permite forjar membresía en cualquier grupo y leer los watchlists compartidos de sus miembros reales. Requiere que la función de grupos ya tenga datos reales (hoy aparenta no ser el caso). | SECURITY_AUDIT_RLS.md § Riesgo 3 |
| SEC-4 | **Medio** | 6 políticas para 4 comandos en `watchlist` — superposición que dificulta razonar cuál gobierna el acceso real. Ya identificada cuál es la laxa (`own_data`). | SECURITY_AUDIT_RLS.md § Riesgo 4 |

**Corrección de método aplicada durante la auditoría (transparencia):** el documento citaba como "comprobado" que `user_id` admite `NULL` vía un chequeo de schema OpenAPI que en realidad requiere `service_role` y nunca devolvió datos reales. Se corrigió en el propio documento; la conclusión de fondo se sostiene por evidencia independiente (código fuente + texto SQL real de las políticas), no por ese chequeo.

---

## 4. Producto — hallazgos

Detalle completo en [PRODUCT_AUDIT.md](PRODUCT_AUDIT.md). Severidad asignada acá según impacto en integridad de producto/datos, no como riesgo de seguridad.

| ID | Severidad | Hallazgo | Documento |
|---|---|---|---|
| PROD-1 | **Alto** | Auto-registro abierto no refleja la intención de producto (archivo personal + grupo cerrado). Es además la precondición común de SEC-2 y SEC-3. | PRODUCT_AUDIT.md § 4, § 5 |
| PROD-2 | **Alto** | Modelo de datos permite pertenecer a varios grupos (`group_members` muchos-a-muchos); la UI asume uno solo (`loadCurrentGroup` toma el primer resultado). | PRODUCT_AUDIT.md § 4 |
| PROD-3 | **Medio** | `tmdb_rating` almacena semánticas distintas según tipo de ítem (rating TMDB para películas/series, rating Google Books ×1-5 para libros). | PRODUCT_AUDIT.md § 3, § 4 |
| PROD-4 | **Medio** | "Directores favoritos" (ADN) es una funcionalidad muerta — ningún camino de alta completa `item.director`. | PRODUCT_AUDIT.md § 6 |
| PROD-5 | **Medio** | Importar CSV no soporta libros, pese a que son ciudadanos de primera clase en el resto de la app. | PRODUCT_AUDIT.md § 4 |
| PROD-6 | **Medio** | El feature Grupo/Compatibilidad da por sentado un aislamiento de datos que hoy no existe (depende de SEC-1/SEC-3). | PRODUCT_AUDIT.md § 4 |
| PROD-7 | **Bajo** | Sin exportación de datos — solo existe importación. | PRODUCT_AUDIT.md § 6 |
| PROD-8 | **Bajo** | PWA sin service worker — instalable, sin capacidad offline real. | PRODUCT_AUDIT.md § 6 |
| PROD-9 | **Bajo** | Sin paginación en `loadItems()` — no es problema al volumen actual. | PRODUCT_AUDIT.md § 3 |
| PROD-10 | **Bajo** | Caché de recomendaciones (`discCache`) se invalida por completo en cada escritura, sin persistencia entre recargas. | PRODUCT_AUDIT.md § 3 |
| PROD-11 | **Bajo** | Parser de CSV propio no maneja comillas escapadas (`""`). | PRODUCT_AUDIT.md § 3 |
| PROD-12 | **Bajo** | Alias de CSS legacy (`--bg-2`, `--t-1`, etc.) coexistiendo permanentemente con la paleta v3. | PRODUCT_AUDIT.md § 3 |
| PROD-13 | **Bajo** | Scripts de migración con credenciales hardcodeadas (mismo hallazgo que EP-2, ángulo de deuda técnica en vez de credencial expuesta). | PRODUCT_AUDIT.md § 3 — ver también EP-2 |

---

## 5. Pendientes que requieren decisión o validación tuya

Nada de esto bloquea el cierre de la auditoría — quedan registrados para resolver en la etapa de diseño o cuando corresponda.

**Nota (30-jul-2026):** la mayoría de lo que estaba abierto acá se resolvió durante el Bloque 0 (visión de producto) y el Bloque A (housekeeping) — ver [DESIGN_MAP.md](DESIGN_MAP.md) para el detalle de cada decisión. Esta sección queda solo con lo genuinamente pendiente hoy.

### Resueltas (referencia, detalle en DESIGN_MAP.md)
- Tracking de `main` → recableado a `root` (Bloque A).
- `.bak` → eliminados, `.gitignore` creado (Bloque A).
- Auto-registro abierto → resuelto direccionalmente por Bloque 0 (visión: invitación y confianza, nunca registro público abierto).
- Alcance de Grupo → resuelto por Bloque 0 (no se usa hoy, no debe condicionar prioridades).
- Ambición de producto / escala esperada → resueltas por Bloque 0 (ver DESIGN_MAP.md).
- Texto SQL de las políticas de `profiles`/`groups`/`group_members` → obtenido y revisado (diseño de Bloque B).
- Conteo real de filas en `watchlist` para `user_id IS NULL` → confirmado en 0 (diseño de Bloque C).

### Todavía requieren tu decisión
- **Destino de `origin`/`cinelog.git`:** deprecar el deploy, mantenerlo sincronizado, o borrar el repo — **diferido explícitamente por Diego (30-jul)**, no bloquea el proyecto.
- **Los 3 scripts `.py` con credenciales:** dejarlos ignorados en `.gitignore` tal cual, o sanearlos para leer credenciales de variables de entorno.

### Todavía requieren tu validación en el dashboard de Supabase
- Conteo real de filas en `profiles`, `groups`, `group_members` (Table Editor) — sigue sin confirmarse (distinto del conteo de `watchlist`, que sí se hizo).
- Si "Confirm email" está deshabilitado en Authentication → Settings (inferido del código, no confirmado).
- **Nuevo (EP-7): regenerar la `service_role` key** expuesta en `import_movies.py` — Settings → API → service_role secret.

---

## 6. Cómo usar este documento en una sesión futura

1. Leer este tablero primero — no los documentos de detalle.
2. Si vas a tocar algo relacionado a un hallazgo, abrir el documento de detalle correspondiente por su ID (ej. "SEC-2" → `SECURITY_AUDIT_RLS.md` § Riesgo 2).
3. Los pendientes de la sección 5 **no bloquean el diseño en general**. Cada bloque de diseño debe declarar explícitamente de qué decisiones o validaciones pendientes depende — solo esas bloquean ese bloque puntual. Un bloque que no depende de ningún pendiente de la sección 5 avanza sin esperar nada.
4. Metodología vigente para lo que sigue: diseño → revisión → documento de implementación → implementación por etapas → validación → recién después integración/deploy (misma que FitCoach). Cada bloque de diseño arranca declarando sus dependencias de la sección 5, si las tiene.
