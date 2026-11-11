# 09 - Backlog: miStorePe

Basado en `08-architecture.md` (v1.0), `07-user-stories-refinadas.md`, `03-reglas-negocio.md`, `02-alcance.md`. Fecha: 2026-11-10 (versión aprobada tras las auditorías QA v2 y v3).

Reglas de este backlog:
- Alcance: solo MVP (F-01 a F-06, HU-001 a HU-012). Todas las tareas son **Must**. No hay tareas de Should, Could ni Won't: la arquitectura v0.1 solo cubre el MVP.
- La arquitectura v1.0, los ADR 0002 a 0013 y las dependencias de la sección 6.5 de `08-architecture.md` ya están aprobados por Sergio (2026-11-10); T-000 solo los copia al repositorio.
- Principio de producto: la primera versión es simple y no bloquea al usuario más de lo necesario; ante la duda, la pantalla ofrece un valor por defecto en lugar de pedir datos.
- Toda dependencia nueva (sección 6.5 de la arquitectura) requiere aprobación previa de Sergio (PRINCIPIO-008). Los agentes no hacen merge, no despliegan ni tocan bases reales.
- Lo marcado **[BLOQUEADO]** en la arquitectura queda fuera de las tareas (ver sección final).
- Las tareas están numeradas en orden topológico: cada tarea solo depende de tareas con número menor (DAG sin ciclos).
- Los IDs `T-NNN` de este backlog no son los criterios transversales `T-01` a `T-07` de las HU; cuando se citan esos criterios se escriben con dos dígitos.

---

## 1. Setup e infraestructura base

### T-000: Crear el repositorio con la documentación
- **HU relacionada:** Todas (PRINCIPIO-009, PRINCIPIO-012)
- **Componente:** Documentación
- **Prioridad:** Must
- **Tamaño:** S
- **Dependencias:** Ninguna
- **Descripción:** Tarea mecánica, sin decisiones: copiar el Spec Pack (`00` a `09`) a `docs/specs/` y separar los ADR en `docs/adr/NNNN-titulo.md`: el 0002 (archivo `0002-admin-without-tenant.md`) y del 0003 al 0013 (sección 5 de `08-architecture.md`, un archivo por ADR, estado Aprobado). No se crean especificaciones aparte (ADR-0013). Sergio aprueba el PR inicial en el repositorio.
- **Criterio de completitud:** ¿Existen `docs/specs/` con el pack y `docs/adr/` con los 12 ADR, y Sergio aprobó el PR?

### T-001: Crear el proyecto backend con módulos y entorno local
- **HU relacionada:** Infraestructura
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-000
- **Descripción:** Crear el proyecto Maven con Java 21 y Spring Boot 4.x con los paquetes `pe.mistore.shared`, `identity`, `catalog`, `inventory`, `sales` y `lifecycle`. Agregar `docker-compose.yml` con PostgreSQL 18 para `local`, configuración por variables de entorno (`${DB_PASSWORD}`) y `.env` en `.gitignore`. Solo dependencias aprobadas por Sergio.
- **Criterio de completitud:** ¿`mvn verify` pasa y la app arranca contra el PostgreSQL 18 de Docker Compose sin secretos en el repositorio?

### T-002: Crear la SPA Angular liviana
- **HU relacionada:** Infraestructura
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-000
- **Descripción:** Crear la app Angular con standalone components, signals y rutas lazy por módulo funcional, locale es-PE, archivo central de textos en español y pipe de montos con formato `S/ 1,250.50`. Configurar presupuestos de build de 250 kB gzip inicial y 100 kB por ruta (ADR-0006). Incluir los formatos visibles: montos negativos `-S/ 4.00` (RN-148), fechas `DD/MM/AAAA` y horas `HH:mm` en hora de Lima (RN-149), singular y plural de unidades (RN-150) y entrada de montos con coma o punto (RN-152).
- **Criterio de completitud:** ¿El build falla si se superan los presupuestos y el pipe muestra `1250.5` como `S/ 1,250.50` en un test unitario? ¿Los tests unitarios muestran `-4` como `-S/ 4.00`, `2026-11-10T21:30-05:00` como `10/11/2026 21:30`, 1 como "1 unidad" y aceptan "29,90"?

### T-003: Configurar el pipeline de CI
- **HU relacionada:** Infraestructura (criterio T-06)
- **Componente:** C-07 `shared` / plataforma (CI)
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-001, T-002
- **Descripción:** Pipeline escrito como scripts independientes de la plataforma (Maven, npm, gitleaks), para no depender de la elección de D-04; el archivo propio de la plataforma se agrega al elegirla. Compila backend y frontend, corre tests unitarios y de integración, exige 80% de cobertura de líneas en `domain` y `application` (JaCoCo) y en el frontend, y corre gitleaks en cada PR (PRINCIPIO-007, PRINCIPIO-010).
- **Criterio de completitud:** ¿Un PR con cobertura de 79% o con un secreto de prueba falla el CI?

### T-004: Implementar Money, Clock, errores y política de logs
- **HU relacionada:** Infraestructura (criterios T-01, T-04, T-05)
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-001
- **Descripción:** Crear el value object `Money` (`BigDecimal`, `HALF_UP`, rechaza más de 2 decimales), un `Clock` inyectable en America/Lima y el manejador global de errores con `ProblemDetail` (RFC 9457), `code` en inglés y `title`/`detail` en español. Configurar Logback para registrar IDs internos y nunca correo, nombre, contraseña, token ni cookie.
- **Criterio de completitud:** ¿Los tests verifican que `Money` redondea 10.005 a 10.01, rechaza 35.999 y que un error de validación devuelve `application/problem+json` en español?

### T-005: Migración V1 de cuentas, sesiones e idempotencia
- **HU relacionada:** HU-001, HU-002
- **Componente:** C-09 Base de datos
- **Prioridad:** Must
- **Tamaño:** S
- **Dependencias:** T-001
- **Descripción:** Crear con Flyway las tablas `account` (con `closure_reason`), `user_session` (sin `user_agent_label`) e `idempotency_record` según la sección 3.2, con `email` UNIQUE, `token_hash` UNIQUE y PK compuesta (`tenant_id`, `idempotency_key`).
- **Criterio de completitud:** ¿La migración se aplica en un PostgreSQL 18 vacío (Testcontainers) y los constraints existen?

### T-006: Migración V2 del catálogo
- **HU relacionada:** HU-005, HU-006
- **Componente:** C-09 Base de datos
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-005
- **Descripción:** Crear `product`, `variant`, `sku_counter` y `variant_audit` con `tenant_id NOT NULL`, `CHECK (stock >= 0)`, UNIQUE (`tenant_id`, `sku`) y los índices parciales de unicidad por categoría (Ropa, Perfume con `NULLS NOT DISTINCT` propuesto por M-12, Única).
- **Criterio de completitud:** ¿Un test de integración confirma que insertar dos variantes Ropa M/Negro del mismo producto viola el índice y que `stock = -1` viola el CHECK?

### T-007: Migración V3 de movimientos y ventas
- **HU relacionada:** HU-007 a HU-012
- **Componente:** C-09 Base de datos
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-006
- **Descripción:** Crear `stock_entry`, `sale` (con `sale_at timestamptz`), `sale_line` y `stock_adjustment` con sus CHECK de rangos (1..9999, 0.01..99999.99, 0..9999), UNIQUE (`sale_id`, `variant_id`), FK `ON DELETE RESTRICT` y la secuencia global `movement_seq`. Sin columnas de comprador ni de pago.
- **Criterio de completitud:** ¿La migración aplica y un test confirma que `quantity = 10000` en `stock_entry` viola el CHECK?

### T-008: Row-Level Security y roles de base de datos
- **HU relacionada:** Infraestructura (criterio T-02)
- **Componente:** C-09 Base de datos
- **Prioridad:** Must
- **Tamaño:** XL
- **Dependencias:** T-007
- **Descripción:** Migración V4 que activa RLS con la política `tenant_id = current_setting('app.tenant_id')::bigint` en las 10 tablas de la sección 3.4, crea el rol de la app (no dueño, sin `BYPASSRLS`) y el rol de borrado con `BYPASSRLS` limitado a `DELETE` (ADR-0003). Agregar un test que recorre el catálogo de tablas y falla si una tabla de negocio no tiene `tenant_id` o RLS.
- **Criterio de completitud:** ¿Con `app.tenant_id = 1` el rol de la app no ve filas del tenant 2 y el test de catálogo de tablas pasa?

### T-009: TenantContext y SET LOCAL por transacción
- **HU relacionada:** Infraestructura (criterio T-02)
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-004, T-008
- **Descripción:** Implementar `TenantContext` que toma el tenant solo de la sesión autenticada y ejecuta `SET LOCAL app.tenant_id` al abrir cada transacción. Los repositorios reciben el tenant y lo usan en cada consulta (DP-02, DP-03).
- **Criterio de completitud:** ¿Un test con dos transacciones seguidas en la misma conexión del pool confirma que la segunda no hereda el `app.tenant_id` de la primera?

### T-010: Filtro de idempotencia
- **HU relacionada:** HU-007, HU-008, HU-009, HU-010, HU-012
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-009
- **Descripción:** Implementar el `IdempotencyFilter` (ADR-0005): exige `Idempotency-Key` UUID en los endpoints indicados en 4.1, guarda el SHA-256 del cuerpo y la respuesta por 48 h, devuelve la respuesta original en el reintento y `422` si la misma clave llega con otro cuerpo.
- **Criterio de completitud:** ¿Dos POST con la misma clave y el mismo cuerpo crean un solo registro y devuelven la misma respuesta, y con otro cuerpo devuelven 422?

---

## 2. Features backend

### T-011: Registro de cuenta
- **HU relacionada:** HU-001
- **Componente:** C-03 `identity`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-004, T-005
- **Descripción:** Implementar `POST /api/auth/register`: normaliza el correo (`trim` + minúsculas), valida el formato del correo (RN-133), exige contraseña de 12 a 128 caracteres (RN-002, RN-134) y `privacyAccepted = true`, guarda hash Argon2id y `privacy_accepted_at`, y responde `409 EMAIL_ALREADY_REGISTERED` apoyado en el UNIQUE. El nombre es opcional, máximo 100 caracteres (RN-141). El inicio de sesión automático tras el registro (RN-153) se conecta en T-012, que ya tiene el servicio de sesiones. Nunca asigna el rol `ADMIN`.
- **Criterio de completitud:** ¿Pasan los tests de HU-001: 12 caracteres crea y 11 no, ` Ana@Gmail.com ` choca con `ana@gmail.com`, el hash empieza con `$argon2id$` y dos registros simultáneos crean una sola cuenta? ¿`ana@gmail` da `EMAIL_INVALID` y una contraseña de 129 caracteres da `PASSWORD_TOO_LONG`?

### T-012: Login, logout y sesiones seguras
- **HU relacionada:** HU-002
- **Componente:** C-03 `identity`
- **Prioridad:** Must
- **Tamaño:** XL
- **Dependencias:** T-009, T-011
- **Descripción:** Implementar `POST /api/auth/login` y `/logout` con token opaco de 256 bits en cookie `MSP_SESSION` (solo su SHA-256 en base), CSRF cookie + cabecera, contador de fallos con bloqueo de 900 s al quinto, inactividad de 28,800 s con `last_activity_at` actualizado en cada petición (RN-132, ADR-0007), `423` con "Demasiados intentos. Intenta de nuevo en 15 minutos" sin sumar ni extender el bloqueo (RN-130, RN-131), `401` para cuentas con 30 días o más desde `closed_at` (RN-128) y `GET /api/auth/session`. El log usa el ID de la cuenta, nunca el correo. Conectar el registro de T-011 para que, al crear la cuenta, emita la sesión y la cookie sin pedir de nuevo el correo y la contraseña (RN-153).
- **Criterio de completitud:** ¿Pasan los tests de HU-002 con `Clock` fijo: rechazo a 899 s y aceptación a 901 s del quinto fallo, un intento durante el bloqueo no lo extiende, sesión válida a 28,799 s de la última petición y cerrada a 28,801 s, correo inexistente sin sumar intentos, login de una cuenta cerrada hace 30 días da 401 y `GET /api/auth/session` devuelve el estado de la cuenta? ¿Tras `POST /api/auth/register` la respuesta trae la cookie y `GET /api/auth/session` devuelve la cuenta activa?

### T-013: Cambio de contraseña
- **HU relacionada:** HU-003
- **Componente:** C-03 `identity`
- **Prioridad:** Must
- **Tamaño:** S
- **Dependencias:** T-012
- **Descripción:** Implementar `PUT /api/account/password`: valida la contraseña actual, guarda la nueva con Argon2id y revoca las demás sesiones del tenant manteniendo la actual (RN-110). Aplica el mínimo de 12 y el máximo de 128 caracteres (RN-105 aprobada, RN-134). Agregar `PATCH /api/account` para escribir o corregir el nombre (opcional, máximo 100 caracteres, RN-141).
- **Criterio de completitud:** ¿Tras el cambio desde la sesión A, la sesión B recibe 401, la A sigue activa y la contraseña anterior ya no permite entrar? ¿Una contraseña nueva de 11 caracteres se rechaza y una de 12 se acepta? ¿Un nombre de 100 caracteres se guarda y uno de 101 da `NAME_TOO_LONG`?

### T-014: Reapertura de cuenta y bloqueo de cuentas cerradas
- **HU relacionada:** HU-004
- **Componente:** C-03 `identity`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-012
- **Descripción:** Implementar `POST /api/account/reopen` (válido si `now() - closed_at < 2,592,000 s`, si no `422 REOPEN_WINDOW_EXPIRED`) y que el login de una cuenta `CLOSED` solo permita reabrir y cerrar sesión. Cualquier otra operación de una cuenta cerrada responde 403 (RN-007). No incluye `POST /api/account/close` (bloqueado por A-02).
- **Criterio de completitud:** ¿Reabrir a 2,591,999 s funciona, fuera de plazo da 422 y una venta desde una cuenta cerrada da 403 sin cambiar el stock?

### T-015: Reglas de variantes por categoría y generador de SKU
- **HU relacionada:** HU-005
- **Componente:** C-04 `catalog`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-004, T-006
- **Descripción:** Implementar `VariantRules` en el dominio (atributos obligatorios y prohibidos por categoría, talla en mayúsculas hasta 10 caracteres, color "Negro" hasta 30, ml entero > 0, vencimiento `AAAA-MM-DD`, precio 0.01..99,999.99) y el generador de SKU con `UPDATE sku_counter ... RETURNING` por tenant y categoría, sin reutilización.
- **Criterio de completitud:** ¿Hay un test unitario por regla de HU-005 y un test de integración donde dos creaciones simultáneas reciben `PER-00001` y `PER-00002`? ¿Una presentación de 9,999 ml se acepta y una de 10,000 ml se rechaza (RN-146)?

### T-016: Crear productos y variantes
- **HU relacionada:** HU-005
- **Componente:** C-04 `catalog`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-012, T-015
- **Descripción:** Implementar `GET /api/products` (lista paginada con filtro `active`), `POST /api/products`, `POST /api/products/{id}/variants` y `GET /api/products/{id}` con la variante "Única" automática en accesorios sin variantes, stock inicial 0 y sin campo de stock en el request. Errores según 4.3.
- **Criterio de completitud:** ¿Pasan los tests de HU-005: ` xl `/` NEGRO ` se guardan `XL`/`Negro`, primer SKU `ROP-00001`, accesorio sin variantes crea "Única", duplicado da 409 y producto ajeno da 404? ¿Dos perfumes de 100 ml sin vencimiento en el mismo producto dan 409 (RN-135) y `GET /api/products?active=false` lista los desactivados?

### T-017: Editar, desactivar y eliminar en el catálogo
- **HU relacionada:** HU-006
- **Componente:** C-04 `catalog`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-016
- **Descripción:** Implementar `PATCH /api/products/{id}`, `PATCH /api/variants/{id}` y `DELETE /api/variants/{id}` con `If-Match`, categoría inmutable, atributos bloqueados con movimientos (RN-098), eliminación solo sin movimientos (RN-099), advertencia `PRICE_BELOW_COST` y fila en `variant_audit` al cambiar el vencimiento. Eliminar la única variante responde `422 LAST_VARIANT` (RN-136); desactivar con stock está permitido y conserva el stock (RN-137).
- **Criterio de completitud:** ¿Pasan los tests de HU-006: precio 18.49 con costo 18.50 advierte y 18.50 no, cambio de talla con ventas da 422, versión vieja da 409 y `PATCH` de un producto ajeno da 404? ¿Eliminar la única variante da `LAST_VARIANT`, desactivar con stock 3 y reactivar deja stock 3, y el SKU no cambia al editar el precio (RN-066)?

### T-018: Búsqueda y consulta de variantes
- **HU relacionada:** HU-009, HU-012
- **Componente:** C-04 `catalog`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-016
- **Descripción:** Implementar `GET /api/variants?q=&category=&active=&page=&size=` paginado (máximo 50) con búsqueda por nombre, SKU, talla y color sin distinguir mayúsculas (RN-138), el campo `expired` calculado con la fecha de hoy en America/Lima y lista vacía con `200`.
- **Criterio de completitud:** ¿Un tenant sin productos recibe `"items": []` con 200, los productos desactivados se excluyen con `active=true` y nunca aparecen variantes de otro tenant?

### T-019: Entradas de mercadería con costo promedio ponderado
- **HU relacionada:** HU-007
- **Componente:** C-05 `inventory`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-007, T-010, T-016
- **Descripción:** Implementar el servicio de inventario como único dueño de `variant.stock` y `avg_cost`, y `POST /api/stock-entries` con `SELECT ... FOR UPDATE`, cálculo de la sección 3.3, guardado de `previous_avg_cost`, `previous_stock` y `movement_seq`, y marca `has_movements`. Exponer una interfaz Java para que `sales` descuente y devuelva stock. Rechaza con `422 PRODUCT_INACTIVE` y "Reactiva el producto para registrar mercadería" si el producto está desactivado (RN-140); la entrada es de una sola variante con la fecha y hora del registro (RN-142).
- **Criterio de completitud:** ¿Pasan los ejemplos de HU-007: 10 a 18.00 + 10 a 20.00 = 19.00; 1 a 10.00 + 1 a 10.01 = 10.01; 10 a 18.00 + 5 a 20.01 = 18.67; costo 0.00 rechazado sin cambiar nada; variante ajena da 404? ¿Una entrada en un producto desactivado da `PRODUCT_INACTIVE` y no cambia el stock?

### T-020: Anulación de entradas
- **HU relacionada:** HU-008
- **Componente:** C-05 `inventory`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-019
- **Descripción:** Implementar `POST /api/stock-entries/{id}/void`: plazo de 86,400 s desde `created_at`, rechazo con `LATER_MOVEMENTS_EXIST` si hay entrada, línea de venta o ajuste vigente con `movement_seq` mayor, restauración de stock y `avg_cost = previous_avg_cost`, y `409 ALREADY_VOIDED` sin tocar el stock.
- **Criterio de completitud:** ¿Pasan los tests de HU-008: anular a 86,400 s funciona y a 86,401 s no, una venta posterior anulada no impide anular y una doble anulación no cambia el stock?

### T-021: Ajustes por conteo físico e historial de movimientos
- **HU relacionada:** HU-012
- **Componente:** C-05 `inventory`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-019
- **Descripción:** Implementar `POST /api/stock-adjustments` (motivo obligatorio, cantidad contada 0..9999, requiere una entrada vigente, `EXPIRY_WASTE` solo en Perfume, sin cambiar `avg_cost`) y `GET /api/variants/{id}/movements` paginado. Cualquier `PUT`/`PATCH` sobre el stock responde 405. Motivo de 1 a 200 caracteres sin solo espacios (RN-143), diferencia 0 permitida (RN-144) y sin endpoint para anular un ajuste (RN-145). Permitido en productos desactivados (RN-147).
- **Criterio de completitud:** ¿Pasan los tests de HU-012: de 8 a 6 queda diferencia -2, sin entrada vigente da `NO_ACTIVE_ENTRY`, el costo promedio no cambia y fijar el stock directamente da 405? ¿Un motivo de solo espacios o de 201 caracteres se rechaza, un ajuste con diferencia 0 se registra y un ajuste en un producto desactivado se acepta?

### T-022: Registro de ventas con varias líneas
- **HU relacionada:** HU-009
- **Componente:** C-06 `sales`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-018, T-019
- **Descripción:** Implementar `POST /api/sales` en una transacción: fusión de líneas con igual precio, bloqueo de variantes ordenadas por `id`, fecha y hora de venta (`sale_at`: por defecto el momento del registro; si se envía, entre hoy - 7 días calendario de Lima y ahora, RN-076 a RN-078, RN-126), validación de producto activo, vencimiento contra hoy, stock, una sola confirmación de pérdida con la suma de las líneas (`LOSS_CONFIRMATION_REQUIRED`, RN-073), todo o nada, y congelado de `unit_cost` y `profit` por línea. Descuenta stock solo a través de la interfaz de `inventory`. Rechaza con `422 LINE_QUANTITY_EXCEEDED` una línea fusionada de más de 9,999 unidades (RN-151).
- **Criterio de completitud:** ¿Pasan los tests de HU-009: venta de 4 líneas con una sin stock no cambia ningún stock; vencido ayer se rechaza y hoy se vende; dos líneas con pérdida piden una sola confirmación de S/ 5.50; sin `saleAt` se guarda el momento del registro; el 2026-11-10 se acepta el 2026-11-03 a las 08:00 y se rechazan el 2026-11-02 a las 23:59 y una hora futura?

### T-023: Anulación de ventas y de líneas
- **HU relacionada:** HU-010
- **Componente:** C-06 `sales`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-021, T-022
- **Descripción:** Implementar `POST /api/sales/{id}/void` y `POST /api/sales/{saleId}/lines/{lineId}/void` con plazo de 86,400 s desde el registro, devolución de stock vía `inventory`, recálculo de `total_profit`, venta `VOIDED` al anular su última línea y bloqueo de la venta para que dos anulaciones simultáneas devuelvan el stock una sola vez. Si alguna variante tiene un ajuste posterior a la venta, responde `422 LATER_ADJUSTMENT_EXISTS` (RN-139). Permite anular ventas de productos desactivados dentro de las 24 horas (RN-147).
- **Criterio de completitud:** ¿Pasan los tests de HU-010: anular una línea de S/ 16.00 deja la ganancia en S/ 77.00, dos anulaciones simultáneas suben el stock una sola vez y anular a 86,401 s da 422? ¿Una venta con un ajuste posterior de su variante no se anula y el stock no cambia?

### T-024: Historial de ventas y ganancia
- **HU relacionada:** HU-011
- **Componente:** C-06 `sales`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-022
- **Descripción:** Implementar `GET /api/sales` paginado con filtros `from`, `to` y `status` (orden por `sale_at` descendente) y `GET /api/sales/{id}` con las líneas y su ganancia congelada. Las ventas anuladas aparecen con `VOIDED` y no suman.
- **Criterio de completitud:** ¿Pasan los tests de HU-011: 3 a 35.00 con costo 18.67 da `"48.99"`, una entrada posterior más cara no cambia la ganancia, historial vacío da 200 y venta ajena da 404?

### T-025: Job de borrado de cuentas y limpieza
- **HU relacionada:** HU-004 (criterio T-07)
- **Componente:** C-08 `lifecycle`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-008, T-014
- **Descripción:** **[BLOQUEADO por P-61 (M-07) solo en el borrado definitivo]** Implementar el borrado definitivo, con el rol de borrado y en el orden de la sección 3.4, de todas las filas de las cuentas con `closed_at` de más de 30 días, con el mecanismo y la frecuencia que Sergio decida. La limpieza diaria de sesiones vencidas y de claves de idempotencia de más de 48 h no está bloqueada y se implementa ya.
- **Criterio de completitud:** ¿Un test automatizado confirma que tras el job no queda ninguna fila con el `tenant_id` de Ana y que Luis conserva sus 20 productos y 15 ventas?

---

## 3. Features frontend (integración con la API)

### T-026: Pantallas de cuenta y acceso
- **HU relacionada:** HU-001, HU-002, HU-003, HU-004
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-002, T-013, T-014
- **Descripción:** Crear las pantallas de registro (con casilla de política de privacidad; el enlace lo agrega T-038), login, cerrar sesión, "Mi cuenta" con cambio de contraseña, y la pantalla de cuenta cerrada que solo ofrece "Reabrir cuenta". Al cargar, la SPA consulta `GET /api/auth/session`. Agregar el interceptor que lleva al login ante un 401 y muestra los errores `ProblemDetail` en español.
- **Criterio de completitud:** ¿Un usuario puede registrarse, entrar, cambiar su contraseña y salir desde un viewport de 360 px sin scroll horizontal?

### T-027: Pantallas del catálogo
- **HU relacionada:** HU-005, HU-006
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-017, T-026
- **Descripción:** Crear la lista de productos (tarjetas apiladas, con filtro de activos y desactivados) y el formulario de producto con campos de variante según la categoría, sin campo de stock. Permitir editar precio y vencimiento, desactivar y reactivar productos, eliminar variantes sin movimientos y mostrar la advertencia "El precio de venta es menor que el costo".
- **Criterio de completitud:** ¿Desde el celular se puede crear un perfume con dos presentaciones, ver sus SKU y desactivarlo, con todos los textos en español?

### T-028: Pantallas de entradas, stock y ajustes
- **HU relacionada:** HU-007, HU-008, HU-012
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-018, T-020, T-021, T-026
- **Descripción:** Crear la consulta y búsqueda de stock, el historial de movimientos por variante, el formulario de entrada con costo y su anulación, y el formulario de ajuste con el motivo predefinido "Merma por vencimiento" para perfumes (RN-075), motivo libre y la advertencia "No escribas datos de tus clientes" (B-06). Las escrituras envían `Idempotency-Key`.
- **Criterio de completitud:** ¿Registrar una entrada de 10 a S/ 20.00 muestra el nuevo stock y costo promedio, y un ajuste aparece en el historial con su diferencia?

### T-029: Pantalla de venta rápida para lives
- **HU relacionada:** HU-009
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** XL
- **Dependencias:** T-022, T-026
- **Descripción:** Crear la venta en máximo 4 toques para el caso común (buscar variante, cantidad, precio prellenado, registrar), con objetivos táctiles de 44x44 px. La venta registra por defecto la fecha y hora actuales sin pedir nada (RN-126); un enlace "¿Fue otro día?" abre un selector de día (hasta 7 días atrás) y hora aproximada (RN-076). Permite agregar más variantes a la misma venta (RN-080), fusiona la misma variante con igual precio (RN-084), avisa si se repite con otro precio (RN-125), marca la línea con problema y deja quitarla y registrar el resto (RN-085). Borrador en memoria, reintento automático con la misma `Idempotency-Key` (1 s, 2 s, 4 s y luego "Reintentar"), mensaje "Sin conexión. Tu venta no se perdió..." y una sola confirmación de pérdida con una nueva clave. Sin campos de comprador.
- **Criterio de completitud:** ¿Con la red cortada la venta se registra una sola vez al volver la conexión; una venta sin tocar la fecha queda con la hora actual; una venta del día anterior a las 21:00 se registra con esa fecha y hora; y una venta de 2 líneas con una sin stock permite quitar esa línea y registrar la otra?

### T-030: Pantallas de historial y anulación de ventas
- **HU relacionada:** HU-010, HU-011
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-023, T-024, T-029
- **Descripción:** Crear el historial paginado de ventas, el detalle con la ganancia por línea y total (formato `S/ 1,250.50`), y las acciones de anular la venta o una línea con confirmación, mostrando el estado "Anulada", la fecha y hora de cada venta y el mensaje de RN-139 cuando no se puede anular. Los montos negativos se muestran como `-S/ 4.00`.
- **Criterio de completitud:** ¿Anular una línea actualiza la ganancia mostrada y la venta anulada aparece como "Anulada" sin sumar?

---

## 4. Integración de infraestructura

### T-031: Configuración de Nginx
- **HU relacionada:** Infraestructura (HU-001, HU-002)
- **Componente:** C-02 Reverse proxy
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-002
- **Descripción:** Versionar la configuración de Nginx (ADR-0008): TLS 1.2+, HSTS `max-age=31536000; includeSubDomains`, redirección 80 a 443, gzip/brotli, estáticos de Angular, proxy de `/api` a `127.0.0.1:8080` y límite de 10 peticiones por minuto por IP en `/api/auth/*`. Sin secretos en el repositorio.
- **Criterio de completitud:** ¿`nginx -t` pasa y un script de prueba con `curl` confirma el 301 desde el puerto 80, la cabecera HSTS y el 429 al superar el límite en un entorno de prueba?

### T-032: Script de backups cifrados
- **HU relacionada:** HU-004 (PRINCIPIO-006)
- **Componente:** C-10 Backups
- **Prioridad:** Must
- **Tamaño:** S
- **Dependencias:** T-008
- **Descripción:** Crear el script de `pg_dump` diario cifrado con AES-256 (clave por variable de entorno) que usa un rol de solo lectura cuya política RLS excluye los `tenant_id` de cuentas cerradas y vuelca `account` sin esas filas (RN-129), y la rotación que borra backups de más de 30 días, más su entrada de cron documentada. Sergio lo instala; el agente no lo ejecuta en el servidor real.
- **Criterio de completitud:** ¿En un entorno local el script genera un archivo cifrado, borra uno de hace 31 días, y un backup tomado después del cierre de la cuenta de Ana no contiene ninguna fila con su `tenant_id`?

---

## 5. Testing

### T-033: Tests de aislamiento entre emprendedores en todos los endpoints
- **HU relacionada:** HU-005 a HU-012 (criterio T-02)
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-017, T-020, T-021, T-023, T-024
- **Descripción:** Escribir un test de integración por cada endpoint que recibe un ID (en ruta o en el cuerpo) donde el usuario A lee o modifica un recurso del usuario B. Todos deben responder 404 con el mismo cuerpo que un recurso inexistente y sin cambiar los datos de B (PRINCIPIO-003).
- **Criterio de completitud:** ¿Existe un test 404 cruzado para cada endpoint con ID de la sección 4 y todos pasan en el CI?

### T-034: Tests de concurrencia y consistencia del stock
- **HU relacionada:** HU-009, HU-010, HU-012
- **Componente:** C-06 `sales`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-021, T-023
- **Descripción:** Escribir un test de dos ventas simultáneas de 1 unidad con stock 1 (una pasa y otra recibe `INSUFFICIENT_STOCK`), uno de dos anulaciones simultáneas y un test de propiedad que verifica `stock = entradas vigentes - líneas vigentes + diferencias de ajustes` tras secuencias aleatorias de movimientos (ADR-0004).
- **Criterio de completitud:** ¿Los tres tests pasan contra PostgreSQL 18 en Testcontainers y el stock nunca queda negativo?

### T-035: Tests de logs sin datos personales y de dependencias entre módulos
- **HU relacionada:** HU-001, HU-002 (criterio T-05)
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-012, T-023
- **Descripción:** Con un appender de prueba, verificar que registro fallido, login fallido y venta no escriben correo, nombre, contraseña, token ni cookie. Si Sergio aprueba ArchUnit, agregar el test que impide a `sales` escribir en tablas de `inventory` o `catalog`.
- **Criterio de completitud:** ¿El test de logs falla si se loguea el correo y pasa con el código actual?

### T-036: Tests E2E del flujo principal en 360 px con accesibilidad
- **HU relacionada:** HU-001 a HU-012 (criterios T-01, T-03, T-06)
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** L
- **Dependencias:** T-003, T-026, T-027, T-028, T-030
- **Descripción:** Con Playwright y axe-core en viewport 360x640, cubrir el flujo principal (registro, login, crear producto, registrar entrada, vender, ver ganancia, anular y ajustar stock) y escenarios propios de HU-003 (cambiar contraseña), HU-004 (reabrir la cuenta; el cierre lo agrega T-037), HU-006 (editar, desactivar y reactivar) y HU-008 (anular una entrada). Además, recorrer todas las rutas de la SPA con axe-core (PRINCIPIO-011). Verificar que no hay scroll horizontal, ni violaciones serias o críticas, ni textos en inglés en las pantallas clave.
- **Criterio de completitud:** ¿El E2E pasa en el CI y el CI falla si se agrega una violación seria de axe-core? ¿El CI falla si una ruta de la SPA no pasa por axe-core?

### T-037: Cierre de cuenta
- **HU relacionada:** HU-004
- **Componente:** C-03 `identity` y C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-014, T-026, T-036
- **Descripción:** Implementar `POST /api/account/close` (valida la contraseña actual, registra `closed_at` y `closure_reason = USER` y revoca todas las sesiones, incluida la actual, RN-127) y el botón "Cerrar cuenta" en "Mi cuenta", que pide la contraseña y explica que puede reabrirla durante 30 días. Agregar su escenario E2E con axe-core a la suite de T-036.
- **Criterio de completitud:** ¿Con la contraseña correcta la cuenta queda cerrada y las sesiones del celular y de la laptop reciben 401; con una incorrecta da `CURRENT_PASSWORD_INVALID` y la cuenta sigue activa; y el E2E de cerrar y reabrir pasa en el CI?

### T-038: Política de privacidad
- **HU relacionada:** HU-001 (RN-003, PRINCIPIO-005)
- **Componente:** C-01 `web-app`
- **Prioridad:** Must
- **Tamaño:** S
- **Dependencias:** T-026, T-036
- **Descripción:** Redactar en español claro un borrador de política de privacidad (datos que se guardan, para qué, plazo de borrado de 30 días, derechos según la Ley N° 29733 y contacto) para que Sergio lo revise y apruebe, y publicarlo en una página pública enlazada desde el registro, con su escenario E2E y axe-core en la suite de T-036.
- **Criterio de completitud:** ¿La página se abre desde el formulario de registro sin iniciar sesión, pasa axe-core en 360 px y Sergio aprobó el texto?

### T-039: Comando de restablecimiento manual de contraseña
- **HU relacionada:** HU-003 (SUP-07)
- **Componente:** C-03 `identity`
- **Prioridad:** Must
- **Tamaño:** S
- **Dependencias:** T-013
- **Descripción:** Crear un comando de consola (por ejemplo, `java -jar app.jar reset-password --account 4821`) que solo ejecuta Sergio en el servidor: pide la nueva contraseña sin mostrarla, aplica RN-105 y RN-134, guarda el hash Argon2id, reinicia el contador de fallos y revoca todas las sesiones. Nunca escribe la contraseña en logs. El agente no lo ejecuta contra la base real (PRINCIPIO-008).
- **Criterio de completitud:** ¿Un test de integración confirma que tras el comando la contraseña anterior no entra, la nueva sí y los logs no la contienen?

### T-040: Procedimiento de incidentes y detección mínima
- **HU relacionada:** Infraestructura (PRINCIPIO-005, Ley N° 29733)
- **Componente:** C-07 `shared` / plataforma
- **Prioridad:** Must
- **Tamaño:** M
- **Dependencias:** T-012, T-033
- **Descripción:** Escribir `docs/runbooks/incidente-seguridad.md` (cómo detectar, contener, quién decide y plantilla de notificación a la ANPD dentro de 48 horas) y agregar un registro de eventos de seguridad con el ID de la cuenta (bloqueos, 404 cruzados, cambios de contraseña y cierres de cuenta) que el runbook usa como fuente.
- **Criterio de completitud:** ¿El runbook está aprobado por Sergio y un test verifica que un bloqueo y un 404 cruzado quedan registrados sin correo ni nombre?

---

## 6. Resumen

**Total de tareas:** 41 (T-000 a T-040)

### Por prioridad

| Prioridad | Tareas |
|---|---|
| Must | 41 |
| Should | 0 (fuera del lote de arquitectura) |
| Could | 0 (fuera del lote de arquitectura) |

### Por componente

| Componente | Tareas | Cantidad |
|---|---|---|
| Documentación | T-000 | 1 |
| C-01 `web-app` | T-002, T-026, T-027, T-028, T-029, T-030, T-036, T-038 | 8 |
| C-02 Reverse proxy | T-031 | 1 |
| C-03 `identity` | T-011, T-012, T-013, T-014, T-037, T-039 | 6 |
| C-04 `catalog` | T-015, T-016, T-017, T-018 | 4 |
| C-05 `inventory` | T-019, T-020, T-021 | 3 |
| C-06 `sales` | T-022, T-023, T-024, T-034 | 4 |
| C-07 `shared` / plataforma | T-001, T-003, T-004, T-009, T-010, T-033, T-035, T-040 | 8 |
| C-08 `lifecycle` | T-025 | 1 |
| C-09 Base de datos | T-005, T-006, T-007, T-008 | 4 |
| C-10 Backups | T-032 | 1 |

### Por tipo

| Tipo | Tareas | Cantidad |
|---|---|---|
| Documentación | T-000 | 1 |
| Setup | T-001 a T-010 | 10 |
| Features (backend y frontend) | T-011 a T-030, T-037, T-038, T-039 | 23 |
| Integración de infraestructura | T-031, T-032 | 2 |
| Testing y operación | T-033 a T-036, T-040 | 5 |

### Por tamaño

| Tamaño | Tareas | Cantidad |
|---|---|---|
| S (< 30 min) | T-000, T-005, T-013, T-032, T-038, T-039 | 6 |
| M (30 min - 1 h) | T-001, T-002, T-003, T-004, T-006, T-007, T-009, T-010, T-011, T-014, T-015, T-018, T-020, T-021, T-024, T-025, T-030, T-031, T-034, T-035, T-037, T-040 | 22 |
| L (1-2 h) | T-016, T-017, T-019, T-022, T-023, T-026, T-027, T-028, T-033, T-036 | 10 |
| XL (2-4 h) | T-008, T-012, T-029 | 3 |

Esfuerzo máximo estimado: 6 x 0.5 h + 22 x 1 h + 10 x 2 h + 3 x 4 h = **57 h**. T-008, T-012 y T-029 se reestimaron como XL porque concentran RLS con dos roles, sesiones con CSRF y bloqueo, y la pantalla de venta con fecha pasada y reintentos.

### Ruta crítica

```
T-000 → T-001 → T-005 → T-006 → T-007 → T-008 → T-009 → T-012 → T-016 → T-019 → T-022 → T-023 → T-030 → T-036
```

14 tareas, hasta ~24 h. Se pueden avanzar en paralelo: T-002, T-003, T-004, T-011, T-015, T-031 y T-032.

---

## 7. Fuera de este backlog o bloqueado

| Elemento | Motivo | Qué falta |
|---|---|---|
| Borrado definitivo de cuentas (parte de T-025) | **[BLOQUEADO]** por P-61 (M-07) | Que Sergio elija el mecanismo y la frecuencia. RN-128 ya impide entrar a la cuenta vencida |
| Elegir hosting, dominio/TLS y plataforma de CI (D-01, D-03, D-04; P-62) | No bloquea el desarrollo | ADR antes de desplegar el piloto |
| Implementar F-16 con el ADR 0002 (aprobado) | Fuera del MVP | Arquitectura y HU del siguiente lote |
| F-07 a F-17 (Should y Could) | La arquitectura v0.2 solo cubre el MVP | Arquitectura y HU del siguiente lote |
| W-01 a W-12 | Won't | - |

Todas las decisiones anteriores están resueltas y escritas como reglas en `03-reglas-negocio.md` (secciones "Auditoría QA v2" y "Auditoría QA v3"). Los únicos pendientes son P-61 (mecanismo del borrado) y P-62 (proveedores de despliegue).
