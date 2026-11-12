# 05 - Preguntas abiertas: miStorePe

Preguntas que el Spec Pack no puede resolver sin una decisión del Responsable del producto. Ningún agente implementa un caso listado como **pendiente** hasta que tenga respuesta (PRINCIPIO-009).

Actualizado el 2026-11-10 tras las auditorías QA v2 y v3, el 2026-11-11 tras la validación del prototipo (auditoría QA v4, P-64) y la auditoría del Spec Finalizer (P-65 a P-73, que consolidan aquí los pendientes que estaban repartidos en otros archivos), y el 2026-11-12 tras la segunda auditoría del Spec Finalizer (estados sincronizados y P-74 a P-77). El Responsable del producto aprobó la arquitectura v1.0 y delegó en QA los casos restantes. Criterio de las respuestas: la primera versión es simple y amigable; ante la duda se elige un valor por defecto en vez de pedir datos o bloquear al usuario. Las preguntas respondidas pasan a las secciones 2 y 3 y no se borran sus números, para no romper referencias.

---

## 1. Pendientes

Estado al 2026-11-12. **Ninguna pregunta pendiente bloquea el desarrollo del MVP**: P-74 a P-77 se respondieron el 2026-11-12 (sección 2). Bloquea el despliegue del piloto, no el desarrollo: P-62, y la validación legal de P-49. El resto es de funcionalidades Should o no bloquea nada.

| ID | Tema | Bloquea |
|---|---|---|
| P-62 | Proveedores de hosting, dominio/TLS y CI | Despliegue del piloto (T-041) |
| P-69 | Reapertura y sesiones de cuentas suspendidas o cerradas por falta de pago | F-16 (Should) |
| P-70 | Sesión del Administrador frente a `tenant_id` | F-16 (Should) |
| P-71 | Ubicación del campo "Stock mínimo" | F-07 (Should) |
| P-72 | Dividir HU-005 y HU-009 (INVEST) | Nada |
| P-73 | Coherencia de KPI-02 y KPI-04 | Nada (métrica del piloto) |

Las respondidas el 2026-11-11 (P-61, P-64 a P-68) y el 2026-11-12 (P-74 a P-77) están en la sección 2.

### P-62 - Proveedores de hosting, dominio/TLS y CI
- Origen: M-14 de la auditoría QA (MEDIA); `02-alcance.md`, sección 6 (D-01, D-03, D-04).
- Pregunta: ¿qué proveedores se usan? Cada uno requiere un ADR (PRINCIPIO-002).
- Bloquea: el despliegue del piloto (T-041). No bloquea el desarrollo.
- Respuesta: pendiente.

### P-69 - Cuentas suspendidas o cerradas por falta de pago
- Origen: A-03 y M-02 de la auditoría QA (antes solo citados en `06` y `07`).
- Preguntas: (1) ¿quién reabre una cuenta cerrada por falta de pago y cómo se distingue del cierre voluntario (`closure_reason`)? (2) ¿Al suspender una cuenta se cierran sus sesiones abiertas?
- Recomendación: (1) solo el Administrador la reactiva; (2) sí, se cierran todas.
- Bloquea: F-16 (Should). No bloquea el MVP.
- Respuesta: pendiente.

### P-70 - Sesión del Administrador frente a `tenant_id`
- Origen: auditoría del Spec Finalizer.
- Pregunta: `user_session.tenant_id` es NOT NULL y apunta a `account.id`, pero el Administrador no tiene `tenant_id` (ADR 0002). ¿Cómo se guarda su sesión?
- Recomendación: con F-16, permitir `tenant_id` NULL solo cuando el rol es `ADMIN` (con CHECK), mediante un ADR nuevo.
- Bloquea: F-16 (Should). En el MVP no existe la pantalla del Administrador.
- Respuesta: pendiente.

### P-71 - Ubicación del campo "Stock mínimo"
- Origen: UI-Q2 de `09-ui-spec.md`.
- Pregunta: ¿el campo va en P-007 (Editar variante) o en una pantalla aparte?
- Recomendación: en P-007, valor por defecto 0 (P-52).
- Bloquea: F-07 (Should).
- Respuesta: pendiente.

### P-72 - Dividir HU-005 y HU-009
- Origen: validación INVEST de `06` ("No cumple Small").
- Pregunta: ¿se dividen (HU-005 por categoría; HU-009 en una línea, varias líneas y fecha pasada) o se aceptan como están? El backlog ya divide la pantalla de venta (T-029a a T-029d).
- Recomendación: aceptarlas como están; las tareas ya son pequeñas.
- Bloquea: nada.
- Respuesta: pendiente.

### P-73 - Coherencia de KPI-02 y KPI-04
- Origen: `01-project-brief.md`, sección 5.
- Pregunta: KPI-02 acepta que 2 de 3 validen el piloto, pero KPI-04 exige que 3 de 3 paguen. ¿Se ajusta KPI-04 a 2 de 3?
- Bloquea: nada.
- Respuesta: pendiente. (Corrección 2026-11-12: el texto que figuraba aquí era la respuesta de P-61 y se movió a la sección 2.)

---

## 2. Respondidas (2026-11-11 y 2026-11-12)

### P-74 - Lectura de la sesión con RLS activa
- Origen: auditoría del Spec Finalizer (2026-11-12, B-3).
- Reglas y decisiones afectadas: ADR-0003, ADR-0007, PRINCIPIO-003, RN-097, RN-132.
- Pregunta: `user_session` tiene RLS por `tenant_id`, pero en cada petición la sesión se busca por `token_hash` antes de conocer el tenant; sin `app.tenant_id` la consulta no devuelve filas. ¿Cómo se resuelve la sesión?
  - Opción A (recomendada): función de base de datos `resolve_session(token_hash)` con `SECURITY DEFINER`, propiedad del dueño del esquema, que devuelve solo `session_id`, `tenant_id`, `last_activity_at`, `revoked_at` y el estado de la cuenta, sin otras columnas. Después la app ejecuta `SET LOCAL app.tenant_id` y todo lo demás (incluida la actualización de `last_activity_at`) pasa por RLS.
  - Opción B: sacar `user_session` de RLS y filtrar solo en servicio, como `account`.
- Criterio: A mantiene la doble protección de ADR-0003 en todas las tablas con `tenant_id` y expone una sola lectura acotada; B es más simple pero deja una tabla de negocio sin RLS, lo que hoy hace fallar el test de catálogo de tablas de T-008.
- Bloquea: T-008b, T-009 y T-012a.
- Respuesta (2026-11-12): **Opción A aprobada por Sergio.** Registrada en ADR-0014 (Aprobado). T-008b, T-009 y T-012a sin bloqueo.

### P-75 - Roles de base de datos para jobs, comando y backup
- Origen: auditoría del Spec Finalizer (2026-11-12, B-3).
- Reglas y decisiones afectadas: ADR-0003, ADR-0011, RN-006, RN-129, PRINCIPIO-006, PRINCIPIO-008.
- Pregunta: ADR-0003 define dos roles (app sin `BYPASSRLS` y borrado con `BYPASSRLS`). Pero (1) la limpieza diaria de T-025a borra sesiones e idempotencia de todos los tenants; (2) el comando de T-039 revoca sesiones de una cuenta sin sesión iniciada; y (3) ADR-0011 pide volcar `account` sin las cuentas cerradas, aunque `account` no tiene RLS. ¿Con qué rol y mecanismo se hace cada uno?
- Recomendación:
  1. T-025a y T-039 usan el rol de la app y recorren los tenants: leen los IDs de `account` (sin RLS) y, por cada uno, abren una transacción con `SET LOCAL app.tenant_id`. Con 3 cuentas el costo es despreciable y no se agrega otro rol con `BYPASSRLS`.
  2. Se crea un tercer rol, `backup`, de solo lectura y sin `BYPASSRLS`. Sus políticas RLS en las tablas de negocio excluyen los `tenant_id` de cuentas con `status = CLOSED`. Se activa RLS en `account` con dos políticas: `USING (true)` para el rol de la app y `USING (status <> 'CLOSED')` para el rol `backup`. El volcado usa `pg_dump --enable-row-security`.
  3. El rol de borrado de ADR-0003 no cambia.
- Bloquea: T-008b, T-025a, T-032 y T-039.
- Respuesta (2026-11-12): **Recomendación aprobada por Sergio** (puntos 1 a 3). Registrada en ADR-0014 (Aprobado). T-008b, T-025a, T-032 y T-039 sin bloqueo.

### P-76 - Desbloqueo exacto a los 900 s
- Origen: nota "QA:" de HU-002 en `06`; auditoría del Spec Finalizer (B-5).
- Reglas afectadas: RN-004, RN-130.
- Pregunta: si la cuenta se bloquea a las 10:00:00, ¿un login correcto a las 10:15:00 (900 s exactos) entra?
- Recomendación: sí. El bloqueo es el intervalo semiabierto [quinto fallo, quinto fallo + 900 s), igual que los intervalos de RN-031 y RN-051, y coincide con `locked_until` de `07`.
- Bloquea: el criterio de borde de T-012b (hoy prueba 899 s y 901 s).
- Respuesta (2026-11-12): **Recomendación aprobada por Sergio.** Sí entra: el bloqueo es [quinto fallo, quinto fallo + 900 s). Queda como RN-155.

### P-77 - Contraseña nueva igual a la actual y fallos al cambiarla
- Origen: nota "QA:" de HU-003 en `06`; auditoría del Spec Finalizer (B-5).
- Reglas afectadas: RN-004, RN-105, RN-110.
- Preguntas: (1) ¿se acepta una contraseña nueva igual a la actual? (2) ¿Escribir mal la contraseña actual al cambiarla suma intentos fallidos al bloqueo de RN-004?
- Recomendación: (1) se rechaza con "La nueva contraseña debe ser distinta de la actual"; (2) no suma: RN-004 habla de intentos de inicio de sesión, y el usuario ya está autenticado.
- Bloquea: un criterio de T-013.
- Respuesta (2026-11-12): **Recomendación aprobada por Sergio.** (1) Se rechaza con "La nueva contraseña debe ser distinta de la actual" (RN-156, código `PASSWORD_SAME_AS_CURRENT`). (2) No suma al bloqueo (RN-157).


### P-61 - Mecanismo del borrado definitivo
- Origen: M-07 de la auditoría QA (MEDIA). Reglas afectadas: RN-006, RN-046, PRINCIPIO-006.
- Pregunta: ¿el borrado definitivo a los 30 días se hace con un job (diario, cada hora) o con otro mecanismo? ¿Qué tolerancia se acepta respecto del plazo de 30 días?
- Respuesta (2026-11-11): **Job diario con Spring `@Scheduled`**, mismo componente C-08 `lifecycle` que ya gestiona la limpieza de sesiones (T-025a). Tolerancia: hasta 24 h por encima del plazo de 30 días (máximo el día 31). RN-128 ya impide el acceso desde el día 30, por lo que la ventana de exposición es solo de backups (cubierta por RN-129). El test de T-025b verifica que tras el job no queda ninguna fila del tenant. Resuelto por Sergio. T-025b ya puede implementarse (el rol de la base quedó definido en P-75, 2026-11-12).

### P-64 - Costo promedio al anular la primera entrada
- Origen: O-04 de `10-validation-log.md` (severidad ALTA). Reglas afectadas: RN-088, RN-118, RN-154, RN-062.
- Pregunta: al anular la única entrada vigente de una variante no hay un costo promedio anterior que restaurar. ¿Qué costo promedio queda? Opción A: sin costo promedio, como una variante nueva. Opción B: se conserva el último costo registrado.
- Respuesta (2026-11-11): **Opción A aprobada por Sergio.** La variante queda sin costo promedio (`avg_cost = NULL`). Se muestra "Costo promedio: sin entradas" en P-006, P-007, P-008 y P-013. Coherente con `07`, sección 3.3.
- Cambios resultantes: RN-154 aprobada; T-020 y T-028 sin bloqueo; criterio [AUDITORÍA v4] de HU-008 vigente; `newAvgCost: null` documentado en `07`.

### P-65 - Alcance de "Merma por vencimiento"
- Origen: nota "confirmar" de `03` (Revisión QA) y de HU-012. Reglas afectadas: RN-075.
- Pregunta: ¿el motivo "Merma por vencimiento" se ofrece en todo ajuste de perfume o solo en perfumes vencidos? Opción A: en todo ajuste de perfume (texto actual de RN-075). Opción B: solo en variantes de perfume con fecha de vencimiento anterior a hoy.
- Respuesta (2026-11-11): **Opción A confirmada por el Spec Finalizer** (delegada por Sergio al criterio de simplicidad). RN-075 queda como está.

### P-66 - Ampliación de contratos de API para la UI
- Origen: auditoría del Spec Finalizer, observaciones O-02 y O-10 de `10-validation-log.md`.
- Pregunta: ¿se aprueban `GET /api/variants/{id}`, los campos nuevos de `GET /api/variants/{id}/movements` (`unitCost`, `saleAt`, `reasonType`, `voidable`, `voidBlockedReason` y cabecera de la variante), `hasMovements` en `GET /api/products/{id}` y `voidedAmount`/`voidedProfit` en las ventas anuladas?
- Respuesta (2026-11-11): **Aprobada por Sergio.** Forman parte de la especificación; los implementan T-017, T-018, T-021, T-024, T-027, T-028 y T-030.

### P-67 - Correo de contacto para derechos ARCO
- Origen: UI-Q1 de `09-ui-spec.md`; P-49.
- Respuesta (2026-11-11): **sergioc9916@gmail.com**, leído de la variable de entorno `ARCO_CONTACT_EMAIL` en el backend y expuesto como configuración en la API (no se escribe en el código). Resuelto por Sergio. Desbloquea T-038 y los textos de P-015 y P-016.

### P-68 - Ampliar RN-005
- Origen: A-05 de la auditoría QA.
- Respuesta (2026-11-11): **Texto aprobado por el Spec Finalizer** (delegado por Sergio). RN-005 se reescribió en `03`: modificar, anular o citar por ID en el cuerpo un recurso ajeno también responde HTTP 404 sin cambiar datos.

## 3. Respondidas (2026-11-10)

| ID | Tema | Respuesta | Dónde quedó |
|---|---|---|---|
| A-01 | Anular una venta con un ajuste posterior | No se anula; mensaje "No se puede anular: hay un ajuste de stock posterior a esta venta" | RN-139 |
| A-02 | Cómo se cierra la cuenta | Se confirma con la contraseña; se registra `closed_at` y se cierran todas las sesiones | RN-127 |
| A-04 | Confirmar RN-056, RN-088, RN-115, RN-118 | Confirmadas | `03` |
| P-37 | Stock guardado o calculado | Se guarda en la variante; RN-026 es un invariante verificado por test | ADR-0004 |
| P-38 | Vencimiento en ventas pasadas | Se compara con la fecha de hoy | RN-074 |
| P-39 | Hora de la venta y plazo de anulación | La venta guarda fecha y hora: por defecto el momento del registro; para ventas olvidadas, el usuario elige el día (hasta 7 días calendario atrás) y una hora aproximada. Las 24 horas para anular se cuentan desde el registro | RN-076 a RN-078, RN-087, RN-126 |
| P-40 | Pérdida en varias líneas | Una sola confirmación por venta, con la suma de las pérdidas | RN-073 |
| P-41 | Productos desactivados | Se pueden desactivar con stock y el stock se conserva. No se registran ventas ni entradas mientras esté desactivado ("Reactiva el producto para registrar mercadería"); las anulaciones dentro de las 24 horas y los ajustes sí se permiten. No aparece en alertas (F-07, F-10) | RN-016, RN-137, RN-140, RN-147 |
| P-42 | Desactivar una variante sola | Fuera del MVP: se desactiva el producto completo | - |
| P-43 | Nombre en el registro | Opcional, máximo 100 caracteres; si falta, la app saluda con el correo. Se puede corregir en "Mi cuenta" | RN-141 |
| P-44 | Registro con el correo de una cuenta cerrada | Se responde "Este correo ya tiene una cuenta" hasta el borrado definitivo; dentro de los 30 días, el usuario inicia sesión y reabre | RN-001, RN-090 |
| P-45 | Login de cuentas suspendidas y cerradas | Intencional: la cerrada entra solo para ver "Reabrir cuenta" (o "Contacta al administrador"); la suspendida no entra | RN-007, RN-063, RN-104 |
| P-46 | Intentos durante el bloqueo | Mensaje "Demasiados intentos. Intenta de nuevo en 15 minutos"; no suman ni alargan el bloqueo. Se acepta el riesgo de revelar que un correo existe: el registro ya lo revela y son 3 usuarios conocidos | RN-130, RN-131 |
| P-47 | Qué es actividad | Toda petición del usuario a la app; no hay consultas automáticas en segundo plano en el MVP | RN-132 |
| P-48 | Aviso de fin de la gratuidad | Fuera de la app: Sergio avisa directamente a los 3 emprendedores | - |
| P-49 | Incidentes y derechos ARCO | Runbook de incidentes con notificación a la ANPD en 48 horas (T-040). Los derechos ARCO se ejercen escribiendo al correo de contacto publicado en la política de privacidad, y Sergio responde a mano. La app ya permite rectificar el nombre, cambiar la contraseña y cerrar la cuenta. Validar con un especialista legal antes del piloto | T-038, T-040 |
| P-50 | Fecha y variantes de una entrada | Una entrada = una variante, con la fecha y hora del registro; la pantalla permite registrar varias seguidas sin salir | RN-142 |
| P-51 | Búsqueda de stock | Nombre del producto, SKU, talla y color, sin distinguir mayúsculas | RN-138 |
| P-52 | Stock mínimo por defecto | 0 (sin alerta). Aplica con F-07 | - |
| P-53 | Productos sin movimiento | Las ventas anuladas no cuentan y se usa la fecha y hora de venta. Reformular RN-030 junto con F-08 | - |
| P-54 | Costo en la línea de venta | Cada línea guarda el costo usado | RN-028, RN-079 |
| P-55 | Variante "Única" | Campo `is_single` de la variante. Si luego se agregan variantes al accesorio, "Única" se mantiene como una más | `07` |
| P-56 | Ajustes | Se permite diferencia 0 (confirma el conteo). Motivo de 1 a 200 caracteres, sin solo espacios. Un ajuste no se anula: se corrige con otro ajuste | RN-143, RN-144, RN-145 |
| P-57 | Límites de nombre y presentación | Nombre de producto de 1 a 100 caracteres, puede repetirse. Presentación de 1 a 9,999 ml | RN-011, RN-093, RN-146 |
| P-58 | Formatos visibles | Montos negativos `-S/ 4.00`. Fechas `10/11/2026` y horas `21:30` (formato es-PE). Al escribir montos se acepta punto o coma decimal | RN-148, RN-149, RN-152 |
| P-59 | Varios enlaces de recuperación | Al pedir uno nuevo, el anterior deja de ser válido. Aplica con F-11 | - |
| P-60 | Mermas como salida | Sí: el glosario ya indica que la merma será una salida con F-17; en el MVP se retira con un ajuste. Actualizar el brief cuando se planifique F-17 | `04` |
| M-01 | Última variante | No se elimina la única variante; se ofrece desactivar el producto | RN-136 |
| M-06 | Correo y contraseña | Formato `nombre@dominio.ext`, máximo 254; contraseña de 12 a 128 caracteres | RN-133, RN-134 |
| M-12 | Perfumes sin vencimiento | Misma presentación y ambos sin vencimiento = duplicado | RN-135 |
| Backups | RN-046 frente a PRINCIPIO-006 | Los backups no incluyen cuentas cerradas | RN-129 |
| RN-105, RN-106 | Propuestas QA | Aprobadas | `03` |
| B-02 | Singular de unidades | "queda 1 unidad", "quedan 0 unidades" | RN-150 |
| B-08 | Línea fusionada mayor que 9,999 | Se rechaza con "La cantidad máxima por línea es 9,999" | RN-151 |
| UX registro | Sesión tras registrarse | Se inicia sola | RN-153 |
| P-63 | ADR 0002 (Administrador sin `tenant_id`) | Aprobado; PRINCIPIO-003 actualizado | `0002-admin-without-tenant.md` |
| Arquitectura | Versión 0.2 | Aprobada como v1.0, junto con los ADR 0003 a 0013 y las dependencias de la sección 6.5 | `07` |
| UI-Q3 | Angular CDK | Aprobada como dependencia el 2026-11-10 | `07`, sección 6.5 |

### Referencias históricas

Hallazgos de auditorías anteriores que se citan en otros archivos y ya están resueltos:

| ID | Tema | Estado | Dónde quedó |
|---|---|---|---|
| M-01 | Contradicción RN-099 y RN-058 al eliminar la última variante | Resuelto | RN-136 |
| M-07 | Mecanismo del borrado | Pasó a P-61 | P-61 |
| M-10 | Ejemplos con "ganancia del día" sin pantalla en F-06 | Resuelto: criterios ajustados en HU-011 y HU-012; el resumen por periodo es F-09 | `06` |
| P-31 | Atributos del accesorio | Resuelto: sin atributos obligatorios | SUP-08, RN-060 |
| B-06 | El motivo del ajuste es texto libre y podría tener datos de clientes | Mitigado con el aviso "No escribas datos de tus clientes" | P-014, T-028 |
| A-03, M-02 | Cuentas por falta de pago | Pasaron a P-69 | P-69 |
| A-05 | Ampliar RN-005 | Pasó a P-68 | P-68 |
| M-14 | Proveedores de hosting, dominio/TLS y CI | Pasó a P-62 | P-62 |
| UI-Q1 | Correo ARCO | Pasó a P-67 | P-67 |
| UI-Q2 | Stock mínimo | Pasó a P-71 | P-71 |

Las respuestas delegadas por Sergio a QA con el criterio de simplicidad ya son reglas en `03-reglas-negocio.md`.
