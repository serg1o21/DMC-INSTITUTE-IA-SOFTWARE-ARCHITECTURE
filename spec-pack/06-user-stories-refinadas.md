# 06 - Historias de usuario: miStorePe (enriquecidas por QA)

Basado en `00-constitucion.md`, `01-project-brief.md`, `02-alcance.md` y `03-reglas-negocio.md` (Spec Pack v1.0). Fecha: 2026-11-10.

Convenciones:
- Actores: Visitante (sin sesión) y Emprendedor (titular de la cuenta; en el MVP es también el responsable de registrar las ventas).
- Todos los textos visibles están en español (RN-035). Fechas y horas en America/Lima (UTC-05:00). Montos con el formato de RN-036.
- Las preguntas abiertas se refieren a `05-preguntas-abiertas.md`. Los hallazgos A-nn y M-nn son los de la última auditoría QA. Ningún agente implementa un caso afectado por una pregunta abierta hasta que se responda (PRINCIPIO-009).
- Marcas de la revisión QA: `[AUDITORÍA v2]` y `[AUDITORÍA v3]` son criterios agregados tras las decisiones del 2026-11-10 (auditorías QA v2 y v3); `[AUDITORÍA v4]` es un criterio agregado tras la validación del prototipo (2026-11-11); `[AUDITORÍA v5]` es un criterio agregado tras las decisiones del 2026-11-12 (P-76, P-77); `[NUEVO]` es un criterio agregado; `[AJUSTADO]` es un criterio original corregido; los criterios sin marca son los originales, sin cambios. Cada criterio nuevo indica su tipo (Positivo, Negativo o Borde) y la regla en la que se apoya. QA no inventó reglas: lo que no tiene regla queda anotado como "QA:" en las notas de la HU.
- Este lote cubre solo las funcionalidades Must (F-01 a F-06). Las Should y Could (F-07 a F-17) quedan para el siguiente lote. No hay HU para ningún Won't (W-01 a W-12).

## Resumen y orden de implementación

Las 12 HU son Must del MVP. El orden responde a dependencias técnicas, no a recortes: todas deben estar listas antes del piloto.

| Orden | HU | Épica | Feature | Depende de |
|---|---|---|---|---|
| 1 | HU-001 | Cuenta y acceso | F-01 | - |
| 2 | HU-002 | Cuenta y acceso | F-01 | HU-001 |
| 3 | HU-005 | Catálogo | F-02 | HU-002 |
| 4 | HU-007 | Entradas y costo | F-03 | HU-005 |
| 5 | HU-009 | Ventas y ganancia | F-04 | HU-007 |
| 6 | HU-011 | Ventas y ganancia | F-06 | HU-009 |
| 7 | HU-012 | Stock | F-05 | HU-007 |
| 8 | HU-006 | Catálogo | F-02 | HU-005 |
| 9 | HU-008 | Entradas y costo | F-03 | HU-007 |
| 10 | HU-010 | Ventas y ganancia | F-04 | HU-009 |
| 11 | HU-003 | Cuenta y acceso | F-01 | HU-002 |
| 12 | HU-004 | Cuenta y acceso | F-01 | HU-002 |

---

## Validación INVEST (revisión QA)

| HU | Resultado | Problema | Corrección sugerida (no aplicada) |
|---|---|---|---|
| HU-001 | Cumple | - | - |
| HU-002 | Cumple | Depende de HU-001 | - |
| HU-003 | Cumple | - (RN-105 aprobada) | - |
| HU-004 | Cumple (A-02 resuelto con RN-127) | Reúne cerrar, reabrir y borrar | Opcional: dividir en HU-004a "Cerrar mi cuenta" y HU-004b "Reabrir mi cuenta y borrado a los 30 días" |
| HU-005 | No cumple Small | Reúne tres categorías, 20 criterios | Dividir en HU-005a (Ropa), HU-005b (Perfume) y HU-005c (Accesorio) |
| HU-006 | Parcial en Small | Mezcla editar, desactivar, eliminar y auditar | Separar "Editar una variante" de "Desactivar y reactivar un producto" |
| HU-007 | Cumple | Depende de HU-005 | - |
| HU-008 | Cumple | Depende de HU-007 | - |
| HU-009 | No cumple Small | 21 criterios | Dividir en venta de una línea, de varias líneas y con fecha pasada |
| HU-010 | Cumple (A-01 resuelto con RN-139) | - | - |
| HU-011 | Cumple | El criterio de la ganancia mensual era de F-09 (ajustado) | - |
| HU-012 | Parcial en Small | Consulta y ajuste son dos objetivos | Dividir en "Consultar mi stock" y "Ajustar el stock por conteo físico" |

Valores concretos: todos los criterios originales tienen fechas, montos o cantidades. Los únicos ajustes fueron dos criterios que usaban una "ganancia del día" o una "ganancia mensual" que no existe en este lote (HU-011 y HU-012) y uno sin resultado observable (HU-012, RN-024).

## Criterios transversales [NUEVO]

Aplican a todas las HU y no se repiten en cada una.

- [NUEVO] (Positivo) T-01. DADO QUE el emprendedor usa cualquier pantalla o recibe cualquier mensaje, CUANDO el sistema muestra un texto, ENTONCES está en español (por ejemplo, el botón dice "Registrar venta" y no "Save sale"). (RN-035, PRINCIPIO-001)
- [NUEVO] (Negativo) T-02. DADO QUE existe un endpoint que recibe un ID, CUANDO el usuario A lo pide o intenta modificarlo con un recurso del usuario B, ENTONCES el sistema responde HTTP 404; cada endpoint tiene un test de integración que lo verifica. (PRINCIPIO-003, RN-005)
- [NUEVO] (Borde) T-03. DADO QUE una pantalla se abre en un celular de 360 px de ancho, CUANDO el usuario la recorre, ENTONCES no aparece scroll horizontal y los tests E2E con axe-core no reportan violaciones de severidad seria o crítica. (PRINCIPIO-011)
- [NUEVO] (Positivo) T-04. DADO QUE el sistema muestra un monto, CUANDO el valor es 1250.5, ENTONCES se muestra "S/ 1,250.50". (RN-036)
- [NUEVO] (Negativo) T-05. DADO QUE el sistema escribe un log, CUANDO registra cualquier evento, ENTONCES no contiene el correo, el nombre, la contraseña ni tokens de ningún emprendedor. (RN-120, PRINCIPIO-007)
- [NUEVO] (Borde) T-06. DADO QUE se corre el CI, CUANDO la cobertura de líneas de la lógica de negocio baja de 80% o falta un test E2E del flujo principal de una HU, ENTONCES el CI falla y el pull request no se fusiona. (PRINCIPIO-010)
- [NUEVO] (Positivo) T-07. DADO QUE existe el proceso automático de borrado de cuentas cerradas, CUANDO se corre el CI, ENTONCES un test automatizado lo verifica. (PRINCIPIO-006)

---

## Épica 1: Cuenta y acceso (F-01)

### HU-001: Registrarme con mi correo y una contraseña

**Prioridad:** Must

**Como** visitante, **quiero** crear mi cuenta con mi correo y una contraseña, **para** empezar a controlar mi inventario sin depender del cuaderno.

**Criterios de aceptación:**
- DADO QUE no existe ninguna cuenta con `ana@gmail.com`, CUANDO el visitante envía el formulario con ese correo, la contraseña `TiendaLima2026` (14 caracteres) y la casilla "Acepto la política de privacidad" marcada, ENTONCES el sistema crea la cuenta, guarda `privacy_accepted_at` con la fecha y hora del envío (por ejemplo, `2026-11-10T12:40:00-05:00`) y guarda la contraseña como hash que empieza con `$argon2id$`.
- DADO QUE ya existe una cuenta con `ana@gmail.com`, CUANDO el visitante envía el formulario con ` Ana@Gmail.com ` (con mayúsculas y espacios), ENTONCES el sistema no crea otra cuenta y muestra "Este correo ya tiene una cuenta".
- DADO QUE el visitante escribe la contraseña `Lima2026` (8 caracteres), CUANDO envía el formulario, ENTONCES el sistema no crea la cuenta.
- DADO QUE el visitante deja sin marcar la casilla "Acepto la política de privacidad", CUANDO envía el formulario, ENTONCES el sistema no crea la cuenta.
- [NUEVO] (Positivo) DADO QUE Ana se registró escribiendo `Ana@Gmail.com ` (con mayúsculas y un espacio al final), CUANDO inicia sesión con `ana@gmail.com`, ENTONCES entra a su cuenta porque el correo se guardó como `ana@gmail.com`. (RN-048)
- [NUEVO] (Borde) DADO QUE el visitante escribe una contraseña de exactamente 12 caracteres (`ClaveLima123`) y completa lo demás, CUANDO envía el formulario, ENTONCES el sistema crea la cuenta; CUANDO escribe una de 11 caracteres (`ClaveLima12`), ENTONCES no la crea. (RN-002)
- [NUEVO] (Borde) DADO QUE dos visitantes envían a la vez el formulario con `luis@gmail.com`, CUANDO el sistema procesa ambos envíos, ENTONCES crea una sola cuenta y el otro visitante ve "Este correo ya tiene una cuenta". (RN-001, RN-048, RN-094)
- [NUEVO] (Negativo) DADO QUE el visitante envía el formulario con la contraseña `Lima2026` (8 caracteres), CUANDO el sistema rechaza el envío y escribe sus logs, ENTONCES los logs no contienen la contraseña ni el correo escrito. (RN-120, PRINCIPIO-007)
- [AUDITORÍA v2] (Negativo) DADO QUE el visitante escribe el correo `ana@gmail`, CUANDO envía el formulario, ENTONCES el sistema no crea la cuenta y muestra "Revisa el correo". (RN-133)
- [AUDITORÍA v2] (Borde) DADO QUE el visitante escribe una contraseña de 128 caracteres, CUANDO envía el formulario, ENTONCES el sistema crea la cuenta; CUANDO tiene 129, ENTONCES no la crea. (RN-134)
- [AUDITORÍA v3] (Positivo) DADO QUE Ana se registra sin escribir su nombre, CUANDO envía el formulario, ENTONCES el sistema crea la cuenta y la app la saluda con su correo. (RN-141)
- [AUDITORÍA v3] (Borde) DADO QUE el visitante escribe un nombre de 100 caracteres, CUANDO envía el formulario, ENTONCES el sistema lo guarda; CUANDO tiene 101, ENTONCES no lo guarda. (RN-141)
- [AUDITORÍA v3] (Positivo) DADO QUE Ana completa el registro con `ana@gmail.com` y `TiendaLima2026`, CUANDO el sistema crea su cuenta, ENTONCES su sesión queda iniciada y ve su lista de productos sin volver a escribir sus datos. (RN-153)

**Reglas de negocio relacionadas:** RN-001, RN-002, RN-003, RN-035, RN-048, RN-094, RN-120, RN-133, RN-134, RN-141, RN-153 (Constitución: PRINCIPIO-004, PRINCIPIO-005, PRINCIPIO-007)

**Notas:**
- El rol Administrador nunca se asigna desde este formulario (RN-071, parte de F-16, fuera de este lote).
- P-43 resuelto con RN-141; P-44 resuelto: se muestra "Este correo ya tiene una cuenta" hasta el borrado (RN-001). M-06 resuelto con RN-133 y RN-134.
- QA: no se prueba la verificación de que el correo sea del visitante (fuera del MVP).
- `privacy_accepted_at` es un dato de la cuenta reconocido por ADR-0010; PRINCIPIO-005 ya lo incluye.

---

### HU-002: Iniciar sesión y mantener mi sesión segura

**Prioridad:** Must

**Como** emprendedor, **quiero** iniciar sesión con mi correo y contraseña y que mi sesión se cierre sola tras 8 horas sin uso, **para** proteger los datos de mi negocio si dejo la app abierta en el celular o la laptop.

**Criterios de aceptación:**
- DADO QUE existe la cuenta `ana@gmail.com`, CUANDO Ana inicia sesión con ` Ana@Gmail.com ` y la contraseña correcta, ENTONCES entra a su cuenta y su contador de intentos fallidos consecutivos vuelve a 0.
- DADO QUE la cuenta de Ana acumuló 5 intentos fallidos consecutivos y el quinto ocurrió a las 10:00:00, CUANDO Ana intenta entrar con la contraseña correcta a las 10:10:00, ENTONCES el sistema rechaza el inicio de sesión; CUANDO lo intenta a las 10:15:01 (más de 900 segundos después), ENTONCES lo acepta.
- DADO QUE Ana falló 3 veces, entró a la cuarta y al día siguiente falla 2 veces, CUANDO intenta entrar de nuevo, ENTONCES la cuenta no está bloqueada porque su contador está en 2.
- DADO QUE nadie tiene una cuenta con `noexiste@gmail.com`, CUANDO alguien intenta iniciar sesión con ese correo, ENTONCES el sistema no suma ningún intento fallido a ninguna cuenta.
- DADO QUE la sesión de Ana no tiene actividad desde las 10:00, CUANDO llegan las 18:01 (más de 28,800 segundos después), ENTONCES la sesión está cerrada y Ana debe iniciar sesión de nuevo.
- DADO QUE un inicio de sesión falla, CUANDO el sistema escribe el log, ENTONCES el log contiene el identificador interno de la cuenta (por ejemplo, "Intento fallido para la cuenta 4821") y no el correo.
- [NUEVO] (Positivo) DADO QUE Ana inició sesión hace 2 horas y consultó su stock hace 1 hora, CUANDO abre la lista de productos, ENTONCES la app le muestra sus datos sin pedirle iniciar sesión. (RN-097)
- [NUEVO] (Negativo) DADO QUE el contador de intentos fallidos de Ana es 0, CUANDO ingresa una contraseña incorrecta, ENTONCES el sistema rechaza el inicio de sesión y el contador pasa a 1. (RN-004, RN-049)
- [NUEVO] (Borde) DADO QUE Ana falló 4 veces consecutivas, CUANDO intenta entrar con la contraseña correcta, ENTONCES entra y su contador vuelve a 0; si en cambio falla una quinta vez, ENTONCES la cuenta se bloquea. (RN-004, RN-049)
- [NUEVO] (Borde) DADO QUE el quinto intento fallido de Ana ocurrió a las 10:00:00, CUANDO intenta entrar con la contraseña correcta a las 10:14:59 (899 segundos después), ENTONCES el sistema rechaza el inicio de sesión. (RN-004)
- [NUEVO] (Borde) DADO QUE la última actividad de Ana fue a las 10:00:00, CUANDO realiza una acción propia (por ejemplo, consultar su stock) a las 17:59:00 (28,740 segundos después), ENTONCES la sesión sigue abierta y el plazo de 28,800 segundos se cuenta desde esa acción. (RN-097)
- [AUDITORÍA v2] (Negativo) DADO QUE el quinto fallo de Ana fue a las 10:00:00, CUANDO intenta entrar a las 10:05:00, ENTONCES ve "Demasiados intentos. Intenta de nuevo en 15 minutos" y el bloqueo sigue terminando a las 10:15:00. (RN-130, RN-131)
- [AUDITORÍA v2] (Borde) DADO QUE Ana abrió la lista de productos a las 17:59:30, CUANDO llegan las 01:59:29 del día siguiente sin otra acción, ENTONCES su sesión sigue abierta. (RN-097, RN-132)
- [AUDITORÍA v5] (Borde) DADO QUE el quinto intento fallido de Ana ocurrió a las 10:00:00, CUANDO intenta entrar con la contraseña correcta a las 10:15:00 (900 segundos exactos), ENTONCES entra. (RN-155)

**Reglas de negocio relacionadas:** RN-004, RN-048, RN-049, RN-097, RN-103, RN-120, RN-130, RN-131, RN-132, RN-155 (Constitución: PRINCIPIO-004, PRINCIPIO-007)

**Notas:**
- Las reglas de cuentas suspendidas o cerradas por falta de pago (RN-063, RN-067, RN-104) son Should y están en F-16, fuera de este lote.
- P-46 resuelto con RN-130 y RN-131; P-47 resuelto con RN-132. La suspensión y las sesiones abiertas se deciden con F-16 (P-69).
- El instante exacto de 900 segundos quedó definido el 2026-11-12 (pregunta P-76, RN-155): a las 10:15:00 la cuenta ya está desbloqueada.

---

### HU-003: Cambiar mi contraseña

**Prioridad:** Must

**Como** emprendedor con sesión iniciada, **quiero** cambiar mi contraseña desde "Mi cuenta", **para** mantener mi cuenta segura sin pedirle ayuda a nadie.

**Criterios de aceptación:**
- DADO QUE Ana tiene sesión iniciada, CUANDO ingresa su contraseña actual correcta y la nueva `NuevaClaveLima2026` (18 caracteres), ENTONCES el sistema cambia la contraseña.
- DADO QUE Ana tiene sesión iniciada, CUANDO ingresa una contraseña actual incorrecta, ENTONCES el sistema no cambia la contraseña.
- DADO QUE Ana tiene sesión iniciada, CUANDO ingresa su contraseña actual correcta y una nueva de 8 caracteres (`Lima2027`), ENTONCES el sistema no cambia la contraseña.
- DADO QUE Ana tiene sesiones abiertas en el celular y en la laptop, CUANDO cambia su contraseña desde el celular, ENTONCES la sesión de la laptop se cierra y la sesión del celular sigue abierta.
- [NUEVO] (Positivo) DADO QUE Ana cambió su contraseña a `NuevaClaveLima2026`, CUANDO inicia sesión con la contraseña anterior, ENTONCES el sistema la rechaza; CUANDO inicia sesión con `NuevaClaveLima2026`, ENTONCES entra y su `password_hash` empieza con `$argon2id$`. (RN-095, PRINCIPIO-004)
- [NUEVO] (Negativo) DADO QUE la sesión de Ana se cerró por inactividad, CUANDO intenta cambiar su contraseña, ENTONCES el sistema le pide iniciar sesión y no cambia la contraseña. (RN-095, RN-097)
- [NUEVO] (Negativo) DADO QUE Ana tiene sesión iniciada, CUANDO envía el formulario con la contraseña actual vacía y la nueva `NuevaClaveLima2026`, ENTONCES el sistema no cambia la contraseña. (RN-096)
- [NUEVO] (Borde) DADO QUE Ana ingresa su contraseña actual correcta, CUANDO la nueva tiene exactamente 12 caracteres (`ClaveLima123`), ENTONCES el sistema la cambia; CUANDO tiene 11 (`ClaveLima12`), ENTONCES no la cambia. (RN-105)
- [AUDITORÍA v2] (Borde) DADO QUE Ana ingresa su contraseña actual correcta, CUANDO la nueva tiene 129 caracteres, ENTONCES el sistema no la cambia. (RN-134)
- [AUDITORÍA v3] (Positivo) DADO QUE Ana tiene sesión iniciada, CUANDO escribe "Ana" como nombre en "Mi cuenta", ENTONCES el sistema lo guarda y la app la saluda con ese nombre. (RN-141)
- [AUDITORÍA v5] (Negativo) DADO QUE la contraseña de Ana es `TiendaLima2026` y tiene sesiones en el celular y la laptop, CUANDO escribe `TiendaLima2026` como nueva, ENTONCES ve "La nueva contraseña debe ser distinta de la actual" y ninguna sesión se cierra. (RN-156)
- [AUDITORÍA v5] (Borde) DADO QUE el contador de intentos fallidos de Ana está en 0, CUANDO escribe mal su contraseña actual 5 veces en "Mi cuenta", ENTONCES su contador sigue en 0 y puede iniciar sesión sin bloqueo. (RN-157)

**Reglas de negocio relacionadas:** RN-002, RN-095, RN-096, RN-105, RN-110, RN-134, RN-141, RN-156, RN-157 (Constitución: PRINCIPIO-004)

**Notas:**
- RN-105 (mínimo de 12 caracteres al cambiar la contraseña) fue aprobada el 2026-11-10.
- La recuperación por correo es F-11 (Could, fuera de este lote). Hasta entonces, Sergio restablece las contraseñas a mano (SUP-07) con un comando de consola que solo él ejecuta (tarea T-039 del backlog).
- La contraseña nueva igual a la actual y los errores en la contraseña actual quedaron definidos el 2026-11-12 (pregunta P-77, RN-156 y RN-157).

---

### HU-004: Cerrar mi cuenta y poder reabrirla dentro de 30 días

**Prioridad:** Must

**Como** emprendedor, **quiero** cerrar mi cuenta y poder reabrirla si me arrepiento dentro de los 30 días, **para** dejar la app con la seguridad de que mis datos se borran, sin perder mi inventario por un error.

**Criterios de aceptación:**
- DADO QUE la cuenta de Ana tiene `closed_at = 2026-12-01T10:00:00-05:00`, CUANDO Ana inicia sesión el 2026-12-05, ENTONCES solo ve la opción "Reabrir cuenta" y no puede ver ni modificar su inventario.
- DADO QUE la cuenta de Ana tiene `closed_at = 2026-12-01T10:00:00-05:00`, CUANDO el 2026-12-10 Ana confirma "Reabrir cuenta", ENTONCES el sistema reactiva la cuenta, cancela el borrado y Ana recupera su inventario completo.
- DADO QUE pasaron 2,592,000 segundos desde el `closed_at` de Ana (2026-12-31T10:00:00-05:00), CUANDO se ejecuta el proceso automático de borrado, ENTONCES no existen la cuenta de Ana (correo, nombre, hash de contraseña e intentos fallidos) ni ningún registro con su `tenant_id`.
- DADO QUE existe un backup con datos de Ana, CUANDO pasan 2,592,000 segundos desde su `closed_at`, ENTONCES ese backup ya no existe.
- [NUEVO] (Positivo) DADO QUE Ana reabrió su cuenta el 2026-12-10, CUANDO llega el 2026-12-31T10:00:00-05:00, ENTONCES su cuenta y su inventario siguen existiendo. (RN-090)
- [NUEVO] (Negativo) DADO QUE pasaron más de 2,592,000 segundos desde el `closed_at` de Ana, CUANDO Ana intenta iniciar sesión, ENTONCES el sistema la rechaza porque la cuenta no existe y no suma intentos fallidos a ninguna cuenta. (RN-006, RN-103)
- [NUEVO] (Negativo) DADO QUE la cuenta de Ana está cerrada, CUANDO su sesión intenta registrar una venta de 1 unidad, ENTONCES el sistema no la registra y el stock no cambia. (RN-007)
- [NUEVO] (Borde) DADO QUE `closed_at` es 2026-12-01T10:00:00-05:00, CUANDO Ana confirma "Reabrir cuenta" el 2026-12-31T09:59:59-05:00 (2,591,999 segundos después), ENTONCES el sistema reactiva la cuenta con todo su inventario. (RN-090)
- [NUEVO] (Borde) DADO QUE Luis tiene 20 productos y 15 ventas, CUANDO el proceso de borrado elimina la cuenta de Ana, ENTONCES Luis conserva sus 20 productos y sus 15 ventas. (PRINCIPIO-003, RN-006)
- [NUEVO] (Borde) DADO QUE Ana tenía entradas, ajustes y auditorías de vencimiento, CUANDO se borra su cuenta, ENTONCES no queda ninguna fila con su `tenant_id` en ninguna tabla. (RN-006, RN-115)
- [AUDITORÍA v2] (Positivo) DADO QUE Ana tiene sesiones abiertas en el celular y en la laptop, CUANDO confirma "Cerrar cuenta" con su contraseña el 2026-12-01T10:00:00-05:00, ENTONCES la cuenta queda con ese `closed_at` y ambas sesiones se cierran. (RN-127)
- [AUDITORÍA v2] (Negativo) DADO QUE Ana ingresa una contraseña incorrecta, CUANDO confirma "Cerrar cuenta", ENTONCES la cuenta sigue activa. (RN-127)
- [AUDITORÍA v2] (Negativo) DADO QUE `closed_at` es 2026-12-01T10:00:00-05:00 y el borrado aún no se ejecutó, CUANDO Ana intenta entrar el 2026-12-31T10:00:01-05:00, ENTONCES el sistema responde igual que con un correo inexistente. (RN-128)
- [AUDITORÍA v2] (Negativo) DADO QUE la cuenta de Ana está cerrada, CUANDO se genera el backup diario, ENTONCES el backup no contiene ningún registro con su `tenant_id`. (RN-129)

**Reglas de negocio relacionadas:** RN-006, RN-007, RN-046, RN-090, RN-103, RN-115, RN-127, RN-128, RN-129 (Constitución: PRINCIPIO-003, PRINCIPIO-006)

**Notas:**
- A-02 resuelto con RN-127: el emprendedor cierra su cuenta confirmando con su contraseña.
- La reapertura de RN-090 aplica solo al cierre voluntario. El cierre por falta de pago se define con F-16 (P-69).
- El borrado usa un job diario `@Scheduled` con tolerancia de 24 h (P-61, resuelto 2026-11-11, T-025b). Mientras tanto, RN-128 impide entrar a la cuenta vencida y RN-129 resuelve los backups.
- QA: el instante exacto 2,592,000 segundos no se prueba el job diario tolera hasta 24 h, aceptado en P-61; por eso el borde se prueba a 2,591,999 segundos.

---

## Épica 2: Catálogo de productos (F-02)

### HU-005: Crear un producto con sus variantes según su categoría

**Prioridad:** Must

**Como** emprendedor, **quiero** crear un producto de ropa, perfume o accesorio con sus variantes y su precio de venta, **para** saber en todo momento qué tallas, colores o presentaciones tengo.

**Criterios de aceptación:**
- DADO QUE Ana crea "Polo oversize básico" con categoría Ropa, CUANDO agrega una variante con talla ` xl ` y color ` NEGRO ` y precio S/ 35.90, ENTONCES el sistema guarda la talla `XL` y el color `Negro`.
- DADO QUE Ana crea un producto de categoría Ropa, CUANDO guarda una variante sin talla, o con una talla de 40 caracteres, o guarda el producto sin ninguna variante, ENTONCES el sistema no lo guarda.
- DADO QUE Ana crea "Perfume Floral Rosé" con categoría Perfume, CUANDO agrega una variante de 100 ml con vencimiento `2028-06-30`, ENTONCES el sistema la guarda; CUANDO agrega otra de 100 ml sin vencimiento, ENTONCES también la guarda; CUANDO agrega una de 0 ml o de -50 ml, ENTONCES no la guarda.
- DADO QUE Ana guarda "Collar dorado" con categoría Accesorio y sin variantes, CUANDO el sistema lo guarda, ENTONCES crea la variante "Única" sin talla, sin color, sin presentación y sin vencimiento.
- DADO QUE Ana crea la primera variante de Ropa de su cuenta, CUANDO el sistema la guarda, ENTONCES le asigna el SKU `ROP-00001` y stock 0; la siguiente variante de perfume de Ana recibe `PER-00001`, y el primer perfume de Luis (otro emprendedor) también recibe `PER-00001`.
- DADO QUE "Polo oversize básico" ya tiene la variante talla M color Negro, CUANDO Ana crea otra talla M color Negro, ENTONCES el sistema no la guarda; CUANDO crea talla M color Blanco, ENTONCES la guarda; CUANDO agrega una variante de Ropa de 100 ml o una de Perfume con talla M, ENTONCES el sistema no la guarda.
- DADO QUE Ana guarda un precio de venta, CUANDO ingresa S/ 35.90, ENTONCES el sistema lo guarda; CUANDO ingresa S/ 0.00, S/ 35.999 o S/ 150,000.00, ENTONCES no lo guarda; CUANDO el nombre del producto tiene 250 caracteres, ENTONCES no lo guarda.
- [NUEVO] (Negativo) DADO QUE Ana crea un producto, CUANDO lo guarda sin elegir categoría, ENTONCES el sistema no lo guarda. (RN-008)
- [NUEVO] (Positivo) DADO QUE Ana guarda "Collar dorado" (Accesorio) con una variante de color `Dorado` y sin talla, CUANDO el sistema la guarda, ENTONCES la acepta y no crea la variante "Única". (RN-059, RN-060)
- [NUEVO] (Positivo) DADO QUE el perfume de 100 ml con vencimiento `2028-06-30` ya existe, CUANDO Ana crea otra variante de 100 ml con vencimiento `2029-01-31`, ENTONCES el sistema la guarda como una variante distinta; CUANDO crea otra de 100 ml con vencimiento `2028-06-30`, ENTONCES no la guarda. (RN-050)
- [NUEVO] (Negativo) DADO QUE "Collar dorado" ya tiene la variante "Única", CUANDO Ana crea otra variante "Única" en el mismo accesorio, ENTONCES el sistema no la guarda. (RN-050)
- [NUEVO] (Negativo) DADO QUE Ana crea una variante de perfume, CUANDO escribe el vencimiento `30/06/28`, ENTONCES el sistema no la guarda; CUANDO escribe `2028-06-30`, ENTONCES la guarda. (RN-012)
- [NUEVO] (Borde) DADO QUE Ana agrega una variante de perfume, CUANDO escribe 1 ml, ENTONCES el sistema la guarda; CUANDO escribe 100.5 ml, ENTONCES no la guarda. (RN-011)
- [NUEVO] (Borde) DADO QUE Ana agrega una variante de Ropa, CUANDO la talla tiene exactamente 10 caracteres (`TALLA-GRND`), ENTONCES la guarda; CUANDO tiene 11 (`TALLA-GRAND`), ENTONCES no la guarda. (RN-122)
- [NUEVO] (Borde) DADO QUE Ana agrega una variante de Ropa, CUANDO el color tiene exactamente 30 caracteres (`Verde militar con rayas negras`), ENTONCES la guarda; CUANDO tiene 31 (`Verde militar con rayas blancas`), ENTONCES no la guarda. (RN-123)
- [NUEVO] (Borde) DADO QUE Ana guarda un producto, CUANDO su nombre tiene exactamente 100 caracteres, ENTONCES el sistema lo guarda; CUANDO tiene 101, ENTONCES no lo guarda. (RN-093)
- [NUEVO] (Borde) DADO QUE Ana guarda el precio de una variante, CUANDO ingresa S/ 0.01 o S/ 99,999.99, ENTONCES el sistema lo guarda; CUANDO ingresa S/ 100,000.00, ENTONCES no lo guarda. (RN-013, RN-091)
- [NUEVO] (Borde) DADO QUE Ana tiene la variante `PER-00003` sin entradas, ventas ni ajustes, CUANDO la elimina y crea otra variante de perfume, ENTONCES la nueva recibe `PER-00004` y no `PER-00003`. (RN-099, RN-108)
- [NUEVO] (Borde) DADO QUE Ana crea dos variantes de perfume a la vez, CUANDO el sistema las guarda, ENTONCES reciben SKU distintos (por ejemplo, `PER-00001` y `PER-00002`). (RN-065, RN-108)
- [NUEVO] (Negativo) DADO QUE Ana crea una variante, CUANDO ve el formulario, ENTONCES no existe un campo de stock inicial: el stock se carga con una entrada (HU-007). (RN-024, RN-061)
- [AUDITORÍA v2] (Negativo) DADO QUE "Perfume Floral Rosé" tiene una variante de 100 ml sin vencimiento, CUANDO Ana crea otra de 100 ml sin vencimiento, ENTONCES el sistema no la guarda. (RN-135)
- [AUDITORÍA v3] (Borde) DADO QUE Ana crea una variante de perfume, CUANDO ingresa una presentación de 9,999 ml, ENTONCES el sistema la guarda; CUANDO ingresa 10,000 ml, ENTONCES no la guarda. (RN-146)

**Reglas de negocio relacionadas:** RN-008, RN-009, RN-010, RN-011, RN-012, RN-013, RN-024, RN-041, RN-050, RN-058, RN-059, RN-060, RN-061, RN-064, RN-065, RN-091, RN-093, RN-099, RN-108, RN-111, RN-112, RN-113, RN-121, RN-122, RN-123, RN-124, RN-135, RN-146 (Constitución: PRINCIPIO-003, PRINCIPIO-005)

**Notas:**
- Es la HU más grande: reúne las tres categorías para respetar el máximo de 12 HU. Si el equipo la considera grande para un sprint, puede dividirse en Ropa, Perfume y Accesorio sin cambiar los criterios.
- Un perfume de 100 ml con otro vencimiento es otra variante (RN-050); no hay manejo de lotes.
- El costo no se define aquí: se registra en cada entrada (HU-007).
- Resueltos: P-31 (el accesorio no tiene atributos obligatorios, SUP-08), P-55 ("Única" es el campo `is_single` de la variante) y P-57 (presentación máxima de 9,999 ml, RN-146; los nombres de producto pueden repetirse). La auditoría señaló que RN-099 y RN-058 se contradicen al eliminar la última variante (M-01).
- QA: no cumple Small. Corrección sugerida: dividirla en HU-005a (Ropa), HU-005b (Perfume) y HU-005c (Accesorio) con los mismos criterios, más un criterio compartido de SKU y precio.
- M-12 resuelto con RN-135; M-01 resuelto con RN-136 y RN-137 (ver HU-006).

---

### HU-006: Editar, desactivar y reactivar mis productos

**Prioridad:** Must

**Como** emprendedor, **quiero** corregir el precio de una variante y desactivar o reactivar un producto, **para** mantener mi catálogo al día sin perder el historial de lo que ya vendí.

**Criterios de aceptación:**
- DADO QUE la variante `ROP-00001` (talla M, Negro) tiene ventas, CUANDO Ana cambia su precio de S/ 35.00 a S/ 32.00, ENTONCES el sistema lo guarda y el SKU sigue siendo `ROP-00001`; CUANDO intenta cambiar la talla de M a L, ENTONCES no se lo permite.
- DADO QUE una variante tiene costo promedio S/ 18.50, CUANDO Ana guarda un precio de venta de S/ 15.00, ENTONCES el sistema lo guarda y muestra "El precio de venta es menor que el costo"; DADO QUE otra variante no tiene entradas, CUANDO Ana guarda su precio de S/ 35.00, ENTONCES no muestra la advertencia.
- DADO QUE "Collar dorado" tuvo 4 ventas en octubre de 2026, CUANDO Ana lo desactiva, ENTONCES no aparece en la lista para registrar ventas y sus 4 ventas siguen en el historial; CUANDO Ana lo reactiva, ENTONCES vuelve a aparecer con su historial intacto.
- DADO QUE una variante tiene ventas, CUANDO Ana intenta eliminarla, ENTONCES el sistema no la elimina y le ofrece desactivar el producto; CUANDO intenta cambiar la categoría de un producto, ENTONCES el sistema no se lo permite.
- DADO QUE la variante de 100 ml de "Perfume Floral Rosé" tiene ventas y vencimiento `2028-06-30`, CUANDO Ana lo corrige a `2027-06-30`, ENTONCES el sistema lo guarda y registra una auditoría con la fecha y hora del cambio, el valor anterior y el valor nuevo.
- DADO QUE el producto 77 pertenece a Luis, CUANDO Ana pide `GET /api/products/77`, ENTONCES el sistema responde HTTP 404.
- [NUEVO] (Positivo) DADO QUE "Perfume Floral Rosé" tiene 2 variantes y la de 50 ml no tiene entradas, ventas ni ajustes, CUANDO Ana la elimina, ENTONCES el sistema la elimina y el producto conserva su otra variante. (RN-099)
- [NUEVO] (Positivo) DADO QUE la variante talla M de "Polo oversize básico" no tiene entradas, ventas ni ajustes y no existe la talla L, CUANDO Ana cambia la talla de M a L, ENTONCES el sistema la guarda; si ya existe la talla L con el mismo color, ENTONCES no la guarda. (RN-050, RN-098)
- [NUEVO] (Negativo) DADO QUE la variante de 100 ml tiene ventas, CUANDO Ana intenta cambiar su presentación a 200 ml, ENTONCES el sistema no se lo permite. (RN-098)
- [NUEVO] (Negativo) DADO QUE "Collar dorado" está desactivado y tiene stock 3, CUANDO se intenta registrar una venta de 1 unidad de su variante, ENTONCES el sistema no la registra y el stock sigue en 3. (RN-016)
- [NUEVO] (Negativo) DADO QUE el producto 77 pertenece a Luis, CUANDO Ana intenta modificarlo o desactivarlo con `PATCH /api/products/77`, ENTONCES el sistema responde HTTP 404 y el producto de Luis no cambia. (RN-005, PRINCIPIO-003)
- [NUEVO] (Negativo) DADO QUE existen dos variantes de 100 ml del mismo perfume, una con vencimiento `2028-06-30` y otra con `2029-01-31`, CUANDO Ana cambia el vencimiento de la segunda a `2028-06-30`, ENTONCES el sistema no lo guarda. (RN-050)
- [NUEVO] (Borde) DADO QUE Ana edita el precio de una variante, CUANDO ingresa S/ 99,999.99, ENTONCES el sistema lo guarda; CUANDO ingresa S/ 100,000.00 o S/ 35.999, ENTONCES no lo guarda. (RN-041, RN-091)
- [NUEVO] (Borde) DADO QUE el costo promedio de una variante es S/ 18.50, CUANDO Ana guarda el precio S/ 18.50, ENTONCES el sistema lo guarda sin advertencia; CUANDO guarda S/ 18.49, ENTONCES lo guarda y muestra "El precio de venta es menor que el costo". (RN-015)
- [NUEVO] (Borde) DADO QUE una variante de perfume tiene solo 1 entrada y ninguna venta ni ajuste, CUANDO Ana cambia su vencimiento de `2028-06-30` a `2027-06-30`, ENTONCES el sistema lo guarda y registra la auditoría con ambos valores. (RN-115)
- [AUDITORÍA v2] (Negativo) DADO QUE "Collar dorado" solo tiene la variante "Única" y no tiene movimientos, CUANDO Ana intenta eliminarla, ENTONCES el sistema no la elimina y le ofrece desactivar el producto. (RN-136)
- [AUDITORÍA v2] (Positivo) DADO QUE "Cartera negra" tiene stock 3, CUANDO Ana la desactiva y luego la reactiva, ENTONCES sigue con stock 3. (RN-137)
- [AUDITORÍA v3] (Positivo) DADO QUE "Collar dorado" está desactivado y tiene stock 4, CUANDO Ana registra un ajuste por conteo físico de 3 unidades, ENTONCES el sistema lo registra. (RN-147)

**Reglas de negocio relacionadas:** RN-005, RN-015, RN-016, RN-017, RN-050, RN-062, RN-066, RN-098, RN-099, RN-100, RN-115, RN-119, RN-136, RN-137, RN-147 (Constitución: PRINCIPIO-003)

**Notas:**
- Se puede desactivar con stock (RN-137); no se elimina la única variante (RN-136). Eliminar productos completos no está en el MVP.
- P-41 resuelto: en un producto desactivado no se venden ni se registran entradas, pero sí se anulan y se ajustan (RN-016, RN-140, RN-147). P-42 queda fuera del MVP: se desactiva el producto completo.
- RN-005 cubre pedir, modificar o anular recursos ajenos y citarlos por ID en el cuerpo (ampliada el 2026-11-11, pregunta P-68). T-033 lo prueba.
- RN-115 está confirmada por Sergio (A-04 cerrado).
- QA: mezcla editar precio, desactivar, eliminar y corregir vencimiento (Small parcial). Corrección sugerida: separar "Editar una variante" de "Desactivar y reactivar un producto" si el sprint lo pide.
- QA: no se prueba eliminar productos ni desactivar una sola variante (P-42), porque no están en el MVP. La auditoría del vencimiento (RN-115) agrega una tabla que debe llevar `tenant_id` (PRINCIPIO-003) y borrarse con la cuenta (PRINCIPIO-006).

---

## Épica 3: Entradas de mercadería y costo (F-03)

### HU-007: Registrar una entrada de mercadería con su costo

**Prioridad:** Must

**Como** emprendedor, **quiero** registrar las unidades que me llegan de un proveedor con su costo unitario, **para** que mi stock y mi costo promedio estén al día.

**Criterios de aceptación:**
- DADO QUE una variante tiene stock 10 y costo promedio S/ 18.00, CUANDO Ana registra una entrada de 10 unidades a S/ 20.00, ENTONCES el stock queda en 20 y el costo promedio en S/ 19.00.
- DADO QUE una variante tiene stock 1 y costo promedio S/ 10.00, CUANDO Ana registra una entrada de 1 unidad a S/ 10.01, ENTONCES el costo promedio queda en S/ 10.01 (10.005 redondeado hacia arriba).
- DADO QUE una variante nueva tiene stock 0, CUANDO Ana registra una entrada de 12 unidades a S/ 18.50, ENTONCES el stock queda en 12.
- DADO QUE Ana registra una entrada, CUANDO ingresa una cantidad de 12, ENTONCES el sistema la registra; CUANDO ingresa 0, 2.5 o 12,000, ENTONCES no la registra.
- DADO QUE Ana registra una entrada, CUANDO ingresa un costo de S/ 18.50, ENTONCES el sistema lo registra; CUANDO ingresa S/ 0.00, S/ -5.00, S/ 18.505 o S/ 150,000.00, ENTONCES no lo registra.
- [NUEVO] (Positivo) DADO QUE una variante nueva tiene stock 0 y ninguna entrada, CUANDO Ana registra una entrada de 12 unidades a S/ 18.50, ENTONCES el costo promedio queda en S/ 18.50. (RN-020, RN-061)
- [NUEVO] (Positivo) DADO QUE una variante tiene stock 10 y costo promedio S/ 18.00, CUANDO Ana registra una entrada de 5 unidades a S/ 20.01, ENTONCES el stock queda en 15, el costo promedio en S/ 18.67 y la entrada guarda "costo promedio anterior: S/ 18.00". (RN-020, RN-118)
- [NUEVO] (Negativo) DADO QUE una variante tiene stock 10 y costo promedio S/ 18.00, CUANDO Ana intenta registrar una entrada de 5 unidades con costo S/ 0.00, ENTONCES el sistema la rechaza, el stock sigue en 10 y el costo promedio sigue en S/ 18.00. (RN-014, RN-019, RN-020)
- [NUEVO] (Negativo) DADO QUE la variante 88 pertenece a Luis, CUANDO Ana registra una entrada de 5 unidades para la variante 88, ENTONCES el sistema responde HTTP 404 y el stock de Luis no cambia. (RN-005, PRINCIPIO-003)
- [NUEVO] (Borde) DADO QUE Ana registra una entrada, CUANDO ingresa 1 unidad o 9,999 unidades, ENTONCES el sistema la registra; CUANDO ingresa 10,000, ENTONCES no la registra. (RN-018, RN-092)
- [NUEVO] (Borde) DADO QUE Ana registra una entrada, CUANDO ingresa un costo de S/ 0.01 o S/ 99,999.99, ENTONCES el sistema la registra; CUANDO ingresa S/ 100,000.00, ENTONCES no la registra. (RN-014, RN-091)
- [AUDITORÍA v3] (Negativo) DADO QUE "Collar dorado" está desactivado, CUANDO Ana intenta registrar una entrada de 5 unidades, ENTONCES el sistema no la registra y muestra "Reactiva el producto para registrar mercadería"; CUANDO reactiva el producto, ENTONCES puede registrarla. (RN-140)
- [AUDITORÍA v3] (Positivo) DADO QUE Ana registra una entrada de la variante talla M negro, CUANDO la guarda, ENTONCES queda con la fecha y hora del registro, sin campo para elegir otra fecha, y solo para esa variante. (RN-142)
- [AUDITORÍA v3] (Positivo) DADO QUE Ana escribe el costo "18,50", CUANDO guarda la entrada, ENTONCES el costo unitario es S/ 18.50. (RN-152)

**Reglas de negocio relacionadas:** RN-005, RN-014, RN-018, RN-019, RN-020, RN-042, RN-047, RN-061, RN-091, RN-092, RN-118, RN-140, RN-142, RN-152 (Constitución: PRINCIPIO-003)

**Notas:**
- Cada entrada guarda el costo promedio que tenía la variante justo antes de registrarla (RN-118); HU-008 lo necesita para anular.
- P-50 resuelto con RN-142: una entrada es de una variante y lleva la fecha y hora del registro.
- QA: no se prueban dos entradas simultáneas de la misma variante (cómo se calcula el costo promedio) (las entradas a un producto desactivado están en RN-140).

---

### HU-008: Anular una entrada registrada por error

**Prioridad:** Must

**Como** emprendedor, **quiero** anular una entrada que registré por error, **para** que mi stock y mi costo promedio vuelvan a ser los correctos sin tocar nada a mano.

**Criterios de aceptación:**
- DADO QUE una variante tenía stock 10 y costo promedio S/ 18.00, Ana registró una entrada de 10 unidades a S/ 20.00 (stock 20, costo promedio S/ 19.00) y no hubo movimientos posteriores, CUANDO Ana la anula, ENTONCES el stock queda en 10, el costo promedio vuelve a S/ 18.00 y la entrada aparece en el historial con el estado "Anulada".
- DADO QUE después de esa entrada se registró una venta de 2 unidades de la misma variante, CUANDO Ana intenta anular la entrada, ENTONCES el sistema la rechaza con "No se puede anular: hay movimientos posteriores a esta entrada".
- DADO QUE una entrada se registró el 2026-11-01, CUANDO Ana intenta anularla el 2026-11-10, ENTONCES el sistema no se lo permite; DADO QUE se registró el 2026-11-10T20:00-05:00, CUANDO la anula el 2026-11-11T09:00-05:00, ENTONCES el sistema la anula.
- DADO QUE una entrada ya está "Anulada", CUANDO Ana intenta anularla otra vez, ENTONCES el sistema no la anula de nuevo y el stock no cambia.
- [NUEVO] (Positivo) DADO QUE después de la entrada se registró una venta de 2 unidades que Ana anuló, CUANDO Ana anula la entrada, ENTONCES el sistema la anula porque ya no hay ventas vigentes posteriores. (RN-056)
- [NUEVO] (Negativo) DADO QUE después de la entrada se registró un ajuste por conteo físico de la misma variante, CUANDO Ana intenta anular la entrada, ENTONCES el sistema la rechaza con "No se puede anular: hay movimientos posteriores a esta entrada". (RN-056)
- [NUEVO] (Negativo) DADO QUE después de la entrada se registró otra entrada de la misma variante, CUANDO Ana intenta anular la primera, ENTONCES el sistema la rechaza con "No se puede anular: hay movimientos posteriores a esta entrada". (RN-056)
- [NUEVO] (Negativo) DADO QUE la entrada 41 pertenece a Luis, CUANDO Ana intenta anularla, ENTONCES el sistema responde HTTP 404 y la entrada de Luis no cambia. (RN-005, PRINCIPIO-003)
- [NUEVO] (Borde) DADO QUE una entrada se registró el 2026-11-10T20:00:00-05:00, CUANDO Ana la anula el 2026-11-11T20:00:00-05:00 (exactamente 86,400 segundos), ENTONCES el sistema la anula; CUANDO intenta anularla a las 20:00:01 (86,401 segundos), ENTONCES no se lo permite. (RN-087)
- [NUEVO] (Borde) DADO QUE una variante nueva tenía stock 0 y Ana registró una entrada de 12 unidades por error, CUANDO la anula de inmediato, ENTONCES el stock vuelve a 0 y la entrada aparece "Anulada" en el historial. (RN-054, RN-055)
- [AUDITORÍA v4] (Borde) DADO QUE una variante nueva tiene una sola entrada de 12 unidades a S/ 18.50 y ningún otro movimiento, CUANDO Ana la anula, ENTONCES el stock queda en 0, la variante muestra "Costo promedio: sin entradas" y no tiene costo promedio. (RN-088, RN-154; aprobadas)

**Reglas de negocio relacionadas:** RN-005, RN-054, RN-055, RN-056, RN-057, RN-087, RN-088, RN-118, RN-154 (Constitución: PRINCIPIO-003)

**Notas:**
- RN-056 y RN-088 se redactaron en la revisión QA con la solución más simple: solo se anula una entrada si no hay movimientos posteriores. Para corregir una entrada ya vendida, se registra una entrada nueva con el costo correcto.
- La auditoría señaló que la redacción de RN-056 ("después de una entrada") debe precisar de cuál (M-12), RN-056, RN-088 y RN-118 están confirmadas por Sergio (A-04 cerrado).
- Auditoría QA v4 (2026-11-11): RN-154 aprobada (P-64). El criterio [AUDITORÍA v4] de HU-008 ya se implementa en T-020.

---

## Épica 4: Ventas y ganancia (F-04, F-06)

### HU-009: Registrar una venta con una o más variantes

**Prioridad:** Must

**Como** emprendedor, **quiero** registrar una venta con una o más variantes y el precio al que realmente vendí, **para** que el stock se descuente al instante y no tenga que recontar después del live.

**Criterios de aceptación:**
- DADO QUE el perfume A de 100 ml tiene stock 15 y precio de venta S/ 35.00, CUANDO Ana registra una venta de 2 unidades a S/ 30.00 cada una, ENTONCES la venta queda con precio final S/ 30.00 por unidad y el stock queda en 13.
- DADO QUE Ana registra una venta con 4 líneas (polo A verde talla L x1, polo A marrón talla XS x1, perfume A de 200 ml x1 y perfume A de 100 ml x1), CUANDO la confirma, ENTONCES el sistema descuenta 1 unidad del stock de cada una de las 4 variantes.
- DADO QUE el polo A marrón XS tiene stock 0 y la venta tiene 4 líneas, CUANDO Ana intenta registrarla, ENTONCES el sistema no la registra y marca esa línea con "Stock insuficiente"; CUANDO Ana quita esa línea, ENTONCES puede registrar las otras 3. DADO QUE una variante tiene stock 1 y se registran a la vez 1 unidad desde el celular y 1 desde la laptop, ENTONCES una venta se registra y la otra recibe "Stock insuficiente: quedan 0 unidades".
- DADO QUE la venta tiene el perfume A de 100 ml a S/ 35.00, CUANDO Ana lo agrega otra vez a S/ 35.00, ENTONCES la venta muestra una sola línea "Perfume A 100 ml x2"; CUANDO lo agrega a S/ 30.00, ENTONCES el sistema no lo agrega y muestra "Esta variante ya está en la venta con otro precio. Edita esa línea".
- DADO QUE hoy es 2026-11-10 y el perfume vence el 2026-11-09, CUANDO Ana intenta venderlo, ENTONCES el sistema lo rechaza con "Este perfume está vencido y no se puede vender"; si vence el 2026-11-10, la venta se registra.
- DADO QUE una variante tiene costo promedio S/ 19.00, CUANDO Ana registra 1 unidad a S/ 15.00, ENTONCES el sistema muestra "Esta venta genera una pérdida de S/ 4.00" y la registra solo cuando Ana confirma.
- DADO QUE Ana registra una línea, CUANDO ingresa una cantidad de 2 y un precio final de S/ 29.90, ENTONCES el sistema la acepta; CUANDO ingresa cantidad 0 o 10,000, o precio final S/ 0.00 o S/ 29.995, ENTONCES no la acepta.
- [AJUSTADO v2] DADO QUE hoy es 2026-11-10 a las 10:00, CUANDO Ana registra una venta eligiendo el 2026-11-04 a las 21:00, ENTONCES el sistema la acepta; CUANDO elige el 2026-10-01 o el 2026-11-12, ENTONCES no la acepta. (RN-076, RN-077, RN-078)
- [NUEVO] (Positivo) DADO QUE Ana agrega el perfume A de 100 ml y el perfume A de 200 ml, CUANDO ve la venta, ENTONCES aparecen 2 líneas separadas, una por cada variante. (RN-107)
- [NUEVO] (Negativo) DADO QUE la venta tiene 4 líneas y el polo A marrón XS tiene stock 0, CUANDO Ana intenta registrarla, ENTONCES el sistema no registra la venta y el stock de las otras 3 variantes sigue igual. (RN-081)
- [NUEVO] (Negativo) DADO QUE Ana confirma una venta sin ninguna línea, CUANDO el sistema la valida, ENTONCES no la registra. (RN-080)
- [NUEVO] (Negativo) DADO QUE "Collar dorado" está desactivado, CUANDO Ana busca su variante para agregarla a una venta, ENTONCES no aparece y no se puede vender. (RN-016)
- [NUEVO] (Negativo) DADO QUE el costo promedio es S/ 19.00 y Ana intenta vender 1 unidad a S/ 15.00, CUANDO el sistema pide confirmación y Ana la rechaza, ENTONCES no se registra la venta y el stock no cambia. (RN-073)
- [NUEVO] (Negativo) DADO QUE la variante 88 pertenece a Luis, CUANDO Ana incluye esa variante en una línea de venta, ENTONCES el sistema responde HTTP 404 y no registra la venta. (RN-005, PRINCIPIO-003)
- [NUEVO] (Negativo) DADO QUE Ana registra una venta, CUANDO ve el formulario, ENTONCES no existen campos para el nombre, el teléfono, la dirección ni el DNI del comprador. (PRINCIPIO-005)
- [NUEVO] (Borde) DADO QUE una variante tiene stock 2, CUANDO Ana registra una venta de 2 unidades, ENTONCES el sistema la registra y el stock queda en 0; CUANDO intenta vender 3, ENTONCES la rechaza con "Stock insuficiente: quedan 2 unidades". (RN-023)
- [NUEVO] (Borde) DADO QUE una variante tiene stock 9,999, CUANDO Ana registra una línea de 9,999 unidades, ENTONCES el sistema la registra y el stock queda en 0. (RN-092)
- [NUEVO] (Borde) DADO QUE Ana registra una línea, CUANDO ingresa un precio final de S/ 0.01 o S/ 99,999.99, ENTONCES el sistema lo acepta; CUANDO ingresa S/ 100,000.00, ENTONCES no lo acepta. (RN-025, RN-091)
- [NUEVO] (Borde) DADO QUE el costo promedio es S/ 19.00, CUANDO Ana vende 1 unidad a S/ 19.00, ENTONCES el sistema la registra sin pedir confirmación y con ganancia S/ 0.00; CUANDO la vende a S/ 18.99, ENTONCES pide confirmación con "Esta venta genera una pérdida de S/ 0.01". (RN-072, RN-073)
- [NUEVO] (Borde) DADO QUE una variante tiene stock 1, CUANDO Ana agrega esa variante dos veces a S/ 35.00, ENTONCES el sistema rechaza la línea fusionada de 2 unidades por falta de stock y muestra "Stock insuficiente" con 1 unidad disponible. (RN-023, RN-084)
- [AJUSTADO v2] (Borde) DADO QUE hoy es 2026-11-10 a las 10:00, CUANDO Ana registra una venta con fecha 2026-11-10 a las 09:30, ENTONCES el sistema la acepta; CUANDO elige las 18:00 de ese mismo día, ENTONCES no la acepta. (RN-078)
- [AUDITORÍA v2] (Positivo) DADO QUE Ana no cambia la fecha ni la hora, CUANDO registra una venta el 2026-11-10 a las 21:30:12, ENTONCES la venta queda con fecha y hora 2026-11-10T21:30:12-05:00. (RN-126)
- [AUDITORÍA v2] (Borde) DADO QUE hoy es 2026-11-10, CUANDO Ana registra una venta con fecha 2026-11-03 a las 08:00, ENTONCES el sistema la acepta; CUANDO elige el 2026-11-02 a las 23:59, ENTONCES no la acepta. (RN-077)
- [AUDITORÍA v2] (Positivo) DADO QUE una venta tiene dos líneas con pérdidas de S/ 4.00 y S/ 1.50, CUANDO Ana la registra, ENTONCES el sistema pide una sola confirmación con "Esta venta genera una pérdida de S/ 5.50". (RN-073)
- [AUDITORÍA v3] (Negativo) DADO QUE una línea tiene 9,990 unidades de una variante con stock suficiente, CUANDO Ana agrega 20 unidades más de la misma variante al mismo precio, ENTONCES el sistema no las agrega y muestra "La cantidad máxima por línea es 9,999". (RN-151)
- [AUDITORÍA v3] (Positivo) DADO QUE una variante tiene stock 1, CUANDO Ana intenta vender 2 unidades, ENTONCES el sistema muestra "Stock insuficiente: queda 1 unidad". (RN-023, RN-150)
- [AUDITORÍA v3] (Positivo) DADO QUE Ana escribe el precio final "29,90", CUANDO registra la venta, ENTONCES la línea queda con S/ 29.90. (RN-152)

**Reglas de negocio relacionadas:** RN-005, RN-016, RN-021, RN-022, RN-023, RN-025, RN-043, RN-045, RN-072, RN-073, RN-074, RN-076, RN-077, RN-078, RN-080, RN-081, RN-084, RN-085, RN-091, RN-092, RN-106, RN-107, RN-125, RN-126, RN-150, RN-151, RN-152 (Constitución: PRINCIPIO-003)

**Notas:**
- Es la HU más grande del lote por la riqueza de la venta con varias líneas; puede dividirse en "venta de una línea", "venta con varias líneas" y "venta con fecha pasada" sin cambiar los criterios. Es el flujo de mayor riesgo del piloto (R-01): durante un live debe poder completarse en pocos toques.
- No se guardan datos del comprador (PRINCIPIO-005, W-04).
- Resueltos: P-38 (el vencimiento se compara con hoy, RN-074), P-39 (la venta guarda fecha y hora; por defecto el momento del registro, RN-126), P-40 (una sola confirmación, RN-073). La auditoría señaló que sumar líneas puede superar 9,999 unidades (B-08).
- QA: no cumple Small. Corrección sugerida: dividirla en HU-009a "Registrar una venta de una línea", HU-009b "Registrar una venta con varias líneas" y HU-009c "Registrar una venta con fecha pasada".
- QA: el singular con 1 unidad está en RN-150 (B-02) y la línea fusionada mayor que 9,999 en RN-151 (B-08). El criterio 3 original junta dos escenarios; se mantiene para no perder trazabilidad.

---

### HU-010: Anular una venta o una línea registrada por error

**Prioridad:** Must

**Como** emprendedor, **quiero** anular una venta completa o solo una de sus líneas, **para** corregir un error de registro sin dejar el stock ni la ganancia mal.

**Criterios de aceptación:**
- DADO QUE una venta de 4 líneas con ganancia S/ 93.00 se registró hace 2 horas, CUANDO Ana la anula, ENTONCES las 4 líneas quedan "Anulada", cada variante recupera la cantidad de su línea y los S/ 93.00 dejan de sumarse a la ganancia.
- DADO QUE una venta de 4 líneas se registró hace 2 horas, CUANDO Ana anula solo el polo A verde talla L, ENTONCES ese polo vuelve al stock y las otras 3 líneas siguen vigentes.
- DADO QUE una venta tiene 2 líneas, CUANDO Ana anula la primera, ENTONCES la venta sigue vigente; CUANDO anula la segunda, ENTONCES la venta pasa a "Anulada".
- DADO QUE una venta se registró el 2026-11-10T20:00-05:00, CUANDO Ana la anula el 2026-11-11T09:00-05:00, ENTONCES el sistema la anula; DADO QUE se registró el 2026-11-01, CUANDO Ana intenta anularla el 2026-11-10, ENTONCES no se lo permite.
- DADO QUE una venta de 2 unidades ya está "Anulada", CUANDO Ana intenta anularla otra vez, ENTONCES el stock no sube otra vez y la venta sigue "Anulada".
- [NUEVO] (Positivo) DADO QUE el stock de una variante es 13 por una venta de 2 unidades registrada por error, CUANDO Ana anula esa venta, ENTONCES el stock queda en 15. (RN-052)
- [NUEVO] (Positivo) DADO QUE una venta de 4 líneas tiene ganancia S/ 93.00 y la línea del polo A verde talla L aporta S/ 16.00, CUANDO Ana anula solo esa línea, ENTONCES la ganancia de la venta pasa a S/ 77.00. (RN-053, RN-082, RN-086)
- [NUEVO] (Negativo) DADO QUE una línea de venta se registró hace 2 días, CUANDO Ana intenta anularla, ENTONCES el sistema no se lo permite. (RN-087)
- [NUEVO] (Negativo) DADO QUE la línea del polo A verde talla L ya está "Anulada", CUANDO Ana intenta anularla otra vez, ENTONCES el stock no cambia. (RN-057)
- [NUEVO] (Negativo) DADO QUE se envían a la vez dos anulaciones de la misma venta de 2 unidades, CUANDO el sistema las procesa, ENTONCES el stock sube 2 una sola vez. (RN-057)
- [NUEVO] (Negativo) DADO QUE la venta 305 pertenece a Luis, CUANDO Ana intenta anularla, ENTONCES el sistema responde HTTP 404 y la venta de Luis no cambia. (RN-005, PRINCIPIO-003)
- [NUEVO] (Borde) DADO QUE una venta se registró el 2026-11-10T20:00:00-05:00, CUANDO Ana la anula el 2026-11-11T20:00:00-05:00 (exactamente 86,400 segundos), ENTONCES el sistema la anula; CUANDO intenta anularla a las 20:00:01, ENTONCES no se lo permite. (RN-087)
- [NUEVO] (Borde) DADO QUE una venta de una sola línea está vigente, CUANDO Ana anula esa línea, ENTONCES la venta pasa a "Anulada". (RN-086, RN-109)
- [NUEVO] (Borde) DADO QUE el 2026-11-10T10:00:00-05:00 Ana registró una venta con fecha de venta 2026-11-05, CUANDO la anula el 2026-11-10T15:00:00-05:00, ENTONCES el sistema la anula porque las 24 horas se cuentan desde el registro. (RN-087)
- [AUDITORÍA v2] (Negativo) DADO QUE Ana vendió 2 polos a las 20:00 y ajustó el stock de esa variante a las 21:00, CUANDO intenta anular la venta o esa línea a las 21:30, ENTONCES el sistema no la anula y muestra "No se puede anular: hay un ajuste de stock posterior a esta venta". (RN-139)
- [AUDITORÍA v3] (Positivo) DADO QUE "Collar dorado" fue desactivado después de una venta registrada hace 2 horas, CUANDO Ana anula esa venta, ENTONCES el sistema la anula y el stock sube. (RN-147)

**Reglas de negocio relacionadas:** RN-005, RN-052, RN-053, RN-054, RN-057, RN-083, RN-086, RN-087, RN-109, RN-139, RN-147 (Constitución: PRINCIPIO-003)

**Notas:**
- A-01 resuelto con RN-139: no se anula una venta con un ajuste posterior de alguna de sus variantes.
- El plazo de 24 horas se cuenta desde el registro, no desde la fecha y hora de venta elegida (RN-087).

---

### HU-011: Consultar el historial de ventas y la ganancia de cada una

**Prioridad:** Must

**Como** emprendedor, **quiero** ver cuánto gané en cada venta y revisar mi historial de ventas, **para** saber qué me conviene reponer y cuánto gano por lo que vendo.

**Criterios de aceptación:**
- DADO QUE Ana vendió 2 polos a S/ 35.00 por unidad con costo promedio S/ 19.00, CUANDO abre la venta, ENTONCES ve la ganancia "S/ 32.00".
- DADO QUE una venta tiene 4 líneas con ganancias S/ 16.00, S/ 12.00, S/ 40.00 y S/ 25.00, CUANDO Ana abre la venta, ENTONCES ve la ganancia total "S/ 93.00".
- DADO QUE la venta del 2026-11-10 tuvo ganancia S/ 32.00, CUANDO el 2026-11-15 entra mercadería más cara, ENTONCES esa venta sigue mostrando "S/ 32.00".
- DADO QUE el 2026-11-09 el costo promedio era S/ 18.00 y el 2026-11-10 es S/ 19.00, CUANDO Ana registra el 2026-11-10 una venta con fecha 2026-11-09, ENTONCES su ganancia se calcula con S/ 19.00.
- DADO QUE un polo con costo promedio S/ 19.00 se remató a S/ 15.00, CUANDO Ana abre la venta, ENTONCES ve la ganancia "S/ -4.00".
- DADO QUE una venta está "Anulada", CUANDO Ana revisa su historial, ENTONCES la ve con el estado "Anulada" y su ganancia no se suma.
- [AJUSTADO] DADO QUE la ganancia de una venta es 1250.5, CUANDO Ana la ve en pantalla, ENTONCES se muestra "S/ 1,250.50". (El criterio original hablaba de ganancia mensual, que es F-09.)
- [NUEVO] (Positivo) DADO QUE Ana vendió 3 polos a S/ 35.00 por unidad con costo promedio S/ 18.67, CUANDO abre la venta, ENTONCES ve la ganancia "S/ 48.99". (RN-027)
- [NUEVO] (Negativo) DADO QUE la venta 305 pertenece a Luis, CUANDO Ana la pide, ENTONCES el sistema responde HTTP 404. (RN-005, PRINCIPIO-003)
- [NUEVO] (Borde) DADO QUE Ana vendió 1 polo a S/ 19.00 con costo promedio S/ 19.00, CUANDO abre la venta, ENTONCES ve la ganancia "S/ 0.00". (RN-027)
- [NUEVO] (Borde) DADO QUE una venta tiene una sola línea con ganancia S/ 32.00, CUANDO Ana abre la venta, ENTONCES la ganancia de la venta es "S/ 32.00". (RN-082)
- [NUEVO] (Borde) DADO QUE Ana no registró ninguna venta, CUANDO abre su historial de ventas, ENTONCES ve el historial sin ventas y sin errores. (RN-035)
- [AUDITORÍA v3] (Positivo) DADO QUE una venta tiene ganancia -S/ 4.00 y se registró el 2026-11-10 a las 21:30, CUANDO Ana ve el historial, ENTONCES muestra "-S/ 4.00" y "10/11/2026 21:30". (RN-148, RN-149)

**Reglas de negocio relacionadas:** RN-005, RN-027, RN-028, RN-035, RN-036, RN-053, RN-054, RN-072, RN-079, RN-082, RN-148, RN-149 (Constitución: PRINCIPIO-003)

**Notas:**
- El resumen de ganancias por semana y por mes es F-09 (Should, fuera de este lote). Los ejemplos de las reglas que mencionan "ganancia del día" no tienen una pantalla propia en F-06 (M-10).
- P-54 resuelto: cada línea guarda el costo usado (RN-028, RN-079). P-58 resuelto con RN-148, RN-149 y RN-152.
- QA: el criterio original sobre la ganancia mensual era de F-09 (Should, fuera de este lote); se ajustó para usar la ganancia de una venta.
- QA: no se prueban el orden del historial ni los filtros (no hay regla) (el orden es `sale_at` descendente).

---

## Épica 5: Stock (F-05)

### HU-012: Consultar mi stock y corregirlo con un ajuste por conteo físico

**Prioridad:** Must

**Como** emprendedor, **quiero** ver cuántas unidades me quedan de cada variante y corregir el stock cuando mi conteo físico no coincide, **para** dejar de recontar toda mi mercadería y confiar en lo que muestra la app.

**Criterios de aceptación:**
- DADO QUE una variante tuvo entradas de 12 y 6 unidades, ventas de 2, 3 y 1 unidades y un ajuste de -1, CUANDO Ana consulta su stock, ENTONCES ve 11 unidades.
- DADO QUE el stock en la app es 8, CUANDO Ana registra un ajuste con cantidad contada 6 y el motivo "Conteo físico: venta del live del 2026-11-09 no registrada", ENTONCES el stock queda en 6 y el historial muestra una diferencia de -2.
- DADO QUE Ana registra un ajuste, CUANDO deja el motivo vacío, ENTONCES el sistema no lo registra; CUANDO ingresa una cantidad contada de -3 o de 10,000, ENTONCES no lo registra; CUANDO ingresa 0, ENTONCES lo registra.
- [AJUSTADO] DADO QUE el costo promedio es S/ 19.00 y la venta del 2026-11-10 tiene ganancia S/ 32.00, CUANDO Ana registra un ajuste de -2 unidades, ENTONCES el costo promedio sigue en S/ 19.00, la ganancia de esa venta sigue en S/ 32.00 y el ajuste no aparece en el historial de ventas. (El criterio original usaba una "ganancia del día" que ninguna HU de este lote muestra.)
- DADO QUE una variante no tiene entradas vigentes y tiene stock 0, CUANDO Ana registra un ajuste con cantidad contada 3, ENTONCES el sistema lo rechaza con "Registra primero una entrada con su costo"; DADO QUE otra variante tiene una entrada vigente de 10 unidades y stock 8, CUANDO Ana registra un ajuste con cantidad contada 9, ENTONCES el sistema lo registra.
- DADO QUE Ana ajusta de 4 a 1 unidad un perfume de 100 ml vencido el 2026-11-09, CUANDO elige el motivo "Merma por vencimiento", ENTONCES el ajuste queda en el historial con ese motivo.
- [AJUSTADO] DADO QUE el stock de una variante es 15, CUANDO se envía una petición que intenta fijar el stock directamente en 13, ENTONCES el sistema la rechaza y el stock sigue en 15; el stock solo cambia con una entrada, una venta, un ajuste o una anulación. (RN-024)
- [NUEVO] (Positivo) DADO QUE una variante tuvo una entrada de 12 unidades anulada, una entrada de 6, una venta de 2 unidades, una venta de 3 unidades anulada y un ajuste de -1, CUANDO Ana consulta su stock, ENTONCES ve 3 unidades. (RN-026)
- [NUEVO] (Positivo) DADO QUE el stock en la app es 8, CUANDO Ana registra un ajuste con cantidad contada 9 y un motivo, ENTONCES el stock queda en 9 y el historial muestra una diferencia de +1. (RN-037)
- [NUEVO] (Negativo) DADO QUE la variante 88 pertenece a Luis, CUANDO Ana intenta ajustar su stock, ENTONCES el sistema responde HTTP 404 y el stock de Luis no cambia. (RN-005, PRINCIPIO-003)
- [NUEVO] (Negativo) DADO QUE una variante tiene todas sus entradas anuladas y stock 0, CUANDO Ana registra un ajuste con cantidad contada 2, ENTONCES el sistema lo rechaza con "Registra primero una entrada con su costo". (RN-116)
- [NUEVO] (Borde) DADO QUE el stock en la app es 8, CUANDO Ana registra un ajuste con cantidad contada 0, ENTONCES el stock queda en 0 con diferencia -8; CUANDO intenta vender 1 unidad, ENTONCES la venta se rechaza con "Stock insuficiente: quedan 0 unidades". (RN-023, RN-037)
- [NUEVO] (Borde) DADO QUE Ana registra un ajuste, CUANDO ingresa una cantidad contada de 9,999, ENTONCES el sistema lo registra. (RN-040, RN-092)
- [NUEVO] (Borde) DADO QUE una variante tiene una entrada vigente de 10 unidades que se vendieron todas (stock 0), CUANDO Ana registra un ajuste con cantidad contada 1, ENTONCES el sistema lo registra porque la variante sí tiene una entrada vigente. (RN-116)
- [NUEVO] (Borde) DADO QUE Ana no tiene ningún producto, CUANDO consulta su stock, ENTONCES ve la lista vacía sin errores. (RN-035)
- [AUDITORÍA v2] (Positivo) DADO QUE Ana tiene la variante `ROP-00001` talla M negro de "Polo oversize básico", CUANDO busca "polo negro" o "rop-00001", ENTONCES la encuentra. (RN-138)
- [AUDITORÍA v3] (Negativo) DADO QUE Ana registra un ajuste con el motivo "   " (solo espacios) o de 201 caracteres, CUANDO lo envía, ENTONCES el sistema no lo registra; con 1 y con 200 caracteres sí lo registra. (RN-143)
- [AUDITORÍA v3] (Borde) DADO QUE el stock es 8, CUANDO Ana registra un ajuste con cantidad contada 8, ENTONCES el sistema lo registra con diferencia 0. (RN-144)
- [AUDITORÍA v3] (Negativo) DADO QUE existe un ajuste en el historial de una variante, CUANDO Ana lo revisa, ENTONCES no hay opción de anularlo y lo corrige con otro ajuste. (RN-145)
- [AUDITORÍA v3] (Positivo) DADO QUE una variante tiene stock 1, CUANDO Ana la consulta, ENTONCES ve "1 unidad"; con stock 0 ve "0 unidades". (RN-150)

**Reglas de negocio relacionadas:** RN-005, RN-023, RN-024, RN-026, RN-037, RN-038, RN-039, RN-040, RN-044, RN-075, RN-089, RN-092, RN-116, RN-138, RN-143, RN-144, RN-145, RN-150 (Constitución: PRINCIPIO-003)

**Notas:**
- La búsqueda está definida por RN-138 (P-51 resuelto).
- RN-075 ofrece "Merma por vencimiento" en todo ajuste de perfume, porque el sistema no puede saber el motivo; el alcance quedó confirmado el 2026-11-11 (pregunta P-65, opción A). El registro de mermas con costo es F-17 (Should, fuera de este lote).
- P-37 resuelto: el stock se guarda en la variante (ADR-0004). P-56 resuelto con RN-143, RN-144 y RN-145. El motivo es texto libre; P-014 muestra el aviso "No escribas datos de tus clientes" (B-06, mitigado).
- La alerta de stock bajo y el stock mínimo son F-07 (Should, fuera de este lote).
- QA: no cumple Small del todo (consulta y ajuste son dos objetivos). Corrección sugerida: dividirla en HU-012a "Consultar mi stock" y HU-012b "Ajustar el stock por conteo físico".

---

## Mapa de especificaciones por funcionalidad

La especificación de cada funcionalidad del MVP es la unión de sus historias (con sus criterios y reglas relacionadas, más abajo), el contrato y los datos de `07-architecture.md` (sección 7 indica endpoints y tablas por historia) y sus tareas de `08-backlog.md` (ADR-0013). No hay archivos de especificación aparte.

| Funcionalidad | Nombre | Historias | Tareas propias |
|---|---|---|---|
| F-01 | Cuenta del emprendedor | HU-001, HU-002, HU-003, HU-004 | T-005, T-011, T-012a, T-012b, T-012c, T-013, T-014, T-025a, T-025b, T-026, T-032, T-035, T-037, T-038, T-039 |
| F-02 | Catálogo con variantes | HU-005, HU-006 | T-006, T-015, T-016, T-017, T-027 |
| F-03 | Entradas de mercadería | HU-007, HU-008 | T-010, T-019, T-020, T-028 |
| F-04 | Salidas por venta | HU-009, HU-010 | T-010, T-018, T-022, T-023, T-029a, T-029b, T-029c, T-029d, T-030, T-034 |
| F-05 | Consulta y ajuste de stock | HU-012 | T-010, T-018, T-021, T-028, T-034 |
| F-06 | Ganancia por venta | HU-011 | T-024, T-030 |

Tareas transversales que aplican a todas: T-000 a T-004 (incluidas T-002a y T-002b, base de la UI), T-008 a T-010 (base técnica), T-031 a T-036, T-040 y T-041 (infraestructura, pruebas y despliegue). Las pantallas de cada historia están en `09-ui-spec.md` (sección 1) y su tarea en el mapa pantalla a tarea de `08-backlog.md`.

## Matriz de trazabilidad

| HU | Feature | Reglas de negocio |
|---|---|---|
| HU-001 | F-01 | RN-001, RN-002, RN-003, RN-035, RN-048, RN-094, RN-120, RN-133, RN-134, RN-141, RN-153 |
| HU-002 | F-01 | RN-004, RN-048, RN-049, RN-097, RN-103, RN-120, RN-130 a RN-132, RN-155 |
| HU-003 | F-01 | RN-002, RN-095, RN-096, RN-105, RN-110, RN-134, RN-141, RN-156, RN-157 |
| HU-004 | F-01 | RN-006, RN-007, RN-046, RN-090, RN-103, RN-115, RN-127 a RN-129 |
| HU-005 | F-02 | RN-008 a RN-013, RN-024, RN-041, RN-050, RN-058 a RN-061, RN-064, RN-065, RN-091, RN-093, RN-099, RN-108, RN-111 a RN-113, RN-121 a RN-124, RN-135, RN-146 |
| HU-006 | F-02 | RN-005, RN-015 a RN-017, RN-050, RN-062, RN-066, RN-098 a RN-100, RN-115, RN-119, RN-136, RN-137, RN-147 |
| HU-007 | F-03 | RN-005, RN-014, RN-018 a RN-020, RN-042, RN-047, RN-061, RN-091, RN-092, RN-118, RN-140, RN-142, RN-152 |
| HU-008 | F-03 | RN-005, RN-054 a RN-057, RN-087, RN-088, RN-118, RN-154 |
| HU-009 | F-04 | RN-005, RN-016, RN-021 a RN-023, RN-025, RN-043, RN-045, RN-072 a RN-074, RN-076 a RN-078, RN-080, RN-081, RN-084, RN-085, RN-091, RN-092, RN-106, RN-107, RN-125, RN-126, RN-150, RN-151, RN-152 |
| HU-010 | F-04 | RN-005, RN-052 a RN-054, RN-057, RN-083, RN-086, RN-087, RN-109, RN-139, RN-147 |
| HU-011 | F-06 | RN-005, RN-027, RN-028, RN-035, RN-036, RN-053, RN-054, RN-072, RN-079, RN-082, RN-148, RN-149 |
| HU-012 | F-05 | RN-005, RN-023, RN-024, RN-026, RN-037 a RN-040, RN-044, RN-075, RN-089, RN-092, RN-116, RN-138, RN-143, RN-144, RN-145, RN-150 |

## Verificación contra la constitución

Ningún criterio contradice la constitución. Puntos a vigilar:

| Principio | Hallazgo | Dónde |
|---|---|---|
| PRINCIPIO-001 | Todos los mensajes de los criterios están en español; los nombres técnicos (`/api/products`, `password_hash`, `closed_at`) están en inglés como exige el código. | Todas |
| PRINCIPIO-003 | Todo endpoint con ID tiene su criterio de 404 (T-02 y un criterio por HU). RN-005 cubre pedir, modificar y anular recursos ajenos y los IDs dentro del cuerpo de la petición (ampliada el 2026-11-11, pregunta P-68). RN-114 (Administrador sin `tenant_id`) está respaldada por el ADR 0002, aprobado; la sesión del Administrador queda en P-70. | HU-006 a HU-012, HU-013 |
| PRINCIPIO-004 | El hash `$argon2id$`, los 12 caracteres y el bloqueo de 900 segundos están en HU-001, HU-002 y HU-003. HSTS, TLS y el cifrado AES-256 no tienen criterio de usuario: son pruebas de infraestructura. | HU-001 a HU-003 |
| PRINCIPIO-005 | `privacy_accepted_at` es un dato de cuenta reconocido por el ADR 0010 (PRINCIPIO-005 actualizado). El motivo del ajuste es texto libre; se mitiga con el aviso de P-014 (B-06). | HU-001, HU-012 |
| PRINCIPIO-006 | El borde exacto de 2,592,000 segundos no es verificable con un proceso diario; se prueba a 2,591,999 segundos. Las auditorías de vencimiento (RN-115) deben llevar `tenant_id` para borrarse con la cuenta. | HU-004, HU-006 |
| PRINCIPIO-007 | Los logs no contienen correo, nombre ni contraseña (HU-001, HU-002, T-05). | HU-001, HU-002 |

## Reglas sin HU

- Reglas MUST sin HU: ninguna. Todas las reglas con prioridad MUST de `03-reglas-negocio.md` están referenciadas por al menos una HU, y cada una tiene ahora al menos un criterio.
- Reglas Should y Could sin HU: RN-029 a RN-034, RN-051, RN-063, RN-067 a RN-071, RN-101, RN-102, RN-104 y RN-114. Están cubiertas por las HU sugeridas de abajo.
- Funcionalidades sin ninguna regla de negocio (no se pueden escribir HU sin inventar reglas): F-12 (exportar reportes), F-13 (gráficos), F-14 (ayudantes de solo lectura), F-15 (proveedores) y F-17 (mermas con costo).

---

## HUs sugeridas

Las HU-013 a HU-016 son Should y las HU-017 y HU-018 son Could. Siguen el mismo formato; todas sus reglas ya existen, y las preguntas abiertas que las afectan están en las notas.

### HU-013: Administrar las cuentas que no pagan

**Prioridad:** Should

**Como** Administrador (Responsable del producto), **quiero** ver las cuentas y suspender o reactivar las que no pagan, **para** cobrar la mensualidad de S/ 25 sin entrar al inventario de nadie.

**Criterios de aceptación:**
- (Positivo) DADO QUE existe la cuenta de Ana, CUANDO el Administrador abre la pantalla de administración, ENTONCES ve el nombre, el correo y el estado de la cuenta de Ana, y no ve sus productos, variantes, entradas, ventas ni ajustes.
- (Positivo) DADO QUE la gratuidad de Ana terminó el 2027-01-31 sin pago, CUANDO el Administrador marca su cuenta "Suspendida" el 2027-02-01, ENTONCES el 2027-02-02 Ana no puede iniciar sesión y ve "Tu cuenta está suspendida por falta de pago".
- (Positivo) DADO QUE Ana pagó el 2027-02-05, CUANDO el Administrador reactiva su cuenta, ENTONCES Ana inicia sesión con su inventario intacto.
- (Negativo) DADO QUE Ana no tiene el rol Administrador, CUANDO intenta suspender la cuenta de Luis, ENTONCES el sistema lo rechaza.
- (Negativo) DADO QUE un visitante se registra, CUANDO el sistema crea su cuenta, ENTONCES la cuenta nunca tiene el rol Administrador.
- (Borde) DADO QUE la cuenta de Ana se suspendió el 2027-02-01T00:00:00-05:00, CUANDO pasan 2,592,000 segundos (2027-03-03T00:00:00-05:00), ENTONCES la cuenta pasa a cerrada y su borrado definitivo vence el 2027-04-02T00:00:00-05:00.
- (Negativo) DADO QUE la cuenta de Luis pasó a cerrada por falta de pago, CUANDO Luis inicia sesión, ENTONCES ve "Contacta al administrador" y no puede reabrirla.

**Reglas de negocio relacionadas:** RN-063, RN-067, RN-068, RN-069, RN-070, RN-071, RN-104, RN-114 (Constitución: PRINCIPIO-003, PRINCIPIO-008)

**Notas:**
- Falta una regla que diga quién reabre una cuenta cerrada por falta de pago y cómo se distingue del cierre voluntario: pendiente en P-69. Depende del ADR 0002 (aprobado), de P-45 y de P-70.

---

### HU-014: Ver una alerta cuando el stock de una variante es bajo

**Prioridad:** Should

**Como** emprendedor, **quiero** definir un stock mínimo por variante y ver una alerta cuando baja de ese mínimo, **para** saber qué reponer antes de quedarme sin mercadería.

**Criterios de aceptación:**
- (Positivo) DADO QUE el stock mínimo de una variante es 3, CUANDO su stock actual es 3, ENTONCES aparece la alerta "Stock bajo: quedan 3 unidades".
- (Negativo) DADO QUE el stock mínimo es 3, CUANDO el stock actual es 4, ENTONCES no aparece ninguna alerta; DADO QUE el stock mínimo es 0 y el stock es 0, ENTONCES tampoco aparece alerta.
- (Borde) DADO QUE Ana define un stock mínimo, CUANDO ingresa 0 o 9,999, ENTONCES el sistema lo guarda; CUANDO ingresa -1, 2.5 o 10,000, ENTONCES no lo guarda.

**Reglas de negocio relacionadas:** RN-029, RN-101, RN-102

**Notas:**
- P-52 resuelto: el stock mínimo por defecto es 0 (sin alerta). El singular con 1 unidad está en RN-150 (B-02).

---

### HU-015: Ver qué productos no se están vendiendo

**Prioridad:** Should

**Como** emprendedor, **quiero** ver la lista de productos sin ventas en los últimos 30 días, **para** decidir qué no volver a comprar a mis proveedores.

**Criterios de aceptación:**
- (Positivo) DADO QUE hoy es 2026-11-10 y la última venta de "Cartera negra" fue el 2026-10-05, CUANDO Ana abre la lista de productos sin movimiento, ENTONCES "Cartera negra" aparece.
- (Negativo) DADO QUE la última venta de "Cartera negra" fue el 2026-11-08, CUANDO Ana abre la lista, ENTONCES no aparece; DADO QUE un producto está desactivado, ENTONCES tampoco aparece.
- (Borde) DADO QUE un producto tuvo su primera entrada el 2026-11-01 y no tiene ventas, CUANDO Ana abre la lista el 2026-11-10 (9 días después), ENTONCES no aparece.

**Reglas de negocio relacionadas:** RN-030

**Notas:**
- P-53 resuelto: las ventas anuladas no cuentan y se usa la fecha y hora de venta; el borde exacto de 30 días se prueba al implementar F-08.

---

### HU-016: Ver mis ganancias por semana y por mes

**Prioridad:** Should

**Como** emprendedor, **quiero** ver el total de ventas, costo y ganancia por semana y por mes, **para** saber cómo va mi negocio sin sumar a mano.

**Criterios de aceptación:**
- (Positivo) DADO QUE Ana registró una venta el domingo 2026-11-15 a las 23:30, CUANDO ve el resumen semanal, ENTONCES la venta se suma a la semana del 2026-11-09 al 2026-11-15.
- (Negativo) DADO QUE una venta está "Anulada", CUANDO Ana ve cualquier resumen, ENTONCES su ganancia no se suma; DADO QUE Ana registró un ajuste por conteo físico, ENTONCES la ganancia del periodo no cambia.
- (Borde) DADO QUE Ana registró una venta el lunes 2026-11-16 a las 00:00:00, CUANDO ve el resumen semanal, ENTONCES la venta se suma a la semana siguiente; DADO QUE registró una venta el 2026-11-30 a las 23:30, ENTONCES se suma al resumen de noviembre de 2026, y una del 2026-12-01 a las 00:00:00 se suma a diciembre.

**Reglas de negocio relacionadas:** RN-031, RN-044, RN-051, RN-053

---

### HU-017: Ver los perfumes que están por vencer

**Prioridad:** Could

**Como** emprendedor, **quiero** ver los perfumes que vencen en los próximos 30 días, **para** venderlos antes de que se pierdan.

**Criterios de aceptación:**
- (Positivo) DADO QUE hoy es 2026-11-10, CUANDO una variante con stock 4 vence el 2026-12-01, ENTONCES aparece en la alerta de vencimiento.
- (Negativo) DADO QUE una variante vence el 2026-12-01 y tiene stock 0, CUANDO Ana abre la alerta, ENTONCES no aparece.
- (Borde) DADO QUE hoy es 2026-11-10, CUANDO una variante con stock mayor que 0 vence el 2026-12-10 (hoy + 30 días), ENTONCES aparece; CUANDO vence el 2026-12-11, ENTONCES no aparece; CUANDO ya venció, ENTONCES aparece.

**Reglas de negocio relacionadas:** RN-032

---

### HU-018: Recuperar mi contraseña por correo

**Prioridad:** Could

**Como** emprendedor, **quiero** restablecer mi contraseña con un enlace enviado a mi correo, **para** volver a entrar si la olvido sin pedirle ayuda a Sergio.

**Criterios de aceptación:**
- (Positivo) DADO QUE el enlace se envió a las 10:00:00, CUANDO Ana lo usa a las 10:20:00, ENTONCES el sistema lo acepta.
- (Negativo) DADO QUE el enlace se envió a las 10:00:00, CUANDO alguien lo usa a las 10:45:00, ENTONCES el sistema no lo acepta; DADO QUE Ana ya cambió su contraseña con ese enlace, CUANDO lo abre de nuevo, ENTONCES ve "Este enlace ya fue usado".
- (Borde) DADO QUE el enlace se envió a las 10:00:00, CUANDO se usa a las 10:30:00 (1,800 segundos), ENTONCES el sistema lo acepta; CUANDO se usa a las 10:30:01, ENTONCES no lo acepta.

**Reglas de negocio relacionadas:** RN-033, RN-034 (Constitución: PRINCIPIO-004)

**Notas:**
- Requiere un servicio de correo (D-05) y un ADR antes de escribir código (PRINCIPIO-002). P-59 resuelto: al pedir un enlace nuevo, el anterior deja de ser válido.
