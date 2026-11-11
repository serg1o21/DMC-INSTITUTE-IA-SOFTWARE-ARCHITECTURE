# 03 - Reglas de negocio: miStorePe

Basado en `00-constitucion.md`, `01-project-brief.md` y `02-alcance.md`. Fecha: 2026-11-10.

Convenciones:
- Montos en soles con 2 decimales (S/ 0.00). Tiempos en segundos. Fechas en ISO 8601 (AAAA-MM-DD) y zona horaria America/Lima (UTC-05:00).
- Toda regla de este documento está aprobada por el Responsable del producto (2026-11-10), salvo RN-154. Una regla marcada "Propuesta BA" o "Propuesta QA" no se implementa hasta su aprobación (PRINCIPIO-009); hoy solo queda RN-154 (Propuesta QA, pendiente de P-64).
- Las reglas citan la funcionalidad del alcance (F-NN) en su categoría.
- Una venta tiene una o más líneas (RN-080, RN-107). Las reglas de cantidad, precio final, stock y anulación están redactadas por línea (RN-021 a RN-023, RN-025, RN-043, RN-052 a RN-054, RN-057, RN-106). Las reglas de ganancia y vencimiento que dicen "una venta" (RN-027, RN-072 a RN-074) se aplican a cada línea de venta.
- Los IDs son estables: no se renumeran. Las reglas se agrupan por funcionalidad, por eso los números no siempre son consecutivos.
- Las decisiones que faltan están en `06-preguntas-abiertas.md` (revisión QA del 2026-11-10). Ningún agente implementa un caso listado ahí hasta que se responda.

---

## Cuenta y registro (F-01)

**RN-001** | Categoría: Registro (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si un visitante envía el formulario de registro con un correo que ya pertenece a otra cuenta, entonces el sistema NO DEBE crear una cuenta nueva.
✅
**Ejemplo:** no existe ninguna cuenta con `ana@gmail.com`; Ana se registra con ese correo y la cuenta se crea.
❌
**Contraejemplo:** ya existe `ana@gmail.com` y el sistema crea una segunda cuenta con el mismo correo.

**RN-094** | Categoría: UX (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si un visitante intenta registrarse con un correo que ya tiene cuenta, entonces el sistema DEBE mostrar el mensaje "Este correo ya tiene una cuenta".
✅
**Ejemplo:** `ana@gmail.com` ya existe; el visitante ve "Este correo ya tiene una cuenta".
❌
**Contraejemplo:** el formulario falla sin ningún mensaje.

**RN-048** | Categoría: Registro (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Event-driven
Cuando un visitante envíe un correo en el registro o en el inicio de sesión, el sistema DEBE convertirlo a minúsculas y quitar los espacios al inicio y al final antes de compararlo.
✅
**Ejemplo:** `Ana@Gmail.com ` se guarda y se compara como `ana@gmail.com`.
❌
**Contraejemplo:** existen dos cuentas, `Ana@Gmail.com` y `ana@gmail.com`.

**RN-002** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-004)
Patrón EARS: Unwanted
Si un visitante envía el formulario de registro con una contraseña de menos de 12 caracteres, entonces el sistema NO DEBE crear la cuenta.
✅
**Ejemplo:** la contraseña `TiendaLima2026` (14 caracteres) es aceptada.
❌
**Contraejemplo:** la contraseña `Lima2026` (8 caracteres) es aceptada y la cuenta se crea.

**RN-003** | Categoría: Registro (F-01) | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-005), Ley N° 29733
Patrón EARS: Unwanted
Si un visitante envía el formulario de registro sin marcar la aceptación de la política de privacidad, entonces el sistema NO DEBE crear la cuenta.
✅
**Ejemplo:** Ana marca "Acepto la política de privacidad", envía el formulario y la cuenta se crea con `privacy_accepted_at = 2026-11-10T12:40:00-05:00`.
❌
**Contraejemplo:** Ana deja la casilla sin marcar y la cuenta se crea igual.

**RN-004** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-004)
Patrón EARS: Event-driven
Cuando una cuenta acumule 5 intentos fallidos consecutivos de inicio de sesión, el sistema DEBE bloquear el inicio de sesión de esa cuenta durante 900 segundos.
✅
**Ejemplo:** el quinto intento fallido ocurre a las 10:00:00; un intento con la contraseña correcta a las 10:10:00 es rechazado y otro a las 10:15:01 es aceptado.
❌
**Contraejemplo:** tras 5 intentos fallidos, el sexto intento con la contraseña correcta a los 60 segundos es aceptado.

**RN-103** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si alguien intenta iniciar sesión con un correo que no pertenece a ninguna cuenta, entonces el sistema NO DEBE registrar un intento fallido para ninguna cuenta.
✅
**Ejemplo:** un intento con `noexiste@gmail.com` no suma intentos fallidos a ninguna cuenta.
❌
**Contraejemplo:** el intento con `noexiste@gmail.com` suma un fallo a la cuenta `ana@gmail.com`.

**RN-095** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras el emprendedor tenga una sesión iniciada, el sistema DEBE permitirle cambiar su contraseña.
✅
**Ejemplo:** Ana cambia su contraseña desde "Mi cuenta" a `NuevaClaveLima2026`.
❌
**Contraejemplo:** Ana solo puede cambiar su contraseña pidiéndoselo al administrador.

**RN-096** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si el emprendedor intenta cambiar su contraseña sin ingresar correctamente la contraseña actual, entonces el sistema NO DEBE cambiarla.
✅
**Ejemplo:** Ana ingresa su contraseña actual correcta y la nueva: se cambia.
❌
**Contraejemplo:** alguien con la sesión de Ana abierta cambia la contraseña sin conocer la actual.

**RN-105** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10, propuesta QA), basada en Constitución (PRINCIPIO-004)
Patrón EARS: Unwanted
Si el emprendedor intenta cambiar su contraseña por una nueva de menos de 12 caracteres, entonces el sistema NO DEBE cambiarla.
✅
**Ejemplo:** Ana cambia su contraseña a `NuevaClaveLima2026` (18 caracteres) y el cambio se acepta.
❌
**Contraejemplo:** Ana cambia su contraseña a `Lima2027` (8 caracteres) y el cambio se acepta.

**RN-110** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando el emprendedor cambie su contraseña, el sistema DEBE cerrar todas sus demás sesiones abiertas.
✅
**Ejemplo:** Ana cambia su contraseña desde el celular; la sesión de su laptop se cierra y debe iniciar sesión de nuevo, y la sesión del celular sigue abierta.
❌
**Contraejemplo:** Ana cambia su contraseña desde el celular y la sesión de su laptop sigue abierta.

**RN-097** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Event-driven
Cuando hayan pasado 28,800 segundos (8 horas) sin actividad en una sesión, el sistema DEBE cerrar esa sesión.
✅
**Ejemplo:** Ana deja la app abierta a las 10:00 y a las 18:01 debe iniciar sesión de nuevo.
❌
**Contraejemplo:** la sesión de Ana sigue abierta al día siguiente sin actividad.

**RN-049** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Event-driven
Cuando una cuenta inicie sesión con éxito, el sistema DEBE reiniciar a 0 su contador de intentos fallidos consecutivos.
✅
**Ejemplo:** Ana falla 3 veces, entra a la cuarta y al día siguiente falla 2 veces: la cuenta no se bloquea porque el contador está en 2.
❌
**Contraejemplo:** en el mismo caso, la cuenta se bloquea porque el sistema sumó 5 fallos.

**RN-120** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA), Constitución (PRINCIPIO-006, PRINCIPIO-007)
Patrón EARS: Ubiquitous
El sistema NO DEBE registrar en los logs el correo, el nombre ni la contraseña de ningún emprendedor; solo su identificador interno de cuenta.
✅
**Ejemplo:** el log de un inicio de sesión fallido dice "Intento fallido para la cuenta 4821".
❌
**Contraejemplo:** el log dice "Intento fallido para ana@gmail.com".

**RN-005** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-003)
Patrón EARS: Event-driven
Cuando un emprendedor solicite cualquier recurso (producto, variante, entrada, venta o ajuste) que pertenece a otro emprendedor, el sistema DEBE responder con HTTP 404.
✅
**Ejemplo:** Ana pide `GET /api/products/77`, que pertenece a Luis, y recibe 404.
❌
**Contraejemplo:** Ana recibe 200 con los datos del producto 77 de Luis.

**RN-006** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-006)
Patrón EARS: Event-driven
Cuando se cumplan 2,592,000 segundos (30 días) desde el `closed_at` de una cuenta, el sistema DEBE eliminar de forma definitiva la cuenta (incluidos su correo, nombre, hash de contraseña e intentos fallidos) y todos los datos con su `tenant_id`.
✅
**Ejemplo:** Ana cierra su cuenta el 2026-12-01T10:00:00-05:00; el 2026-12-31T10:00:00-05:00 ya no existe su cuenta ni ningún registro con su `tenant_id`.
❌
**Contraejemplo:** el 2027-01-15 los productos de Ana siguen en la base de datos marcados como inactivos.

**RN-046** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-006)
Patrón EARS: Ubiquitous
El sistema NO DEBE conservar ningún backup que contenga datos de una cuenta cerrada más de 2,592,000 segundos después de su `closed_at`.
✅
**Ejemplo:** Ana cerró su cuenta el 2026-12-01T10:00:00-05:00; el 2026-12-31T10:00:00-05:00 ya no existe ningún backup con sus datos.
❌
**Contraejemplo:** el 2027-02-01 todavía existe un backup del 2026-11-30 con los productos de Ana.

**RN-007** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras una cuenta esté cerrada, el sistema NO DEBE mostrar ni modificar su inventario.
✅
**Ejemplo:** Ana cerró su cuenta el 2026-12-01; el 2026-12-05 inicia sesión y solo ve la opción "Reabrir cuenta".
❌
**Contraejemplo:** Ana inicia sesión el 2026-12-05 y registra una venta con la cuenta cerrada.

**RN-090** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor que cerró su propia cuenta inicie sesión antes de que pasen 2,592,000 segundos desde su `closed_at` y confirme "Reabrir cuenta", el sistema DEBE reactivar la cuenta y cancelar su borrado.
✅
**Ejemplo:** Ana cerró su cuenta el 2026-12-01T10:00-05:00 y la reabre el 2026-12-10: recupera su inventario completo.
❌
**Contraejemplo:** Ana la reabre el 2026-12-10 y su inventario ya fue borrado.

**RN-104** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10, revisión QA)
Patrón EARS: Unwanted
Si una cuenta fue cerrada por falta de pago (RN-067), entonces el sistema NO DEBE permitir que el emprendedor la reabra por su cuenta.
✅
**Ejemplo:** la cuenta de Luis pasó a cerrada por falta de pago; al iniciar sesión ve "Contacta al administrador".
❌
**Contraejemplo:** Luis reabre su cuenta cerrada por falta de pago sin haber pagado.

**RN-063** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: State-driven
Mientras una cuenta esté suspendida por falta de pago, el sistema NO DEBE permitir el inicio de sesión con esa cuenta.
✅
**Ejemplo:** la gratuidad de Ana terminó el 2027-01-31 sin pago; el Administrador suspende su cuenta y el 2027-02-02 Ana intenta iniciar sesión y ve "Tu cuenta está suspendida por falta de pago".
❌
**Contraejemplo:** Ana inicia sesión el 2027-02-02 y registra ventas sin haber pagado.

**RN-067** | Categoría: Datos (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando hayan pasado 2,592,000 segundos (30 días) desde que una cuenta fue suspendida por falta de pago, el sistema DEBE cambiar su estado a cerrada y registrar ese instante como su `closed_at`.
✅
**Ejemplo:** la cuenta de Ana se suspende el 2027-02-01T00:00:00-05:00; el 2027-03-03T00:00:00-05:00 pasa a cerrada y su borrado definitivo vence el 2027-04-02T00:00:00-05:00 (RN-006).
❌
**Contraejemplo:** el 2027-05-01 la cuenta de Ana sigue suspendida con todos sus datos.

**RN-068** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: State-driven
Mientras el usuario tenga el rol Administrador, el sistema DEBE permitirle suspender una cuenta por falta de pago.
✅
**Ejemplo:** el Administrador marca la cuenta de Ana como "Suspendida" el 2027-02-01 desde la pantalla de administración.
❌
**Contraejemplo:** Ana, sin rol Administrador, suspende la cuenta de Luis.

**RN-069** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Constitución (PRINCIPIO-003)
Patrón EARS: Ubiquitous
El rol Administrador NO DEBE ver productos, variantes, entradas, ventas ni ajustes de ningún emprendedor.
✅
**Ejemplo:** en la pantalla de administración el Administrador ve solo nombre, correo y estado de la cuenta de Ana.
❌
**Contraejemplo:** el Administrador abre el inventario de Ana desde la pantalla de administración.

**RN-070** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras el usuario tenga el rol Administrador, el sistema DEBE permitirle reactivar una cuenta suspendida.
✅
**Ejemplo:** Ana paga el 2027-02-05 y el Administrador reactiva su cuenta; Ana vuelve a iniciar sesión con su inventario intacto.
❌
**Contraejemplo:** una cuenta suspendida solo puede reactivarse creando una cuenta nueva.

**RN-071** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El sistema NO DEBE asignar el rol Administrador mediante el formulario de registro.
✅
**Ejemplo:** el Responsable del producto inserta manualmente el rol Administrador en su propia cuenta, fuera de la app y sin intervención de agentes de IA (PRINCIPIO-008).
❌
**Contraejemplo:** un visitante se registra y obtiene el rol Administrador.

**RN-114** | Categoría: Seguridad (F-16) | Prioridad: SHOULD | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA), Constitución (PRINCIPIO-003)
Patrón EARS: Ubiquitous
La cuenta con el rol Administrador NO DEBE tener `tenant_id` ni inventario propio.
✅
**Ejemplo:** la cuenta del Responsable del producto no tiene `tenant_id` y no aparece como emprendedor.
❌
**Contraejemplo:** el Administrador tiene un `tenant_id` y registra productos como si fuera un emprendedor.

**RN-127** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (A-02)
Patrón EARS: Event-driven
Cuando el emprendedor confirme "Cerrar cuenta" ingresando su contraseña actual, el sistema DEBE registrar ese instante como `closed_at` de la cuenta y cerrar todas sus sesiones.
✅
**Ejemplo:** Ana confirma "Cerrar cuenta" con su contraseña el 2026-12-01T10:00:00-05:00; la cuenta queda con ese `closed_at` y sus sesiones del celular y de la laptop se cierran.
❌
**Contraejemplo:** Ana cierra su cuenta sin ingresar su contraseña, o su sesión de la laptop sigue abierta.

**RN-128** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (M-07)
Patrón EARS: Unwanted
Si han pasado 2,592,000 segundos o más desde el `closed_at` de una cuenta, entonces el sistema NO DEBE permitir el inicio de sesión con esa cuenta y DEBE responder igual que con un correo que no existe, aunque sus datos aún no se hayan borrado.
✅
**Ejemplo:** `closed_at` es 2026-12-01T10:00:00-05:00; el 2026-12-31T10:00:01-05:00 Ana intenta entrar y ve el mismo mensaje que un correo inexistente.
❌
**Contraejemplo:** el borrado se retrasa y Ana entra el 2026-12-31T12:00:00-05:00 y reabre su cuenta.

**RN-129** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (C-03, PRINCIPIO-006)
Patrón EARS: Unwanted
Si una cuenta está cerrada cuando se genera un backup, entonces el sistema NO DEBE incluir en ese backup ningún dato de esa cuenta.
✅
**Ejemplo:** Ana cerró su cuenta el 2026-12-01; el backup del 2026-12-29 no contiene su cuenta ni ningún registro con su `tenant_id`.
❌
**Contraejemplo:** el backup del 2026-12-29 contiene los productos de Ana y se conserva hasta el 2027-01-28.

**RN-130** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (P-46)
Patrón EARS: State-driven
Mientras una cuenta esté bloqueada por RN-004, el sistema DEBE rechazar todo inicio de sesión con el mensaje "Demasiados intentos. Intenta de nuevo en 15 minutos".
✅
**Ejemplo:** el quinto fallo de Ana fue a las 10:00:00; a las 10:05:00 intenta entrar y ve ese mensaje.
❌
**Contraejemplo:** Ana ve "Correo o contraseña incorrectos" aunque escribió bien su contraseña y no sabe que debe esperar.

**RN-131** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (P-46)
Patrón EARS: State-driven
Mientras una cuenta esté bloqueada por RN-004, el sistema NO DEBE sumar intentos fallidos ni extender el bloqueo.
✅
**Ejemplo:** el quinto fallo fue a las 10:00:00; Ana falla otra vez a las 10:05:00 y a las 10:15:01 puede entrar.
❌
**Contraejemplo:** el intento de las 10:05:00 reinicia el bloqueo hasta las 10:20:00.

**RN-132** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (P-47)
Patrón EARS: Ubiquitous
Toda petición que el emprendedor haga a la app con su sesión DEBE contar como actividad para el plazo de RN-097.
✅
**Ejemplo:** Ana abre la lista de productos a las 17:59:30; su sesión sigue abierta hasta las 01:59:30 del día siguiente.
❌
**Contraejemplo:** la sesión se cierra a las 01:59:00 porque esa consulta no se tomó en cuenta.

**RN-133** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (M-06)
Patrón EARS: Unwanted
Si un visitante envía el registro con un correo sin el formato `nombre@dominio.ext` o de más de 254 caracteres, entonces el sistema NO DEBE crear la cuenta y DEBE mostrar "Revisa el correo".
✅
**Ejemplo:** `ana@gmail.com` se acepta.
❌
**Contraejemplo:** se crea una cuenta con el correo `ana@gmail`.

**RN-134** | Categoría: Seguridad (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (M-06)
Patrón EARS: Unwanted
Si una contraseña nueva (en el registro o al cambiarla) tiene más de 128 caracteres, entonces el sistema NO DEBE aceptarla.
✅
**Ejemplo:** se acepta una contraseña de 128 caracteres.
❌
**Contraejemplo:** se acepta una contraseña de 5,000 caracteres.

**RN-141** | Categoría: Datos (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-43)
Patrón EARS: Optional
Donde el visitante escriba un nombre al registrarse, el sistema DEBE guardarlo con un máximo de 100 caracteres; el nombre es opcional y el emprendedor puede corregirlo en "Mi cuenta".
✅
**Ejemplo:** Ana se registra sin nombre y la app la saluda con su correo; luego escribe "Ana" en "Mi cuenta" y se guarda.
❌
**Contraejemplo:** el registro exige un nombre, o acepta un nombre de 300 caracteres.

**RN-153** | Categoría: UX (F-01) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (UX del registro)
Patrón EARS: Event-driven
Cuando el visitante complete el registro con éxito, el sistema DEBE iniciar su sesión sin pedirle que escriba de nuevo su correo y contraseña.
✅
**Ejemplo:** Ana se registra con `ana@gmail.com` y llega directo a su lista de productos vacía.
❌
**Contraejemplo:** tras registrarse, Ana ve el formulario de inicio de sesión y debe escribir otra vez sus datos.

---

## Catálogo de productos (F-02)

**RN-008** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Alcance (F-02)
Patrón EARS: Ubiquitous
Cada producto DEBE tener exactamente una categoría entre Ropa, Perfume y Accesorio.
✅
**Ejemplo:** "Polo oversize básico" tiene la categoría Ropa.
❌
**Contraejemplo:** "Polo oversize básico" se guarda sin categoría o con las categorías Ropa y Accesorio a la vez.

**RN-119** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
El sistema NO DEBE permitir cambiar la categoría de un producto después de crearlo.
✅
**Ejemplo:** Ana creó "Polo oversize básico" como Accesorio por error; al editarlo ve la categoría sin poder cambiarla, desactiva el producto y crea uno nuevo como Ropa. El historial del producto desactivado se conserva (RN-017).
❌
**Contraejemplo:** Ana cambia la categoría de "Polo oversize básico" de Accesorio a Ropa y sus variantes conservan SKU `ACC-00001` y sin talla ni color.

**RN-058** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
Cada producto DEBE tener al menos una variante.
✅
**Ejemplo:** "Polo oversize básico" tiene las variantes talla M negro y talla L negro.
❌
**Contraejemplo:** se guarda "Polo oversize básico" sin ninguna variante.

**RN-059** | Categoría: UX (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor guarde un producto de categoría Accesorio sin variantes, el sistema DEBE crear automáticamente una variante llamada "Única".
✅
**Ejemplo:** se guarda "Collar dorado" sin variantes y queda con la variante "Única", lista para registrar entradas y ventas.
❌
**Contraejemplo:** el sistema rechaza el guardado de "Collar dorado" con el mensaje "Debe crear al menos una variante".

**RN-060** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Optional
Donde una variante pertenezca a un producto de categoría Accesorio, el sistema DEBE permitir guardarla sin talla y sin color.
✅
**Ejemplo:** la variante de "Cartera negra" se guarda sin talla y sin color.
❌
**Contraejemplo:** el sistema exige una talla para guardar la variante de "Cartera negra".

**RN-009** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Brief (segmento Ropa)
Patrón EARS: Ubiquitous
Cada variante de un producto de categoría Ropa DEBE tener una talla.
✅
**Ejemplo:** la variante de "Polo oversize básico" tiene talla `M`.
❌
**Contraejemplo:** se guarda una variante de "Polo oversize básico" sin talla.

**RN-010** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Brief (segmento Ropa)
Patrón EARS: Ubiquitous
Cada variante de un producto de categoría Ropa DEBE tener un color.
✅
**Ejemplo:** la variante talla `M` de "Polo oversize básico" tiene color `Negro`.
❌
**Contraejemplo:** se guarda la variante talla `M` sin color.

**RN-111** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando el emprendedor guarde una variante con talla, el sistema DEBE guardar la talla como texto en mayúsculas y sin espacios al inicio ni al final.
✅
**Ejemplo:** el emprendedor escribe ` xl ` y la talla se guarda como `XL`.
❌
**Contraejemplo:** existen dos tallas distintas, `xl` y `XL`, para el mismo producto.

**RN-112** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
Una variante de un producto de categoría Ropa NO DEBE tener presentación en mililitros ni fecha de vencimiento.
✅
**Ejemplo:** la variante talla `M` color `Negro` de "Polo oversize básico" se guarda sin presentación.
❌
**Contraejemplo:** se guarda una variante de "Polo oversize básico" con presentación `100` ml.

**RN-113** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
Una variante de un producto de categoría Perfume NO DEBE tener talla ni color.
✅
**Ejemplo:** la variante de 100 ml de "Perfume Floral Rosé" se guarda sin talla y sin color.
❌
**Contraejemplo:** se guarda una variante de "Perfume Floral Rosé" con talla `M`.

**RN-121** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando el emprendedor guarde una variante con color, el sistema DEBE guardar el color con la primera letra en mayúscula, el resto en minúsculas y sin espacios al inicio ni al final.
✅
**Ejemplo:** el emprendedor escribe ` AZUL marino ` y el color se guarda como `Azul marino`.
❌
**Contraejemplo:** existen dos colores distintos, `Negro` y `negro`, para el mismo producto.

**RN-122** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
La talla de una variante NO DEBE tener más de 10 caracteres.
✅
**Ejemplo:** se guarda la talla `XXL`.
❌
**Contraejemplo:** se guarda una talla de 40 caracteres.

**RN-123** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
El color de una variante NO DEBE tener más de 30 caracteres.
✅
**Ejemplo:** se guarda el color `Azul marino`.
❌
**Contraejemplo:** se guarda un color de 80 caracteres.

**RN-124** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
Una variante de un producto de categoría Accesorio NO DEBE tener presentación en mililitros ni fecha de vencimiento.
✅
**Ejemplo:** la variante "Única" de "Collar dorado" se guarda sin presentación y sin vencimiento.
❌
**Contraejemplo:** se guarda una variante de "Collar dorado" con presentación `50` ml.

**RN-011** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Brief (segmento Perfumes)
Patrón EARS: Ubiquitous
Cada variante de un producto de categoría Perfume DEBE tener una presentación en mililitros expresada como número entero mayor que 0.
✅
**Ejemplo:** la variante de "Perfume Floral Rosé" tiene presentación `100` ml.
❌
**Contraejemplo:** la variante se guarda con presentación `0` ml o `-50` ml.

**RN-012** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Optional
Donde una variante de un producto de categoría Perfume tenga fecha de vencimiento, esa fecha DEBE estar en formato AAAA-MM-DD.
✅
**Ejemplo:** la variante de 100 ml de "Perfume Floral Rosé" se guarda con vencimiento `2028-06-30`; otra variante sin fecha impresa se guarda sin vencimiento.
❌
**Contraejemplo:** se guarda la fecha de vencimiento como `30/06/28`.

**RN-115** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando el emprendedor modifique la fecha de vencimiento de una variante que ya tiene entradas, ventas o ajustes, el sistema DEBE registrar en una auditoría la fecha y hora del cambio, el valor anterior y el valor nuevo.
✅
**Ejemplo:** Ana corrige el vencimiento de `2028-06-30` a `2027-06-30` por un error de registro; queda registrado el cambio con su fecha y hora y los dos valores.
❌
**Contraejemplo:** Ana cambia el vencimiento y no queda ningún rastro del valor anterior.

**RN-013** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El precio de venta de cada variante DEBE ser mayor que S/ 0.00.
✅
**Ejemplo:** el polo talla M negro tiene precio de venta S/ 35.00.
❌
**Contraejemplo:** se guarda un precio de venta de S/ 0.00.

**RN-041** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El precio de venta de cada variante NO DEBE tener más de 2 decimales.
✅
**Ejemplo:** se guarda un precio de venta de S/ 35.90.
❌
**Contraejemplo:** se guarda un precio de venta de S/ 35.999.

**RN-091** | Categoría: Datos (F-02, F-03, F-04) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
Todo precio de venta, costo unitario y precio final NO DEBE superar S/ 99,999.99.
✅
**Ejemplo:** se registra un perfume con precio S/ 450.00.
❌
**Contraejemplo:** se registra un precio de S/ 150,000.00 por error de tipeo.

**RN-092** | Categoría: Datos (F-03, F-04, F-05) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
Toda cantidad de entrada, línea de venta o ajuste NO DEBE superar 9,999 unidades.
✅
**Ejemplo:** se registra una entrada de 120 polos.
❌
**Contraejemplo:** se registra una entrada de 12,000 polos por un cero de más.

**RN-093** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El nombre de un producto NO DEBE superar 100 caracteres.
✅
**Ejemplo:** "Polo oversize básico algodón pima" (33 caracteres) se guarda.
❌
**Contraejemplo:** se guarda un nombre de 250 caracteres.

**RN-014** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El costo unitario de cada entrada de mercadería DEBE ser mayor que S/ 0.00.
✅
**Ejemplo:** se registra una entrada de polos con costo unitario S/ 18.50.
❌
**Contraejemplo:** se registra una entrada con costo unitario S/ -5.00.

**RN-042** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El costo unitario de cada entrada de mercadería NO DEBE tener más de 2 decimales.
✅
**Ejemplo:** se registra una entrada con costo unitario S/ 18.50.
❌
**Contraejemplo:** se registra una entrada con costo unitario S/ 18.505.

**RN-015** | Categoría: UX (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si el emprendedor guarda un precio de venta menor que el costo unitario promedio de la variante, entonces el sistema DEBE mostrar la advertencia "El precio de venta es menor que el costo".
✅
**Ejemplo:** costo promedio S/ 18.50, precio de venta S/ 15.00: el sistema guarda el precio y muestra la advertencia.
❌
**Contraejemplo:** costo promedio S/ 18.50, precio de venta S/ 15.00: el sistema guarda el precio sin ninguna advertencia.

**RN-016** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras un producto esté desactivado, el sistema NO DEBE permitir registrar ventas de sus variantes.
✅
**Ejemplo:** "Collar dorado" está desactivado y no aparece en la lista para registrar una venta.
❌
**Contraejemplo:** se registra una venta de 1 unidad de "Collar dorado" estando desactivado.

**RN-017** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor desactive un producto, el sistema DEBE conservar el historial (entradas, ventas y ajustes) de ese producto.
✅
**Ejemplo:** "Collar dorado" tuvo 4 ventas en octubre 2026; tras desactivarlo, esas 4 ventas siguen en el historial.
❌
**Contraejemplo:** al desactivar "Collar dorado" desaparecen sus 4 ventas del historial.

**RN-098** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si una variante tiene entradas, ventas o ajustes, entonces el sistema NO DEBE permitir modificar su talla, color ni presentación.
✅
**Ejemplo:** la variante talla M negro tiene ventas; Ana solo puede cambiar su precio de venta.
❌
**Contraejemplo:** Ana cambia la talla M a L y las ventas pasadas quedan como talla L.

**RN-099** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si una variante tiene entradas, ventas o ajustes, entonces el sistema NO DEBE permitir eliminarla.
✅
**Ejemplo:** Ana intenta eliminar la variante con ventas y el sistema le ofrece desactivar el producto.
❌
**Contraejemplo:** la variante se elimina y sus ventas quedan sin variante.

**RN-100** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras un producto esté desactivado, el sistema DEBE permitir al emprendedor reactivarlo.
✅
**Ejemplo:** Ana reactiva "Collar dorado" y vuelve a aparecer para registrar ventas con su historial intacto.
❌
**Contraejemplo:** para volver a vender "Collar dorado" Ana debe crearlo de nuevo.

**RN-050** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si el emprendedor intenta crear o modificar una variante dejándola con los mismos atributos (talla y color, o presentación y fecha de vencimiento) que otra variante del mismo producto, entonces el sistema NO DEBE guardarla.
✅
**Ejemplo:** "Polo oversize básico" tiene la variante talla M negro; se crea talla M blanco. Un perfume de 100 ml que vence el 2028-06-30 y otro de 100 ml que vence el 2029-01-31 son dos variantes distintas del mismo producto.
❌
**Contraejemplo:** se crea una segunda variante talla M negro en "Polo oversize básico", o una segunda variante "Única" en el mismo accesorio.

**RN-135** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (M-12)
Patrón EARS: Unwanted
Si el emprendedor intenta crear una variante de perfume con la misma presentación que otra variante del mismo producto y ambas sin fecha de vencimiento, entonces el sistema NO DEBE guardarla (RN-050).
✅
**Ejemplo:** "Perfume Floral Rosé" tiene 100 ml sin vencimiento; se crea 50 ml sin vencimiento y se guarda.
❌
**Contraejemplo:** se guardan dos variantes de 100 ml sin vencimiento en el mismo perfume.

**RN-136** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (M-01)
Patrón EARS: Unwanted
Si el emprendedor intenta eliminar la única variante de un producto, entonces el sistema NO DEBE eliminarla y DEBE ofrecerle desactivar el producto.
✅
**Ejemplo:** "Collar dorado" solo tiene la variante "Única" sin movimientos; Ana intenta eliminarla y ve "Un producto necesita al menos una variante. Puedes desactivarlo".
❌
**Contraejemplo:** el producto queda sin variantes (RN-058).

**RN-137** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (M-01, P-41)
Patrón EARS: Ubiquitous
El sistema DEBE permitir desactivar un producto aunque sus variantes tengan stock; el stock se conserva y vuelve a estar disponible al reactivarlo.
✅
**Ejemplo:** "Cartera negra" tiene stock 3; Ana la desactiva y al reactivarla sigue con stock 3.
❌
**Contraejemplo:** el sistema obliga a ajustar el stock a 0 antes de desactivar.

**RN-146** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-57)
Patrón EARS: Ubiquitous
La presentación en mililitros de una variante de perfume NO DEBE superar 9,999.
✅
**Ejemplo:** se guarda una variante de 100 ml; también una de 9,999 ml.
❌
**Contraejemplo:** se guarda una presentación de 100,000 ml por error de tipeo.

**RN-147** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-41)
Patrón EARS: State-driven
Mientras un producto esté desactivado, el sistema DEBE permitir anular sus ventas y entradas dentro del plazo de RN-087 y registrar ajustes por conteo físico de sus variantes.
✅
**Ejemplo:** "Collar dorado" está desactivado; Ana anula una venta de hace 2 horas y el stock vuelve a subir.
❌
**Contraejemplo:** Ana no puede corregir una venta de hace 2 horas porque el producto está desactivado.

---

## Entradas de mercadería (F-03)

**RN-018** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
La cantidad de cada entrada de mercadería DEBE ser un número entero mayor o igual que 1.
✅
**Ejemplo:** se registra una entrada de 12 polos talla M negro.
❌
**Contraejemplo:** se registra una entrada de 0 unidades o de 2.5 unidades.

**RN-019** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Alcance (F-03)
Patrón EARS: Event-driven
Cuando el emprendedor registre una entrada de N unidades de una variante, el sistema DEBE aumentar en N el stock de esa variante.
✅
**Ejemplo:** stock 3, entrada de 12 unidades: el stock queda en 15.
❌
**Contraejemplo:** stock 3, entrada de 12 unidades: el stock queda en 12.

**RN-061** | Categoría: Datos (F-02, F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor cree una variante, el sistema DEBE crearla con stock 0.
✅
**Ejemplo:** se crea "Polo oversize básico" talla M negro con stock 0 y luego se registra una entrada de 12 unidades a S/ 18.50: el stock queda en 12.
❌
**Contraejemplo:** al crear la variante se ingresa directamente stock 12 sin registrar una entrada ni su costo.

**RN-064** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor cree una variante, el sistema DEBE asignarle automáticamente un SKU.
✅
**Ejemplo:** se crea "Polo oversize básico" talla M negro y recibe el SKU `ROP-00001` sin que el emprendedor lo escriba.
❌
**Contraejemplo:** la variante se guarda sin SKU o el sistema pide al emprendedor que lo escriba.

**RN-065** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El SKU DEBE tener el formato `PPP-NNNNN`, donde `PPP` es `ROP`, `PER` o `ACC` según la categoría y `NNNNN` es un correlativo de 5 dígitos por emprendedor y categoría.
✅
**Ejemplo:** el tercer perfume de Ana recibe `PER-00003`; el primer perfume de Luis recibe `PER-00001`.
❌
**Contraejemplo:** una variante de perfume de Ana recibe el SKU `PERF-3`.

**RN-108** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Separada de RN-065 (revisión QA 2026-11-10)
Patrón EARS: Ubiquitous
El SKU de una variante NO DEBE repetirse entre las variantes de un mismo emprendedor ni reasignarse a otra variante, aunque la variante original se elimine.
✅
**Ejemplo:** Ana y Luis tienen cada uno una variante `PER-00001`; es válido porque son emprendedores distintos.
❌
**Contraejemplo:** dos variantes de Ana tienen el SKU `PER-00003`, o Ana elimina `PER-00003` (sin movimientos) y la siguiente variante de perfume vuelve a recibir `PER-00003` en lugar de `PER-00004`.

**RN-066** | Categoría: Datos (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El SKU de una variante NO DEBE cambiar después de asignado.
✅
**Ejemplo:** Ana cambia el precio de `ROP-00001` de S/ 35.00 a S/ 32.00 y el SKU sigue siendo `ROP-00001`.
❌
**Contraejemplo:** al cambiar el precio, el SKU pasa a `ROP-00007`.

**RN-062** | Categoría: UX (F-02) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras una variante no tenga entradas no anuladas, el sistema NO DEBE mostrar la advertencia de precio menor que el costo (RN-015).
✅
**Ejemplo:** una variante nueva con precio S/ 35.00 y sin entradas se guarda sin advertencia.
❌
**Contraejemplo:** la variante nueva muestra "El precio de venta es menor que el costo" porque toma el costo como S/ 0.00 o indefinido.

**RN-020** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor registre una entrada, el sistema DEBE recalcular el costo unitario promedio de la variante como (stock anterior x costo promedio anterior + unidades nuevas x costo nuevo) / stock resultante, redondeado a 2 decimales.
✅
**Ejemplo:** stock 10 a S/ 18.00 y entrada de 10 a S/ 20.00: el costo promedio queda en S/ 19.00.
❌
**Contraejemplo:** en el mismo caso, el costo promedio queda en S/ 20.00 (se toma solo el último costo).

**RN-047** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El redondeo del costo unitario promedio DEBE usar el modo "mitad hacia arriba" (HALF_UP) a 2 decimales.
✅
**Ejemplo:** (10 x S/ 18.00 + 5 x S/ 20.01) / 15 = S/ 18.67 (18.6700); (1 x S/ 10.00 + 1 x S/ 10.01) / 2 = 10.005, que se redondea a S/ 10.01.
❌
**Contraejemplo:** 10.005 se redondea a S/ 10.00 (truncado).

**RN-089** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
Un ajuste por conteo físico NO DEBE modificar el costo unitario promedio de la variante.
✅
**Ejemplo:** costo promedio S/ 19.00; un ajuste de +2 unidades deja el promedio en S/ 19.00.
❌
**Contraejemplo:** tras el ajuste de +2 el promedio baja como si las unidades costaran S/ 0.00.

**RN-088** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando el emprendedor anule una entrada, el sistema DEBE restaurar el costo unitario promedio que tenía la variante justo antes de registrar esa entrada.
✅
**Ejemplo:** la variante tiene stock 10 y costo promedio S/ 18.00; una entrada de 10 unidades a S/ 20.00 lleva el promedio a S/ 19.00; se anula esa entrada: el promedio vuelve a S/ 18.00.
❌
**Contraejemplo:** tras anular la entrada el promedio sigue en S/ 19.00, o se recalcula con una fórmula que da S/ 18.01 por redondeo.

**RN-118** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Ubiquitous
El sistema DEBE guardar con cada entrada el costo unitario promedio que la variante tenía justo antes de registrarla.
✅
**Ejemplo:** la entrada de 10 unidades a S/ 20.00 guarda "costo promedio anterior: S/ 18.00".
❌
**Contraejemplo:** la entrada no guarda el costo promedio anterior y no se puede restaurar al anularla.
Nota (auditoría QA v4): si la variante no tenía costo promedio antes de la entrada (no había entradas vigentes), la entrada guarda el costo promedio anterior vacío (sin valor), no S/ 0.00 (ver RN-154).

**RN-154** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Propuesta QA (auditoría QA v4, 2026-11-11), pendiente de aprobación del Responsable del producto (P-64)
Patrón EARS: Event-driven
Cuando el emprendedor anule una entrada y la variante no tenga otras entradas vigentes, el sistema DEBE dejar la variante sin costo unitario promedio, igual que una variante sin entradas (RN-062).
✅
**Ejemplo:** una variante nueva recibe 12 unidades a S/ 18.50; Ana anula esa entrada; la variante queda con stock 0 y muestra "Costo promedio: sin entradas".
❌
**Contraejemplo:** tras anular, la variante muestra "Costo promedio S/ 0.00" y una venta posterior calcula su ganancia con S/ 0.00.

**RN-140** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-41)
Patrón EARS: State-driven
Mientras un producto esté desactivado, el sistema NO DEBE permitir registrar entradas de sus variantes y DEBE mostrar "Reactiva el producto para registrar mercadería".
✅
**Ejemplo:** "Collar dorado" está desactivado; Ana intenta registrar una entrada de 5 unidades y ve el mensaje; reactiva el producto y la registra.
❌
**Contraejemplo:** se registra una entrada de 5 unidades en un producto desactivado.

**RN-142** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-50)
Patrón EARS: Ubiquitous
Cada entrada DEBE corresponder a una sola variante y guardar como fecha y hora de la entrada el momento de su registro, sin permitir elegir otra fecha.
✅
**Ejemplo:** Ana registra 10 unidades de polo talla M negro; la entrada queda con la fecha y hora del registro, y para otra variante registra otra entrada.
❌
**Contraejemplo:** una entrada agrupa dos variantes, o Ana elige la fecha 2026-11-01 a mano.

---

## Salidas por venta (F-04)

**RN-021** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
La cantidad de cada línea de venta DEBE ser un número entero mayor o igual que 1.
✅
**Ejemplo:** se registra una venta de 2 perfumes de 100 ml.
❌
**Contraejemplo:** se registra una venta de 0 unidades.

**RN-080** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
Cada venta DEBE tener al menos una línea de venta.
✅
**Ejemplo:** una venta al mismo cliente tiene 4 líneas: polo A verde talla L (1), polo A marrón talla XS (1), perfume A 200 ml (1) y perfume A 100 ml (1).
❌
**Contraejemplo:** se registra una venta sin líneas.

**RN-107** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Separada de RN-080 (revisión QA 2026-11-10)
Patrón EARS: Ubiquitous
Cada línea de venta DEBE corresponder a una sola variante.
✅
**Ejemplo:** la línea "Perfume A 100 ml x2" corresponde solo a la variante de 100 ml.
❌
**Contraejemplo:** una sola línea agrupa el perfume A de 100 ml y el de 200 ml.

**RN-084** | Categoría: UX (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor agregue a una venta una variante que ya está en otra línea de la misma venta con el mismo precio final por unidad, el sistema DEBE sumar la cantidad a la línea existente.
✅
**Ejemplo:** se agrega dos veces el perfume A 100 ml: la venta muestra una línea "Perfume A 100 ml x2".
❌
**Contraejemplo:** la venta queda con dos líneas de 1 unidad del perfume A 100 ml.

**RN-125** | Categoría: UX (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Unwanted
Si el emprendedor agrega a una venta una variante que ya está en otra línea de la misma venta con un precio final por unidad distinto, entonces el sistema NO DEBE agregarla ni modificar la línea existente.
✅
**Ejemplo:** la venta tiene "Polo A talla M x1" a S/ 30.00; el emprendedor intenta agregar otro a S/ 35.00 y el sistema muestra "Esta variante ya está en la venta con otro precio. Edita esa línea".
❌
**Contraejemplo:** la venta toma S/ 35.00 para las dos unidades sin avisar, o conserva S/ 30.00 sin avisar.

**RN-081** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: State-driven
Mientras alguna línea de una venta no pueda registrarse (por stock insuficiente o perfume vencido), el sistema NO DEBE registrar la venta.
✅
**Ejemplo:** la venta tiene 4 líneas y el polo marrón XS no tiene stock: el botón "Registrar venta" no confirma y se marca esa línea con "Stock insuficiente".
❌
**Contraejemplo:** se registran 3 de las 4 líneas y el polo marrón XS queda fuera sin aviso.

**RN-085** | Categoría: UX (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando una línea de una venta no pueda registrarse, el sistema DEBE permitir al emprendedor quitar esa línea y registrar el resto de la venta.
✅
**Ejemplo:** el emprendedor quita la línea del polo marrón XS y registra la venta con las otras 3 líneas.
❌
**Contraejemplo:** el sistema obliga a borrar toda la venta y empezar de nuevo.

**RN-082** | Categoría: Datos (F-06) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
La ganancia de una venta DEBE ser la suma de las ganancias de sus líneas.
✅
**Ejemplo:** líneas con ganancias S/ 16.00, S/ 12.00, S/ 40.00 y S/ 25.00: la ganancia de la venta es S/ 93.00.
❌
**Contraejemplo:** la ganancia de la venta muestra solo la de la primera línea, S/ 16.00.

**RN-083** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor anule una venta, el sistema DEBE anular todas sus líneas.
✅
**Ejemplo:** se anula la venta de 4 líneas: las 4 variantes recuperan su stock y la ganancia de S/ 93.00 se descuenta.
❌
**Contraejemplo:** se anula la venta y solo la primera línea devuelve su stock.

**RN-086** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
El sistema DEBE permitir anular una sola línea de una venta sin afectar las demás líneas de esa venta. Su efecto en el stock y la ganancia está en RN-052 y RN-053.
✅
**Ejemplo:** en una venta de 4 líneas se anula el polo A verde talla L registrado por error: ese polo vuelve al stock y las otras 3 líneas siguen vigentes.
❌
**Contraejemplo:** para corregir el polo hay que anular la venta completa y registrar de nuevo las 4 líneas.

**RN-022** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Alcance (F-04)
Patrón EARS: Event-driven
Cuando el emprendedor registre una venta, el sistema DEBE disminuir el stock de la variante de cada línea en la cantidad de esa línea.
✅
**Ejemplo:** stock 15, venta de 2 unidades: el stock queda en 13.
❌
**Contraejemplo:** stock 15, venta de 2 unidades: el stock sigue en 15.

**RN-023** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Alcance (F-04)
Patrón EARS: Unwanted
Si una línea de venta tiene una cantidad mayor que el stock actual de su variante, entonces el sistema NO DEBE registrar la venta (RN-081).
✅
**Ejemplo:** stock 2, venta de 3 unidades: el sistema rechaza la venta con el mensaje "Stock insuficiente: quedan 2 unidades".
❌
**Contraejemplo:** stock 2, venta de 3 unidades: la venta se registra y el stock queda en -1.

**RN-074** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Unwanted
Si el emprendedor intenta registrar una venta de una variante de perfume cuya fecha de vencimiento es anterior a la fecha actual en hora de Lima, entonces el sistema NO DEBE registrar la venta.
✅
**Ejemplo:** hoy es 2026-11-10 y el perfume vence el 2026-11-09: el sistema rechaza la venta con el mensaje "Este perfume está vencido y no se puede vender". Si vence el 2026-11-10, la venta se registra.
❌
**Contraejemplo:** hoy es 2026-11-10, el perfume vence el 2026-11-09 y la venta se registra.

**RN-076** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10; precisada en la auditoría QA v2)
Patrón EARS: Ubiquitous
El sistema DEBE permitir registrar una venta con una fecha y hora de venta anteriores al momento actual, elegidas por el emprendedor (día y hora aproximada), dentro del límite de RN-077.
✅
**Ejemplo:** el 2026-11-10 Ana registra una venta olvidada del live del 2026-11-09 eligiendo ese día y la hora aproximada 21:00; queda con fecha y hora 2026-11-09T21:00-05:00.
❌
**Contraejemplo:** el sistema obliga a registrarla con la fecha 2026-11-10.

**RN-077** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10; reescrita en días calendario en la auditoría QA v2)
Patrón EARS: Unwanted
Si el día de la fecha de venta indicada es anterior a hoy menos 7 días calendario, en hora de Lima, entonces el sistema NO DEBE registrar la venta.
✅
**Ejemplo:** el 2026-11-10 se registra una venta con fecha 2026-11-03 a cualquier hora: se acepta.
❌
**Contraejemplo:** el 2026-11-10 se registra una venta con fecha 2026-11-02 o 2026-10-01.

**RN-078** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si la fecha y hora de venta indicadas son posteriores al momento actual, entonces el sistema NO DEBE registrar la venta.
✅
**Ejemplo:** el 2026-11-10 a las 10:00 se registra una venta con fecha y hora 2026-11-10 09:30: se acepta.
❌
**Contraejemplo:** el 2026-11-10 a las 10:00 se registra una venta con fecha 2026-11-12, o con fecha 2026-11-10 a las 18:00.

**RN-079** | Categoría: Datos (F-06) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
La ganancia de una venta con fecha pasada DEBE calcularse con el costo unitario promedio vigente en el momento en que se registra, no en la fecha indicada.
✅
**Ejemplo:** el 2026-11-09 el costo promedio era S/ 18.00 y el 2026-11-10 es S/ 19.00; una venta registrada el 2026-11-10 con fecha 2026-11-09 usa S/ 19.00.
❌
**Contraejemplo:** el sistema recalcula el costo promedio histórico del 2026-11-09 y modifica ventas ya registradas.

**RN-045** | Categoría: Datos (F-03, F-04) | Prioridad: MUST | Fuente: Alcance (F-04), derivada de RN-023 y RN-056 (reescrita en revisión QA 2026-11-10)
Patrón EARS: Ubiquitous
El stock de una variante NO DEBE quedar por debajo de 0 tras ninguna operación, incluso cuando varias ventas o anulaciones de entradas de esa variante se procesan a la vez.
✅
**Ejemplo:** stock 1; desde el celular y la laptop se registra a la vez 1 unidad: una venta se registra y la otra recibe "Stock insuficiente: quedan 0 unidades".
❌
**Contraejemplo:** en el mismo caso se registran las dos ventas y el stock queda en -1.

**RN-024** | Categoría: Datos (F-04, F-05) | Prioridad: MUST | Fuente: Brief, Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
El sistema NO DEBE modificar el stock de una variante por un motivo distinto a una entrada, una venta, un ajuste por conteo físico o la anulación de una entrada o venta.
✅
**Ejemplo:** el stock baja de 15 a 13 solo al registrar la venta de 2 unidades.
❌
**Contraejemplo:** el emprendedor edita el número de stock de 15 a 13 en la ficha del producto sin registrar una venta ni un ajuste.

**RN-025** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
El precio final por unidad de cada línea de venta DEBE ser mayor que S/ 0.00.
✅
**Ejemplo:** se registra una venta de 2 polos con precio final S/ 30.00 por unidad.
❌
**Contraejemplo:** se registra una venta con precio final S/ 0.00.

**RN-106** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10, propuesta QA), basada en las convenciones de este documento (montos con 2 decimales)
Patrón EARS: Ubiquitous
El precio final por unidad de cada línea de venta NO DEBE tener más de 2 decimales.
✅
**Ejemplo:** se registra una línea con precio final S/ 29.90.
❌
**Contraejemplo:** se registra una línea con precio final S/ 29.995.

**RN-043** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
El sistema DEBE permitir registrar una línea de venta con un precio final por unidad distinto del precio de venta de la variante.
✅
**Ejemplo:** el polo tiene precio de venta S/ 35.00 y en un live se vende a S/ 30.00 por unidad; la venta se registra con S/ 30.00.
❌
**Contraejemplo:** el sistema reemplaza los S/ 30.00 ingresados por S/ 35.00.

**RN-126** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (P-39)
Patrón EARS: Event-driven
Cuando el emprendedor registre una venta sin cambiar la fecha y hora de venta, el sistema DEBE guardar como fecha y hora de venta el momento del registro en hora de Lima.
✅
**Ejemplo:** Ana registra una venta el 2026-11-10 a las 21:30:12 sin tocar la fecha; la venta queda con 2026-11-10T21:30:12-05:00.
❌
**Contraejemplo:** el sistema exige que Ana escriba la fecha y la hora en cada venta.

**RN-151** | Categoría: UX (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (B-08)
Patrón EARS: Unwanted
Si al fusionar una línea (RN-084) la cantidad resultante supera 9,999 unidades, entonces el sistema NO DEBE agregar la cantidad y DEBE mostrar "La cantidad máxima por línea es 9,999".
✅
**Ejemplo:** la línea tiene 9,990 unidades; Ana agrega 20 más y ve el mensaje; la línea sigue en 9,990.
❌
**Contraejemplo:** la línea queda con 10,010 unidades.

---

## Stock y ganancia (F-05, F-06)

**RN-026** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Alcance (F-05)
Patrón EARS: Ubiquitous
El stock actual de cada variante DEBE ser igual a la suma de unidades de sus entradas no anuladas, menos la suma de unidades de sus ventas no anuladas, más la suma de las diferencias de sus ajustes por conteo físico.
✅
**Ejemplo:** entradas de 12 y 6 unidades, ventas de 2, 3 y 1 unidades y un ajuste de -1: el stock mostrado es 11.
❌
**Contraejemplo:** en el mismo caso, el stock mostrado es 12 (se ignora el ajuste).

**RN-027** | Categoría: Datos (F-06) | Prioridad: MUST | Fuente: Alcance (F-06), Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor registre una venta, el sistema DEBE calcular su ganancia como (precio final por unidad - costo unitario promedio de la variante en ese momento) x cantidad.
✅
**Ejemplo:** 2 unidades a S/ 35.00 por unidad con costo promedio S/ 19.00: ganancia S/ 32.00.
❌
**Contraejemplo:** en el mismo caso, la ganancia mostrada es S/ 70.00 (no se resta el costo).

**RN-072** | Categoría: Datos (F-06) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
El sistema DEBE permitir registrar una venta cuya ganancia sea negativa.
✅
**Ejemplo:** un polo con costo promedio S/ 19.00 se remata a S/ 15.00: la venta se registra con ganancia S/ -4.00 y la ganancia del día baja en S/ 4.00.
❌
**Contraejemplo:** el sistema rechaza la venta porque el precio final es menor que el costo.

**RN-073** | Categoría: UX (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Unwanted
Si una o más líneas de una venta tienen un precio final por unidad menor que el costo unitario promedio de su variante, entonces el sistema DEBE pedir una sola confirmación antes de registrarla, mostrando "Esta venta genera una pérdida de S/ X", donde X es la suma de las pérdidas de esas líneas.
✅
**Ejemplo:** remate de 1 polo a S/ 15.00 con costo S/ 19.00: el sistema muestra "Esta venta genera una pérdida de S/ 4.00" y la registra al confirmar.
Con dos líneas que pierden S/ 4.00 y S/ 1.50, el sistema pide una sola confirmación con "Esta venta genera una pérdida de S/ 5.50".
❌
**Contraejemplo:** el polo se registra a S/ 3.50 por error de tipeo sin ningún aviso, o el sistema pide una confirmación por cada línea.

**RN-028** | Categoría: Datos (F-06) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El sistema NO DEBE modificar la ganancia de una venta ya registrada cuando cambie el costo unitario promedio de la variante.
✅
**Ejemplo:** la venta del 2026-11-10 tuvo ganancia S/ 32.00; el 2026-11-15 entra mercadería más cara y esa venta sigue mostrando S/ 32.00.
❌
**Contraejemplo:** tras la entrada del 2026-11-15, la venta del 2026-11-10 pasa a mostrar S/ 28.00.

---

## Ajuste por conteo físico (F-05)

**RN-037** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor registre un ajuste por conteo físico con la cantidad contada de una variante, el sistema DEBE dejar el stock de esa variante igual a la cantidad contada.
✅
**Ejemplo:** el stock en la app es 8, el conteo físico da 6 y tras el ajuste el stock queda en 6, con una diferencia registrada de -2.
❌
**Contraejemplo:** el stock en la app es 8, el conteo da 6 y tras el ajuste el stock queda en 2 (se restó la cantidad contada).

**RN-038** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Unwanted
Si el emprendedor intenta registrar un ajuste por conteo físico sin motivo, entonces el sistema NO DEBE registrar el ajuste.
✅
**Ejemplo:** el ajuste se guarda con el motivo "Conteo físico: venta del live del 2026-11-09 no registrada".
❌
**Contraejemplo:** el ajuste se guarda con el campo de motivo vacío.

**RN-075** | Categoría: UX (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Optional
Donde el ajuste por conteo físico sea de una variante de perfume, el sistema DEBE ofrecer el motivo predefinido "Merma por vencimiento".
✅
**Ejemplo:** Ana ajusta de 4 a 1 unidad el perfume de 100 ml vencido el 2026-11-09 y elige el motivo "Merma por vencimiento"; queda en el historial con ese motivo.
❌
**Contraejemplo:** Ana solo puede escribir el motivo a mano y registra "vencido", "venc." y "merma" en tres ajustes distintos, sin forma de agruparlos.

**RN-039** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
Un ajuste por conteo físico NO DEBE generar una venta.
✅
**Ejemplo:** un ajuste de -2 unidades no aparece en el historial de ventas.
❌
**Contraejemplo:** un ajuste de -2 unidades aparece como venta de 2 unidades.

**RN-044** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
Un ajuste por conteo físico NO DEBE modificar la ganancia de ningún periodo.
✅
**Ejemplo:** tras un ajuste de -2 unidades el 2026-11-10, la ganancia de ese día sigue en S/ 32.00.
❌
**Contraejemplo:** tras el ajuste, la ganancia del día baja a S/ -38.00 por el costo de las 2 unidades.

**RN-040** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
La cantidad contada de un ajuste por conteo físico DEBE ser un número entero mayor o igual que 0.
✅
**Ejemplo:** el conteo físico da 0 unidades y el ajuste se registra.
❌
**Contraejemplo:** se registra un ajuste con cantidad contada -3.

**RN-116** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Unwanted
Si el emprendedor intenta registrar un ajuste por conteo físico cuya cantidad contada es mayor que el stock actual de una variante sin entradas vigentes, entonces el sistema NO DEBE registrar el ajuste.
✅
**Ejemplo:** la variante nueva tiene stock 0 y sin entradas; Ana ingresa una cantidad contada de 3: el sistema rechaza el ajuste con el mensaje "Registra primero una entrada con su costo". Con una entrada vigente de 10 unidades y stock 8, un conteo de 9 se acepta.
❌
**Contraejemplo:** la variante sin entradas queda con stock 3 por un ajuste y no tiene costo unitario promedio.

**RN-138** | Categoría: UX (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (P-51)
Patrón EARS: Event-driven
Cuando el emprendedor busque una variante, el sistema DEBE buscar el texto en el nombre del producto, el SKU, la talla y el color, sin distinguir mayúsculas ni minúsculas.
✅
**Ejemplo:** Ana escribe "polo negro" o "rop-00001" y encuentra la variante talla M negro de "Polo oversize básico".
❌
**Contraejemplo:** la búsqueda "Polo" no encuentra "polo oversize básico".

**RN-143** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-56)
Patrón EARS: Unwanted
Si el motivo de un ajuste por conteo físico está vacío, tiene solo espacios o supera 200 caracteres (sin contar los espacios al inicio y al final), entonces el sistema NO DEBE registrar el ajuste (RN-038).
✅
**Ejemplo:** el motivo es "Conteo físico: venta del live no registrada" (45 caracteres) y el ajuste se guarda.
❌
**Contraejemplo:** el motivo es "   " (solo espacios) o tiene 500 caracteres y el ajuste se guarda.

**RN-144** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-56)
Patrón EARS: Ubiquitous
El sistema DEBE permitir registrar un ajuste por conteo físico cuya cantidad contada sea igual al stock actual (diferencia 0).
✅
**Ejemplo:** el stock es 8, Ana cuenta 8 y registra el ajuste con diferencia 0 para dejar constancia del conteo.
❌
**Contraejemplo:** el sistema rechaza el ajuste porque "no hay cambios".

**RN-145** | Categoría: Datos (F-05) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-56)
Patrón EARS: Ubiquitous
El sistema NO DEBE permitir anular un ajuste por conteo físico; el emprendedor lo corrige registrando otro ajuste.
✅
**Ejemplo:** Ana se equivocó al contar 6 en lugar de 8; registra un nuevo ajuste con cantidad contada 8 y ambos quedan en el historial.
❌
**Contraejemplo:** el historial ofrece un botón "Anular" en el ajuste.

---

## Anulación de ventas y entradas (F-03, F-04)

**RN-052** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor anule una venta o una línea de venta, el sistema DEBE aumentar el stock de la variante de cada línea anulada en la cantidad de esa línea.
✅
**Ejemplo:** stock 13; se anula una venta de 2 unidades registrada por error: el stock queda en 15.
❌
**Contraejemplo:** tras anular la venta el stock sigue en 13.

**RN-053** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Ubiquitous
Una venta o línea de venta anulada NO DEBE sumarse a la ganancia de ningún periodo.
✅
**Ejemplo:** la venta anulada tenía ganancia S/ 32.00; la ganancia del día baja de S/ 50.00 a S/ 18.00.
❌
**Contraejemplo:** la ganancia del día sigue en S/ 50.00 tras la anulación.

**RN-054** | Categoría: Datos (F-03, F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor anule una venta, una línea de venta o una entrada, el sistema DEBE conservarla en el historial con el estado "Anulada".
✅
**Ejemplo:** la venta del 2026-11-10 aparece en el historial con el estado "Anulada".
❌
**Contraejemplo:** la venta anulada desaparece del historial.

**RN-055** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10)
Patrón EARS: Event-driven
Cuando el emprendedor anule una entrada de N unidades, el sistema DEBE disminuir en N el stock de esa variante.
✅
**Ejemplo:** stock 15; se anula una entrada de 12 unidades: el stock queda en 3.
❌
**Contraejemplo:** tras anular la entrada el stock sigue en 15.

**RN-056** | Categoría: Datos (F-03) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Unwanted
Si la variante tiene entradas vigentes, ventas vigentes o ajustes por conteo físico registrados después de una entrada (según su hora de registro, no la fecha de venta), entonces el sistema NO DEBE anular esa entrada.
✅
**Ejemplo:** Ana registra una entrada de 12 unidades por error y de inmediato la anula: se acepta. Si después de esa entrada se registró una venta de 2 unidades, el sistema rechaza la anulación con el mensaje "No se puede anular: hay movimientos posteriores a esta entrada".
❌
**Contraejemplo:** se anula una entrada de 12 unidades aunque después se registró una venta de 2 unidades de esa variante.

**RN-057** | Categoría: Datos (F-03, F-04) | Prioridad: MUST | Fuente: Derivada de las reglas de anulación (RN-052 a RN-056)
Patrón EARS: Unwanted
Si el emprendedor intenta anular una venta, una línea de venta o una entrada que ya está anulada, entonces el sistema NO DEBE aplicar la anulación otra vez.
✅
**Ejemplo:** la venta ya está "Anulada" y el botón "Anular" no está disponible.
❌
**Contraejemplo:** se anula dos veces la misma venta de 2 unidades y el stock sube en 4.

**RN-087** | Categoría: Datos (F-03, F-04) | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si han pasado más de 86,400 segundos (24 horas) desde que se registró una venta, línea de venta o entrada, entonces el sistema NO DEBE permitir anularla.
✅
**Ejemplo:** la venta se registró el 2026-11-10T20:00-05:00 y se anula el 2026-11-11T09:00-05:00: se acepta.
❌
**Contraejemplo:** la venta se registró el 2026-11-01 y se anula el 2026-11-10.

**RN-109** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, revisión QA)
Patrón EARS: Event-driven
Cuando todas las líneas de una venta queden anuladas, el sistema DEBE cambiar el estado de la venta a "Anulada".
✅
**Ejemplo:** una venta tiene 2 líneas; Ana anula la primera y la venta sigue vigente; al anular la segunda, la venta pasa a "Anulada".
❌
**Contraejemplo:** se anulan las 2 líneas y la venta sigue vigente sin ninguna línea vigente.

**RN-139** | Categoría: Datos (F-04) | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, auditoría QA v2) (A-01)
Patrón EARS: Unwanted
Si alguna variante de una venta o línea de venta tiene un ajuste por conteo físico registrado después de esa venta, entonces el sistema NO DEBE anular esa venta o línea y DEBE mostrar "No se puede anular: hay un ajuste de stock posterior a esta venta".
✅
**Ejemplo:** Ana vende 2 polos a las 20:00, cuenta el stock y lo ajusta a las 21:00; a las 21:30 intenta anular la venta y ve ese mensaje. Una venta sin ajustes posteriores se anula normalmente.
❌
**Contraejemplo:** se anula la venta y el stock sube 2 unidades por encima de lo que Ana contó.

---

## Funcionalidades Should y Could (F-07 a F-11)

**RN-029** | Categoría: Notificaciones (F-07) | Prioridad: SHOULD | Fuente: Alcance (F-07)
Patrón EARS: State-driven
Mientras el stock mínimo de una variante sea mayor que 0 y su stock sea menor o igual que ese mínimo, el sistema DEBE mostrar una alerta de stock bajo para esa variante (ver RN-102).
✅
**Ejemplo:** stock mínimo 3 y stock actual 3: la variante aparece con la alerta "Stock bajo: quedan 3 unidades".
❌
**Contraejemplo:** stock mínimo 3 y stock actual 2: no se muestra ninguna alerta.

**RN-101** | Categoría: Datos (F-07) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El stock mínimo de una variante DEBE ser un número entero entre 0 y 9,999.
✅
**Ejemplo:** Ana define stock mínimo 3 para el polo talla M negro.
❌
**Contraejemplo:** se guarda un stock mínimo de -1 o de 2.5.

**RN-102** | Categoría: Notificaciones (F-07) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras el stock mínimo de una variante sea 0, el sistema NO DEBE mostrar alerta de stock bajo para esa variante.
✅
**Ejemplo:** stock mínimo 0 y stock 0: no aparece alerta.
❌
**Contraejemplo:** stock mínimo 0 y stock 0: aparece "Stock bajo: quedan 0 unidades".

**RN-030** | Categoría: Notificaciones (F-08) | Prioridad: SHOULD | Fuente: Alcance (F-08), Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras un producto activo no tenga ventas en los últimos 30 días contados desde hoy o desde su primera entrada, lo que sea más reciente, el sistema DEBE mostrarlo en la lista de productos sin movimiento.
✅
**Ejemplo:** hoy es 2026-11-10 y la última venta de "Cartera negra" fue el 2026-10-05: aparece en la lista.
❌
**Contraejemplo:** la última venta de "Cartera negra" fue el 2026-11-08 y aparece en la lista de productos sin movimiento.

**RN-031** | Categoría: Datos (F-09) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El resumen semanal de ganancias DEBE considerar las ventas desde el lunes 00:00:00 (incluido) hasta el lunes siguiente 00:00:00 (excluido), en hora de Lima (UTC-05:00).
✅
**Ejemplo:** una venta del domingo 2026-11-15 a las 23:30 se suma a la semana del 2026-11-09 al 2026-11-15.
❌
**Contraejemplo:** esa venta se suma a la semana siguiente porque se usó la hora UTC (2026-11-16T04:30Z).

**RN-051** | Categoría: Datos (F-09) | Prioridad: SHOULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
El resumen mensual de ganancias DEBE considerar las ventas desde el día 1 a las 00:00:00 (incluido) hasta el día 1 del mes siguiente a las 00:00:00 (excluido), en hora de Lima (UTC-05:00).
✅
**Ejemplo:** una venta del 2026-11-30 a las 23:30 se suma al resumen de noviembre 2026.
❌
**Contraejemplo:** esa venta se suma a diciembre porque se usó la hora UTC (2026-12-01T04:30Z).

**RN-032** | Categoría: Notificaciones (F-10) | Prioridad: COULD | Fuente: Alcance (F-10), Responsable del producto (aprobación 2026-11-10)
Patrón EARS: State-driven
Mientras una variante de perfume con stock mayor que 0 tenga fecha de vencimiento anterior o igual a hoy + 30 días (incluidas las ya vencidas), el sistema DEBE mostrarla en la alerta de vencimiento.
✅
**Ejemplo:** hoy es 2026-11-10, la variante vence el 2026-12-01 y tiene stock 4: aparece en la alerta.
❌
**Contraejemplo:** la variante vence el 2026-12-01 con stock 4 y no aparece en la alerta.

**RN-033** | Categoría: Seguridad (F-11) | Prioridad: COULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si alguien usa un enlace de recuperación de contraseña después de 1800 segundos desde su envío, entonces el sistema NO DEBE aceptarlo.
✅
**Ejemplo:** el enlace se envía a las 10:00:00 y se usa a las 10:20:00: se acepta.
❌
**Contraejemplo:** el enlace se envía a las 10:00:00 y se acepta a las 10:45:00.

**RN-034** | Categoría: Seguridad (F-11) | Prioridad: COULD | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Unwanted
Si alguien usa un enlace de recuperación de contraseña que ya se usó para cambiar la contraseña, entonces el sistema NO DEBE aceptarlo.
✅
**Ejemplo:** Ana cambia su contraseña con el enlace; al abrirlo de nuevo ve "Este enlace ya fue usado".
❌
**Contraejemplo:** Ana usa el mismo enlace dos veces para cambiar su contraseña.

---

## Experiencia de usuario (transversal)

**RN-035** | Categoría: UX | Prioridad: MUST | Fuente: Constitución (PRINCIPIO-001)
Patrón EARS: Ubiquitous
Todo texto visible para el usuario DEBE estar en español.
✅
**Ejemplo:** el botón para registrar una venta dice "Registrar venta".
❌
**Contraejemplo:** el botón dice "Save sale".

**RN-036** | Categoría: UX | Prioridad: MUST | Fuente: Responsable del producto (aprobación 2026-11-10)
Patrón EARS: Ubiquitous
Todo monto visible para el usuario DEBE mostrarse con el prefijo "S/ ", separador de miles "," y 2 decimales con ".".
✅
**Ejemplo:** una ganancia mensual de 1250.5 se muestra como "S/ 1,250.50".
❌
**Contraejemplo:** se muestra como "1250.5" o "PEN 1250,50".

**RN-148** | Categoría: UX | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-58)
Patrón EARS: Ubiquitous
Todo monto negativo visible para el usuario DEBE mostrarse con el signo antes del prefijo, como "-S/ 4.00".
✅
**Ejemplo:** una venta con ganancia negativa muestra "-S/ 4.00".
❌
**Contraejemplo:** se muestra "S/ -4.00" o "(S/ 4.00)".

**RN-149** | Categoría: UX | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-58)
Patrón EARS: Ubiquitous
Toda fecha visible para el usuario DEBE mostrarse como DD/MM/AAAA y toda hora en formato de 24 horas HH:mm, en hora de Lima.
✅
**Ejemplo:** una venta registrada el 2026-11-10 a las 21:30 se muestra "10/11/2026 21:30".
❌
**Contraejemplo:** se muestra "2026-11-10 9:30 PM" o "11/10/2026".

**RN-150** | Categoría: UX | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (B-02)
Patrón EARS: Event-driven
Cuando el sistema muestre una cantidad de unidades, DEBE usar el singular para 1 ("queda 1 unidad") y el plural para cualquier otra cantidad ("quedan 0 unidades", "quedan 2 unidades").
✅
**Ejemplo:** "Stock insuficiente: quedan 2 unidades" y "Stock bajo: queda 1 unidad".
❌
**Contraejemplo:** "quedan 1 unidades".

**RN-152** | Categoría: UX | Prioridad: MUST | Fuente: Responsable del producto (decisión 2026-11-10, delegada a QA; auditoría QA v3) (P-58)
Patrón EARS: Event-driven
Cuando el emprendedor escriba un monto, el sistema DEBE aceptar el punto o la coma como separador decimal y guardar el valor con punto.
✅
**Ejemplo:** Ana escribe "29,90" o "29.90" y ambos se guardan como 29.90.
❌
**Contraejemplo:** el sistema rechaza "29,90" con "Monto inválido".

---

## Glosario

El glosario del dominio está en `04-glosario.md`.

---

## Decisiones tomadas por el Responsable del producto (2026-11-10)

1. **Método de costo (RN-020, RN-027):** costo unitario promedio ponderado.
2. **Ajuste por conteo físico (RN-024, RN-037 a RN-040):** permitido con motivo obligatorio y cantidad contada entera de 0 o más; no genera venta ni ganancia.
3. **Precio final (RN-025, RN-043):** puede ser distinto del precio de venta configurado.
4. **Variantes (RN-012, RN-058 a RN-060):** todo producto tiene al menos una variante; un Accesorio sin variantes recibe la variante "Única" y no exige atributos; Ropa exige talla y color; Perfume exige presentación en ml y la fecha de vencimiento es opcional.
5. **Stock inicial (RN-061):** toda variante nace con stock 0 y se carga con una entrada que incluye su costo.
6. **Cuenta impaga (RN-063, RN-067):** al terminar la gratuidad sin pago, la cuenta queda suspendida sin acceso; a los 30 días pasa a cerrada y se borra 30 días después. La suspensión la hace manualmente el Administrador (RN-068 a RN-071).
7. **SKU (RN-064 a RN-066):** cada variante recibe un SKU automático.
8. **Ganancia negativa (RN-072, RN-073):** se permite vender por debajo del costo (remates), con confirmación previa.
9. **Perfume vencido (RN-074, RN-075):** no se puede vender; se retira del stock con un ajuste con motivo "Merma por vencimiento". El registro de mermas con costo queda como F-17 (Should).
10. **Ventas con fecha pasada (RN-076 a RN-079):** se permiten para registrar ventas olvidadas.
11. **Ventas con varias variantes (RN-080 a RN-084):** una venta puede tener varias líneas, una por variante; se puede anular la venta completa o una sola línea.
12. **Corrección de errores (RN-052 a RN-057):** las ventas y entradas registradas por error se anulan; la anulación revierte el stock y la ganancia y queda en el historial.

El 2026-11-10 el Responsable del producto aprobó todas las reglas propuestas y todas las recomendaciones pendientes (RN-087 a RN-104).

---

## Revisión QA (2026-11-10)

Cambios hechos a partir de la auditoría QA del Spec Pack. Los marcados "Propuesta QA" (RN-105, RN-106) fueron aprobados por el Responsable del producto el 2026-11-10. Los demás son correcciones de redacción o decisiones que el Responsable del producto tomó el 2026-11-10.

| Tipo de cambio | Reglas |
|---|---|
| Reglas MUST/SHOULD reescritas de "PUEDE" a "DEBE permitir" | RN-043, RN-060, RN-068, RN-070, RN-075, RN-076, RN-086, RN-095, RN-100 |
| Reglas reescritas por línea de venta | RN-021, RN-022, RN-023, RN-025, RN-052, RN-053, RN-054, RN-057 |
| Reglas divididas por ser compuestas | RN-065 (nueva RN-108), RN-080 (nueva RN-107) |
| Reglas nuevas "Propuesta QA" (aprobadas el 2026-11-10) | RN-105 (contraseña mínima al cambiarla), RN-106 (decimales del precio final) |
| Patrón EARS corregido | RN-002, RN-003, RN-033, RN-034, RN-084 |
| Contradicción resuelta con RN-102 | RN-029 |
| Borrado de la cuenta incluido según PRINCIPIO-006 | RN-006 |
| Concurrencia expresada como invariante | RN-045 |
| Intervalos de tiempo semiabiertos | RN-031, RN-051 |
| Decisiones de la revisión QA sobre hallazgos altos (2026-11-10) | RN-063, RN-067 y RN-104 pasan a SHOULD junto con F-16 (la suspensión es manual); RN-120 (los logs no llevan correo ni nombre); RN-050 y RN-121 a RN-124 (atributos de variante; un perfume con otra fecha de vencimiento es otra variante); RN-084 y RN-125 (líneas con la misma variante) |
| Decisiones del Responsable del producto (2026-11-10) | RN-109 (venta sin líneas vigentes queda anulada), RN-110 (cambio de contraseña cierra las demás sesiones), RN-108 (el SKU no se reutiliza), RN-111 a RN-113 (talla en mayúsculas; atributos por categoría), RN-114 y RN-071 (Administrador sin inquilino, rol insertado manualmente), RN-115 (auditoría del vencimiento), RN-119 (la categoría de un producto no se cambia), RN-116 (no se sube stock por ajuste sin entradas vigentes), RN-056, RN-088 y RN-118 (anular una entrada solo si no hay movimientos posteriores; se restaura el costo promedio anterior) |

Otras decisiones del 2026-11-10 que no cambian reglas:
- La suspensión de cuentas la hace manualmente el Administrador (RN-068).
- Los logs del servidor no se borran al cerrar una cuenta; por eso no guardan datos personales (RN-120).
- Se quitaron las referencias a documentos que no forman parte del pack.

RN-075 ahora ofrece el motivo en todo ajuste de perfume, porque el sistema no puede saber si el ajuste es para retirar vencidos; confirmar.

Los casos que requieren una decisión están en `06-preguntas-abiertas.md`.

---

## Auditoría QA v2 (2026-11-10)

Decisiones del Responsable del producto tras la auditoría QA v2. Criterio general: la primera versión debe ser simple y no bloquear al usuario más de lo necesario.

| Decisión | Reglas |
|---|---|
| RN-105 y RN-106 aprobadas | RN-105, RN-106 |
| Cerrar la cuenta con la contraseña (A-02) | RN-127 |
| Login imposible tras 30 días del cierre, aunque el borrado se retrase; el mecanismo de borrado se decide después (M-07) | RN-128 |
| Los backups no incluyen cuentas cerradas (C-03) | RN-129 |
| Mensaje y efecto del bloqueo (P-46) | RN-130, RN-131 |
| Toda petición cuenta como actividad (P-47) | RN-132 |
| Formato de correo y máximo de contraseña (M-06) | RN-133, RN-134 |
| Perfumes sin vencimiento duplicados (M-12) | RN-135 |
| Última variante y desactivar con stock (M-01, P-41) | RN-136, RN-137 |
| La venta guarda fecha y hora; por defecto, el momento del registro; las ventas pasadas eligen día y hora aproximada (P-39) | RN-076, RN-077, RN-078 (reescritas), RN-126 |
| Campos de búsqueda (P-51) | RN-138 |
| No se anula una venta con un ajuste posterior (A-01) | RN-139 |
| Una sola confirmación de pérdida por venta (P-40) | RN-073 (precisada) |
| El vencimiento se compara con hoy (P-38); el stock se guarda en la variante (P-37); cada línea congela su costo (P-54) | Sin cambios: RN-074, RN-024, RN-028, RN-079 |

---

## Auditoría QA v3 (2026-11-10)

El Responsable del producto aprobó la arquitectura y delegó en QA la decisión de los casos restantes, con el criterio de simplicidad. Los valores por defecto elegidos pasan a ser reglas.

| Decisión | Reglas |
|---|---|
| Nombre del emprendedor opcional, máximo 100 caracteres (P-43) | RN-141 |
| Producto desactivado: sin entradas; sí anulaciones y ajustes (P-41) | RN-140, RN-147 |
| Una entrada = una variante y fecha del registro (P-50) | RN-142 |
| Ajustes: motivo de 1 a 200 caracteres, diferencia 0 permitida, no se anulan (P-56) | RN-143, RN-144, RN-145 |
| Presentación máxima de 9,999 ml (P-57) | RN-146 |
| Formatos visibles (P-58, B-02) | RN-148, RN-149, RN-150, RN-152 |
| Tras registrarse, la sesión se inicia sola | RN-153 |
| Línea fusionada mayor que 9,999 (B-08) | RN-151 |

## Auditoría QA v4 (2026-11-11)

Hallazgo del Prototype Reviewer (`10-validation-log.md`, O-04): RN-088 no define qué costo promedio queda al anular la primera entrada de una variante, porque no hay un costo anterior que restaurar.

| Decisión | Reglas |
|---|---|
| Al anular la única entrada vigente, la variante queda sin costo promedio (propuesta, pendiente de P-64) | RN-154, nota en RN-118 |
