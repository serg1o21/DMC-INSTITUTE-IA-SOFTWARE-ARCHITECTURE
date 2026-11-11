# 04 - Glosario: miStorePe

Lenguaje ubicuo del dominio. Es la única fuente del glosario del proyecto. Todo documento, regla y código del proyecto usa estos términos con este significado. Los nombres en código están en inglés según el PRINCIPIO-001 de `00-constitucion.md`.

Basado en `01-project-brief.md`, `02-alcance.md` y `03-reglas-negocio.md`. Fecha: 2026-11-10. Revisión QA: 2026-11-10 (términos nuevos al final de la tabla). Auditoría QA v2: 2026-11-10 (términos en código alineados con `08-architecture.md`; definiciones actualizadas con RN-126 a RN-139).

| Término (español) | Término en código | Definición |
|---|---|---|
| Emprendedor | `Entrepreneur` | Persona dueña de un pequeño negocio que vende ropa, perfumes y/o accesorios por redes sociales y que tiene una cuenta en miStorePe. En el MVP, un emprendedor tiene exactamente una cuenta y un inquilino. |
| Cuenta | `Account` | Registro de acceso del emprendedor, identificado por un correo único y una contraseña. |
| Inquilino | `Tenant` | Espacio de datos aislado de un emprendedor; todo dato de negocio pertenece a un solo inquilino mediante `tenant_id`. |
| Responsable de ventas | `SalesOwner` | Única persona del negocio que registra las salidas por venta; en el MVP es el titular de la cuenta. |
| Producto | `Product` | Artículo que vende el emprendedor, con nombre y una categoría. |
| Categoría | `Category` | Clasificación del producto: Ropa (`CLOTHING`), Perfume (`PERFUME`) o Accesorio (`ACCESSORY`). |
| Variante | `Variant` | Versión concreta de un producto que se controla por separado en el stock (por ejemplo, talla M color negro, o 100 ml con su fecha de vencimiento: dos presentaciones iguales con vencimientos distintos son variantes distintas). Todo producto tiene al menos una; un Accesorio sin variantes recibe la variante "Única". |
| Talla | `Size` | Medida de una prenda de ropa, guardada como texto en mayúsculas (por ejemplo, S, M, XL, 28, 30) (RN-111). |
| Color | `Color` | Color de una variante de ropa o accesorio, guardado con la primera letra en mayúscula (por ejemplo, Negro, Azul marino) (RN-121). |
| Presentación | `VolumeMl` | Contenido en mililitros de una variante de perfume. |
| Fecha de vencimiento | `ExpiryDate` | Fecha (AAAA-MM-DD) de vencimiento indicada en el perfume; es opcional. Un perfume con fecha anterior a la actual no se puede vender (RN-074). |
| Stock | `Stock` | Unidades disponibles de una variante: entradas no anuladas menos ventas no anuladas, más las diferencias de los ajustes por conteo físico (RN-026). |
| Stock mínimo | `MinimumStock` | Cantidad definida por el emprendedor a partir de la cual se muestra la alerta de stock bajo. |
| Entrada | `StockEntry` | Registro del ingreso de unidades de una variante con su costo unitario. |
| Proveedor | `Supplier` | Persona o negocio que vende la mercadería al emprendedor (por ejemplo, en Gamarra). |
| Venta (salida) | `Sale` | Registro de una venta a un cliente final; contiene una o más líneas de venta. |
| Línea de venta | `SaleLine` | Parte de una venta que corresponde a una sola variante, con su cantidad, precio final por unidad y ganancia. |
| Costo unitario | `UnitCost` | Monto en soles que pagó el emprendedor por una unidad en una entrada. |
| Costo unitario promedio | `AverageCost` | Costo ponderado de las unidades en stock de una variante, recalculado en cada entrada. |
| Precio de venta | `SalePrice` | Precio en soles configurado para una variante; es una referencia y puede diferir del precio final. |
| Ajuste por conteo físico | `StockAdjustment` | Corrección del stock de una variante a la cantidad contada físicamente, con motivo obligatorio; no es una venta. |
| Precio final | `UnitPrice` | Precio en soles al que realmente se vendió una unidad en una línea de venta concreta. |
| Ganancia | `Profit` | (Precio final por unidad - costo unitario promedio) x cantidad, calculada al registrar la venta (RN-027). Puede ser negativa en remates (RN-072). |
| Producto sin movimiento | `SlowMovingProduct` | Producto activo que cumple la condición de RN-030 (sin ventas en los últimos 30 días). |
| Producto desactivado | `InactiveProduct` | Producto que ya no se vende pero conserva su historial. |
| Cierre de cuenta | `AccountClosure` | Acción del emprendedor, confirmada con su contraseña, que registra `closed_at`, cierra sus sesiones e inicia el plazo de 30 días para el borrado definitivo de sus datos (RN-127). |
| Live | `Live` | Transmisión en vivo en TikTok donde el emprendedor ofrece y vende sus productos. |
| Visitante | `Visitor` | Persona que usa la app sin haber iniciado sesión (por ejemplo, en el formulario de registro). |
| Política de privacidad | `PrivacyPolicy` | Texto que informa cómo se tratan los datos del emprendedor según la Ley N° 29733; su aceptación es obligatoria para registrarse. |
| Intento fallido | `FailedLoginAttempt` | Inicio de sesión rechazado por correo o contraseña incorrectos. |
| Enlace de recuperación | `PasswordResetLink` | Enlace temporal enviado al correo para restablecer la contraseña. |
| Cliente final | `EndCustomer` | Persona que le compra al emprendedor; el sistema no guarda sus datos personales. |
| Historial | `MovementHistory` | Lista cronológica de entradas, ventas y ajustes de una variante. No confundir con el historial de ventas. |
| Motivo | `AdjustmentReason` | Texto obligatorio que explica por qué se hizo un ajuste por conteo físico. |
| Diferencia | `AdjustmentDifference` | Cantidad contada menos stock previo en un ajuste por conteo físico; puede ser positiva o negativa. |
| Anulación | `Voiding` | Acción que deja sin efecto una venta, una línea de venta o una entrada registrada por error: revierte su efecto en el stock y la ganancia y la conserva en el historial con el estado "Anulada". |
| SKU | `Sku` | Código único de una variante, generado automáticamente por el sistema al crearla (por ejemplo, `ROP-00001`); no cambia después de asignado. |
| Administrador | `Admin` | Rol del Responsable del producto para ver cuentas y suspenderlas o reactivarlas; no ve el inventario de ningún emprendedor y no tiene inquilino (RN-114). |
| Cuenta suspendida | `SuspendedAccount` | Cuenta sin acceso por falta de pago; a los 30 días pasa a cerrada. |
| Remate | `Clearance` | Venta por debajo del costo para liquidar mercadería; genera ganancia negativa. |
| Merma | `Shrinkage` | Unidades que se pierden o ya no se pueden vender (por ejemplo, perfumes vencidos). En el MVP se retiran con un ajuste por conteo físico; F-17 la registrará como salida con costo (P-60). |
| Sesión | `Session` | Periodo en que el emprendedor está identificado en la app; se cierra tras 28,800 s (8 h) sin actividad. |
| Reapertura de cuenta | `AccountReopening` | Reactivación de una cuenta cerrada por el propio emprendedor dentro de los 30 días, antes del borrado. |
| Historial de ventas | `SalesHistory` | Lista de las ventas del emprendedor, incluidas las anuladas con su estado (F-06, RN-054). |
| Estado de cuenta | `AccountStatus` | Situación de una cuenta: Activa (`ACTIVE`), Suspendida (`SUSPENDED`) o Cerrada (`CLOSED`). |
| Cuenta cerrada | `ClosedAccount` | Cuenta cerrada por el emprendedor (cierre de cuenta) o por falta de pago (RN-067); sus datos se borran 30 días después de su `closed_at` (RN-006). |
| Estado de movimiento | `MovementStatus` | Situación de una venta, línea de venta o entrada: Vigente (`ACTIVE`) o Anulada (`VOIDED`). |
| Fecha y hora de venta | `SaleAt` | Momento en que ocurrió una venta. Por defecto es el momento del registro (RN-126); para una venta olvidada, el emprendedor elige el día y una hora aproximada, hasta 7 días calendario atrás (RN-076 a RN-078). |
| Cantidad contada | `CountedQuantity` | Unidades contadas físicamente que se ingresan en un ajuste por conteo físico; entero de 0 o más (RN-040). |
| Etapa gratuita | `FreePeriod` | Periodo inicial sin cobro de la suscripción; su duración es un pendiente del brief. |
| Rol | `Role` | Tipo de usuario: Emprendedor (`ENTREPRENEUR`) o Administrador (`ADMIN`). |
| Responsable del producto | `ProductOwner` | Sergio, responsable del proyecto: único aprobador de especificaciones, reglas y ADR; tiene el rol Administrador. |
