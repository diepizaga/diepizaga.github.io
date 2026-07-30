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
| EP-3 | **Alto** | Confusión de repos: dos remotes locales (`origin`→`cinelog.git`, `root`→`diepizaga.github.io.git`) con historias divergentes (22 commits únicos en uno, 43 en el otro), sirviendo dos URLs públicas distintas con contenido distinto. | **Resuelto** (identificación) — Diego confirmó que `root`/`diepizaga.github.io.git` es el repo canónico (la PWA que usa a diario). Ningún trabajo se perdió: el rediseño v3 ya estaba commiteado ahí. `origin`/`cinelog.git` está confirmado como stale (sin actividad real desde 07/08-jun). **Recableo de tracking y destino de `origin` quedan pendientes — ver sección 5.** |
| EP-4 | **Medio** | `index.html.bak` / `index_old_backup.html`: verificados idénticos entre sí y con su contenido completamente cubierto por HEAD + el v3 actual (comparación por conjunto de funciones, no solo por hash). | Verificado, seguro de borrar — pendiente de ejecución (ver sección 5). |
| EP-5 | **Medio** | `books_rating_backup_2026-06-12.json` vacío (`[]`). | Verificado como comportamiento esperado — el script solo escribe al backup los libros con match exitoso; cero matches ese día, no una migración sin respaldo. No requiere acción. |
| EP-6 | **Bajo** | `titulos_backup_2026-06-12.json` (922 registros) confirma que la migración es-ES→es-AR de títulos **sí corrió** en junio. | Corrige un supuesto anterior ("migración pendiente"); no requiere acción. |

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

### Requieren tu decisión (no son técnicos, son de producto/alcance)
- **Tracking de `main`:** recablear la rama local para que trackee `root` (`diepizaga.github.io.git`) en vez de `origin`.
- **Destino de `origin`/`cinelog.git`:** deprecar el deploy (apagar Pages), mantenerlo sincronizado, o borrar el repo.
- **Los 3 scripts `.py` con credenciales:** dejarlos ignorados en `.gitignore` tal cual, o sanearlos para leer credenciales de variables de entorno.
- **Auto-registro abierto (PROD-1 / precondición de SEC-2 y SEC-3):** mantenerlo abierto o restringirlo — define buena parte del diseño de la corrección de seguridad.
- **Alcance de Grupo:** ¿se usa hoy? Cambia la prioridad relativa de SEC-3/PROD-2/PROD-6.
- **Ambición de producto:** ¿estrictamente personal + círculo cerrado, o hay intención de que otros lo usen? Incide sobre PROD-1, PROD-7 (exportación) y la superficie de auth en general.
- **Expectativa de escala:** ¿cientos o miles de ítems? Incide sobre la prioridad real de PROD-9.

### Requieren tu validación en el dashboard de Supabase (no pude verificarlas desde acá)
- Conteo real de filas en `profiles`, `groups`, `group_members` (Table Editor).
- Si "Confirm email" está deshabilitado en Authentication → Settings (inferido del código, no confirmado).
- Texto SQL exacto (`USING`/`WITH CHECK`) de las políticas de `profiles`, `groups`, `group_members` — solo se revisó nombre y comando, no el texto completo (a diferencia de `watchlist`, que sí se revisó completo).

### Ejecución mecánica pendiente (diseñada, no ejecutada — no requiere nueva decisión, solo el ok para tocar el repo)
- Borrar `index.html.bak` e `index_old_backup.html` (EP-4).
- Crear `.gitignore` (backups, scripts con credenciales, `.bak`).

---

## 6. Cómo usar este documento en una sesión futura

1. Leer este tablero primero — no los documentos de detalle.
2. Si vas a tocar algo relacionado a un hallazgo, abrir el documento de detalle correspondiente por su ID (ej. "SEC-2" → `SECURITY_AUDIT_RLS.md` § Riesgo 2).
3. Los pendientes de la sección 5 **no bloquean el diseño en general**. Cada bloque de diseño debe declarar explícitamente de qué decisiones o validaciones pendientes depende — solo esas bloquean ese bloque puntual. Un bloque que no depende de ningún pendiente de la sección 5 avanza sin esperar nada.
4. Metodología vigente para lo que sigue: diseño → revisión → documento de implementación → implementación por etapas → validación → recién después integración/deploy (misma que FitCoach). Cada bloque de diseño arranca declarando sus dependencias de la sección 5, si las tiene.
