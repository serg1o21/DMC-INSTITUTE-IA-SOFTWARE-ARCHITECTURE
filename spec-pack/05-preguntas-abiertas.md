# 06 - Preguntas abiertas: miStorePe

Preguntas que el Spec Pack no puede resolver sin una decisión del Responsable del producto. Ningún agente implementa un caso listado como **pendiente** hasta que tenga respuesta (PRINCIPIO-009).

Actualizado el 2026-11-10 tras las auditorías QA v2 y v3, y el 2026-11-11 tras la validación del prototipo (auditoría QA v4, P-64). El Responsable del producto aprobó la arquitectura v1.0 y delegó en QA los casos restantes. Criterio de las respuestas: la primera versión es simple y amigable; ante la duda se elige un valor por defecto en vez de pedir datos o bloquear al usuario. Las preguntas respondidas pasan a la tabla de la sección 2 y no se borran sus números, para no romper referencias.

---

## 1. Pendientes (solo P-64 bloquea un caso del MVP)

### P-61 - Mecanismo del borrado definitivo
- Origen: M-07 de la auditoría QA (MEDIA).
- Reglas afectadas: RN-006, RN-046, PRINCIPIO-006
- Pregunta: ¿el borrado definitivo a los 30 días se hace con un job (diario, cada hora) o con otro mecanismo? ¿Qué tolerancia se acepta respecto del plazo de 30 días?
- Mitigación vigente: RN-128 impide entrar a una cuenta con 30 días o más desde el cierre, aunque el borrado se retrase; RN-129 saca las cuentas cerradas de los backups.
- Bloquea: solo el borrado definitivo de T-025.
- Respuesta: pendiente.

### P-62 - Proveedores de hosting, dominio/TLS y CI
- Origen: M-14 de la auditoría QA (MEDIA); `02-alcance.md`, sección 6 (D-01, D-03, D-04).
- Pregunta: ¿qué proveedores se usan? Cada uno requiere un ADR (PRINCIPIO-002).
- Bloquea: el despliegue del piloto. No bloquea el desarrollo.
- Respuesta: pendiente.

### P-64 - Costo promedio al anular la primera entrada
- Origen: O-04 de `10-validation-log.md` (validación del prototipo, 2026-11-11; severidad ALTA). Nota QA de HU-008.
- Reglas afectadas: RN-088, RN-118, RN-154 (propuesta), RN-062
- Pregunta: al anular la única entrada vigente de una variante no hay un costo promedio anterior que restaurar. ¿Qué costo promedio queda?
  - Opción A (recomendada): la variante queda sin costo promedio, igual que una variante nueva, y se muestra "Costo promedio: sin entradas" (RN-154).
  - Opción B: se conserva el último costo registrado.
- Criterio: la opción A es coherente con RN-062 (sin entradas no hay advertencia de precio) y RN-116 (sin entradas no se ajusta hacia arriba), y evita que una venta futura calcule su ganancia con S/ 0.00.
- Mitigación vigente: ninguna; el caso no se implementa hasta la respuesta (PRINCIPIO-009).
- Bloquea: el criterio [AUDITORÍA v4] de HU-008 en T-020. El resto de HU-008 no se bloquea.
- Respuesta: pendiente.

---

## 2. Respondidas (2026-11-10)

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
| P-55 | Variante "Única" | Campo `is_single` de la variante. Si luego se agregan variantes al accesorio, "Única" se mantiene como una más | `08` |
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
| Arquitectura | Versión 0.2 | Aprobada como v1.0, junto con los ADR 0003 a 0012 y las dependencias de la sección 6.5 | `08` |

Las respuestas delegadas por Sergio a QA con el criterio de simplicidad ya son reglas en `03-reglas-negocio.md`.
