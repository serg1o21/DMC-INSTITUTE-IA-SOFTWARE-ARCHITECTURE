# 07 - Arquitectura inicial: miStorePe

| Campo | Valor |
|---|---|
| Producto | miStorePe |
| Versión del documento | 1.0 (aprobada) |
| Fecha | 2026-11-10 |
| Basado en | `00-constitucion.md`, `01-project-brief.md`, `02-alcance.md`, `03-reglas-negocio.md`, `06-user-stories-refinadas.md` |
| Alcance | MVP: F-01 a F-06, HU-001 a HU-012 |
| Estado | **Aprobada por el Responsable del producto el 2026-11-10**, junto con los ADR 0002 a 0013 y las dependencias de la sección 6.5; ADR-0014 aprobado el 2026-11-12. Cualquier cambio requiere un ADR nuevo aprobado por Sergio (PRINCIPIO-009, PRINCIPIO-012) |

Convenciones:
- Los identificadores técnicos (tablas, columnas, rutas, códigos de error) están en inglés; los textos visibles, en español (PRINCIPIO-001).
- Las fechas se guardan en UTC (`timestamptz`) y se muestran en America/Lima (UTC-05:00).
- Los montos se guardan como `NUMERIC(12,2)` y se manejan como `BigDecimal` en Java y como cadena (`"35.90"`) en JSON, nunca como `double`.
- Las preguntas abiertas (A-nn, M-nn, P-nn) se refieren a `05-preguntas-abiertas.md` y a la auditoría QA. Lo marcado **[BLOQUEADO]** no se implementa hasta que se responda. P-61, P-64, P-66 y P-67 fueron resueltas el 2026-11-11. P-74 y P-75 se resolvieron el 2026-11-12 con ADR-0014 (Aprobado). No hay decisiones pendientes que bloqueen el desarrollo; P-62 bloquea solo el despliegue del piloto. Ver sección 6.4.

---

## 1. Overview del sistema

### 1.1 Descripción

miStorePe es una aplicación web responsiva que permite a pequeños emprendedores de Lima controlar el stock por variante (talla y color, presentación en ml), registrar entradas con su costo y ventas con su precio final, y ver la ganancia de cada venta. Es un monolito modular en Java 21 con Spring Boot 4.x que expone una API REST consumida por una SPA en Angular, con PostgreSQL 18 como única fuente de verdad, desplegado en hosting propio. Cada emprendedor es un tenant aislado por `tenant_id`, con un doble control: filtro obligatorio en la capa de servicio y Row-Level Security en PostgreSQL. El diseño prioriza celulares Android de gama baja y conectividad intermitente: interfaz liviana, respuestas pequeñas y operaciones de escritura idempotentes que se pueden reintentar sin duplicar ventas. Con 3 emprendedores en el piloto y 30-40 productos cada uno, la arquitectura evita a propósito la complejidad distribuida.

### 1.2 Diagrama de arquitectura

```
 Celular Android (gama baja) / Laptop
 +-------------------------------------+
 | Navegador |
 | Angular SPA (es-PE) |
 | - App shell en caché (ADR-0006) |
 | - Borrador de venta en memoria |
 | - Reintento con Idempotency-Key |
 +------------------+------------------+
 | HTTPS (TLS 1.2+, HSTS)
 | Cookie de sesión HttpOnly + XSRF-TOKEN
 v
+--------------------------- Servidor de hosting propio ---------------------------+
| |
| +-------------------------------+ |
| | Reverse proxy (ADR-0008) | - Termina TLS, redirige 80 -> 443, HSTS |
| | - Sirve estáticos de Angular | - Compresión gzip/brotli |
| | - Proxy /api -> backend | - Límite de peticiones en /api/auth/* |
| +---------------+---------------+ |
| | HTTP local (127.0.0.1) |
| v |
| +--------------------------------------------------------------------------+ |
| | Backend: Spring Boot 4.x / Java 21 (monolito modular) | |
| | | |
| | [web/api] Controllers REST, ProblemDetail (RFC 9457), validación | |
| | [shared] TenantContext, IdempotencyFilter, Clock (America/Lima), | |
| | Money, auditoría, logs sin datos personales | |
| | | |
| | +-----------+ +-----------+ +-------------+ +-----------+ | |
| | | identity | | catalog | | inventory | | sales | | |
| | | cuentas, | | productos,| | entradas, | | ventas, | | |
| | | sesiones, | | variantes,| | ajustes, | | líneas, | | |
| | | bloqueo | | SKU | | stock, | | ganancia, | | |
| | | | | | | costo prom. | | anulación | | |
| | +-----------+ +-----------+ +-------------+ +-----------+ | |
| | +--------------------------------------------------------+ | |
| | | lifecycle: borrado a 30 días (mecanismo por decidir), | | |
| | | limpieza de sesiones y claves de idempotencia) | | |
| | +--------------------------------------------------------+ | |
| +-----------------------------------+--------------------------------------+ |
| | JDBC (TLS local) + SET LOCAL app.tenant_id|
| v |
| +--------------------------------------------------------------------------+ |
| | PostgreSQL 18 (disco cifrado AES-256, RLS por tenant_id) | |
| +-----------------------------------+--------------------------------------+ |
| | pg_dump diario |
| v |
| +--------------------------------------------------------------------------+ |
| | Backups cifrados AES-256, sin cuentas cerradas, retención 30 días | |
| +--------------------------------------------------------------------------+ |
+----------------------------------------------------------------------------------+

Fuera del sistema (W-01 a W-04): pagos, SUNAT, redes sociales, datos de compradores.
```

### 1.3 Principios de diseño

| ID | Principio | Por qué |
|---|---|---|
| DP-01 | Monolito modular, un solo despliegue | Una sola persona desarrolla y opera; 3 usuarios en el piloto. Los módulos se comunican por interfaces Java, no por red. |
| DP-02 | El tenant sale siempre de la sesión | PRINCIPIO-003. Ningún endpoint acepta `tenantId` en ruta, query ni cuerpo. |
| DP-03 | Defensa en profundidad del aislamiento | Filtro en servicio + RLS en PostgreSQL (ADR-0003). Un olvido en una consulta no expone datos de otro emprendedor. |
| DP-04 | El stock solo cambia por movimientos | RN-024. No existe endpoint para fijar el stock; cada cambio deja una fila de entrada, venta, ajuste o anulación. |
| DP-05 | Transacciones cortas con bloqueo por variante | Evita ventas simultáneas que dejen stock negativo (HU-009) y dobles anulaciones (HU-010). |
| DP-06 | Escrituras idempotentes | Con conectividad intermitente el usuario reintenta; un reintento nunca registra dos veces la misma venta (ADR-0005). |
| DP-07 | Valores históricos congelados | La ganancia de una venta no cambia cuando entra mercadería más cara (HU-011): cada línea guarda el costo unitario usado. |
| DP-08 | Liviano para gama baja | Presupuesto de bundle, lazy loading, paginación y respuestas JSON pequeñas (sección 6.3). |
| DP-09 | Mínimos datos personales | PRINCIPIO-005: solo nombre, correo, hash y fecha de aceptación de la política. Nada de compradores ni pagos. |
| DP-10 | Preguntar antes de asumir | Las decisiones no cerradas quedan marcadas como [BLOQUEADO] o "Propuesto" (PRINCIPIO-009). |

---

## 2. Componentes principales

| # | Componente | Responsabilidad | Tecnología | HUs que implementa |
|---|---|---|---|---|
| C-01 | `web-app` (SPA) | Pantallas en español, responsivas desde 360 px, WCAG 2.2 AA. Formularios de registro, login, catálogo, entradas, venta rápida para lives, historial, stock y ajustes. Maneja el borrador de la venta, el reintento con `Idempotency-Key` y el formato `S/ 1,250.50`. | Angular (standalone components, signals, lazy routes), `HttpClient` con soporte XSRF nativo, CSS propio sin librería de componentes (ADR-0006) | HU-001 a HU-012 (todas las pantallas), T-01, T-03, T-04 |
| C-02 | Reverse proxy | Termina TLS 1.2+, HSTS, redirección 80 a 443, compresión, sirve los estáticos de Angular, limita la tasa de `/api/auth/*`. | Nginx (aprobada, ADR-0008 y sección 6.5) | HU-001, HU-002 (protección del login); transversal PRINCIPIO-004 |
| C-03 | `identity` (módulo backend) | Registro, normalización del correo (`trim` + minúsculas), hash Argon2id, login, contador de intentos y bloqueo de 900 s, sesiones con inactividad de 28,800 s, cambio de contraseña que cierra las demás sesiones, cierre de cuenta con contraseña (RN-127), reapertura y rechazo del login a los 30 días del cierre (RN-128). | Spring Security 7, `Argon2PasswordEncoder`, Spring Data JPA | HU-001, HU-002, HU-003, HU-004 |
| C-04 | `catalog` (módulo backend) | Productos y variantes por categoría (RN-058 a RN-060), variante "Única" automática, SKU por tenant y categoría sin reutilización, edición con restricciones (RN-098), desactivación/reactivación, auditoría de cambios de vencimiento (RN-115), advertencia de precio menor que costo. | Spring Boot, Spring Data JPA, Bean Validation | HU-005, HU-006 |
| C-05 | `inventory` (módulo backend) | Entradas con costo, costo promedio ponderado con redondeo `HALF_UP` a 2 decimales, anulación de entradas (24 h y sin movimientos posteriores), ajustes por conteo físico con motivo, consulta y búsqueda de stock. Único dueño de la columna `variant.stock`. | Spring Boot, JPA, `SELECT ... FOR UPDATE` | HU-007, HU-008, HU-012 |
| C-06 | `sales` (módulo backend) | Ventas con una o más líneas, fusión de líneas con igual precio, validación de stock, vencimiento y fecha (hoy - 7 días a hoy), confirmación de pérdida, cálculo y congelado de la ganancia por línea, anulación de venta o línea en 24 h, historial paginado. Descuenta stock a través de la interfaz de `inventory`. | Spring Boot, JPA | HU-009, HU-010, HU-011 |
| C-07 | `shared` / plataforma | `TenantContext` (tenant desde la sesión), `SET LOCAL app.tenant_id` por transacción, filtro de idempotencia, `Clock` inyectable, manejo de errores con `ProblemDetail` y mensajes en español, logs con ID interno y sin correo, nombre, contraseña ni tokens. | Spring Boot, SLF4J/Logback | T-02, T-05; transversal a HU-001 a HU-012 |
| C-08 | `lifecycle` (jobs) | Borrado definitivo de cuentas con `closed_at` de más de 30 días y todas sus filas por `tenant_id` (job diario `@Scheduled`, tolerancia 24 h, aprobado P-61, T-025b); limpieza diaria (T-025a) de sesiones vencidas y claves de idempotencia de más de 48 h. | `@Scheduled` de Spring, ShedLock no requerido (una sola instancia) | HU-004, T-07 |
| C-09 | Base de datos | Fuente única de verdad, constraints de unicidad e integridad, RLS por `tenant_id`, migraciones versionadas. | PostgreSQL 18, Flyway (aprobada, sección 6.5) | HU-001 a HU-012 |
| C-10 | Backups | `pg_dump` diario cifrado con AES-256 que excluye las filas de cuentas con `status = CLOSED` (RN-129), retención rodante máxima de 30 días. | Script del servidor (cron) | HU-004 (criterio de backups), PRINCIPIO-006 |

Reglas de dependencia entre módulos backend:
- `sales` llama a `inventory` (descontar y devolver stock) y a `catalog` (leer variante); nunca escribe en sus tablas directamente.
- `inventory` llama a `catalog` solo para leer la variante.
- `identity` no depende de ningún otro módulo de negocio. `lifecycle` borra por `tenant_id` en todas las tablas.
- Se verifica con un test de arquitectura en el CI (ArchUnit, aprobada en la sección 6.5).

Estructura de paquetes sugerida (backend):

```
pe.mistore
 ├─ shared (tenancy, idempotency, error, money, time, audit, logging)
 ├─ identity (api, application, domain, infrastructure)
 ├─ catalog (api, application, domain, infrastructure)
 ├─ inventory (api, application, domain, infrastructure)
 ├─ sales (api, application, domain, infrastructure)
 └─ lifecycle (jobs)
```

---

## 3. Modelo de datos

### 3.1 Diagrama de relaciones

```
account (1) ──< user_session
account (1) ──< product (1) ──< variant (1) ──< stock_entry
 │ │ └──< sale_line >── (1) sale
 │ │ └──< stock_adjustment
 │ │ └──< variant_audit
 └──< sku_counter (por categoría)
 └──< idempotency_record

tenant_id = account.id en todas las tablas de negocio.
```

### 3.2 Entidades

Todas las tablas de negocio llevan `tenant_id BIGINT NOT NULL REFERENCES account(id)`, índice que empieza por `tenant_id` y política RLS (ADR-0003). Todas tienen `created_at timestamptz NOT NULL DEFAULT now()`.

#### `account`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK (identity) | También es el `tenant_id` |
| `email` | `VARCHAR(254)` UNIQUE NOT NULL | Guardado en minúsculas y sin espacios (RN-048); unicidad garantizada por constraint (RN-094, registro simultáneo) |
| `name` | `VARCHAR(100)` NULL | Opcional; máximo 100 caracteres; se corrige en "Mi cuenta" (RN-141) |
| `password_hash` | `VARCHAR(255)` NOT NULL | Prefijo `$argon2id$` (PRINCIPIO-004) |
| `privacy_accepted_at` | `timestamptz` NOT NULL | Ley N° 29733 |
| `failed_login_count` | `SMALLINT` NOT NULL DEFAULT 0 | Se reinicia a 0 al entrar (RN-049) |
| `locked_until` | `timestamptz` NULL | `último fallo + 900 s` al quinto fallo (RN-004) |
| `status` | `VARCHAR(20)` NOT NULL | `ACTIVE`, `CLOSED` (MVP). `SUSPENDED` llega con F-16 |
| `closed_at` | `timestamptz` NULL | Inicio del plazo de 30 días (RN-127) |
| `closure_reason` | `VARCHAR(20)` NULL | `USER` en el MVP; `NON_PAYMENT` llega con F-16 (A-03) |
| `role` | `VARCHAR(20)` NOT NULL DEFAULT `'ENTREPRENEUR'` | `ENTREPRENEUR` o `ADMIN` (glosario: Rol). `ADMIN` nunca se asigna desde el registro (RN-071); una cuenta `ADMIN` no actúa como inquilino: `TenantContext` le niega todo endpoint de inventario con 404 (ADR 0002, RN-114) |

Nota: `account` no tiene RLS por tenant (el login la consulta antes de conocer el tenant); solo la usa el módulo `identity`. ADR-0014 (aprobado, P-75) le agrega RLS solo para excluir las cuentas cerradas del rol de backup.

#### `user_session`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `UUID` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `token_hash` | `CHAR(64)` UNIQUE NOT NULL | SHA-256 del token aleatorio de 256 bits; el token en claro solo viaja en la cookie |
| `last_activity_at` | `timestamptz` NOT NULL | Se actualiza en cada petición (RN-132). Sesión válida si `now() - last_activity_at <= 28,800 s` (RN-097) |
| `revoked_at` | `timestamptz` NULL | Logout o cambio de contraseña (RN-110) |

#### `product`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `name` | `VARCHAR(100)` NOT NULL | 1 a 100 caracteres (RN-093) |
| `category` | `VARCHAR(20)` NOT NULL | `CLOTHING`, `PERFUME`, `ACCESSORY`; inmutable (HU-006) |
| `active` | `BOOLEAN` NOT NULL DEFAULT true | Desactivado no se vende (RN-016) |
| `updated_at` | `timestamptz` NOT NULL | |

#### `variant`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `product_id` | `BIGINT` NOT NULL FK | |
| `sku` | `VARCHAR(12)` NOT NULL | `ROP-00001`, `PER-00001`, `ACC-00001`; UNIQUE (`tenant_id`, `sku`) |
| `size` | `VARCHAR(10)` NULL | Obligatoria en Ropa; mayúsculas (RN-111), máximo 10 (RN-122) |
| `color` | `VARCHAR(30)` NULL | Obligatorio en Ropa; formato "Negro" (RN-121), máximo 30 (RN-123) |
| `volume_ml` | `INTEGER` NULL | Obligatorio en Perfume, entero mayor que 0 (RN-011) |
| `expiry_date` | `DATE` NULL | Solo Perfume, formato `AAAA-MM-DD` (RN-012) |
| `is_single` | `BOOLEAN` NOT NULL DEFAULT false | Variante "Única" (decidido: campo de la variante) |
| `sale_price` | `NUMERIC(12,2)` NOT NULL | 0.01 a 99,999.99 (RN-013, RN-091) |
| `stock` | `INTEGER` NOT NULL DEFAULT 0 CHECK (`stock >= 0`) | Nace en 0 (RN-061); solo lo cambia `inventory` |
| `avg_cost` | `NUMERIC(12,2)` NULL | NULL hasta la primera entrada vigente |
| `has_movements` | `BOOLEAN` NOT NULL DEFAULT false | Bloquea cambios de atributos y eliminación (RN-098, RN-099) |
| `version` | `BIGINT` NOT NULL | Bloqueo optimista para ediciones del catálogo |

Unicidad (RN-050), con índices parciales:
- Ropa: UNIQUE (`product_id`, `size`, `color`).
- Perfume: UNIQUE NULLS NOT DISTINCT (`product_id`, `volume_ml`, `expiry_date`). Trata dos vencimientos vacíos como iguales (RN-135).
- Única: UNIQUE (`product_id`) WHERE `is_single`.
- CHECK por categoría no es posible entre tablas; la validación de atributos por categoría vive en el dominio (`VariantRules`) con test unitario por regla.

#### `sku_counter`

| Columna | Tipo | Regla |
|---|---|---|
| `tenant_id` | `BIGINT` | PK compuesta (`tenant_id`, `category`) |
| `category` | `VARCHAR(20)` | |
| `last_value` | `INTEGER` NOT NULL | `UPDATE ... SET last_value = last_value + 1 RETURNING last_value`; nunca se reutiliza (RN-108) y es seguro ante creaciones simultáneas (RN-065) |

#### `stock_entry`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `variant_id` | `BIGINT` NOT NULL FK | |
| `quantity` | `INTEGER` NOT NULL CHECK 1..9999 | RN-018, RN-092 |
| `unit_cost` | `NUMERIC(12,2)` NOT NULL CHECK 0.01..99999.99 | RN-014, RN-091 |
| `previous_avg_cost` | `NUMERIC(12,2)` NULL | Para restaurar al anular (RN-118) |
| `previous_stock` | `INTEGER` NOT NULL | Trazabilidad |
| `status` | `VARCHAR(10)` NOT NULL | `ACTIVE`, `VOIDED` |
| `voided_at` | `timestamptz` NULL | |
| `movement_seq` | `BIGINT` NOT NULL | Secuencia global de movimientos para saber qué hubo "después" (RN-056) |

#### `sale`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `sale_at` | `timestamptz` NOT NULL | Fecha y hora de venta. Por defecto, el momento del registro (RN-126); el usuario puede elegir día y hora aproximada desde hoy - 7 días calendario de Lima hasta ahora (RN-076 a RN-078) |
| `status` | `VARCHAR(10)` NOT NULL | `ACTIVE`, `VOIDED` (pasa a `VOIDED` cuando todas sus líneas lo están, RN-109) |
| `total_amount` | `NUMERIC(14,2)` NOT NULL | Suma de líneas vigentes |
| `total_profit` | `NUMERIC(14,2)` NOT NULL | Suma de ganancias de líneas vigentes (RN-082) |
| `loss_confirmed` | `BOOLEAN` NOT NULL DEFAULT false | RN-073 |

Sin columnas de comprador (PRINCIPIO-005, W-04).

#### `sale_line`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `sale_id` | `BIGINT` NOT NULL FK | |
| `variant_id` | `BIGINT` NOT NULL FK | UNIQUE (`sale_id`, `variant_id`) (RN-084, RN-107) |
| `quantity` | `INTEGER` NOT NULL CHECK 1..9999 | |
| `unit_price` | `NUMERIC(12,2)` NOT NULL CHECK 0.01..99999.99 | Precio final real (RN-025) |
| `unit_cost` | `NUMERIC(12,2)` NOT NULL | Costo promedio al momento del registro, congelado (RN-028, RN-079) |
| `profit` | `NUMERIC(14,2)` NOT NULL | `(unit_price - unit_cost) * quantity` (RN-027); puede ser negativo |
| `status` | `VARCHAR(10)` NOT NULL | `ACTIVE`, `VOIDED` |
| `voided_at` | `timestamptz` NULL | |
| `movement_seq` | `BIGINT` NOT NULL | |

#### `stock_adjustment`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | |
| `variant_id` | `BIGINT` NOT NULL FK | |
| `previous_stock` | `INTEGER` NOT NULL | |
| `counted_quantity` | `INTEGER` NOT NULL CHECK 0..9999 | RN-040 |
| `difference` | `INTEGER` NOT NULL | `counted - previous` (RN-037) |
| `reason_type` | `VARCHAR(30)` NOT NULL | `PHYSICAL_COUNT`, `EXPIRY_WASTE` ("Merma por vencimiento", RN-075) |
| `reason_text` | `VARCHAR(200)` NOT NULL | De 1 a 200 caracteres, sin solo espacios (RN-143). La UI advierte "No escribas datos de tus clientes" (PRINCIPIO-005) |
| `movement_seq` | `BIGINT` NOT NULL | |

El ajuste no modifica `avg_cost` ni ninguna ganancia (HU-012). Requiere al menos una entrada vigente en la variante (RN-116).

#### `variant_audit`

| Columna | Tipo | Regla |
|---|---|---|
| `id` | `BIGINT` PK | |
| `tenant_id` | `BIGINT` NOT NULL | Se borra con la cuenta (PRINCIPIO-006) |
| `variant_id` | `BIGINT` NOT NULL FK | |
| `field` | `VARCHAR(30)` NOT NULL | En el MVP: `expiry_date` |
| `old_value` | `VARCHAR(100)` NULL | |
| `new_value` | `VARCHAR(100)` NULL | |
| `changed_at` | `timestamptz` NOT NULL | RN-115 |

#### `idempotency_record`

| Columna | Tipo | Regla |
|---|---|---|
| `tenant_id` | `BIGINT` | PK compuesta (`tenant_id`, `idempotency_key`) |
| `idempotency_key` | `UUID` | Generado por el cliente |
| `request_hash` | `CHAR(64)` NOT NULL | SHA-256 del cuerpo; misma clave con otro cuerpo da 422 |
| `response_status` | `SMALLINT` NOT NULL | |
| `response_body` | `JSONB` NOT NULL | Se devuelve tal cual en el reintento |
| `expires_at` | `timestamptz` NOT NULL | 48 h |

### 3.3 Reglas de cálculo

- Costo promedio ponderado (RN-020): `nuevo = round_half_up((stock * avg_cost + cantidad * costo_entrada) / (stock + cantidad), 2)`. Si `avg_cost` es NULL o `stock = 0`, `nuevo = costo_entrada`. Ejemplos verificables: 10 a 18.00 + 10 a 20.00 = 19.00; 1 a 10.00 + 1 a 10.01 = 10.01; 10 a 18.00 + 5 a 20.01 = 18.67.
- Anular una entrada sin movimientos posteriores: `stock -= quantity`, `avg_cost = previous_avg_cost` (si es NULL, la variante vuelve a no tener costo y el precio de venta deja de compararse con el costo, RN-062). Este caso lo cubre RN-154, aprobada el 2026-11-11 (P-64).
- Ganancia de línea: `(unit_price - unit_cost) * quantity` con `unit_cost` congelado.
- "Movimiento posterior" (RN-056): cualquier entrada, línea de venta o ajuste **vigente** de la misma variante con `movement_seq` mayor. Una venta anulada no cuenta (HU-008, criterio positivo).

### 3.4 Notas de seguridad del modelo

- RLS activa en `user_session`, `product`, `variant`, `sku_counter`, `stock_entry`, `sale`, `sale_line`, `stock_adjustment`, `variant_audit` e `idempotency_record`, con la política `tenant_id = current_setting('app.tenant_id')::bigint` (ADR-0003). El usuario de base de datos de la aplicación no es dueño de las tablas ni tiene `BYPASSRLS`.
- El job de borrado usa un rol separado con `BYPASSRLS` limitado a `DELETE`, cuyas credenciales se inyectan por variable de entorno (PRINCIPIO-007).
- La lectura de la sesión por `token_hash` (función `resolve_session`), la limpieza diaria (T-025a), el comando de restablecimiento (T-039) y el rol de backup siguen ADR-0014 (aprobado 2026-11-12).
- Las FK usan `ON DELETE RESTRICT`; el borrado de una cuenta elimina en orden explícito (líneas, ventas, ajustes, entradas, auditoría, variantes, productos, contadores, sesiones, idempotencia, cuenta) dentro de una transacción, verificado por test (HU-004, T-07).
- Ninguna columna guarda datos de compradores, de pago ni IP completas (PRINCIPIO-005).
- El token de sesión solo se guarda como hash; las contraseñas solo como Argon2id.
- Disco de la base de datos y backups cifrados con AES-256 (PRINCIPIO-004).
- El backup se genera con un rol de solo lectura cuya política RLS excluye los `tenant_id` de cuentas cerradas, y vuelca `account` sin esas filas (RN-129). Un test verifica que un backup tomado después de `closed_at` no contiene el tenant.

---

## 4. API Contracts

### 4.1 Convenciones comunes

| Aspecto | Definición |
|---|---|
| Base | `https://<dominio>/api` |
| Formato | JSON UTF-8. Montos como cadena con 2 decimales (`"35.90"`). Fechas `AAAA-MM-DD`; instantes ISO 8601 con offset `-05:00` |
| Autenticación | Cookie `MSP_SESSION` (HttpOnly, Secure, SameSite=Lax, Path=/api). Expira por inactividad en el servidor |
| CSRF | Cookie `XSRF-TOKEN` + cabecera `X-XSRF-TOKEN` en todo método distinto de GET (soporte nativo de Angular y Spring Security) |
| Tenant | Nunca viaja en la petición; se toma de la sesión. Recurso de otro tenant o inexistente: **404** con el mismo cuerpo (PRINCIPIO-003) |
| Idempotencia | `Idempotency-Key: <UUID v4>` obligatoria en `POST /stock-entries`, `POST /sales`, `POST /stock-adjustments` y en todas las anulaciones. Reintento con la misma clave y el mismo cuerpo devuelve la respuesta original |
| Paginación | `?page=0&size=20` (máximo 50). Respuesta con `items`, `page`, `size`, `totalItems` |
| Errores | `application/problem+json` (RFC 9457), `title` y `detail` en español, `code` estable en inglés para el frontend |
| Concurrencia en catálogo | `If-Match: "<version>"` en `PATCH` de productos y variantes; versión desactualizada da 409 |

Formato de error:

```json
{
 "type": "https://mistore.pe/errors/insufficient-stock",
 "title": "Stock insuficiente",
 "status": 422,
 "detail": "No se pudo registrar la venta.",
 "code": "INSUFFICIENT_STOCK",
 "errors": [
 { "field": "lines[1].quantity", "code": "INSUFFICIENT_STOCK", "message": "Stock insuficiente: quedan 0 unidades", "available": 0 }
 ]
}
```

Códigos de estado usados:

| Código | Uso |
|---|---|
| 200 / 201 / 204 | Éxito |
| 400 | Formato o validación de campos (Bean Validation) |
| 401 | Sin sesión o sesión vencida; credenciales inválidas |
| 403 | CSRF inválido; cuenta cerrada intentando operar (RN-007) |
| 404 | Recurso inexistente o de otro emprendedor |
| 409 | Conflicto de estado o unicidad (correo existente, variante duplicada, versión desactualizada, ya anulado) |
| 422 | Regla de negocio (stock insuficiente, vencido, plazo de 24 h, pérdida sin confirmar, movimientos posteriores) |
| 423 | Cuenta bloqueada temporalmente (RN-130) |
| 429 | Demasiadas peticiones al login |

### 4.2 Cuenta y acceso (C-03)

#### `POST /api/auth/register` (HU-001)

Request:
```json
{
 "email": " Ana@Gmail.com ",
 "password": "TiendaLima2026",
 "name": "Ana",
 "privacyAccepted": true
}
```

Validaciones: `password` de 12 caracteres o más (RN-002); `privacyAccepted` debe ser `true` (RN-003); correo normalizado con `trim().toLowerCase(Locale.ROOT)` (RN-048). Correo con formato `nombre@dominio.ext` y máximo 254 caracteres (RN-133); contraseña de máximo 128 caracteres (RN-134).

Response `201 Created`:
```json
{ "accountId": 4821, "email": "ana@gmail.com", "privacyAcceptedAt": "2026-11-10T12:40:00-05:00" }
```

Errores: `400` (`PASSWORD_TOO_SHORT`, `PASSWORD_TOO_LONG`, `EMAIL_INVALID`, `PRIVACY_NOT_ACCEPTED`); `409` `EMAIL_ALREADY_REGISTERED` con detalle "Este correo ya tiene una cuenta" (garantizado por el UNIQUE en registros simultáneos). Inicia la sesión automáticamente: la respuesta incluye `Set-Cookie: MSP_SESSION=...` para que el emprendedor entre sin volver a escribir su contraseña (RN-153).

#### `POST /api/auth/login` (HU-002)

Request:
```json
{ "email": " Ana@Gmail.com ", "password": "TiendaLima2026" }
```

Response `200 OK` + `Set-Cookie: MSP_SESSION=...; HttpOnly; Secure; SameSite=Lax; Path=/api`:
```json
{ "accountId": 4821, "name": "Ana", "status": "ACTIVE", "idleTimeoutSeconds": 28800 }
```

Si la cuenta tiene `status = CLOSED` (HU-004), responde `200` con `"status": "CLOSED"`, `"closedAt"` y `"reopenDeadline"`; la sesión solo permite `POST /api/account/reopen` y `POST /api/auth/logout`.

Errores: `401` `INVALID_CREDENTIALS` (mismo cuerpo exista o no el correo; un correo inexistente no suma intentos a nadie); `423` `ACCOUNT_LOCKED` con detalle "Demasiados intentos. Intenta de nuevo en 15 minutos" cuando `now() < locked_until`; esos intentos no suman ni extienden el bloqueo (RN-130, RN-131). Una cuenta con `now() - closed_at >= 2,592,000 s` responde `401` `INVALID_CREDENTIALS`, igual que un correo inexistente (RN-128); `429` por límite de tasa. El log registra "Intento fallido para la cuenta 4821", nunca el correo (RN-120).

#### `GET /api/auth/session` (HU-002, HU-004)

Permite a la SPA saber, al cargar o recargar, si hay sesión y en qué estado está la cuenta. Response `200` con `{ "accountId": 4821, "name": "Ana", "status": "ACTIVE" }`, o con `"status": "CLOSED"`, `"closedAt"` y `"reopenDeadline"`. `401` sin sesión.

#### `POST /api/auth/logout`

Sin cuerpo. Response `204`. Revoca la sesión actual.

#### `PUT /api/account/password` (HU-003)

Request:
```json
{ "currentPassword": "TiendaLima2026", "newPassword": "NuevaClaveLima2026" }
```

Response `204`. Revoca todas las demás sesiones del tenant y mantiene la actual (RN-110).

Errores: `400` (`CURRENT_PASSWORD_REQUIRED`, `PASSWORD_TOO_SHORT` sujeto a RN-105); `401` sin sesión; `422` `CURRENT_PASSWORD_INVALID` (no suma al contador de intentos de inicio de sesión, RN-157) y `422` `PASSWORD_SAME_AS_CURRENT` (RN-156).

#### `PATCH /api/account` (HU-001, HU-003)

Request: `{ "name": "Ana" }` (el nombre es opcional, máximo 100 caracteres; `null` o vacío lo borra, RN-141). Response `200` con `{ "accountId": 4821, "name": "Ana" }`. Errores: `400` `NAME_TOO_LONG`; `401` sin sesión. El correo no se puede cambiar (W-12).

#### `POST /api/account/reopen` (HU-004)

Sin cuerpo. Response `200` con `{"status": "ACTIVE"}` si `now() - closed_at < 2,592,000 s` (RN-090). `422` `REOPEN_WINDOW_EXPIRED` en otro caso.

#### `POST /api/account/close` (HU-004)

Request: `{ "currentPassword": "TiendaLima2026" }`. Response `200` con `{ "status": "CLOSED", "closedAt": "2026-12-01T10:00:00-05:00", "reopenDeadline": "2026-12-31T10:00:00-05:00" }`. Registra `closed_at`, `closure_reason = USER` y revoca todas las sesiones, incluida la actual (RN-127). Errores: `400` `CURRENT_PASSWORD_REQUIRED`; `401` sin sesión; `422` `CURRENT_PASSWORD_INVALID`.

### 4.3 Catálogo (C-04)

#### `POST /api/products` (HU-005)

Request:
```json
{
 "name": "Polo oversize básico",
 "category": "CLOTHING",
 "variants": [
 { "size": " xl ", "color": " NEGRO ", "salePrice": "35.90" },
 { "size": "M", "color": "Blanco", "salePrice": "35.90" }
 ]
}
```

Perfume: `{ "volumeMl": 100, "expiryDate": "2028-06-30", "salePrice": "120.00" }`. Accesorio sin variantes: `"variants": []` y `"salePrice"` a nivel de producto (`singleVariantPrice`), que crea la variante "Única". No existe campo de stock (RN-024, RN-061).

Response `201 Created` + `Location: /api/products/301`:
```json
{
 "id": 301,
 "name": "Polo oversize básico",
 "category": "CLOTHING",
 "active": true,
 "version": 1,
 "variants": [
 { "id": 901, "sku": "ROP-00001", "size": "XL", "color": "Negro", "volumeMl": null, "expiryDate": null, "single": false, "salePrice": "35.90", "stock": 0, "avgCost": null, "version": 1 },
 { "id": 902, "sku": "ROP-00002", "size": "M", "color": "Blanco", "volumeMl": null, "expiryDate": null, "single": false, "salePrice": "35.90", "stock": 0, "avgCost": null, "version": 1 }
 ]
}
```

Errores: `400` (`CATEGORY_REQUIRED`, `NAME_TOO_LONG`, `SIZE_REQUIRED`, `SIZE_TOO_LONG`, `COLOR_TOO_LONG`, `VOLUME_INVALID`, `EXPIRY_FORMAT_INVALID`, `PRICE_OUT_OF_RANGE`, `PRICE_SCALE_INVALID`, `ATTRIBUTE_NOT_ALLOWED_FOR_CATEGORY`, `VARIANTS_REQUIRED`); `409` `DUPLICATE_VARIANT`.

#### `GET /api/products?active=true&category=CLOTHING&page=0&size=20` (HU-005, HU-006)

Lista de productos del emprendedor para la pantalla de catálogo, incluidos los desactivados con `active=false` (para reactivarlos, RN-100). Response `200` con `items` de `{ "id", "name", "category", "active", "variantCount", "totalStock" }`. Lista vacía: `200` con `"items": []`.

#### `GET /api/products/{id}` (HU-005, HU-006)

Response `200` con el mismo cuerpo de la creación. `404` si no existe o es de otro emprendedor.

 Cada variante incluye además `"hasMovements": true|false` (P-006 decide si ofrece "Eliminar variante", RN-099).

#### `PATCH /api/products/{id}` (HU-006)

Cabecera `If-Match: "1"`. Request (todos opcionales; `category` no se acepta):
```json
{ "name": "Polo oversize básico 2026", "active": false }
```
Response `200` con el producto. Errores: `400` `CATEGORY_IMMUTABLE`; `404`; `409` `VERSION_CONFLICT`. Se puede desactivar aunque haya stock; el stock se conserva (RN-137).

#### `POST /api/products/{id}/variants` (HU-005)

Request: un objeto de variante como en la creación. Response `201` con la variante (SKU nuevo, nunca reutilizado). Errores: los de la creación.

#### `PATCH /api/variants/{id}` (HU-006)

Cabecera `If-Match`. Request:
```json
{ "salePrice": "15.00", "expiryDate": "2027-06-30" }
```
Response `200`:
```json
{
 "id": 905, "sku": "PER-00001", "salePrice": "15.00", "expiryDate": "2027-06-30", "stock": 10, "avgCost": "18.50", "version": 3,
 "warnings": [ { "code": "PRICE_BELOW_COST", "message": "El precio de venta es menor que el costo" } ]
}
```
Errores: `422` `VARIANT_ATTRIBUTES_LOCKED` al cambiar talla, color o ml con movimientos (RN-098); `409` `DUPLICATE_VARIANT`; `404`. Cambiar `expiryDate` crea una fila en `variant_audit`.

#### `DELETE /api/variants/{id}` (HU-006)

Response `204` si no tiene movimientos (RN-099). `422` `VARIANT_HAS_MOVEMENTS` con sugerencia de desactivar el producto. `422` `LAST_VARIANT` al eliminar la única variante del producto, con sugerencia de desactivarlo (RN-136).

#### `GET /api/variants?q=polo&category=CLOTHING&active=true&page=0&size=20` (HU-009, HU-012)

Búsqueda para la venta y la consulta de stock. Busca en nombre del producto, SKU, talla y color, sin distinguir mayúsculas (RN-138). Response `200`:
```json
{
 "items": [
 { "id": 901, "productId": 301, "productName": "Polo oversize básico", "category": "CLOTHING", "sku": "ROP-00001", "label": "XL / Negro", "salePrice": "35.90", "stock": 11, "avgCost": "19.00", "expiryDate": null, "expired": false, "active": true }
 ],
 "page": 0, "size": 20, "totalItems": 1
}
```
Lista vacía: `200` con `"items": []` (HU-012, borde).

### 4.4 Inventario (C-05)

#### `POST /api/stock-entries` (HU-007)

Cabecera `Idempotency-Key`. Request:
```json
{ "variantId": 901, "quantity": 10, "unitCost": "20.00" }
```
Response `201`:
```json
{
 "id": 41, "variantId": 901, "quantity": 10, "unitCost": "20.00",
 "previousAvgCost": "18.00", "newAvgCost": "19.00",
 "previousStock": 10, "newStock": 20,
 "status": "ACTIVE", "createdAt": "2026-11-10T20:00:00-05:00"
}
```
Errores: `400` (`QUANTITY_OUT_OF_RANGE` para 0, 2.5 o 10,000; `COST_OUT_OF_RANGE`; `COST_SCALE_INVALID`); `404` variante inexistente o ajena. `422` `PRODUCT_INACTIVE` con "Reactiva el producto para registrar mercadería" si el producto está desactivado (RN-140). La entrada es de una sola variante y la fecha y hora son las del registro; no hay campo de fecha (RN-142). Los montos aceptan coma o punto decimal en la pantalla (RN-152).

#### `POST /api/stock-entries/{id}/void` (HU-008)

Cabecera `Idempotency-Key`. Sin cuerpo. Response `200`:
```json
{ "id": 41, "status": "VOIDED", "variantId": 901, "newStock": 10, "newAvgCost": "18.00", "voidedAt": "2026-11-11T09:00:00-05:00" }
```
Si la entrada anulada era la única vigente, `newAvgCost` es `null` y la UI muestra "Costo promedio: sin entradas" (RN-154, aprobada el 2026-11-11, P-64).
Errores: `404`; `409` `ALREADY_VOIDED` (stock sin cambios); `422` `VOID_WINDOW_EXPIRED` (más de 86,400 s desde `created_at`, RN-087); `422` `LATER_MOVEMENTS_EXIST` con detalle "No se puede anular: hay movimientos posteriores a esta entrada".

#### `POST /api/stock-adjustments` (HU-012)

Cabecera `Idempotency-Key`. Request:
```json
{
 "variantId": 901,
 "countedQuantity": 6,
 "reasonType": "PHYSICAL_COUNT",
 "reasonText": "Conteo físico: venta del live del 2026-11-09 no registrada"
}
```
Response `201`:
```json
{ "id": 77, "variantId": 901, "previousStock": 8, "countedQuantity": 6, "difference": -2, "reasonType": "PHYSICAL_COUNT", "createdAt": "2026-11-10T21:15:00-05:00" }
```
Errores: `400` (`REASON_REQUIRED`, `COUNTED_QUANTITY_OUT_OF_RANGE` para -3 o 10,000); `404`; `422` `NO_ACTIVE_ENTRY` con detalle "Registra primero una entrada con su costo" (RN-116). `EXPIRY_WASTE` solo se acepta en variantes de Perfume (RN-075).

No existe `PUT`/`PATCH` sobre `stock`: cualquier intento devuelve `405` (HU-012, RN-024).

#### `GET /api/variants/{id}` (HU-006, HU-012) 

Detalle de una variante para P-007 (editar) y P-013 (historial al llegar desde Stock). Response `200`:
```json
{ "id": 901, "productId": 301, "productName": "Polo oversize básico", "category": "CLOTHING", "productActive": true,
 "sku": "ROP-00001", "label": "XL / Negro", "size": "XL", "color": "Negro", "volumeMl": null, "expiryDate": null, "expired": false,
 "single": false, "salePrice": "35.90", "stock": 11, "avgCost": "19.00", "hasMovements": true, "version": 3 }
```
`avgCost` es `null` si la variante no tiene entradas vigentes. `404` si no existe o es de otro emprendedor.

#### `GET /api/variants/{id}/movements?page=0&size=20` (HU-012)

Historial de la variante (entradas, líneas de venta, ajustes, con su estado). Response `200` con `items` de `{ "type": "ENTRY|SALE|ADJUSTMENT", "id", "quantityDelta", "status", "createdAt", "reasonText" }`.

 Campos adicionales por ítem, para que P-013 no muestre acciones que fallarían (O-02):
- `unitCost` (solo `ENTRY`), `saleId` y `saleAt` (solo `SALE`), `reasonType` (solo `ADJUSTMENT`).
- `voidable` (`true` solo en una entrada vigente, dentro de 86,400 s y sin movimientos posteriores) y `voidBlockedReason`: `VOID_WINDOW_EXPIRED`, `LATER_MOVEMENTS_EXIST`, `ALREADY_VOIDED` o `null`.
- Orden: `movement_seq` descendente.

### 4.5 Ventas (C-06)

#### `POST /api/sales` (HU-009)

Cabecera `Idempotency-Key`. Request:
```json
{
 "saleAt": null,
 "confirmLoss": false,
 "lines": [
 { "variantId": 901, "quantity": 1, "unitPrice": "35.00" },
 { "variantId": 905, "quantity": 2, "unitPrice": "30.00" }
 ]
}
```

Reglas del servidor, en una sola transacción:
0. `saleAt` es opcional: si es `null`, se usa el momento del registro (RN-126); si viene, debe estar entre hoy - 7 días calendario de Lima y ahora (RN-076 a RN-078). La SPA ofrece "Ahora" por defecto y, para ventas pasadas, un selector de día y hora aproximada.
1. Al menos 1 línea (RN-080). Líneas con el mismo `variantId` y el mismo precio se fusionan (RN-084); con distinto precio, `422` `DUPLICATE_VARIANT_DIFFERENT_PRICE` (RN-125).
2. `SELECT ... FOR UPDATE` de las variantes ordenadas por `id` (evita deadlocks).
3. Por línea: variante del tenant (si no, 404), producto activo (RN-016), no vencida respecto de hoy en America/Lima, también en ventas pasadas (RN-074), stock suficiente (RN-023).
4. Si una o más líneas dan pérdida y `confirmLoss` es `false`: `422` sin registrar nada, con la suma de las pérdidas (RN-073).
5. Todo o nada (RN-081): si una línea falla, ninguna variante cambia.

Response `201`:
```json
{
 "id": 305,
 "saleAt": "2026-11-10T21:30:00-05:00",
 "status": "ACTIVE",
 "totalAmount": "95.00",
 "totalProfit": "33.00",
 "createdAt": "2026-11-10T21:30:00-05:00",
 "lines": [
 { "id": 1201, "variantId": 901, "label": "Polo oversize básico XL / Negro", "quantity": 1, "unitPrice": "35.00", "unitCost": "19.00", "profit": "16.00", "status": "ACTIVE", "remainingStock": 10 },
 { "id": 1202, "variantId": 905, "label": "Perfume Floral Rosé 100 ml", "quantity": 2, "unitPrice": "30.00", "unitCost": "21.50", "profit": "17.00", "status": "ACTIVE", "remainingStock": 13 }
 ]
}
```

Respuesta de pérdida sin confirmar `422`:
```json
{
 "title": "La venta genera una pérdida", "status": 422, "code": "LOSS_CONFIRMATION_REQUIRED",
 "detail": "Esta venta genera una pérdida de S/ 4.00",
 "lossAmount": "4.00"
}
```
El cliente reenvía el mismo cuerpo con `"confirmLoss": true` y una **nueva** `Idempotency-Key`. En ventas de varias líneas hay una sola confirmación con la suma de las pérdidas y `losses` con el detalle por línea (RN-073).

Otros errores: `400` (`LINES_REQUIRED`, `QUANTITY_OUT_OF_RANGE`, `PRICE_OUT_OF_RANGE`, `PRICE_SCALE_INVALID`, `SALE_DATE_OUT_OF_RANGE` para fechas y horas anteriores a hoy - 7 días calendario o posteriores a ahora); `404`; `422` `INSUFFICIENT_STOCK` (con `available` por línea), `VARIANT_EXPIRED` ("Este perfume está vencido y no se puede vender"), `PRODUCT_INACTIVE`. `422` `LINE_QUANTITY_EXCEEDED` si una línea fusionada supera 9,999 unidades (RN-151).

#### `GET /api/sales?from=2026-11-01&to=2026-11-10&status=ALL&page=0&size=20` (HU-011)

Orden: `sale_at` descendente. `from` y `to` son días de Lima. Response `200`:
```json
{
 "items": [
 { "id": 305, "saleAt": "2026-11-10T21:30:00-05:00", "status": "ACTIVE", "lineCount": 2, "totalAmount": "95.00", "totalProfit": "33.00" }
 ],
 "page": 0, "size": 20, "totalItems": 1
}
```
Las ventas anuladas aparecen con `"status": "VOIDED"` y no suman en ningún total.

 Para una venta `VOIDED`, `totalAmount` y `totalProfit` son `"0.00"` (suma de líneas vigentes), y la respuesta agrega `voidedAmount` y `voidedProfit` con la suma de sus líneas anuladas, calculadas desde `sale_line`. P-010 y P-011 muestran esos importes tachados con el texto "No suma" (RN-054, O-10). Sin cambios en tablas.

#### `GET /api/sales/{id}` (HU-011)

Response `200` con el cuerpo completo de la venta (como en el `201`). `404` si es ajena.

#### `POST /api/sales/{id}/void` (HU-010)

Cabecera `Idempotency-Key`. Sin cuerpo. Response `200`:
```json
{ "id": 305, "status": "VOIDED", "totalProfit": "0.00", "restoredStock": [ { "variantId": 901, "newStock": 11 }, { "variantId": 905, "newStock": 15 } ] }
```
Errores: `404`; `409` `ALREADY_VOIDED` (stock sin cambios; dos anulaciones simultáneas suben el stock una sola vez gracias al bloqueo de la venta); `422` `VOID_WINDOW_EXPIRED` (86,400 s desde `created_at`, no desde `sale_at`); `422` `LATER_ADJUSTMENT_EXISTS` con detalle "No se puede anular: hay un ajuste de stock posterior a esta venta" si alguna de sus variantes tiene un ajuste con `movement_seq` mayor (RN-139).

#### `POST /api/sales/{saleId}/lines/{lineId}/void` (HU-010)

Igual que la anulación de venta, solo para una línea. Response `200` con la venta actualizada (`totalProfit` recalculado; si era la última línea vigente, la venta pasa a `VOIDED`, RN-109). También aplica `LATER_ADJUSTMENT_EXISTS` para la variante de la línea (RN-139).

---

## 5. ADRs

El ADR 0002 (`0002-admin-without-tenant.md`, Administrador sin `tenant_id`, Aprobado) va completo abajo; los ADR de esta arquitectura empiezan en 0003. El número 0001 no se usa en el pack. ADR-0014 se aprobó el 2026-11-12. Al crear el repositorio (T-000) cada uno se guarda en `docs/adr/NNNN-titulo.md` (PRINCIPIO-012). Estado de 0002 a 0013: **Aprobado** (2026-11-10); ADR-0014: **Aprobado** (2026-11-12). Se agregan al final de esta sección ADR-0010 a ADR-0013, también aprobados.

### ADR-0002: El Administrador no tiene `tenant_id`

- **Estado:** Aprobado (2026-11-10, P-63). Archivo en el repositorio: `docs/adr/0002-admin-without-tenant.md`.
- **Nota de origen (2026-11-12):** el archivo original no forma parte del Spec Pack. Este texto se reconstruyó solo con lo que el pack ya establece (PRINCIPIO-003, RN-071, RN-114, P-63, P-70 y la tabla `account`) para que T-000 pueda crear el archivo. Si Sergio conserva el original, T-000 usa el original; si no, Sergio confirma este texto en el PR de T-000.
- **Contexto:** F-16 necesita una cuenta que vea y suspenda las cuentas que no pagan. PRINCIPIO-003 exige que toda tabla de negocio tenga `tenant_id` NOT NULL y que nadie vea datos de inventario ajenos. Una cuenta de administración con `tenant_id` sería un emprendedor más, y una con acceso a todos los tenants rompería el aislamiento.
- **Decisión:** la cuenta con rol `ADMIN` no tiene `tenant_id` ni inventario propio, y solo accede al nombre, el correo y el estado de las cuentas (PRINCIPIO-003, RN-114). La tabla `account` no es una tabla de negocio. `ADMIN` nunca se asigna desde el registro: se inserta manualmente (RN-071). `TenantContext` le niega todo endpoint de inventario con 404.
- **Alternativas:** no registradas en el pack. Sergio las completa si el original las tenía.
- **Consecuencias:** PRINCIPIO-003 se actualizó para excluir al Administrador y a la tabla `account` del requisito de `tenant_id`. Queda pendiente cómo se guarda la sesión del Administrador, porque `user_session.tenant_id` es NOT NULL (P-70, con F-16). En el MVP no existe la pantalla del Administrador.

### ADR-0003: Aislamiento por tenant con filtro en servicio y Row-Level Security

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** PRINCIPIO-003 exige que cada emprendedor solo vea sus datos y que lo ajeno devuelva 404. Una sola persona desarrolla con ayuda de agentes de IA, y un `WHERE tenant_id` olvidado en una consulta nueva es el error más probable y más grave.
- **Decisión:** (1) `TenantContext` toma el tenant de la sesión autenticada; todos los repositorios reciben el tenant y lo usan en cada consulta. (2) Al abrir cada transacción se ejecuta `SET LOCAL app.tenant_id = :tenant`, y PostgreSQL aplica políticas RLS en todas las tablas de negocio. (3) Un test de integración por endpoint verifica el 404 cruzado (T-02).
- **Alternativas:** solo filtro en servicio (más simple, pero un olvido expone datos); filtros de Hibernate (`@Filter`/`@TenantId`; no cubren SQL nativo); una base o esquema por tenant (exagerado para 3 usuarios y complica el borrado y las migraciones).
- **Enmienda:** ADR-0014 (aprobado 2026-11-12) define la lectura de la sesión bajo RLS y los roles de los jobs y del backup (P-74, P-75).
- **Consecuencias:** doble protección y borrado sencillo por `tenant_id`. Costo: se necesitan dos roles de base de datos (aplicación sin `BYPASSRLS`, job de borrado con `BYPASSRLS`) y cuidar que el pool de conexiones no arrastre el `app.tenant_id` (se usa `SET LOCAL`, que muere con la transacción). Las lecturas de `account` en el login quedan fuera de RLS por diseño.

### ADR-0004: Stock guardado en la variante y movimientos como historial, con bloqueo pesimista

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** HU-009 exige rechazar ventas simultáneas sin stock, HU-012 exige que el stock coincida con la suma de movimientos y P-37 (resuelto) preguntaba si el stock se guarda o se calcula. El celular de gama baja necesita leer el stock de 30-40 productos rápido.
- **Decisión:** `variant.stock` y `variant.avg_cost` se guardan y se actualizan en la misma transacción que crea el movimiento. Toda operación que cambia stock bloquea las filas de `variant` con `SELECT ... FOR UPDATE`, ordenadas por `id`. `CHECK (stock >= 0)` como red de seguridad. Un test de propiedad verifica que `stock = suma(entradas vigentes) - suma(líneas vigentes) + suma(diferencias de ajustes)`.
- **Alternativas:** calcular el stock siempre desde los movimientos (exacto pero más lento y complica el bloqueo de ventas concurrentes); bloqueo optimista con reintentos (más complejo en ventas de varias líneas); event sourcing (desproporcionado).
- **Consecuencias:** lecturas O(1), concurrencia correcta con 1 usuario y 2 dispositivos. Riesgo de desincronización si alguien escribe fuera del módulo `inventory`; se mitiga con el test de arquitectura y el test de propiedad. P-37 cerrado: el stock se guarda.

### ADR-0005: Online primero con escrituras idempotentes, sin registro de ventas offline

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** conectividad intermitente en Lima y celulares de gama baja. El riesgo R-01 es que el responsable no registre ventas en un live. Una venta offline no puede validar el stock real (otra venta pudo agotarlo desde la laptop) y la regla es bloquear ventas sin stock.
- **Decisión:** las escrituras requieren conexión. Toda escritura que cambia stock lleva `Idempotency-Key` (UUID generado al abrir el formulario). Si la red falla, la SPA conserva el borrador de la venta en memoria, muestra "Sin conexión. Tu venta no se perdió" y reintenta con la misma clave; el servidor devuelve la respuesta original si ya la procesó. Las claves viven 48 h.
- **Alternativas:** cola offline con sincronización posterior (stock negativo y conflictos, contradice RN-023); sin idempotencia (riesgo de ventas duplicadas al reintentar con mala señal).
- **Consecuencias:** cero ventas duplicadas y stock siempre válido. Si no hay señal, la venta no se registra hasta que vuelva. Se mide en el piloto con KPI-06. Una nueva tabla `idempotency_record` con `tenant_id` y limpieza diaria.

### ADR-0006: SPA Angular liviana para Android de gama baja

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** PRINCIPIO-002 fija Angular. Los usuarios usan celulares Android de gama baja con datos móviles; PRINCIPIO-011 exige 360 px y WCAG 2.2 AA.
- **Decisión:** standalone components, signals, rutas con lazy loading por módulo funcional, detección de cambios sin zone.js si la versión de Angular lo permite de forma estable, CSS propio con variables (sin librería de componentes completa), presupuesto de build de 250 kB gzip para el bundle inicial y 100 kB por ruta (falla el build si se supera). Service worker de Angular (`@angular/service-worker`) solo para cachear el app shell y los estáticos, nunca respuestas de `/api` con datos del negocio.
- **Alternativas:** Angular Material completo (más peso, aunque acelera); PWA con caché de datos (riesgo de mostrar stock desactualizado y de dejar datos en un dispositivo compartido); app nativa (W-05).
- **Consecuencias:** carga inicial rápida en 3G/4G y segunda carga casi instantánea. Más trabajo propio de componentes accesibles (se usa Angular CDK). Angular CDK y el service worker son dependencias nuevas, aprobadas en la sección 6.5 (PRINCIPIO-008).

### ADR-0007: Sesiones con token opaco en cookie HttpOnly, no JWT

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** HU-002 exige cierre a 28,800 s sin actividad contados desde la última acción; HU-003 exige cerrar las demás sesiones al cambiar la contraseña; PRINCIPIO-007 prohíbe tokens en logs.
- **Decisión:** token aleatorio de 256 bits en la cookie `MSP_SESSION` (HttpOnly, Secure, SameSite=Lax); en la base solo su SHA-256 en `user_session`. Cada petición autenticada valida inactividad y actualiza `last_activity_at` en cada petición (RN-132; con 3 usuarios el costo es despreciable). Logout y cambio de contraseña revocan filas. CSRF con el patrón cookie + cabecera de Spring Security y Angular.
- **Alternativas:** JWT sin estado (no se puede revocar sin lista negra y la inactividad deslizante obliga a reemitir tokens); `HttpSession` en memoria (se pierde al reiniciar); Spring Session JDBC (válido, pero agrega una dependencia y un esquema propio).
- **Consecuencias:** revocación inmediata y control exacto de la inactividad. Una consulta por petición (despreciable con 3 usuarios). Actividad = toda petición del usuario a `/api` (RN-132); no hay sondeos automáticos en el MVP.

### ADR-0008: Reverse proxy Nginx delante de Spring Boot

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** PRINCIPIO-004 exige TLS 1.2+, HSTS y redirección de 80 a 443. Hay que servir los estáticos de Angular comprimidos y limitar la tasa del login. PRINCIPIO-002 exige ADR para cualquier servicio distinto del stack.
- **Decisión:** Nginx termina TLS (certificado de D-03), aplica HSTS (`max-age=31536000; includeSubDomains`), comprime con gzip/brotli, sirve `/` desde los estáticos de Angular, hace proxy de `/api` a `127.0.0.1:8080` y limita `/api/auth/*` a 10 peticiones por minuto por IP. Spring Boot no se expone a Internet.
- **Alternativas:** TLS directamente en Tomcat embebido (menos piezas, pero la renovación del certificado y la compresión de estáticos son menos prácticas); proxy de un tercero en la nube (servicio externo, contradice el hosting propio).
- **Consecuencias:** una pieza más que configurar, pero estándar y documentada. La configuración vive en el repositorio sin secretos. Aprobado como componente de infraestructura (sección 6.5).

### ADR-0009: Dinero en `NUMERIC(12,2)` y redondeo `HALF_UP`

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** HU-007 exige que 10.005 se redondee a 10.01; los montos van de 0.01 a 99,999.99; la ganancia puede ser negativa.
- **Decisión:** `NUMERIC(12,2)` en PostgreSQL (`NUMERIC(14,2)` para totales), `BigDecimal` con `RoundingMode.HALF_UP` en un value object `Money` del módulo `shared`, montos como cadena en JSON y formato `S/ 1,250.50` solo en el frontend. Se rechaza en la API cualquier monto con más de 2 decimales (no se redondea la entrada del usuario).
- **Alternativas:** céntimos en `BIGINT` (válido, pero menos legible en reportes futuros); `double` (errores de precisión).
- **Consecuencias:** cálculos exactos y verificables con los ejemplos de las HU. Los negativos se muestran como `-S/ 4.00` (RN-148).

### ADR-0010: La cuenta guarda la fecha de aceptación de la política de privacidad

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** PRINCIPIO-005 limita los datos de la cuenta a nombre y correo, pero también exige que la política de privacidad sea aceptada en el registro (Ley N° 29733). Para demostrar esa aceptación hace falta guardar cuándo ocurrió.
- **Decisión:** `account.privacy_accepted_at` (`timestamptz`, NOT NULL) guarda la fecha y hora de la aceptación. Es un dato de la cuenta, no de inventario ni de clientes finales. PRINCIPIO-005 se actualiza para incluirlo. No se guardan otros datos (IP, dispositivo, versión del texto).
- **Alternativas:** no guardar la fecha (no se puede demostrar el consentimiento); guardar también IP y dispositivo (más datos personales de los necesarios).
- **Consecuencias:** se puede demostrar la aceptación ante la ANPD con un solo campo. Si la política cambia, un ADR nuevo definirá cómo registrar su versión.

### ADR-0011: Los backups no incluyen cuentas cerradas

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** PRINCIPIO-006 exige que los backups con datos de una cuenta cerrada no se conserven más de 30 días, y RN-046 exige que ningún backup conserve esos datos más de 2,592,000 s después de `closed_at`. Con una retención de 30 días contada desde cada backup, un backup tomado el día 29 después del cierre guardaría los datos hasta el día 59.
- **Decisión:** el backup diario usa un rol de solo lectura cuya política excluye los `tenant_id` de cuentas con `status = CLOSED` y vuelca `account` sin esas filas (RN-129). La retención sigue siendo de 30 días y los backups se cifran con AES-256. Un test verifica que un backup tomado después de `closed_at` no contiene filas del tenant cerrado.
- **Alternativas:** ajustar RN-046 a la letra de PRINCIPIO-006 (simple, pero los datos podrían vivir hasta 60 días y es difícil de defender ante la ANPD); acortar la retención (no resuelve los backups tomados cerca del día 30).
- **Consecuencias:** se cumple el plazo máximo de 30 días. Si una cuenta se reabre, sus datos vuelven a los backups siguientes; un backup anterior a la reapertura no los tiene. El script de backup es algo más complejo.

### ADR-0012: La venta guarda fecha y hora, por defecto el momento del registro

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** el emprendedor vende en lives y necesita registrar rápido. A veces olvida una venta y la registra después. Una fecha sin hora no permite ordenar ventas del mismo día, y exigir la fecha en cada venta frena el flujo.
- **Decisión:** `sale.sale_at` (`timestamptz`). Por defecto es el momento del registro, sin que el usuario haga nada. Para una venta olvidada, elige el día (hasta 7 días calendario de Lima atrás) y una hora aproximada. No se aceptan fechas ni horas futuras. El plazo de 24 horas para anular se cuenta desde el registro (`created_at`), no desde `sale_at`.
- **Alternativas:** guardar solo la fecha (`DATE`), que no distingue ventas del mismo día; exigir siempre fecha y hora, que agrega pasos en el live.
- **Consecuencias:** ventas ordenadas por `sale_at`, resúmenes semanales y mensuales (F-09) con límites a las 00:00 de Lima y un flujo común sin pasos extra.

### ADR-0013: El Spec Pack es la especificación de cada funcionalidad

- **Estado:** Aprobado (2026-11-10).
- **Contexto:** PRINCIPIO-012 pide una especificación aprobada por funcionalidad. El Spec Pack ya contiene, por funcionalidad, sus reglas (`03`), historias con criterios (`06`), contratos y datos (`07`), tareas (`08`), pantallas (`09`) y validación del prototipo (`10`). Copiarlos en archivos `docs/specs/NNN-funcionalidad.md` duplicaría el contenido y obligaría a cambiar cada regla en dos sitios.
- **Decisión:** el Spec Pack completo (`00` a `10`) vive en `docs/specs/` y es la especificación aprobada. La tabla "Mapa de especificaciones" de `06-user-stories-refinadas.md` identifica, para cada funcionalidad F-01 a F-06, sus historias y sus tareas. PRINCIPIO-012 se actualiza con este criterio. No se crean archivos de especificación aparte.
- **Alternativas:** un archivo por funcionalidad con el contenido copiado (se desincroniza con facilidad); un archivo por funcionalidad que solo enlaza (un archivo más por cada una, sin aportar contenido).
- **Consecuencias:** una sola fuente de verdad. Quien implemente una funcionalidad lee su fila del mapa y las reglas e historias que enlaza.

### ADR-0014: Lectura de la sesión bajo RLS y roles de base de datos de jobs y backup

- **Estado:** **Aprobado** (2026-11-12, P-74 y P-75). Enmienda ADR-0003 y ADR-0011.
- **Contexto:** ADR-0003 activa RLS en `user_session`, pero ADR-0007 busca la sesión por `token_hash` en cada petición, antes de conocer el tenant: sin `app.tenant_id` la consulta no devuelve filas. Además, ADR-0003 solo define el rol de la app y el de borrado, mientras que la limpieza diaria (T-025a) y el comando de restablecimiento (T-039) operan sobre varios tenants, y ADR-0011 pide volcar `account` sin las cuentas cerradas aunque `account` no tiene RLS.
- **Decisión:**
  1. Sesión (P-74, opción A): función `resolve_session(token_hash)` con `SECURITY DEFINER`, propiedad del dueño del esquema y con `search_path` fijo, que devuelve solo `session_id`, `tenant_id`, `last_activity_at`, `revoked_at` y el `status` y `closed_at` de la cuenta. El rol de la app solo tiene `EXECUTE` sobre ella. Después, `TenantContext` ejecuta `SET LOCAL app.tenant_id` y la actualización de `last_activity_at`, el logout y la revocación pasan por RLS. El login crea la sesión dentro de una transacción con `SET LOCAL app.tenant_id = account.id`.
  2. Limpieza diaria y comando (P-75, punto 1): usan el rol de la app. Leen los IDs de `account` y, por cada uno, abren una transacción con `SET LOCAL app.tenant_id`. No se crea otro rol con `BYPASSRLS`.
  3. Backup (P-75, punto 2): rol `backup` de solo lectura, sin `BYPASSRLS`. En las tablas de negocio, su política excluye los `tenant_id` de cuentas con `status = CLOSED`. Se activa RLS en `account` con `USING (true)` para el rol de la app y `USING (status <> 'CLOSED')` para el rol `backup`. El volcado usa `pg_dump --enable-row-security`.
  4. El rol de borrado de ADR-0003 no cambia. Quedan tres roles: app, borrado y backup.
- **Alternativas:** sacar `user_session` de RLS y filtrar solo en servicio (P-74, opción B; más simple, pero deja una tabla de negocio sin la segunda protección); un rol de mantenimiento con `BYPASSRLS` para T-025a y T-039 (más privilegios de los necesarios con 3 cuentas); excluir las cuentas cerradas del backup con un script que filtra después del volcado (los datos llegarían a escribirse en disco antes de filtrarse).
- **Consecuencias:** se conserva la doble protección en todas las tablas con `tenant_id`, con una sola función privilegiada y acotada que se prueba con un test (devuelve solo esas columnas y nada para un hash inexistente). El test de catálogo de tablas de T-008 acepta `account` con RLS. El backup se puede verificar con el mismo test de ADR-0011.

---

## 6. Restricciones técnicas

### 6.1 Alineadas con la constitución

| Principio | Restricción técnica | Cómo se verifica |
|---|---|---|
| PRINCIPIO-001 | Código, rutas, columnas, commits y ramas en inglés; textos de UI y `title`/`detail` de errores en español es-PE. Textos de UI centralizados en un archivo de mensajes del frontend | Revisión de PR; test E2E que busca textos en inglés en pantallas clave (T-01) |
| PRINCIPIO-002 | Java 21, Spring Boot 4.x, PostgreSQL 18, Angular, hosting propio. Nginx, Flyway y otras piezas nuevas solo con ADR aprobado | ADRs 0003 a 0009; `pom.xml` y `package.json` revisados por Sergio |
| PRINCIPIO-003 | `tenant_id NOT NULL` en toda tabla de negocio, tenant desde la sesión, RLS, 404 en recursos ajenos | Test de integración de 404 por endpoint; test que recorre el catálogo de tablas y falla si una tabla de negocio no tiene `tenant_id` o RLS |
| PRINCIPIO-004 | Argon2id con `Argon2PasswordEncoder` (parámetros por defecto de Spring Security o superiores), mínimo 12 caracteres, bloqueo de 900 s tras 5 fallos, TLS 1.2+, HSTS, discos y backups AES-256 | Test que verifica el prefijo `$argon2id$`; prueba de infraestructura con `curl` sobre el puerto 80 y la cabecera HSTS |
| PRINCIPIO-005 | Sin tablas ni columnas de compradores o pagos. Solo nombre, correo, hash y `privacy_accepted_at` de la cuenta (reconocido por ADR-0010). Política de privacidad visible en el registro. Procedimiento de notificación a la ANPD en 48 h en `docs/runbooks/` | Revisión de migraciones; el campo libre `reason_text` muestra la advertencia "No escribas datos de tus clientes" |
| PRINCIPIO-006 | Borrado definitivo a 30 días de `closed_at` mediante job diario `@Scheduled` (aprobado P-61, tolerancia 24 h); login rechazado desde el día 30 (RN-128); backups sin cuentas cerradas y con retención máxima de 30 días (RN-129) | Test automatizado del borrado (T-07) y test de backups |
| PRINCIPIO-007 | Secretos por variables de entorno (`${DB_PASSWORD}`, `${DB_PURGE_PASSWORD}`), `.env` en `.gitignore`, gitleaks en cada PR, logs sin correo, nombre, contraseña, token ni cookie | gitleaks en CI; test de logs (T-05) con un appender de prueba |
| PRINCIPIO-008 | Los agentes no despliegan, no ejecutan SQL en bases reales, no tocan dependencias sin aprobación, no hacen merge, no manejan secretos. Los cambios de esquema solo como migraciones `V<n>__<desc>.sql` en un PR | Protección de rama; revisión de Sergio |
| PRINCIPIO-009 | Todo lo marcado [BLOQUEADO] o "pendiente" no se implementa | Lista de la sección 6.4 |
| PRINCIPIO-010 | 80% de cobertura de líneas en `domain` y `application` (JaCoCo y cobertura de Angular), tests unitarios, test E2E del flujo principal por HU, CI en verde | Umbral de JaCoCo y de Angular que falla el build (T-06) |
| PRINCIPIO-011 | WCAG 2.2 AA, sin scroll horizontal desde 360 px, axe-core en los E2E sin violaciones serias o críticas. Objetivos táctiles de al menos 24x24 px (WCAG 2.5.8); en la venta rápida, 44x44 px | E2E con viewport 360x640 y axe-core en cada pantalla (T-03) |
| PRINCIPIO-012 | Documentación en Markdown en el repo: el Spec Pack completo en `docs/specs/` (ADR 0013) y los ADR 0002 a 0014 en `docs/adr/`, aprobados antes de implementar. Los ADR de hosting, dominio y CI se agregan al elegirlos (P-62) | Revisión de PR; tarea T-000 del backlog |

### 6.2 Contexto peruano: conectividad intermitente

- Escrituras idempotentes con `Idempotency-Key` y reintento automático con espera creciente (1 s, 2 s, 4 s; máximo 3 intentos) y luego botón "Reintentar" (ADR-0005).
- El borrador de la venta en curso se conserva en memoria del componente al perder conexión; no se guarda en el dispositivo después de cerrar sesión.
- Timeouts del cliente: 10 s para lecturas, 15 s para escrituras. El servidor responde las escrituras de venta en menos de 300 ms (p95) con 3 usuarios.
- Mensajes claros en español: "Sin conexión. Tu venta no se perdió: reintentaremos en cuanto vuelva la señal."
- Nada de datos de negocio en caché del service worker (evita mostrar stock desactualizado).

### 6.3 Contexto peruano: Android de gama baja

- Bundle inicial de 250 kB gzip como máximo (presupuesto de Angular que falla el build).
- Listas paginadas de 20 elementos; sin tablas anchas (tarjetas apiladas en 360 px).
- Respuestas JSON sin campos innecesarios; compresión en el proxy.
- Sin animaciones pesadas, sin fuentes web externas (fuente del sistema), íconos SVG en línea con `aria-label`.
- Flujo de venta para lives en máximo 4 toques: buscar variante, cantidad, precio (prellenado con el precio de venta), registrar (R-01).
- Objetivo medible en el piloto: Largest Contentful Paint menor a 3 s en un perfil "Moto G" con 4G lento de Lighthouse.

### 6.4 Decisiones pendientes y resueltas

Todas las decisiones de las auditorías QA v2 y v3 están resueltas y registradas en `03-reglas-negocio.md` (RN-126 a RN-153) y `05-preguntas-abiertas.md`. Los pendientes que afectan a la arquitectura son estos (la lista completa está en `05-preguntas-abiertas.md`; ninguna pendiente bloquea el desarrollo):

| ID | Tema | Afecta | Efecto en la arquitectura |
|---|---|---|---|
| ~~P-61~~ | ~~Mecanismo del borrado definitivo~~ | | **Resuelto 2026-11-11**: job diario `@Scheduled` en C-08, tolerancia 24 h. T-025b ya implementable. |
| P-62 (D-01, D-03, D-04) | Proveedores de hosting, dominio/TLS y CI | Despliegue del piloto | No bloquea el desarrollo; se elige con un ADR antes de desplegar |
| ~~P-64~~ | ~~Costo promedio al anular la primera entrada~~ | ~~HU-008, C-05, T-020~~ | **Resuelto 2026-11-11**: opción A aprobada. Ver `05-preguntas-abiertas.md` |
| ~~P-66~~ | ~~Contratos de lectura que necesita la UI~~ | | **Resuelto 2026-11-11**: endpoints y campos aprobados. Ver `05-preguntas-abiertas.md` |
| ~~P-67~~ | ~~Correo de contacto ARCO~~ | | **Resuelto 2026-11-11**: `ARCO_CONTACT_EMAIL` por variable de entorno |
| ~~P-74~~ | ~~Lectura de la sesión con RLS activa~~ | | **Resuelto 2026-11-12**: función `resolve_session` (ADR-0014) |
| ~~P-75~~ | ~~Roles de base de datos para jobs, comando y backup~~ | | **Resuelto 2026-11-12**: jobs y comando con el rol de la app por tenant; rol `backup` (ADR-0014) |
| P-70 | Sesión del Administrador frente a `user_session.tenant_id` NOT NULL | C-03, F-16 | Sin efecto en el MVP; requiere ADR con F-16 |

### 6.5 Dependencias aprobadas por Sergio (PRINCIPIO-008)

Aprobadas el 2026-11-10 con estas condiciones: se usan versiones estables compatibles con Java 21 y Spring Boot 4.x, se fijan en el PR de T-001 (backend) y T-002 (frontend) y no se actualizan sin una nueva aprobación. Cualquier dependencia que no esté en esta tabla requiere aprobación previa.

| Dependencia | Para qué | Decisión |
|---|---|---|
| `spring-boot-starter-security`, `-web`, `-data-jpa`, `-validation` | Base del backend | Aprobada (stack de PRINCIPIO-002) |
| `org.bouncycastle:bcprov-jdk18on` | `Argon2PasswordEncoder` la necesita en tiempo de ejecución | Aprobada |
| `org.flywaydb:flyway-core` + `flyway-database-postgresql` | Migraciones versionadas | Aprobada; se elige Flyway y no Liquibase |
| `org.postgresql:postgresql` | Driver JDBC | Aprobada |
| Testcontainers (PostgreSQL) | Tests de integración contra PostgreSQL 18 real (RLS, constraints) | Aprobada (solo test) |
| JaCoCo (plugin Maven) | Cobertura del 80% | Aprobada (solo build) |
| ArchUnit | Test de dependencias entre módulos | Aprobada (solo test) |
| `@angular/service-worker` | Caché del app shell | Aprobada (ADR-0006) |
| `@angular/cdk` | Componentes accesibles (diálogos, foco) | Aprobada |
| Playwright + `@axe-core/playwright` | E2E y accesibilidad | Aprobada (solo test) |
| Nginx | Reverse proxy | Aprobada (ADR-0008) |

### 6.6 Entornos y despliegue

- Entornos: `local` (Docker Compose con PostgreSQL 18 para desarrollo), `production` en el hosting propio. Un entorno de `staging` se recomienda pero no es obligatorio para el piloto.
- Despliegue manual por Sergio (PRINCIPIO-008): artefacto JAR del backend + estáticos de Angular, servicio `systemd`, migraciones Flyway al arrancar.
- Proveedores de hosting, dominio/TLS, repositorio/CI y correo: pendientes de elegir con ADR (`02-alcance.md`, sección 6; P-62). No bloquean el desarrollo: el CI se escribe como scripts independientes de la plataforma y Nginx se prueba en local.

---

## 7. Trazabilidad

| HU | Componentes | Endpoints | Tablas |
|---|---|---|---|
| HU-001 | C-01, C-02, C-03, C-07 | `POST /api/auth/register` | `account` |
| HU-002 | C-01, C-02, C-03, C-07 | `POST /api/auth/login`, `GET /api/auth/session`, `POST /api/auth/logout` | `account`, `user_session` |
| HU-003 | C-01, C-03 | `PUT /api/account/password`, `PATCH /api/account` | `account`, `user_session` |
| HU-004 | C-01, C-03, C-08, C-10 | `POST /api/account/close`, `POST /api/account/reopen`, `GET /api/auth/session` | `account` y todas por `tenant_id` |
| HU-005 | C-01, C-04 | `GET /api/products`, `POST /api/products`, `POST /api/products/{id}/variants`, `GET /api/products/{id}` | `product`, `variant`, `sku_counter` |
| HU-006 | C-01, C-04 | `PATCH /api/products/{id}`, `GET /api/variants/{id}`, `PATCH /api/variants/{id}`, `DELETE /api/variants/{id}` | `product`, `variant`, `variant_audit` |
| HU-007 | C-01, C-05 | `POST /api/stock-entries` | `stock_entry`, `variant`, `idempotency_record` |
| HU-008 | C-01, C-05 | `GET /api/variants/{id}/movements`, `POST /api/stock-entries/{id}/void` | `stock_entry`, `variant` |
| HU-009 | C-01, C-06, C-05 | `GET /api/variants`, `POST /api/sales` | `sale`, `sale_line`, `variant`, `idempotency_record` |
| HU-010 | C-01, C-06, C-05 | `POST /api/sales/{id}/void`, `POST /api/sales/{saleId}/lines/{lineId}/void` | `sale`, `sale_line`, `variant` |
| HU-011 | C-01, C-06 | `GET /api/sales`, `GET /api/sales/{id}` | `sale`, `sale_line` |
| HU-012 | C-01, C-05 | `GET /api/variants`, `GET /api/variants/{id}`, `GET /api/variants/{id}/movements`, `POST /api/stock-adjustments` | `variant`, `stock_adjustment` |
